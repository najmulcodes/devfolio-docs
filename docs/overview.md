---
label: Overview
icon: telescope
order: 90
---

# Overview

Devfolio Analyzer is a Next.js application that combines GitHub's REST API with an LLM-based scoring pipeline to produce structured, actionable evaluations of developer profiles.

The core insight is simple: GitHub profiles contain a lot of signal, but that signal is buried in raw data. Recruiters don't have time to manually audit 40 repos, and developers rarely know how their profile reads to someone who isn't them. Devfolio Analyzer bridges that gap.

---

## How it works (high level)

```
GitHub Username
     │
     ▼
GitHub REST API (user, repos, events, contributions)
     │
     ▼
Data Normalization Layer (scoring-ready structured data)
     │
     ▼
LLM Analysis (prompt-engineered, structured output)
     │
     ▼
Score + Feedback JSON
     │
     ▼
Frontend UI (Next.js — score cards, feedback list, charts)
```

The process from username submission to result delivery takes approximately 4–8 seconds depending on repo count and LLM response time.

---

## Scoring model

The total score is a weighted composite across five dimensions:

| Dimension | Weight | What it measures |
|-----------|--------|-----------------|
| Project Quality | 30% | README, structure, tests, CI/CD, meaningful commit history |
| Tech Stack | 20% | Language diversity, recency, framework maturity |
| Contribution Consistency | 25% | Activity frequency, gaps, streak length |
| Portfolio Presentation | 15% | Profile README, pinned repos, bio quality, links |
| Community Engagement | 10% | External PRs, issue participation, forks to meaningful repos |

Each dimension produces a sub-score from 0–100. The composite score is the weighted sum.

### Score ranges

| Score | Label | Interpretation |
|-------|-------|----------------|
| 85–100 | Excellent | Strong signal. Profile is polished and active. |
| 70–84 | Good | Solid but has specific gaps worth addressing. |
| 50–69 | Fair | Active developer but presentation or consistency needs work. |
| 30–49 | Weak | Sparse activity or underdeveloped projects. |
| 0–29 | Poor | Minimal public signal. Hard to evaluate technically. |

---

## What the LLM actually does

The raw GitHub data is too noisy to feed directly into an LLM context window. Before the LLM sees anything, the data normalization layer reduces it to a structured summary:

- Top 6 repos by stars + recency
- Aggregate language stats (% breakdown)
- Contribution streak metadata
- Presence flags (has README, has tests, has CI config, has pinned repos)

This normalized object is injected into a prompt template that instructs the model to:

1. Evaluate each scoring dimension independently
2. Return a JSON-formatted response (no markdown, no prose outside the schema)
3. Produce specific, non-generic feedback per dimension

The model is explicitly instructed **not** to award points for quantity. Ten low-effort repos score worse than three well-maintained ones.

---

## Design decisions

**Why not use GraphQL?**
GitHub's REST API v3 is more stable across rate limit scenarios and easier to cache at the edge. The GraphQL API requires more careful query scoping and has stricter rate limits for unauthenticated requests. For the data points Devfolio Analyzer needs, REST is sufficient.

**Why LLM for scoring instead of a deterministic algorithm?**
The qualitative dimensions — "is this README actually useful?" — don't reduce cleanly to boolean checks. An LLM can read a README and evaluate it the same way a human would. The quantitative dimensions (contribution frequency, language counts) are computed deterministically and passed as context.

**Why Next.js API routes instead of a separate backend?**
Deployment simplicity. Everything runs on Vercel — no separate server to manage, no CORS issues in development, and the API routes are close to the frontend. If throughput demands grow, the API layer can be extracted to a standalone service without changing the frontend.
