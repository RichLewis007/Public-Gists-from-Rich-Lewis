# Overview — Public Gists Index (What, Why, How)

This system builds a **daily index of your public GitHub gists** and publishes it in two places:
1) A **GitHub Gist** (single, canonical Markdown file), and
2) A small **project website** at `https://github.richlewis007.com/Public-Gists-from-Rich-Lewis/`.

It runs every morning at **9:00 AM Eastern**. Time stamps in the list are shown in **US Eastern** (EST/EDT).

---

## Moving parts (at a glance)

- **`Create-Gist-List`** — *Automation & generator*  
  Runs a Python script that fetches your public gists, renders a Markdown table, and
  - updates the **list gist**, and
  - commits the same content into the website repo (both the repo root and `docs/index.md`).

- **`Public-Gists-from-Rich-Lewis`** — *Project website*  
  GitHub Pages serves the generated list from the `/docs` folder, with a lightweight theme and a few CSS tweaks.

- **`RichLewis007.github.io`** — *User site*  
  My main site (created using the MkDocs static site generator). Its top‑nav has a “Public Gists” link that points to the project website path.

---

## How each daily run works

1. The **GitHub Action** in `Create-Gist-List` starts (cron or manual).
2. `create-gist-list.py` calls the GitHub API, collects my **public** gists, and renders Markdown.
3. If configured, it **PATCHes the list Gist** with that Markdown file.
4. The workflow checks out `Public-Gists-from-Rich-Lewis` and writes:
   - `Public-Gists-by-Rich-Lewis.md` at the **repo root** (easy to read on GitHub),
   - `docs/index.md` for the **website homepage** (optionally wrapped with a `_header.md` and `_footer.md`).
5. That repo’s **GitHub Pages** rebuilds and serves the page at the project path.

That’s it. It's fully unattended after repo secrets are set.

---

## Where everything lives

- **The list in a Gist:** a single Markdown file named `Public-Gists-by-Rich-Lewis.md`.
- **The website page:** `docs/index.md` in `Public-Gists-from-Rich-Lewis` → published at `/Public-Gists-from-Rich-Lewis/`.
- **Look & feel:** a tiny stylesheet at `docs/assets/css/style.scss` makes the table clean and responsive.
- **Favicons & manifest:** files under `docs/assets/favicons/` + links in `docs/_includes/head-custom.html`.

> Note: The live URL path is **case‑sensitive**: `/Public-Gists-from-Rich-Lewis/`.

---

## Configuration files

- **Schedule:** the Action in `create-gist-list` uses a cron expression (`0 13 * * *`, i.e., 9:00 AM ET).  
- **Tokens:** two secrets in `create-gist-list`:
  - `GIST_TOKEN` (scope: `gist`) lets the workflow update the list Gist.
  - `PUSH_TOKEN` (fine‑grained, Contents: read/write) lets the workflow commit to `Public-Gists-from-Rich-Lewis`.
- **Copy behavior:** the workflow writes both the **root** copy and the **website** copy.  
  Optional intro/outro lives in `docs/_header.md` and `docs/_footer.md` (website only).

---

## Operating notes

- Don’t hand‑edit `docs/index.md` as it’s overwritten by the workflow.
- If you rename the project repo, update any links and the `baseurl` in `docs/_config.yml`.
- If the site 404s at a lowercase path, check the casing (`/Public-Gists-from-Rich-Lewis/`).

---

## Quick troubleshooting

- **Action ran, site didn’t change** → likely no content diff, or pushed to the wrong branch vs Pages settings.
- **Styles missing** → ensure `docs/assets/css/style.scss` begins with the two `---` lines (Jekyll front matter).
- **Table shows raw pipes** → ensure the generator outputs a plain Markdown table with no blocking HTML around it.
- **Deployment blocked** → the `github-pages` environment may require approval or branch allow‑listing.
