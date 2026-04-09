# Architecture

---

## System overview

Devfolio Analyzer is a monorepo Next.js application. All backend logic runs as serverless functions (Next.js API routes), deployed to Vercel's Edge Network. There is no separate backend server.

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Browser                        │
│                   (Next.js + Tailwind CSS)                   │
└─────────────────────────┬───────────────────────────────────┘
                          │ POST /api/analyze
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   Vercel Serverless Function                  │
│                    /api/analyze (Node.js)                     │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Validator  │  │  Cache Check │  │  Rate Limiter    │  │
│  │  (Zod schema)│  │  (KV Store)  │  │  (Upstash Redis) │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────────────┘  │
│         │                 │ miss                             │
│         ▼                 ▼                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              GitHub Data Fetcher                      │   │
│  │  (parallel fetch: user, repos, events, languages)     │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                    │
│                         ▼                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Data Normalization Layer                   │   │
│  │  (scoring signals, repo ranking, streak computation)  │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                    │
│                         ▼                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │               LLM Analysis Pipeline                  │   │
│  │        (prompt builder → API call → parser)           │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                    │
│                         ▼                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Score Composer                           │   │
│  │   (weighted aggregate, label assignment, slug gen)    │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                    │
│                         ▼                                    │
│                   Cache write + Response                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Data flow in detail

### 1. Request validation

Every incoming request is validated against a Zod schema before any external calls are made. Invalid requests are rejected immediately with a structured error — no GitHub API calls are wasted.

```typescript
// lib/validators/analyze.ts
const AnalyzeRequestSchema = z.object({
  username: z.string().min(1).max(39).regex(/^[a-zA-Z0-9-]+$/),
  options: z.object({
    include_forks: z.boolean().default(false),
    max_repos: z.number().int().min(5).max(30).default(20),
    cache: z.boolean().default(true),
  }).optional().default({}),
});
```

### 2. Cache lookup

Before hitting GitHub or the LLM, the cache is checked. Results are stored in Upstash Redis with a 6-hour TTL, keyed by `analyze:{username}:{options_hash}`.

Cache hits skip directly to score composition and return with `"cached": true`. This is important because:

- GitHub API has a 60 req/hour limit for unauthenticated requests (5,000 for authenticated)
- LLM calls add 2–5 seconds and non-trivial cost per analysis
- Profile data doesn't change meaningfully in under 6 hours

### 3. GitHub data fetching

Four GitHub REST API calls are made in parallel using `Promise.all`:

| Call | Endpoint | Data extracted |
|------|----------|----------------|
| User profile | `GET /users/{username}` | Name, bio, avatar, counts |
| Repositories | `GET /users/{username}/repos` | Up to 30 public repos |
| Events | `GET /users/{username}/events` | Last 90 days of activity |
| Contribution calendar | Scraped from GitHub HTML | Weekly contribution counts |

The contribution calendar is not available through the REST or GraphQL API without authentication scoped to the user. The current implementation scrapes the SVG contribution graph from `github.com/{username}`. This is a known fragility — see [Limitations](limitations.md).

Repository language breakdowns are fetched individually (`GET /repos/{owner}/{name}/languages`) for the top 6 repos only, to avoid exhausting rate limits.

### 4. Data normalization

The normalization layer transforms raw GitHub API responses into a deterministic scoring structure:

```typescript
interface NormalizedProfile {
  user: GitHubUserSummary;
  topRepos: NormalizedRepo[];       // top 6 by composite rank
  languageStats: Record<string, number>; // pct by byte count
  contributionMetrics: {
    activeWeeks: number;
    longestStreak: number;
    last90Days: number;
    largestGapDays: number;
  };
  presentationSignals: {
    hasProfileReadme: boolean;
    hasPinnedRepos: boolean;
    pinnedCount: number;
    hasBio: boolean;
    hasAvatar: boolean;
    reposWithDescriptionPct: number;
  };
}
```

Repo ranking uses a composite score: `(stars * 0.4) + (forks * 0.3) + (recencyScore * 0.3)`, where recency score decays over 365 days.

### 5. LLM analysis pipeline

The normalized profile is injected into a prompt template. The prompt is structured into sections:

1. **System instruction** — role, output format requirements, scoring rubric per dimension
2. **Profile data** — the normalized JSON object
3. **Evaluation instruction** — explicit instruction to return only valid JSON, no markdown

The model is called with a low temperature (0.2) to reduce variance in scores across repeated calls for the same profile. Output is parsed and validated against the expected response schema. If parsing fails, the pipeline falls back to a deterministic scoring algorithm (no LLM feedback text).

```typescript
// Simplified prompt structure
const systemPrompt = `
You are a senior engineering hiring evaluator. 
Analyze the following developer profile data and return ONLY a valid JSON object.
Do not include markdown, backticks, or any text outside the JSON.

Scoring rubric:
- project_quality (0-100): README quality, test presence, CI/CD, commit discipline
- tech_stack (0-100): Language recency, diversity, framework knowledge
- contribution_consistency (0-100): Activity regularity, recency weighting, gap penalty
- portfolio_presentation (0-100): Profile README, pinned repos, bio, descriptions
- community_engagement (0-100): External contributions, issues, follower signal

For each dimension, include 2-4 specific feedback strings. Be concrete. 
Do not give generic advice like "add more tests." Reference actual repo names.

Response schema: [schema injected here]
`;
```

### 6. Score composition

The final score is assembled from:

- Deterministically computed metrics (contribution stats, presence signals)
- LLM-assigned per-dimension scores and feedback
- Weighted sum: `total = Σ(dimension_score * weight)`

A unique slug is generated (`{username}-{6 char hex}`) and the full result is written to cache and returned.

---

## Folder structure

```
devfolio-analyzer/
├── app/                          # Next.js app router
│   ├── page.tsx                  # Landing / search page
│   ├── report/[slug]/page.tsx    # Report view
│   └── layout.tsx
├── pages/
│   └── api/
│       ├── analyze.ts            # POST /api/analyze
│       ├── report/[slug].ts      # GET /api/report/[slug]
│       └── health.ts             # GET /api/health
├── lib/
│   ├── github/
│   │   ├── fetcher.ts            # GitHub API client
│   │   ├── normalizer.ts         # Raw → NormalizedProfile
│   │   └── scraper.ts            # Contribution calendar scrape
│   ├── llm/
│   │   ├── client.ts             # LLM API wrapper
│   │   ├── prompt.ts             # Prompt builder
│   │   └── parser.ts             # Response validator/parser
│   ├── scoring/
│   │   ├── composer.ts           # Weighted score assembly
│   │   ├── fallback.ts           # Deterministic fallback scorer
│   │   └── labels.ts             # Score → label mapping
│   ├── cache/
│   │   └── kv.ts                 # Upstash Redis wrapper
│   ├── validators/
│   │   └── analyze.ts            # Zod schemas
│   └── rate-limit.ts
├── components/
│   ├── ScoreDashboard.tsx
│   ├── SubScoreCard.tsx
│   ├── ContributionHeatmap.tsx
│   ├── LanguageBar.tsx
│   └── RepoCard.tsx
├── public/
├── .env.local.example
├── next.config.js
└── package.json
```

---

## Environment dependencies

| Variable | Required | Description |
|----------|----------|-------------|
| `GITHUB_TOKEN` | Recommended | GitHub PAT. Without it, unauthenticated limit (60 req/hr) applies |
| `OPENAI_API_KEY` | Yes | LLM provider API key (or equivalent for other providers) |
| `LLM_MODEL` | No | Model name. Defaults to `gpt-4o-mini` |
| `UPSTASH_REDIS_URL` | Yes | Redis URL from Upstash for caching |
| `UPSTASH_REDIS_TOKEN` | Yes | Auth token for Upstash Redis |
| `RATE_LIMIT_WINDOW_MS` | No | Rate limit window in ms. Default: `3600000` (1 hour) |
| `RATE_LIMIT_ANON_MAX` | No | Max anon requests per window. Default: `5` |
