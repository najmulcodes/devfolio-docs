---
label: Home
icon: home
order: 100
---

# Devfolio Analyzer

**See how your GitHub profile and developer portfolio actually read to someone else.**

Devfolio Analyzer turns public profile data into a clear score, specific feedback, and practical next steps.

It is built for developers who want honest improvement signals and for hiring teams that need faster, cleaner review.

---

## What it evaluates

- **GitHub profile quality**  
  Bio, pinned repos, repo descriptions, profile completeness, and presentation

- **Portfolio quality**  
  Layout clarity, project presentation, live links, and how well the work is explained

- **Project quality**  
  README depth, repo structure, tests, CI/CD, and code organization signals

- **Contribution consistency**  
  Commit frequency, streaks, activity gaps, and recent momentum

- **Tech stack signal**  
  Language mix, framework signals, and how current the stack looks

---

## What you get

- An overall score from 0 to 100
- Five weighted sub-scores
- Strengths and weak points
- Concrete improvement suggestions
- A shareable report page

---

## Example result

**Score:** 78 / 100  
**Label:** Good

**Main issues:**
- Recent activity is inconsistent
- Top repositories need stronger README structure
- Profile presentation is weaker than the projects themselves

**Suggested fixes:**
- Improve top repo documentation
- Add clearer profile messaging
- Pin the most relevant projects
- Make the portfolio easier to scan in under 30 seconds

---

## Why this matters

Most profiles look fine at first glance.

But a real reviewer checks faster and more critically:
what you build, how clearly you present it, whether your work feels current, and whether your public profile makes that obvious.

Devfolio Analyzer makes that signal visible.

---

## How it works

1. Enter a GitHub username
2. Public GitHub data is fetched
3. Signals are normalized into structured inputs
4. AI generates the evaluation
5. A clean report is created

---

## Who it is for

| User | Value |
|------|-------|
| Developers | See how your profile and portfolio read from the outside |
| Hiring managers | Review public signal faster |
| Bootcamp grads | Find gaps before applying |
| Team leads | Benchmark open-source presence and project quality |

---

## Quick start

```http
POST /api/analyze
Content-Type: application/json

{
  "username": "torvalds"
}