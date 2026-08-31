---
description: Security-scan, then push this repo to GitHub with a README, About section, and GitHub Pages deployment
argument-hint: <github-repo-url-or-owner/repo>
---

The user wants to publish this local repo to GitHub, fully configured. The target repo is: $ARGUMENTS

If no repo was provided in $ARGUMENTS, ask the user for the GitHub repo (URL or `owner/repo`) before doing anything else — do not guess or reuse a repo from memory.

Follow these steps in order. Each step gates the next — do not skip the security scan, and do not push before it passes.

## 1. Security scan (must pass before anything is uploaded)

Before staging or pushing any file, scan the working tree for confidential data that must never leave the machine:

- Grep broadly for likely secrets: API keys, tokens, passwords, private keys, connection strings, AWS/GCP/Azure credential patterns, `.pem`/`.p12` files, `.env*` files, and any hardcoded auth headers.
- Check `git status` / `git diff` (or the full untracked file list if this is a fresh repo) so you see everything that would be staged — never rely on `git add -A` blindly.
- Read the contents of any file that looks sensitive by name (`.env`, `secrets.*`, `credentials.*`, `*.key`, `*.pem`, config files with embedded connection strings) even if the user says it's fine to skip.
- Check for this project's specific constraint (see CLAUDE.md if present): no real credential collection, no committed secrets, no accidental PII.
- If anything suspicious is found: stop, do not stage/commit/push it, and tell the user exactly what you found and why it's risky. Ask whether to exclude it (e.g. via `.gitignore`) or whether it's a false positive before proceeding.
- If the scan is clean, briefly tell the user what you checked and that it passed before continuing.

## 2. Screenshot

Capture a current screenshot of the site to use in the README:

- Check whether the Playwright MCP tools (`mcp__playwright__*`) are connected this session. If not available, skip this step entirely, don't block the rest of the publish, and mention in the wrap-up that a screenshot wasn't captured (e.g. Playwright MCP not connected).
- Playwright MCP blocks `file://` URLs, so serve the repo over a local static server first, e.g. `python3 -m http.server <port>` from the repo root, backgrounded.
- Navigate to the main page, resize the viewport to a reasonable desktop size (e.g. 1440x900), and take a full-page screenshot. Save it to `assets/screenshot.png` at the repo root (create `assets/` if missing), overwriting any previous one so it stays current.
- Check console messages for errors; ignore benign ones (e.g. a missing favicon 404), but surface anything else to the user.
- Close the browser page and kill the temporary HTTP server when done.
- Add `.playwright-mcp/` (Playwright MCP's own debug-artifact directory, if it appears) to `.gitignore` — it's session debug output, not part of the deliverable.

## 3. README

Create or update `README.md` at the repo root so it actually describes this project (not just a placeholder echo'd line):

- What the project is, in 1-3 sentences.
- How to run/view it (respect the project's actual setup — e.g. this repo is a dependency-free static `index.html`, no build step).
- Any constraints worth surfacing to a reader (e.g. training/demo disclaimers already present in the page).
- If a screenshot was captured in the previous step, embed it with `![Screenshot](assets/screenshot.png)`. If it wasn't (Playwright MCP unavailable) and the repo has no existing screenshot, don't fabricate one — just skip the image.

Keep it concise. Don't invent features that don't exist in the code — base it on what's actually in the repo.

## 4. Push to GitHub

- If this directory is not yet a git repo, `git init` and set the branch to `main`.
- Add the remote named `origin` pointing at the repo from $ARGUMENTS (use SSH form `git@github.com:owner/repo.git` if an SSH key is already set up and authenticated; otherwise fall back to HTTPS and let auth fail visibly rather than guessing at credentials).
  - If `git@github.com` auth isn't already working (test with `ssh -T git@github.com`) and there's no `gh` auth either, stop and ask the user how they want to authenticate (SSH key setup, `gh auth login`, or PAT) rather than attempting to embed credentials anywhere.
- Stage only the files that passed the security scan, commit with a message describing what's being published, and push with `-u origin main` (or the current branch).

## 5. GitHub Pages via GitHub Action

- Add `.github/workflows/deploy.yml` (skip if one already exists — update it instead) that deploys the repo root as a static site via `actions/configure-pages`, `actions/upload-pages-artifact`, and `actions/deploy-pages`, triggered on push to `main` plus `workflow_dispatch`. Don't introduce a build step unless this project actually has one.
- Commit and push the workflow.
- If the `gh` CLI is installed and authenticated, use it to set the repo's Pages build source to "GitHub Actions" (`gh api` Pages endpoint) so the workflow can deploy without a manual dashboard step, and trigger a run.
- If `gh` isn't available/authenticated, tell the user this one manual step is required: repo Settings → Pages → Source → "GitHub Actions" — and offer to set up `gh` for them if they want it automated next time.

## 6. GitHub About section

- Using `gh repo edit` (description and `--homepage`), set:
  - The repo description, if missing or stale, to a short one-line summary matching the README.
  - The homepage URL to the GitHub Pages URL (`https://<owner>.github.io/<repo>/`).
- If `gh` isn't available/authenticated, tell the user to set these manually via the repo's "About" gear icon (top right of the repo homepage) and give them the exact description text and Pages URL to paste in.

## Wrap-up

End with a short summary: what was pushed, the Pages URL, and any manual step still pending (Pages source toggle or About section) if `gh` wasn't available to automate it.
