# Public Gists List — System Architecture

Daily, automated publication of a **Markdown index of Rich Lewis’s public gists** to:
- A **GitHub Gist** (single canonical file), and
- A **project website** at `https://github.richlewis007.com/Public-Gists-from-Rich-Lewis/`.

Update schedule: **9:00 AM Eastern** (cron: `0 13 * * *` UTC).  
Time stamps in the index are rendered in **US Eastern** (handles EST/EDT).

---

## Repo Map

### 1) `Create-Gist-List` (generator + automation)
- **What it does**
  - Runs `create-gist-list.py` to list public gists, render a Markdown table, and print to `stdout`.
  - If configured, **PATCHes** the list gist.
  - Commits the generated Markdown into the website repo (root copy + `/docs/index.md`).

- **Key files**
  - `create-gist-list.py` — generator (uses `requests`, `zoneinfo`, Eastern timestamps).
  - `.github/workflows/update-gist-lists.yml` — runs daily + on demand (uses **uv**).

- **Secrets (in this repo)**
  - `GIST_TOKEN` → scope: **`gist`** (update the list gist).
  - `INDEX_GIST_ID` → ID of the gist file to overwrite.
  - `PUSH_TOKEN` → fine-grained PAT with **Repository → Contents: Read and write** on `Public-Gists-from-Rich-Lewis`.

- **Important env**
  - `GITHUB_USERNAME` (e.g., `RichLewis007`)
  - `TARGET_MD_FILENAME` = `Public-Gists-by-Rich-Lewis.md` (name in the gist)

---

### 2) `Public-Gists-from-Rich-Lewis` (project website)
- **Purpose**: Serves the generated list via **GitHub Pages** (“Deploy from a branch”, **`/docs`** folder).
- **Key files**
  - `docs/index.md` — **generated** (do not edit).
  - `Public-Gists-by-Rich-Lewis.md` — same content at repo root (for GitHub file browsing).
  - `docs/_header.md` — optional header **prepended** by the workflow (safe place for `{% seo %}`).
  - `docs/_footer.md` — optional footer **appended** by the workflow.
  - `docs/_config.yml` — Jekyll config:
    ```yaml
    title: Public Gists from Rich Lewis
    description: Daily list of public gists
    theme: jekyll-theme-cayman
    url: https://github.richlewis007.com
    baseurl: /Public-Gists-from-Rich-Lewis   # ← case-sensitive
    plugins:
      - jekyll-seo-tag
      - jekyll-sitemap
    ```
  - `docs/assets/css/style.scss`: Cayman overrides (compiled by Jekyll; must start with `---` front matter).
  - `docs/_includes/head-custom.html`: favicon/manifest `<link>`s.
  - `docs/site.webmanifest` + `docs/assets/favicons/*`: icons with paths scoped to the project base URL.

> **Case matters**: the live path is `/Public-Gists-from-Rich-Lewis/`. Links using lowercase will 404. (or maybe not if my workaround works)

---

### 3) `RichLewis007.github.io` (user site)
- **Purpose**: Main site using **MkDocs (Material)**, deployed via **GitHub Actions** to Pages.
- **Custom domain**: `github.richlewis007.com` (CNAME → `richlewis007.github.io`).
- **Nav**: Adds a top-level link to `/Public-Gists-from-Rich-Lewis/`.  
  (Optional JS to open in a new tab; otherwise standard link.)

---

## Data Flow

1. **Workflow triggers** (cron or manual) in `Create-Gist-List`.
3. `create-gist-list.py` **fetches public gists**, renders Markdown (Eastern timestamps).
4. If `INDEX_GIST_ID` + `GIST_TOKEN` exist, the **gist is updated**.
5. Workflow **checks out** `Public-Gists-from-Rich-Lewis` using `PUSH_TOKEN` and writes:
   - `Public-Gists-by-Rich-Lewis.md` (root)
   - `docs/index.md` = `[optional _header.md] + generated Markdown + [optional _footer.md]`
   - (Bootstrap `docs/_config.yml` if missing)
6. Commit + push to that repo’s default branch.
7. GitHub Pages (branch-based) **rebuilds** the project site.
8. The user site links visitors to the project site path.

---

## Workflows (essentials)

### `create-gist-list/.github/workflows/update-gist-lists.yml` (core steps)
```yaml
permissions:
  contents: write   # needed for commits to the target repo

jobs:
  update-list:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - uses: astral-sh/setup-uv@v6
        with:
          enable-cache: true

      - name: Generate Markdown (and update gist)
        env:
          GITHUB_USERNAME: "RichLewis007"
          GITHUB_TOKEN: ${{ secrets.GIST_TOKEN }}
          INDEX_GIST_ID: ${{ secrets.INDEX_GIST_ID }}
          TARGET_MD_FILENAME: "Public-Gists-by-Rich-Lewis.md"
        run: |
          uv run --with requests python gist-index.py > /tmp/Public-Gists-by-Rich-Lewis.md

      - name: Checkout target repo (Public-Gists-from-Rich-Lewis)
        uses: actions/checkout@v4
        with:
          repository: RichLewis007/Public-Gists-from-Rich-Lewis
          fetch-depth: 0
          path: target-repo
          token: ${{ secrets.PUSH_TOKEN }}

      - name: Update target repo (root + docs)
        working-directory: target-repo
        shell: bash
        run: |
          set -euo pipefail
          mkdir -p docs
          cp /tmp/Public-Gists-by-Rich-Lewis.md ./Public-Gists-by-Rich-Lewis.md
          HEADER=docs/_header.md
          FOOTER=docs/_footer.md
          cat ${HEADER:+$HEADER} /tmp/Public-Gists-by-Rich-Lewis.md ${FOOTER:+$FOOTER} > docs/index.md

          if [ ! -f docs/_config.yml ]; then
            printf "%s\n%s\n%s\n%s\n%s\n"               "title: Public Gists from Rich Lewis"               "description: Daily index of public gists"               "theme: jekyll-theme-cayman"               "url: https://github.richlewis007.com"               "baseurl: /Public-Gists-from-Rich-Lewis"               > docs/_config.yml
          fi

          git config user.name  "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add -A
          if git diff --cached --quiet; then
            echo "No changes to commit."
          else
            git commit -m "chore: update gist list (root + docs) [skip ci]"
            git push
          fi
```

---

## Local Usage (with `uv`)

```bash
# env
export GITHUB_USERNAME="RichLewis007"
export INDEX_GIST_ID="<your_gist_id>"
export GITHUB_TOKEN="<gist_scope_pat>"

# run
uv run --with requests python gist-index.py > /tmp/Public-Gists-by-Rich-Lewis.md
```

- Prints the Markdown to `stdout`.
- Updates the gist if both `INDEX_GIST_ID` and `GITHUB_TOKEN` are set.
- Do **not** edit `docs/index.md` by hand; it’s generated by the workflow.

---

## Styling & Assets (project site)

- **SCSS overrides**: `docs/assets/css/style.scss`
  - Must start with:
    ```
    ---
    ---
    ```
  - Imports the theme and overrides table/layout styles.  
- **Favicons/PWA**:
  - Files in `docs/assets/favicons/*`
  - `docs/_includes/head-custom.html` adds `<link rel="icon">` and manifest tags.
  - `docs/site.webmanifest` has `start_url`/`scope` pointing to `/Public-Gists-from-Rich-Lewis/` and icon paths under that prefix.

---

## Troubleshooting

- **404 at `/public-gists-from-Rich-Lewis/`**  
  Use the correct **case**: `/Public-Gists-from-Rich-Lewis/`.

- **Table shows as pipes, not a table**  
  Ensure the generated Markdown contains a plain pipe table, without wrapping HTML that blocks Markdown parsing.

- **Styles not applied**  
  Confirm `docs/assets/css/style.scss` exists and compiles to `/assets/css/style.css`.  
  The SCSS file must include the two `---` front-matter lines.

- **Pages didn’t update**  
  - The workflow committed **no diff** (identical content).  
  - It pushed to the **wrong branch** vs. the Pages branch/folder.  
  - The `github-pages` environment is protecting deployments (approve or allow `main`).

- **Action didn’t trigger**  
  - You pushed to a different branch than the workflow’s `on.push.branches`.  
  - You used `[skip ci]` in the commit message.

- **Caching warning (setup-uv)**  
  Provide `requirements.txt` or `pyproject.toml`/`uv.lock` and set `cache-dependency-glob` accordingly.

---

## Security

- **Never** commit token values.  
- `GIST_TOKEN` scope: **gist** only.  
- `PUSH_TOKEN` scope: **Contents: Read and write** on the **single** target repo.  
- Gist IDs and repo URLs are fine to publish.
