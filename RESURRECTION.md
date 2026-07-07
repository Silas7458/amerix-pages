# RESURRECTION - rebuild this site on a new machine

Written S#280 (2026-07-07). amerix-pages is the lightest app in the fleet: a
pure static HTML site (visualizations + landing pages for Amerix Medical
Consulting). There is no build system, no data pipeline, and no secrets. This
one document plus the two git remotes and the two cloud deploys below restore
full functionality and fidelity. No MANUAL.md exists because there is no
operational logic to manual-ize - editing a page is "edit the HTML, commit,
deploy" and nothing more.

## What survives a dead laptop (nothing to do)

| Piece | Where it lives | Notes |
|---|---|---|
| Site source | GitHub `Silas7458/amerix-pages` (**public**, remote `origin`) + GitLab `Amerix-dev/amerix-pages` (remote `gitlab`, mirror) | Two independent remotes; either restores the other |
| Live site (Vercel) | Vercel project `amerix-pages` -> `https://amerix-pages.vercel.app/` | Keeps serving with the laptop dead |
| Live site (GitHub Pages) | `https://silas7458.github.io/amerix-pages/` | Auto-built by `.github/workflows/deploy.yml` on every push to `main` |
| Deploy secrets | none | Static files only. GitHub Pages uses the repo's own Actions token; Vercel uses the account login. There is nothing to `env pull`. |

Nothing here is truly laptop-bound. The only local-only piece is the Vercel CLI
link (`.vercel/project.json`, gitignored) and the habit of running a manual
Vercel deploy - both trivially rebuilt below.

## Deploy topology - the one fact that bites

There are TWO independent live deploys and they can DRIFT apart:

- **GitHub Pages** auto-deploys from `origin/main`. Push to GitHub `main` and
  the workflow (`.github/workflows/deploy.yml`, `actions/deploy-pages@v4`)
  rebuilds it. What is NOT pushed to GitHub is NOT on GitHub Pages.
- **Vercel** is deployed by hand with `vercel --prod` from the local working
  tree. It is NOT wired to git auto-deploy. Vercel serves whatever was last
  pushed from a laptop, which may be AHEAD of or BEHIND either git remote.

As of this writing that drift is live and observable: `ain-dashboard.html` was
served on Vercel (HTTP 200) while returning 404 on GitHub Pages, because its
commit had been deployed to Vercel manually but never `git push`-ed. Lesson:
after editing, do BOTH `git push origin main` (updates GitHub Pages) AND
`vercel --prod` (updates Vercel), or the two live URLs disagree.

## Rebuild + deploy (10 minutes)

```
# 1. Clone (either remote restores the site)
git clone https://github.com/Silas7458/amerix-pages.git
cd amerix-pages

# 2. Re-link Vercel (regenerates the gitignored .vercel/project.json)
npx vercel link          # choose scope + existing project "amerix-pages"

# 3a. Deploy to Vercel
npx vercel --prod --force

# 3b. Deploy to GitHub Pages (just push - the workflow does the rest)
git push origin main
```

No `npm install`, no build step - the files ARE the site. To preview locally,
open `index.html` in a browser or run any static server
(`npx serve .` / `python -m http.server`).

Restore the GitLab mirror if it is missing:

```
git remote add gitlab https://gitlab.com/Amerix-dev/amerix-pages.git
git push gitlab main
```

## Deploy identity (pin this)

| Field | Value |
|---|---|
| Vercel project name | `amerix-pages` |
| Vercel live URL | `https://amerix-pages.vercel.app/` |
| Vercel org / project IDs | in `.vercel/project.json` - **gitignored, local-only**; regenerate with `npx vercel link` (do not hand-copy the IDs; the CLI writes them) |
| GitHub repo | `Silas7458/amerix-pages` (public), remote `origin` |
| GitHub Pages URL | `https://silas7458.github.io/amerix-pages/` |
| GitHub Pages source | `.github/workflows/deploy.yml` - deploys the repo root on push to `main` |
| GitLab mirror | `Amerix-dev/amerix-pages`, remote `gitlab` |

## Verify end-to-end (5 minutes)

After a rebuild + deploy, confirm both live targets serve the site:

1. `curl -sL https://amerix-pages.vercel.app/` returns HTTP 200 and the
   title `Amerix Medical Consulting`.
2. `curl -sL https://silas7458.github.io/amerix-pages/` returns HTTP 200 and
   the same title.
3. Each linked page loads on Vercel (all should be HTTP 200):
   `/tandem-team.html`, `/toolkit-landscape.html`, `/toolkit.html` (redirects
   to toolkit-landscape), and `/ain-dashboard.html`.
4. Open `index.html` in a browser: the two cards ("Tandem Team" and "Amerix
   Development Toolkit") both navigate to their live pages.
5. If Vercel and GitHub Pages disagree on a page, one deploy was skipped - see
   "Deploy topology" above and run the missing half.

## Content inventory (what the repo contains)

| File | Lines | What it is |
|---|---|---|
| `index.html` | 116 | Landing page. Card grid linking to the two headline visualizations. Title: "Amerix Medical Consulting". |
| `tandem-team.html` | 1440 | "The Tandem Team - How It Works" - canvas-animated interactive walkthrough of the AI team (roles, workflows, comms flows). Self-contained inline CSS/JS. |
| `toolkit-landscape.html` | 834 | "Amerix Development Toolkit - Landscape" - living dashboard of the full dev stack (tools, MCP servers, agents, hooks). Carries its own internal version marker + changelog inside the page. |
| `toolkit.html` | 11 | Meta-refresh redirect to `toolkit-landscape.html` (a stable short-URL alias). |
| `ain-dashboard.html` | 366 | "Amerix Intelligence Network - Dashboard". Standalone page (not linked from `index.html`). |
| `.github/workflows/deploy.yml` | - | GitHub Pages deploy workflow (checkout -> configure-pages -> upload-pages-artifact -> deploy-pages). |
| `.gitignore` | - | Ignores `.vercel/` (the local Vercel link). |
| `.vercel/project.json` | - | Local Vercel link (gitignored). Not in the repo. |

## Editing a page (there is no build)

1. Edit the target `.html` file directly (all styling and scripting is inline;
   there are no shared assets, no bundler, no dependencies).
2. `git commit` the change.
3. Deploy BOTH targets: `git push origin main` (GitHub Pages) and
   `npx vercel --prod` (Vercel). Skipping one leaves the two live URLs out of
   sync (see "Deploy topology").
4. `toolkit-landscape.html` tracks its own version/changelog inside the page -
   bump those in the same edit if you change the toolkit content.

## Credential + account inventory (names only - no values anywhere)

Static hosting needs almost nothing. There are no app secrets, no database, no
env vars to pull.

| Account | Used for | How to recover on a new machine |
|---|---|---|
| Vercel account | `vercel link`, `vercel --prod` deploys of project `amerix-pages` | `vercel login` with Silas's Vercel identity, then `vercel link` |
| GitHub `Silas7458` | `origin` remote + GitHub Pages (Actions runs with the repo's own token) | new HTTPS PAT from github.com -> settings -> tokens |
| GitLab `Amerix-dev` group | `gitlab` mirror remote | new HTTPS token from gitlab.com profile |

There are no secret VALUES in this repo, by design. This is a public repo of
static marketing/visualization pages - keep it that way: never add a database
URL, API key, token, client name, or patient data to any file here.

## Related-but-separate systems (not restored by this doc)

- `kantime-dashboard` (its own repo / Vercel project / Neon DB) - the data app.
  See its `RESURRECTION.md` + `MANUAL.md`.
- Team infrastructure (Claude session-context files, Discord bots, NotebookLM
  auth) - separate backup story, unrelated to this static site.
