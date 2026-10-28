# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Aron Berger's personal portfolio site (https://aronberger.com), built with Hugo and the [Toha](https://github.com/hugo-toha/toha) theme (v4), pulled in as a **Hugo module** (see `go.mod`), not a git submodule or `themes/` directory. There is no application code or test suite; the "build" is the Hugo site build.

## Commands

Tool versions are pinned in `mise.toml` (hugo-extended, node, go). Tasks:

```sh
mise run install        # hugo mod tidy && hugo mod npm pack && npm install
mise run server         # install, then `hugo server -w` (local preview)
mise run update         # hugo mod get -u (update Toha to latest release), then install
mise run update-to-main # pin Toha to its main branch, then install
```

Build the site (what CI runs): `hugo --minify` (output in `public/`, which is gitignored).

`hugo mod npm pack` regenerates `package.json` from the theme's dependencies — the `comments` block in `package.json` maps each dev dependency to the module that requires it. Don't hand-edit those dependency versions; update the theme instead.

## Architecture

- **Content lives in YAML data files, not Markdown.** The homepage is assembled by Toha from `data/en/`:
  - `data/en/author.yaml` — name, contact, summary
  - `data/en/site.yaml` — copyright, meta description, OpenGraph, `customMenus` (the navbar "Resume" link points to `/files/Aron-Berger-Resume.pdf`)
  - `data/en/sections/*.yaml` — one file per homepage section. Each has a `section:` header with `enable` (show/hide) and `weight` (order). Currently enabled: about, education, experiences, skills. The rest (accomplishments, achievements, projects, publications, featured/recent posts) are disabled leftovers from the theme's example site.
- `hugo.yaml` holds site config and theme feature flags (`params.features.*`). Blog and notes are disabled, so `content/posts` and `content/notes` are unused example content. The `bn` (Bengali) language and `data/bn/` are also disabled example leftovers.
- **Theme overrides**: files in `layouts/` shadow the theme's templates of the same path (e.g. `layouts/partials/sections/about.html`, `experiences.html`, `footer.html`, `cards/*.html`). When changing how a section renders, edit/create the override here rather than touching the module. To see the original, look in the Hugo module cache (`hugo mod graph` / `go env GOMODCACHE`).
- **Assets**: images referenced by section YAML (e.g. `/images/sections/education/UC.jpeg`) resolve to `assets/images/...`; SCSS variable overrides are in `assets/styles/variables.scss`. Downloadable files (the resume PDF, logo) live in `static/files/`.
- `hugo.yaml` `module.mounts` maps `node_modules` assets (flag icons, Mulish font, KaTeX fonts) into `static/`, so `npm install` must have run before building.

## Deployment

- Push to `main` → `.github/workflows/merge-to-main.yml` builds and publishes `public/` to the `gh-pages` branch (GitHub Pages, custom domain via `static/CNAME`).
- `.github/workflows/theme-update.yml` runs nightly, runs `hugo mod get -u` + `hugo mod npm pack`, and opens an automated "Update theme" PR (the `[create-pull-request] automated change` commits in history).
- `netlify.toml`, `.gitlab-ci.yml`, and the Netlify/Lighthouse jobs in `pull-request.yml` (which reference `toha-example-site`) are inherited from the theme's example site and aren't the actual deploy path.
