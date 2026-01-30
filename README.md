# Platnova Documentation

This repository contains the [Platnova](https://platnova.com) API and product documentation, built with [Mintlify](https://mintlify.com).

## Prerequisites

- **Node.js** (v18 or later) and **npm**

## Quick start

### 1. Clone the repo

```bash
git clone <repo-url>
cd docs
```

### 2. Install the Mintlify CLI

Install the CLI globally so you can run the dev server and other commands:

```bash
npm install -g mint
```

### 3. Run the docs locally

From the **root of this repo** (the folder that contains `docs.json`):

```bash
mint dev
```

Your browser will open to a local URL (e.g. `http://localhost:3000`). Edits to `.mdx` files and `docs.json` will hot-reload.

## Project structure

| Path | Purpose |
|------|--------|
| `docs.json` | Mintlify config: theme, name, navigation (tabs, groups, pages), logo, navbar, footer |
| `*.mdx` (root) | Top-level pages: `index.mdx`, `authentication.mdx`, `environments.mdx`, `errors.mdx` |
| `customers/`, `card/`, `payments/`, etc. | Guide pages (overview, how-tos). Each group in `docs.json` points to these paths |
| `api-reference/` | API reference: `introduction.mdx`, plus per-endpoint pages that use the OpenAPI spec |
| `api-reference/openapi.json` | OpenAPI 3.x spec; Mintlify uses this to render request/response docs for API Reference pages |

- **Guides** — Written in MDX (Markdown + JSX). Add or edit `.mdx` files and register them under the right group in `docs.json` → `navigation` → `tabs`.
- **API Reference** — Pages under `api-reference/` can reference OpenAPI operations with the `openapi` frontmatter (e.g. `openapi: "POST /v1/payments/send"`). Keep `api-reference/openapi.json` in sync with your API so the docs stay accurate.

## Mintlify basics

- **Navigation** — All nav tabs and sidebar groups are defined in `docs.json` under `navigation.tabs`. Each `pages` array lists file paths without `.mdx` (e.g. `"payments/overview"` → `payments/overview.mdx`).
- **API Reference from OpenAPI** — In an API reference `.mdx` file, set `openapi: "<METHOD> <PATH>"` in the frontmatter to pull in the operation from `openapi.json`. Example: `openapi: "GET /v1/transactions"`.
- **Docs** — Full Mintlify docs: [mintlify.com/docs](https://mintlify.com/docs)

## Publishing changes

1. **Connect the repo** — In the [Mintlify dashboard](https://dashboard.mintlify.com), install the Mintlify GitHub App and connect this repository.
2. **Deploy** — Pushes to the default branch (e.g. `main`) trigger a deploy. Your live docs will update automatically after the build completes.
3. **Custom domain / branding** — Configure domains, redirects, and analytics in the Mintlify dashboard.

## Troubleshooting

| Issue | What to try |
|-------|----------------|
| Dev server won’t start or behaves oddly | Run `mint update` to get the latest CLI, then `mint dev` again. |
| Page shows 404 | Ensure you’re running `mint dev` from the folder that contains `docs.json`. Check that the page path in `docs.json` matches the file (e.g. `payments/overview` → `payments/overview.mdx`). |
| API reference missing or wrong | Confirm `api-reference/openapi.json` is valid OpenAPI 3.x and that the path in the `.mdx` frontmatter (e.g. `openapi: "POST /v1/links"`) matches an operation in the spec. |
| Nav or tabs don’t match expectations | Edit `docs.json` → `navigation` → `tabs`. Each tab has `groups`; each group has a `group` name and a `pages` array of paths. |

For more help, see [Mintlify’s docs](https://mintlify.com/docs) or contact [Platnova Support](mailto:support@platnova.com).
