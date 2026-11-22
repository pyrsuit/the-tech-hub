# CLAUDE.md — thetechhub.io

Personal diary/blog of a software developer and team lead. It collects reflections
on **technical** and **interpersonal** problems met at work. It is a static site built
with **Zensical** and deployed to **GitHub Pages** at **https://thetechhub.io**.

## Stack

- **Zensical** (static site generator from the Material for MkDocs team; Python package with a Rust core).
  It is pre-1.0 and changes quickly, so the version is locked in `uv.lock`. Read the docs at
  https://zensical.org/docs/ before assuming a Material for MkDocs feature works the same way here.
- Config lives in **`zensical.toml`** (TOML, not `mkdocs.yml`).
- The blog uses the built-in **blog plugin** with `blog_dir = "."`, so the **homepage is the post list**.
- **Python 3.15**, managed by **uv** together with all dependencies. Don't use `pip`, `requirements.txt` or a manually created venv.
- Deployment: GitHub Actions → GitHub Pages (`.github/workflows/docs.yml`).

## Project layout

```
.
├─ .github/workflows/docs.yml   # build + deploy on push to main
├─ docs/
│  ├─ index.md                  # homepage intro; post list is rendered below it
│  ├─ about.md                  # "About me" page
│  ├─ CNAME                     # contains: thetechhub.io  (custom domain — do not delete)
│  ├─ assets/images/            # images used in posts
│  └─ posts/                    # one Markdown file per reflection
│     └─ YYYY-MM-DD-short-slug.md
├─ .python-version             # contains: 3.15  (uv installs it automatically)
├─ pyproject.toml               # dependencies (zensical); not a package, no build-system
├─ uv.lock                      # exact locked versions; always commit
├─ zensical.toml
└─ CLAUDE.md
```

The build output goes to `site/`, and uv's environment lives in `.venv/`. Both are generated,
so never edit or commit them (keep both in `.gitignore`).

## Commands

```bash
uv sync                                  # create .venv and install exactly what uv.lock says

uv run zensical serve                    # live preview at http://localhost:8000
uv run zensical build --clean            # production build into site/
uv run zensical build --clean --strict   # fail on warnings (run before committing)

uv add <package>                         # add a dependency (updates pyproject.toml + uv.lock)
uv lock --upgrade-package zensical       # upgrade Zensical (then build and check the site)
```

`pyproject.toml` looks like this:

```toml
[project]
name = "thetechhub"
version = "0.1.0"
requires-python = ">=3.15"
dependencies = [
    "zensical>=0.1.3,<0.2",   # pre-1.0: stay within a minor version, uv.lock pins the exact one
]
```

## Writing a post

File name: `docs/posts/YYYY-MM-DD-short-slug.md`. Required front matter:

```markdown
---
date:
  created: 2026-11-22
categories:
  - Interpersonal        # one of: Technical, Interpersonal, Leadership, Process
tags:
  - feedback
  - 1on1s
slug: giving-feedback-to-a-senior-dev   # optional; otherwise derived from the title
draft: true                             # remove when ready to publish
---

# Giving feedback to a more senior developer

Two or three sentence summary that appears in the post list.

<!-- more -->

Full reflection…
```

Conventions:
- **One H1 per post.** It becomes the title.
- Always put **`<!-- more -->`** after the summary so the homepage shows only an excerpt.
- Use only the categories listed above. Ask before inventing a new one.
- Tags are lowercase and hyphenated (`code-review`, not `Code Review`).
- `draft: true` posts are excluded from the production build, so new posts start as drafts.
- A suggested (not mandatory) structure for reflections: **Situation → What I tried → What happened → What I learned / would do differently.**
- Images go in `docs/assets/images/` and are referenced relative to the post, e.g. `![alt](../assets/images/foo.png)`. Always include alt text.

## Rules for Claude

1. **This is the author's voice.** When editing a post, fix grammar, clarity and structure, but
   do not rewrite opinions, add conclusions or change the tone unless asked. Propose bigger changes; don't just apply them.
2. **Privacy first.** Posts describe real workplace situations. Flag any real names of colleagues,
   employers, clients or identifying details, and suggest anonymized versions ("a senior backend dev",
   "a previous company"). Never publish such details without explicit confirmation.
3. **Don't publish by accident.** Never remove `draft: true` and never push to `main` without being asked,
   because pushing to `main` deploys the live site.
4. Before saying a change is done, run `uv run zensical build --clean --strict` and confirm it passes.
5. Don't modify `docs/CNAME`, `.github/workflows/docs.yml` or the locked Zensical version unless asked.
   When upgrading Zensical, build locally first and check the homepage, a post, and the archive and category pages.
   Always commit `pyproject.toml` and `uv.lock` together.
6. Use `uv add` / `uv remove` / `uv run` only. Never `pip install`, never edit `uv.lock` by hand.
7. Keep `zensical.toml` changes minimal and commented. If unsure whether an option is supported in Zensical
   (versus Material for MkDocs), check https://zensical.org/docs/ first.
8. Commit messages: short imperative subject, e.g. `Add post: giving feedback to a senior dev`.

## Key config (excerpt of zensical.toml)

```toml
[project]
site_name = "The Tech Hub"
site_url = "https://thetechhub.io/"
site_description = "Reflections on technical and interpersonal challenges as a developer and team lead"
site_author = "Zsófia Kálosi"

[project.plugins.blog]
blog_dir = "."          # post list on the homepage; posts live in docs/posts/

[project.plugins.tags]
```

## Deployment

- Pushing to `main` triggers `.github/workflows/docs.yml`, which installs uv, runs
  `uv sync --locked` → `uv run zensical build --clean` → uploads `site/` → `actions/deploy-pages`.
  `--locked` makes CI fail if `uv.lock` is out of date with `pyproject.toml`, so CI builds exactly
  what was tested locally. The workflow:

  ```yaml
  name: Documentation
  on:
    push:
      branches: [main]
    workflow_dispatch:
  permissions:
    contents: read
    pages: write
    id-token: write
  jobs:
    deploy:
      environment:
        name: github-pages
        url: ${{ steps.deployment.outputs.page_url }}
      runs-on: ubuntu-latest
      steps:
        - uses: actions/configure-pages@v6
        - uses: actions/checkout@v7
        - uses: astral-sh/setup-uv@v7      # bump to the latest major when updating actions
        - run: uv sync --locked
        - run: uv run zensical build --clean
        - uses: actions/upload-pages-artifact@v5
          with:
            path: site
        - uses: actions/deploy-pages@v5
          id: deployment
  ```
- One-time GitHub setup: **Settings → Pages → Build and deployment → Source: GitHub Actions**,
  then set the custom domain to `thetechhub.io` and tick **Enforce HTTPS**.
- DNS at the registrar for the apex domain uses A records `185.199.108.153`, `185.199.109.153`,
  `185.199.110.153` and `185.199.111.153` (plus optional AAAA `2606:50c0:8000::153` … `8003::153`),
  and `www` as a CNAME to `pyrsuit.github.io`.
- `docs/CNAME` is copied into `site/` on every build so the custom domain survives redeploys.
