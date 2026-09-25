# Handoff: majincheng.com (JM/OS)

State of the project as of 2026-09-25, for picking the work up in a new
session. Conventions and code layout are in [AGENTS.md](AGENTS.md).

## 1. What's done

### Site
- **majincheng.com is JM/OS**, a Mac OS X Aqua–style desktop in the
  browser. The classic one-page site is retired.
- **Chinese site is offline.** The `zh` copy in `src/i18n/content.ts` and
  `src/content/projects/zh/` are kept for later.
- **Project pages** at `/projects/<slug>/` remain for search and sharing,
  with an "Open in JM/OS" link.
- **Deep links:** `/?open=<about|resume|projects|photos|terminal|<project-slug>|dashboard>`.
- **SEO and accessibility:** the home page ships a visually hidden
  plain-text copy of the content; sitemap, `robots.txt`, `llms.txt` and
  JSON-LD are generated.

### Shell
- Menu bar with per-app menus and a clock.
- Desktop icons: Macintosh HD, About Me, Résumé, Projects, Photos,
  Terminal.
- Dock with magnification, running triangles, a Dashboard toggle and a
  trash can.
- Window manager: drag, resize from five edges, z-order, minimize into
  the Dock, zoom, and windows that grow from the icon that opened them.
- Spotlight (⌘K); ⌥W / ⌥M / ⌥T close, minimize and open a terminal
  (browsers reserve ⌘W and ⌘T).
- Boot screen once per session; light and dark appearance.

### Apps
- **About:** short and long bio.
- **Résumé:** a Pages-style HTML document (grey canvas, white two-column
  page) with zoom and Print…, whose print styles output only the page.
  All PDF entry points are gone.
- **Projects:** Finder-style list filtered by status, plus a detail
  window per project.
- **Browser:** an iframe window for live demos.
- **Terminal:** `help`, `whoami`, `ls`, `open`, `projects`, `contact`,
  `theme`, `neofetch`, tab completion and history.
- **Photos:** the 43 Unsplash photos, fetched at build time from
  Unsplash's public profile endpoint, grouped by year in justified rows.
  The viewer shows the whole photo and resizes the window to its
  proportions.
- **Dashboard:** clock, calendar, San Jose weather (Open-Meteo), recent
  GitHub activity and a sticky note.

### Phones
iOS-style home screen: a four-column icon grid and a four-slot Dock.
Apps open full screen with a "‹ Home" button, and the Dock hides while
an app is open.

### Content
- Projects, in order: ocra (in progress), Assay, majincheng.com.
- Positioning: title is "Software Engineer"; the copy leads with
  bringing AI agents into each stage of quality control and building
  developer tooling, not "mainly testing". The bio covers US experience
  only and mentions bouldering (V6) and photography.

### Automation
- `npm run preview:capture` builds the site, screenshots each project's
  `capture` URL into `src/content/projects/covers/*.jpg`, and captures
  the home page into `public/og.png`.
- Captures use a frozen 9:41 clock and reduced motion, skip pages that
  answer with an HTTP error, and only overwrite images whose pixels
  change by more than 0.1%.
- The "Update project previews" workflow runs the script on macOS after
  every push to `main` and commits changed images.

## 2. Architecture, stack and agreements

### Stack
- Astro 5 + Tailwind 4. The desktop is one `client:only` React 19 island,
  with zustand for state and motion for animation.
- Hosted on Vercel project `jincheng-protafolio`; merging to `main`
  deploys. DNS is at GoDaddy.

### Code map
- `src/os/store.ts`: window map + z-order array; theme, Spotlight and
  Dashboard state.
- `src/os/registry.tsx`: every app, lazily loaded; `dockApps` and
  `mobileDockApps` choose what the Dock shows.
- `src/os/apps/*`: one component per app.
- `src/os/os.css`: the Aqua theme.
- `src/pages/index.astro`: assembles `OSData` at build time from
  `content.ts`, the projects collection and `src/lib/photos.ts`.
- Projects: `src/content/projects/<lang>/<slug>.md` with `order`,
  `status`, `cover`, `capture`, `demo` and `repo`.

### Assets
The icons, fonts (Lucida Grande, Apple Garamond, Monaco) and the stones
wallpaper are copied from ryOS. Using them was a deliberate choice after
discussing the Apple/Adobe copyright risk; `NOTICE` records their origin.
The `src/os` code is original and only borrows architecture ideas from
ryOS (AGPL-3.0).

### Working agreements
- English for commits, PRs, comments and docs; only the Chinese site
  copy is in Chinese.
- Conventional Commits, one logical change per commit.
- Branch → PR → merge after CI passes. The owner sometimes merges PRs
  directly on GitHub, so fetch `main` before assuming a PR is still open.

### Decisions
- No link back to a classic site; Chinese is on hold.
- Don't change ocra for now; it will be redesigned.
- The "Ask me" AI assistant is on hold.

## 3. Open issues and next steps

1. **`/zh/` returns 404 instead of redirecting.** `vercel.json` defines
   308 redirects for `/zh`, `/zh/:path*` and `/os`, but
   `https://www.majincheng.com/zh/` returns 404 NOT_FOUND. Check whether
   Vercel picks up `vercel.json` for this project and whether the rules
   match the trailing-slash form. Verify `/os` at the same time.
2. **Non-browser requests hit a Vercel Security Checkpoint.** `curl`
   against any path gets 403 "Vercel Security Checkpoint". This can block
   link-preview fetchers (LinkedIn, Slack), search engines and `llms.txt`
   readers. Check the project's Firewall settings (Attack Challenge Mode
   or bot protection).
3. **Primary domain direction.** `majincheng.com` currently redirects to
   `www`, while the code's canonical URL is the bare domain. Flip it in
   Vercel → Domains, and update the GoDaddy records to the values Vercel
   recommends.
4. **Assay is down.** `sql-script-depoly.vercel.app` returns 404
   `DEPLOYMENT_NOT_FOUND` and needs fixing in Vercel. Until then the
   capture workflow skips it and keeps the last good cover, but the card's
   demo link leads to a 404.
5. **Cleanup.** Confirm the stray Vercel project `majincheng` has been
   deleted.
6. **Possible next work:** phone polish; persisting windows across
   reloads; automated tests for the window manager; the Chinese site and
   the AI assistant later; an ocra review-replay app once ocra's redesign
   is done.
