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
- **Deep links:** `/?open=<about|resume|projects|photos|stickies|terminal|<project-slug>|dashboard|screensaver>`.
  `?sky=<dawn|day|golden|dusk|night>,<clear|cloudy|overcast|fog|drizzle|rain|storm|snow>`
  pins the desktop's time of day and weather for demos.
- **Canonical URL** is `https://www.majincheng.com`; the bare domain
  redirects to it.
- **SEO and accessibility:** the home page ships a visually hidden
  plain-text copy of the content; sitemap, `robots.txt`, `llms.txt` and
  JSON-LD are generated.

### Shell
- Menu bar with per-app menus and a clock.
- Desktop icons: Macintosh HD, About Me, Résumé, Projects, Photos,
  Stickies, Terminal. Right-clicking the empty desktop offers to change
  or reset the desktop picture, Exposé and the screensaver.
- Dock with magnification, running triangles, a Dashboard toggle and a
  trash can.
- Window manager: drag, resize from five edges, z-order, minimize into
  the Dock with a Genie effect (SVG displacement map; Safari, phones and
  reduced motion get a plain shrink), zoom, windows that grow from the
  icon that opened them, and windows that can be thrown and bounce off
  the screen edges.
- Exposé: F9, the bottom-left hot corner or View → Exposé.
- Screensaver: a Ken Burns slideshow of the Photos library after two
  idle minutes.
- Sky: the wallpaper follows the light and weather in San Jose
  (Open-Meteo), with rain, storms, snow and fog drawn behind the windows
  and the temperature in the menu bar.
- Presence: the menu bar counts who's on the desktop, and other
  visitors' cursors drift across it.
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
- **Photos:** the 43 Unsplash photos, grouped by year in justified rows.
  Fetched at build time; Unsplash blocks Vercel's build servers, so
  builds usually fall back to the committed snapshot in
  `src/data/photos.json` (set `UNSPLASH_ACCESS_KEY` to use the official
  API instead). The viewer shows the whole photo, resizes the window to
  its proportions, and can set it as the desktop picture.
- **Stickies:** a guestbook of notes in the style of Mac OS X Stickies,
  stored in Supabase. Notes show right away; each visitor gets one,
  enforced by a salted hash of their IP address (a unique index in the
  database) plus a browser flag that turns the button into "Note Posted
  ✓". People behind one shared address share a note; if the database
  can't see an address, only the browser flag applies. Checked against
  the live project: a second note from the same IP gets a 409 and a
  friendly "You've already left a note".
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
  every push to `main` and weekly, and commits changed images. It also
  runs `npm run photos:update` to refresh the Photos snapshot.

## 2. Architecture, stack and agreements

### Stack
- Astro 5 + Tailwind 4. The desktop is one `client:only` React 19 island,
  with zustand for state and motion for animation.
- Hosted on Vercel project `jincheng-protafolio`; merging to `main`
  deploys. DNS is at GoDaddy.
- Supabase project `hszogpoyyqgwjuznbegd` backs Stickies and presence.
  Vercel holds its URL and publishable key as `NEXT_PUBLIC_SUPABASE_URL`
  and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (all environments; Astro
  exposes the `NEXT_PUBLIC_` prefix through `vite.envPrefix`). Preview
  deployments use the same project as production, so anything posted
  while testing a PR is real data.
- The project has `supabase/schema.sql` and
  `supabase/migrations/20260925_one_note_per_visitor.sql` applied.
  `schema.sql` always describes the full current state for a new
  project; changes to an existing one go in a new dated file under
  `supabase/migrations/`, written so it can be rerun, and the owner runs
  it by hand in the SQL editor. The editor warns about "destructive
  operations" for `drop policy` / `drop trigger` lines; that's expected.
- Free Supabase projects pause after a week without activity; Stickies
  and presence then hide themselves until it's resumed.

### Code map
- `src/os/store.ts`: window map + z-order array; theme, Spotlight,
  Dashboard, Exposé, screensaver, desktop picture and online count.
- `src/os/Expose.tsx`, `Screensaver.tsx`, `Sky.tsx` + `weather.ts`,
  `genie.ts`, `Presence.tsx` + `social.ts`: the features above.
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
- Merge with merge commits. When PRs are stacked, merge from the bottom
  up. Delete a branch once it's merged; only `main` should be left.
- Try UI changes in a real browser before calling them done. The owner's
  Chrome has reduced motion on, so animation work needs a temporary
  bypass to see (and the reduced-motion fallback checked separately).

### Decisions
- No link back to a classic site; Chinese is on hold.
- Don't change ocra for now; it will be redesigned.
- The "Ask me" AI assistant is on hold.
- Stickies has no review step and no login, by choice; one note per
  visitor is the only rule. To hide a note, set `approved` to false in
  the Supabase Table editor.

## 3. Open issues and next steps

The issues listed in the first handoff are resolved: `/zh/` paths with a
trailing slash redirect, the Security Checkpoint no longer blocks
non-browser requests, the canonical URL matches the `www` domain, Assay's
links point at `assay-sql.vercel.app`, and the stray `majincheng` Vercel
project is gone. PRs #13–#17 are merged and every other branch is
deleted.

1. **Delete the test notes.** Two notes starting `[TEST]` and `[TEST 2]`
   were left while checking Stickies. Visitors can't delete, so remove
   them in Supabase → Table Editor → notes. `[TEST 2]` holds the owner's
   IP slot until it's gone.
2. **Moderation, if Stickies attracts spam.** Options discussed: an email
   (or Telegram) notification per note with signed approve/delete links
   via a Supabase Edge Function; an owner-only review app in JM/OS behind
   Supabase Auth; automatic filtering in front of either.
3. **Photos freshness.** Without `UNSPLASH_ACCESS_KEY`, new Unsplash
   uploads reach the site only when the weekly workflow refreshes the
   snapshot, and only if GitHub's runners aren't blocked too.
4. **Branch hygiene.** Turning on "Automatically delete head branches"
   in the GitHub repo settings would make the cleanup above automatic.
5. **Possible next work:** phone polish; persisting windows across
   reloads; automated tests for the window manager; the Chinese site and
   the AI assistant later; an ocra review-replay app once ocra's redesign
   is done.
