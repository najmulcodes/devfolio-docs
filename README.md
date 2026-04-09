# Devfolio Analyzer — Documentation

This is the documentation site for [Devfolio Analyzer](https://devfolio-analyzer.vercel.app), built with [Retype](https://retype.com).

---

## Local development

### Prerequisites

- Node.js 18+
- npm or yarn
- Retype CLI

### Install Retype

```bash
npm install --global retypeapp
```

Or with dotnet (alternative install method):

```bash
dotnet tool install retypeapp --global
```

Verify the install:

```bash
retype --version
```

### Clone and run

```bash
git clone https://github.com/your-org/devfolio-analyzer.git
cd devfolio-analyzer/devfolio-docs
retype watch
```

Retype starts a local dev server at `http://localhost:5000` with hot reload. Edit any `.md` file and the browser updates immediately.

---

## Project structure

```
devfolio-docs/
├── docs/
│   ├── index.md          # Home page
│   ├── overview.md       # How it works, scoring model
│   ├── features.md       # Feature breakdown
│   ├── api.md            # API reference with request/response examples
│   ├── architecture.md   # Data flow, folder structure, env vars
│   ├── deployment.md     # Vercel deployment, local setup, Redis
│   └── limitations.md    # Known constraints and edge cases
├── static/               # Logo, favicon (add your own)
├── retype.yml            # Site config, navigation, branding
└── README.md             # This file
```

---

## Build for production

```bash
retype build
```

Output is generated to the `.retype/` directory. This is a static site — deploy it anywhere that serves static files.

---

## Deploy to Vercel

The `.retype/` build output is a static site. To deploy it on Vercel:

### Option 1: Deploy the static output

1. Build the docs: `retype build`
2. Create a new Vercel project pointed at the `devfolio-docs` directory
3. Set the **Output Directory** to `.retype`
4. Set **Build Command** to `retype build`
5. Deploy

Vercel will rebuild the docs on every push to the connected branch.

### Option 2: Deploy via Vercel CLI

```bash
npm install -g vercel
cd devfolio-docs
retype build
cd .retype
vercel --prod
```

### Vercel config (optional)

Create a `vercel.json` in `devfolio-docs/`:

```json
{
  "buildCommand": "retype build",
  "outputDirectory": ".retype",
  "installCommand": "npm install -g retypeapp"
}
```

This lets Vercel handle the full build pipeline without manual steps.

---

## Updating content

All documentation content is in the `docs/` directory as Markdown files. Edit any file and commit — Vercel will redeploy automatically if connected to the repository.

Retype supports standard Markdown plus extended syntax for callouts, tabs, code groups, and more. See [retype.com/components](https://retype.com/components) for the full component reference.

---

## Retype documentation

- [Getting started](https://retype.com/guides/getting-started/)
- [Configuration reference](https://retype.com/configuration/project/)
- [Components](https://retype.com/components/)
