---
name: demo-site-quickbuild
description: Build a fast, one-page static demo website from a Claude Design handoff (or a written description) with the absolute minimum tooling and dev time. Use this skill whenever the user asks for a demo site, sample site, prospect/customer preview site, mockup site, or hands off a Claude Design export and wants it live quickly — even if they don't say the word "demo." Prioritizes speed of development and deployment simplicity over production polish. Do NOT use for production sites, sites needing a working backend, or multi-page builds.
---

# Demo Site Quickbuild

Turn a Claude Design handoff (or a plain description) into a one-page demo website as fast as possible. The output is a **single self-contained `index.html` file** that can be dragged onto Netlify Drop, pushed to GitHub Pages, or opened directly in a browser — no build step, no dependencies, no config.

## The Prime Directive

**Speed of development beats everything else.** This site exists to show a prospective customer what their site *could* look like. It is not production software. Every decision should be answered with: "What gets a good-looking, working page finished fastest?"

## Tech Stack (non-negotiable)

- **One file:** `index.html` containing all HTML, CSS (in a `<style>` block), and JS (in a `<script>` block).
- **Vanilla HTML/CSS/JS only.** No React, no Vue, no build tools, no npm, no `package.json`, no bundler, no TypeScript, no Tailwind CLI.
- **No package installation of any kind.** Do not run `npm install`, `pip install`, or create `node_modules`. If you find yourself scaffolding a project, stop — you're off track.
- **CDN allowed only if genuinely needed** (e.g., a Google Fonts `<link>`). Prefer system font stacks to avoid even that.
- **Images:** if the Claude Design handoff includes image assets, either (a) inline small ones as base64 data URIs, or (b) place them in an `assets/` folder next to `index.html` and use relative paths. Never reference temporary, session-scoped, or external hotlinked URLs — those break after deployment.

## Translating a Claude Design Handoff

When given a Claude Design export (often a React/framework project):

1. **Do not run or build the exported project.** Read it as a visual/content spec only.
2. Extract: layout structure, section order, color palette, typography choices, copy/text, and any images.
3. Re-implement it as the single vanilla HTML file. Flatten components into plain markup.
4. If the export includes interactivity (menus, carousels, tabs), reproduce only what's visible on a demo walkthrough. A hamburger menu should open/close; a carousel can be static images side by side if that's faster.
5. If something in the export is ambiguous or missing, make a reasonable choice and move on. Do not ask clarifying questions for cosmetic details.

## Page Requirements

- **One page.** Anchor-link navigation (`#about`, `#services`, `#contact`) instead of separate pages.
- **Responsive.** Must look right on a phone and a desktop:
  - Include `<meta name="viewport" content="width=device-width, initial-scale=1">`.
  - Use flexbox/grid with `flex-wrap` or `auto-fit` grids; avoid fixed pixel widths on containers.
  - Test mentally at ~375px and ~1440px. Nav should collapse or wrap gracefully on mobile.
  - Font sizes and tap targets comfortable on mobile (body ≥16px, buttons ≥44px tall).
- **Fast loading.** No large libraries, no video backgrounds, no web-font families beyond one or two weights. Total page weight should be well under 1 MB excluding photos.
- **Contact form is display-only.** Render the form fields and a submit button, but wire the button to `event.preventDefault()` and show a simple inline "Thanks! This is a demo — form submissions are disabled." message. No backend, no email service, no form provider signup.

## Explicitly Skip

Do not spend any time on:

- SEO/GEO: meta descriptions, Open Graph tags, structured data/JSON-LD, sitemaps, robots.txt, canonical tags
- Analytics or tracking scripts
- Favicons beyond nothing at all (browser default is fine)
- Accessibility audits beyond basic semantic HTML (use real `<button>`, `<nav>`, `<h1>` tags — that's it)
- Performance tooling (Lighthouse, minification, image optimization pipelines)
- Cookie banners, privacy pages, legal footers
- Cross-browser testing beyond modern evergreen browsers

If the user later wants any of these, that's a different (production) task, not this skill.

## Workflow

1. Ingest the design source (handoff files or description).
2. Write `index.html` in one pass. Don't scaffold, don't iterate on architecture — there is no architecture.
3. If there are image assets, copy them into `assets/` and confirm every `src` resolves relatively.
4. Quick self-check: open/read the file and verify (a) viewport meta present, (b) form submit is intercepted, (c) no external references except approved CDNs, (d) layout uses responsive units.
5. Deploy to the shared **Sarecta/demo** repo (see next section) and give the user the live URL.

## Deploying to GitHub Pages (Sarecta/demo shared repo)

All demo sites live in one repo: **`Sarecta/demo`**, which already has GitHub Pages enabled (main branch, root). Deploying a new demo is just committing a new folder — never create a new repo, never touch Pages settings.

**Folder naming:** `<prospect-name>` in kebab-case (e.g., `acme-plumbing`). One folder per prospect, containing `index.html` (and `assets/` if needed) at the folder root.

```bash
# Clone (first time) or pull (repo already cloned)
gh repo clone Sarecta/demo || (cd demo && git pull)
cd demo

# Add the new demo as its own folder
mkdir <prospect-name>
cp /path/to/built/index.html <prospect-name>/
# copy assets/ too if the demo has one

git add <prospect-name>
git commit -m "Add demo: <prospect-name>"
git push
```

The public URL will be:

```
https://sarecta.github.io/demo/<prospect-name>/
```

Pages rebuilds automatically on push and usually goes live within 1–3 minutes. Verify before handing over the link:

```bash
curl -s -o /dev/null -w "%{http_code}" https://sarecta.github.io/demo/<prospect-name>/
```

A `200` means it's live (a `404` right after pushing usually just means the build hasn't finished — wait a minute and retry). Always paste the final URL back to the user as the last line of your response.

**Rules for the shared repo:**
- Only add or modify files inside your own prospect's folder. Never edit other folders, the repo root, or `.nojekyll`.
- If the folder already exists, this is an update to an existing demo — overwrite its contents rather than creating `<prospect-name>-2`.
- Updating a demo later: same workflow — pull, edit the folder, commit, push. Same URL, no reconfiguration.
- The repo is public, so don't commit anything sensitive (real customer data, credentials, internal pricing).

**Troubleshooting:**
- `gh` not authenticated → run `gh auth status`; if logged out, tell the user to run `gh auth login` (needs an account with **write access** to `Sarecta/demo`).
- Push rejected (non-fast-forward) → `git pull --rebase` then push again; someone else added a demo since your clone.
- 404 persists after several minutes → confirm the file is named exactly `index.html` directly inside the prospect folder, not nested deeper, and that the push actually landed on `main`.

**Fallback (no GitHub access):** drag the demo folder onto app.netlify.com/drop for an instant public URL with no account setup.

## Done Means

A prospective customer can open the URL on their phone in the parking lot and it loads instantly, looks professional, and every visible element responds — even though nothing is wired to a real backend.
