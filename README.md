# The Tech Hub

Source for [thetechhub.io](https://thetechhub.io), a blog of reflections on the technical
and interpersonal challenges of working as a software developer and team lead.

The site is built with [Zensical](https://zensical.org) and deployed to GitHub Pages.

## Requirements

- [uv](https://docs.astral.sh/uv/). It installs Python 3.15 and all dependencies for you,
  so you don't need `pip` or a virtual environment of your own.

## Getting started

```bash
git clone git@github.com:pyrsuit/the-tech-hub.git
cd the-tech-hub
uv sync                     # install the exact versions from uv.lock
uv run pre-commit install   # set up the git pre-commit hook (once per clone)
uv run zensical serve       # preview at http://localhost:8000
```

The preview reloads when you save a file and also shows draft posts.

## Pre-commit hooks

[pre-commit](https://pre-commit.com) runs these checks on the staged files before each commit
(configured in `.pre-commit-config.yaml`):

- **Ruff** lints (`ruff check --fix`) and formats (`ruff format`) Python code.
- Basic file checks: trailing whitespace (Markdown line breaks are kept), a final newline,
  valid YAML and TOML, no large files and no merge-conflict markers.
- **uv-lock** makes sure `uv.lock` matches `pyproject.toml`.

If a hook changes a file, the commit stops. Review the changes, `git add` them and commit again.

## Commands

| Command | What it does |
| --- | --- |
| `uv sync` | Install dependencies from `uv.lock` |
| `uv run zensical serve` | Live preview at http://localhost:8000 (drafts included) |
| `uv run zensical build --clean` | Production build into `site/` (drafts excluded) |
| `uv run zensical build --clean --strict` | Same, but fails on warnings. Run before committing. |
| `uv run pre-commit run --all-files` | Run all pre-commit hooks on every file |
| `uv run pre-commit autoupdate` | Update the hook versions in `.pre-commit-config.yaml` |
| `uv run ruff check .` | Lint Python code |
| `uv add <package>` | Add a dependency (updates `pyproject.toml` and `uv.lock`) |
| `uv lock --upgrade-package zensical` | Upgrade Zensical |

## Project layout

```
.
├─ .github/workflows/
│  ├─ docs.yml                  # build + deploy on push to main
│  └─ lint.yml                  # ruff lint on pushes to main and pull requests
├─ docs/
│  ├─ index.md                  # homepage intro; the post list is rendered below it
│  ├─ about.md                  # "About me" page
│  ├─ CNAME                     # custom domain (thetechhub.io), do not delete
│  ├─ assets/images/            # images used in posts
│  └─ posts/                    # one Markdown file per post
├─ .pre-commit-config.yaml      # pre-commit hooks (ruff, file checks, uv-lock)
├─ pyproject.toml               # dependencies
├─ uv.lock                      # locked versions, always commit
└─ zensical.toml                # site configuration
```

`site/` (build output) and `.venv/` (uv environment) are generated and git-ignored.

## Writing a post

Create `docs/posts/YYYY-MM-DD-short-slug.md`:

```markdown
---
date:
  created: 2026-11-22
categories:
  - Interpersonal        # one of: Technical, Interpersonal, Leadership, Process
tags:
  - feedback
slug: giving-feedback-to-a-senior-dev   # optional
draft: true                             # remove when ready to publish
---

# Giving feedback to a more senior developer

A two or three sentence summary shown in the post list.

<!-- more -->

The full post…
```

- Use exactly one `#` heading per post. It becomes the title.
- Put `<!-- more -->` after the summary so the homepage shows only an excerpt.
- Only the four categories above are allowed. Any other category fails the build
  (`categories_allowed` in `zensical.toml`).
- Tags are lowercase and hyphenated, e.g. `code-review`.
- New posts start with `draft: true`. Drafts appear in `serve` but not on the live site.
- Images go in `docs/assets/images/` and are referenced relative to the post,
  e.g. `![Alt text](../assets/images/diagram.png)`.
- Posts describe real workplace situations, so leave out names of colleagues,
  employers and clients.

## Deployment

Every push to `main` runs `.github/workflows/docs.yml`, which builds the site with
`uv sync --locked` and `zensical build` and deploys it to GitHub Pages.
**Pushing to `main` publishes the live site**, so work on a branch and merge when ready.

`--locked` makes the build fail if `uv.lock` doesn't match `pyproject.toml`. Always commit
both files together.

`.github/workflows/lint.yml` runs `ruff check` on every push to `main` and on every pull
request, using the Ruff version locked in `uv.lock`.

### One-time GitHub setup

1. **Settings → Pages → Build and deployment → Source:** GitHub Actions.
2. Set the custom domain to `thetechhub.io` and tick **Enforce HTTPS**.
3. DNS at the registrar:
   - A records for the apex domain: `185.199.108.153`, `185.199.109.153`,
     `185.199.110.153`, `185.199.111.153`
   - `www` as a CNAME to `pyrsuit.github.io`

## Upgrading Zensical

Zensical is pre-1.0 and changes quickly, so its version is pinned in `uv.lock`.

```bash
uv lock --upgrade-package zensical
uv sync
uv run zensical build --clean --strict
uv run zensical serve    # check the homepage, a post, the archive and a category page
```

Commit `pyproject.toml` and `uv.lock` together.

## Copyright

Copyright © 2026 Zsófia Kálosi. All rights reserved.
