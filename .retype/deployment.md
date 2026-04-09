# Deployment

Devfolio Analyzer is designed to deploy on Vercel with zero infrastructure configuration. The only external dependencies are a GitHub token, an LLM API key, and an Upstash Redis instance for caching.

---

## Prerequisites

Before deploying, you need:

- A [Vercel](https://vercel.com) account
- A [GitHub Personal Access Token](https://github.com/settings/tokens) (classic, `read:user` + `public_repo` scopes)
- An OpenAI (or compatible) API key
- An [Upstash](https://upstash.com) Redis database (free tier works fine)

---

## Option 1: Deploy to Vercel (recommended)

### Step 1: Fork the repository

Fork `https://github.com/your-org/devfolio-analyzer` to your own GitHub account.

### Step 2: Create a new Vercel project

1. Go to [vercel.com/new](https://vercel.com/new)
2. Import your forked repository
3. Vercel will detect the Next.js framework automatically
4. **Do not click Deploy yet** — you need to add environment variables first

### Step 3: Set environment variables

In the Vercel project settings → **Environment Variables**, add the following:

| Variable | Example value | Notes |
|----------|--------------|-------|
| `GITHUB_TOKEN` | `ghp_xxxxxxxxxxxx` | GitHub PAT. Required for production. |
| `OPENAI_API_KEY` | `sk-proj-xxxx` | LLM API key |
| `LLM_MODEL` | `gpt-4o-mini` | Optional — defaults to `gpt-4o-mini` |
| `UPSTASH_REDIS_URL` | `https://xxx.upstash.io` | From Upstash dashboard |
| `UPSTASH_REDIS_TOKEN` | `AXxx...` | From Upstash dashboard |

Set all variables for **Production**, **Preview**, and **Development** environments.

### Step 4: Deploy

Click **Deploy**. Vercel will build and deploy the project. First deploy typically takes 60–90 seconds.

Your app will be live at `https://your-project.vercel.app`.

---

## Option 2: Local development

### Clone and install

```bash
git clone https://github.com/your-org/devfolio-analyzer.git
cd devfolio-analyzer
npm install
```

### Configure environment

```bash
cp .env.local.example .env.local
```

Edit `.env.local` and fill in all required values:

```env
GITHUB_TOKEN=ghp_your_token_here
OPENAI_API_KEY=sk-proj-your_key_here
LLM_MODEL=gpt-4o-mini
UPSTASH_REDIS_URL=https://your-db.upstash.io
UPSTASH_REDIS_TOKEN=your_token_here
RATE_LIMIT_ANON_MAX=100
```

Set `RATE_LIMIT_ANON_MAX` higher locally so you're not blocked during development.

### Start the dev server

```bash
npm run dev
```

The app runs at `http://localhost:3000`. The API is available at `http://localhost:3000/api`.

### Run a test analysis

```bash
curl -X POST http://localhost:3000/api/analyze \
  -H "Content-Type: application/json" \
  -d '{"username": "sindresorhus"}'
```

---

## Custom domain

In Vercel → Settings → Domains, add your custom domain and follow the DNS configuration instructions. HTTPS is provisioned automatically.

---

## GitHub API rate limits

The GitHub REST API enforces:

- **Unauthenticated**: 60 requests/hour per IP
- **Authenticated (PAT)**: 5,000 requests/hour per token

Each profile analysis consumes approximately 8–12 GitHub API calls. At 60/hour unauthenticated, you can serve ~5–7 analyses per hour before hitting the limit. **Always configure `GITHUB_TOKEN` in production.**

If you expect high traffic, consider:

- Rotating multiple tokens (requires care to stay within GitHub's ToS)
- Increasing cache TTL to reduce repeat fetches
- Implementing a queue to spread requests over time

---

## Upstash Redis setup

1. Create an account at [upstash.com](https://upstash.com)
2. Create a new Redis database (Global region recommended for low latency)
3. Copy the **REST URL** and **REST Token** from the database dashboard
4. Paste into your environment variables

The free tier (10,000 commands/day, 256 MB) is sufficient for moderate usage. Each analysis writes one cache entry and reads one (on cache hit). At 5,000 analyses/day, you'll use ~10,000 Redis commands — exactly at the free tier limit. Upgrade to the Pay-as-you-go plan if you exceed this.

---

## Monitoring

Vercel provides basic function logs and error tracking out of the box. For production use, consider adding:

- **[Sentry](https://sentry.io)** — error tracking with Next.js SDK (`@sentry/nextjs`)
- **[Axiom](https://axiom.co)** — structured log aggregation (first-party Vercel integration)
- **[Vercel Analytics](https://vercel.com/analytics)** — real user monitoring, no configuration needed

The `/api/health` endpoint can be polled by an external uptime monitor (e.g., Better Uptime, UptimeRobot) to alert on service degradation.

---

## Updating the deployment

All pushes to the `main` branch trigger automatic redeployment on Vercel. There is no manual deployment step after the initial setup.

To update environment variables without a code change:

1. Update the variable in Vercel → Settings → Environment Variables
2. Trigger a redeploy: Vercel dashboard → Deployments → ... → Redeploy

Environment variable changes do not take effect until the next deployment.
