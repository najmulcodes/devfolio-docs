# Devfolio Analyzer — Documentation

Documentation for **Devfolio Analyzer**, an AI-powered tool that evaluates GitHub profiles and developer portfolios with structured scoring and actionable feedback.

👉 Live App: https://devfolioanalyzer.vercel.app
👉 Docs: https://devfoliodocs.vercel.app

---

## What this repo contains

This repository hosts the documentation site built with Retype.

It covers:

* How Devfolio Analyzer works
* Feature breakdown
* API reference
* System architecture
* Deployment setup
* Known limitations

The goal is to provide clear, technical insight into how the system evaluates developer profiles and portfolios.

---

## Local development

### Requirements

* Node.js 18+
* npm (or yarn)
* Retype CLI

---

### Install Retype

```bash
npm install -g retypeapp
```

Verify installation:

```bash
retype --version
```

---

### Run locally

```bash
git clone https://github.com/najmulcodes/devfolio-docs.git
cd devfolio-docs
retype start
```

Open:

```
http://localhost:5000
```

The site will reload automatically when you edit any `.md` file.

---

## Project structure

```
devfolio-docs/
├── docs/
│   ├── index.md          # Homepage
│   ├── overview.md       # System overview & scoring model
│   ├── features.md       # Feature breakdown
│   ├── api.md            # API documentation
│   ├── architecture.md   # System design & data flow
│   ├── deployment.md     # Deployment guide
│   └── limitations.md    # Constraints & edge cases
├── static/               # Logo, favicon
├── retype.yml            # Configuration
└── README.md             # This file
```

---

## Build

Generate the production-ready static site:

```bash
retype build
```

Output will be created in:

```
.retype/
```

This folder contains the full static website.

---

## Deploy (Vercel)

### Recommended setup

1. Push this repo to GitHub
2. Import it into Vercel
3. Use the following settings:

* Build Command:

  ```
  retype build
  ```
* Output Directory:

  ```
  .retype
  ```
* Install Command:

  ```
  npm install -g retypeapp
  ```

Deploy. Vercel will automatically rebuild on every push.

---

## Content updates

All documentation lives inside the `docs/` folder.

To update:

1. Edit any `.md` file
2. Commit and push
3. Vercel redeploys automatically

---

## About Devfolio Analyzer

Devfolio Analyzer evaluates:

* GitHub profile presentation
* Project quality and structure
* Contribution consistency
* Tech stack relevance
* Portfolio clarity

It returns a structured score along with specific feedback and improvement suggestions.

The focus is not on vanity metrics, but on **actual signal**.

---

## Notes

* This documentation site is static and fast to load
* All pages are written in Markdown
* Retype handles layout, navigation, and search

---

## References

* https://retype.com
* https://retype.com/guides/getting-started/
* https://retype.com/configuration/project/
