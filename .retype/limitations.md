# Limitations

Understanding what Devfolio Analyzer cannot do is as important as knowing what it can. This page documents known constraints, data gaps, and scoring edge cases.

---

## GitHub API constraints

### Contribution calendar is not available via API

The contribution calendar (the green squares on a GitHub profile) is not exposed by GitHub's REST or GraphQL API for users other than the authenticated user. Devfolio Analyzer scrapes this data from the rendered HTML at `github.com/{username}`.

**Implications:**
- This scrape can break if GitHub changes their HTML structure (has happened twice in 2023–2024)
- The scrape adds ~800ms to analysis time
- It fails silently for users with private contribution settings — contribution metrics default to `null`

**Workaround:** If you're analyzing your own profile and need accurate contribution data, authenticate via OAuth (coming in a future release) to use the authenticated API endpoint.

### Private repositories are invisible

All analysis is based on public repository data only. Developers who do most of their work in private repos — common for professionals — will score poorly on project quality and contribution consistency regardless of their actual output.

There is no way to fix this without OAuth authentication and explicit user consent. This is by design.

### Forked repos inflate repo counts

GitHub includes forked repositories in the public repo count and the default repo listing. Devfolio Analyzer excludes forks from analysis by default (`include_forks: false`), but the raw `public_repos` count in the profile object reflects GitHub's number, which includes forks.

### Organization contributions are partially visible

Contributions made to organization repositories show up in the contribution calendar but not in the per-repo analysis unless the organization repos are also public. Some developers have significant work in organization-owned repos that won't surface in this analysis.

---

## LLM scoring limitations

### Scores vary slightly across runs

The LLM is called with temperature 0.2, which reduces but does not eliminate variance. Two analyses of the same profile within the same cache window will return identical results (cached). After cache expiry, small score differences (±3–5 points) are possible on re-analysis.

This is not a bug — it's an inherent property of probabilistic language models. The 6-hour cache mitigates this for most use cases.

### Generic profiles are scored conservatively

The LLM is instructed to be specific and reference actual repo names. For profiles with minimal data (no descriptions, no READMEs, no bio), the feedback will be generic because there is nothing specific to reference. The score will be low — which is accurate.

### The model doesn't execute code

The LLM cannot evaluate code quality, architecture decisions, or algorithmic complexity. "Has tests" is detected by file presence, not by whether the tests are meaningful. A repo with a single empty test file will incorrectly get credit for test presence.

### Non-English READMEs

README quality evaluation is done by the LLM, which works best with English content. READMEs in other languages may be evaluated less accurately. This is a known gap with no current mitigation.

---

## Scoring model limitations

### Stars are not a quality signal, but they're used anyway

Repository star count is one input to the repo ranking algorithm (top 6 repos selected for detailed analysis). A popular but low-quality repo will be selected over a high-quality but obscure one. This is a reasonable heuristic but not a perfect one.

### Quantity is deliberately penalized

The scoring model does not reward having more repos. 20 well-maintained repos score the same or lower than 5 excellent ones. Developers with large numbers of small utility scripts, course projects, or abandoned experiments may find their scores lower than expected.

### Account age is not scored

Devfolio Analyzer does not factor account age into scores. A 6-month-old account with strong signals can score higher than a 10-year-old account with stale activity. This is intentional — we score current signal strength, not tenure.

---

## Known edge cases

| Scenario | Behavior |
|----------|----------|
| Username with all private repos | Profile metadata only. Score will be 0–30 range. |
| Bot accounts | May score high on consistency. No bot detection implemented. |
| Organization accounts | Treated as user accounts. Results are often misleading. |
| Suspended GitHub accounts | Returns `USER_NOT_FOUND` error. |
| Very new accounts (<10 repos) | Low scores by default due to sparse data. Expected behavior. |
| Renamed usernames | Old slugs will 404. Re-analyze under the new username. |

---

## What this tool is not

- **Not a hiring decision tool.** The score is a signal, not a verdict. Use it as a starting point for evaluation, not a filter.
- **Not a real-time dashboard.** Results are cached for 6 hours. Do not expect second-by-second accuracy.
- **Not an audit tool for private code.** Everything is based on public GitHub data.
- **Not a replacement for code review.** This tool cannot assess the actual quality of code — only the signals around it.
