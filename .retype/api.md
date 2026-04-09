# API Reference

All API endpoints are Next.js API routes, accessible at `/api/*`. The API accepts and returns JSON.

---

## Authentication

The API works without authentication, but anonymous requests are rate-limited to 5 per IP per hour.

To authenticate, include your API key as a Bearer token:

```http
Authorization: Bearer dfa_live_your_api_key_here
```

API keys are issued from the dashboard at `https://devfolio-analyzer.vercel.app/settings/api`.

---

## Base URL

```
https://devfolio-analyzer.vercel.app/api
```

For local development:

```
http://localhost:3000/api
```

---

## Endpoints

### `POST /api/analyze`

Triggers a full profile analysis for a given GitHub username. This is the primary endpoint.

#### Request

```http
POST /api/analyze
Content-Type: application/json
Authorization: Bearer dfa_live_xxx  (optional)
```

**Request body:**

```json
{
  "username": "gaearon",
  "options": {
    "include_forks": false,
    "max_repos": 20,
    "cache": true
  }
}
```

**Parameters:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `username` | string | Yes | — | GitHub username to analyze |
| `options.include_forks` | boolean | No | `false` | Whether to include forked repos in analysis |
| `options.max_repos` | integer | No | `20` | Max repos fetched. Min 5, max 30. |
| `options.cache` | boolean | No | `true` | Return cached result if available (< 6h old) |

#### Response — Success

**Status:** `200 OK`

```json
{
  "ok": true,
  "cached": false,
  "analyzed_at": "2025-05-30T10:22:14.000Z",
  "expires_at": "2025-05-30T16:22:14.000Z",
  "report_url": "https://devfolio-analyzer.vercel.app/report/gaearon-a3f9c1",
  "profile": {
    "username": "gaearon",
    "name": "Dan Abramov",
    "avatar_url": "https://avatars.githubusercontent.com/u/810438",
    "bio": "Working on @reactjs. Co-author of Redux and Create React App.",
    "public_repos": 251,
    "followers": 91400,
    "following": 171,
    "account_created": "2011-06-06T00:00:00Z",
    "location": "London, UK"
  },
  "score": {
    "total": 91,
    "label": "Excellent",
    "breakdown": {
      "project_quality": {
        "score": 94,
        "label": "Excellent",
        "weight": 0.30,
        "feedback": [
          "All top repos have detailed, well-structured READMEs with usage examples.",
          "Test suites detected in 5 of 6 evaluated repos.",
          "CI/CD configuration present in primary repos (GitHub Actions).",
          "Commit messages follow conventional commits pattern consistently."
        ]
      },
      "tech_stack": {
        "score": 88,
        "label": "Excellent",
        "weight": 0.20,
        "languages": {
          "JavaScript": 61.2,
          "TypeScript": 22.4,
          "HTML": 8.1,
          "CSS": 5.3,
          "Shell": 2.1,
          "Other": 0.9
        },
        "detected_frameworks": ["React", "Redux", "Jest", "Webpack"],
        "feedback": [
          "Strong JavaScript/TypeScript combination — modern and in demand.",
          "Framework authorship (React, Redux) demonstrates deep ecosystem knowledge.",
          "Low language diversity is justified by deep domain specialization.",
          "No Go/Rust/Python exposure — not a gap given specialization level."
        ]
      },
      "contribution_consistency": {
        "score": 78,
        "label": "Good",
        "weight": 0.25,
        "metrics": {
          "active_weeks_last_52": 38,
          "longest_streak_days": 47,
          "contributions_last_90_days": 142,
          "largest_gap_days": 21
        },
        "feedback": [
          "38 of 52 weeks active — strong baseline consistency.",
          "21-day gap in Q1 2025 noted — likely intentional, not a red flag at this level.",
          "Contribution volume is high but has tapered in the last 60 days.",
          "Majority of contributions are to external repos — positive signal."
        ]
      },
      "portfolio_presentation": {
        "score": 96,
        "label": "Excellent",
        "weight": 0.15,
        "signals": {
          "has_profile_readme": true,
          "has_pinned_repos": true,
          "pinned_count": 6,
          "has_bio": true,
          "has_avatar": true,
          "has_social_links": true,
          "repos_with_descriptions_pct": 74
        },
        "feedback": [
          "Profile README is detailed and reflects personal voice — stands out.",
          "All 6 pinned slots used, featuring highest-impact projects.",
          "74% of public repos have descriptions — above average.",
          "Twitter and personal blog linked — good discoverability."
        ]
      },
      "community_engagement": {
        "score": 97,
        "label": "Excellent",
        "weight": 0.10,
        "feedback": [
          "Significant open-source contribution history across major projects.",
          "Active issue discussions — not just code contributions.",
          "91k+ followers reflects substantial community presence.",
          "Regular engagement in RFC discussions for React ecosystem."
        ]
      }
    }
  },
  "top_repos": [
    {
      "name": "react",
      "full_name": "facebook/react",
      "description": "The library for web and native user interfaces.",
      "stars": 224000,
      "forks": 45800,
      "language": "JavaScript",
      "last_pushed": "2025-05-29T18:30:00Z",
      "is_fork": false,
      "has_readme": true,
      "has_tests": true,
      "has_ci": true,
      "url": "https://github.com/facebook/react"
    }
  ]
}
```

#### Response — Errors

**Status: `400 Bad Request`** — Invalid input

```json
{
  "ok": false,
  "error": {
    "code": "INVALID_USERNAME",
    "message": "Username must be 1–39 characters and may only contain alphanumeric characters and hyphens."
  }
}
```

**Status: `404 Not Found`** — GitHub user not found

```json
{
  "ok": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "No GitHub user found with username 'xyz_does_not_exist'."
  }
}
```

**Status: `429 Too Many Requests`** — Rate limit hit

```json
{
  "ok": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Anonymous rate limit reached. Authenticate to increase your quota.",
    "retry_after": 1800
  }
}
```

**Status: `502 Bad Gateway`** — GitHub API or LLM failure

```json
{
  "ok": false,
  "error": {
    "code": "UPSTREAM_ERROR",
    "message": "Failed to fetch data from GitHub API. This is likely a transient error.",
    "details": "GitHub API returned 503"
  }
}
```

---

### `GET /api/report/[slug]`

Retrieves a previously generated report by its unique slug.

#### Request

```http
GET /api/report/gaearon-a3f9c1
```

#### Response — Success

Returns the same JSON structure as `POST /api/analyze`. The `cached` field will always be `true` for this endpoint.

#### Response — Not Found

```json
{
  "ok": false,
  "error": {
    "code": "REPORT_NOT_FOUND",
    "message": "No report found for this slug, or the report has expired."
  }
}
```

---

### `GET /api/health`

Simple health check endpoint. Useful for uptime monitoring.

#### Request

```http
GET /api/health
```

#### Response

```json
{
  "status": "ok",
  "version": "1.4.2",
  "timestamp": "2025-05-30T10:22:14.000Z",
  "dependencies": {
    "github_api": "ok",
    "llm": "ok",
    "cache": "ok"
  }
}
```

If a dependency is degraded, its value will be `"degraded"` or `"down"`. The HTTP status will still be `200` unless the core service itself is down.

---

## Error codes reference

| Code | HTTP Status | Meaning |
|------|-------------|---------|
| `INVALID_USERNAME` | 400 | Username fails format validation |
| `INVALID_OPTIONS` | 400 | Options object contains unknown or out-of-range values |
| `USER_NOT_FOUND` | 404 | GitHub user does not exist |
| `REPORT_NOT_FOUND` | 404 | Report slug expired or invalid |
| `RATE_LIMIT_EXCEEDED` | 429 | IP or API key rate limit exceeded |
| `GITHUB_RATE_LIMITED` | 429 | GitHub API quota exhausted (server-side) |
| `LLM_TIMEOUT` | 504 | LLM did not respond within 30s |
| `UPSTREAM_ERROR` | 502 | GitHub API or LLM returned an error |
| `INTERNAL_ERROR` | 500 | Unexpected server-side failure |
