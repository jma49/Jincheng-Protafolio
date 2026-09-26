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
- **Deep links:** `/?open=<about|resume|projects|photos|ipod|karaoke|stickies|soapbox|terminal|preferences|<project-slug>|dashboard|screensaver>`.
  `?sky=<dawn|day|golden|dusk|night>,<clear|cloudy|overcast|fog|drizzle|rain|storm|snow>`
  pins the desktop's time of day and weather for demos; `?place=<city>`
  pins the visitor's place.
- **Canonical URL** is `https://www.majincheng.com`; the bare domain
  redirects to it.
- **SEO and accessibility:** the home page ships a visually hidden
  plain-text copy of the content; sitemap, `robots.txt`, `llms.txt` and
  JSON-LD are generated.

### Shell
- Menu bar with per-app menus and a clock. It's see-through over the
  desktop picture, with white or black text to suit the picture, and
  opaque over a zoomed window or an app on a phone. The Apple logo takes
  the accent colour, so it follows the desktop picture.
- Desktop icons: Macintosh HD, About Me, Résumé, Projects, Photos,
  Stickies, Terminal. Right-clicking the empty desktop offers to change
  or reset the desktop picture, Exposé and the screensaver.
- Dock: one floating, rounded pane of frosted glass with magnification,
  running dots, a Dashboard toggle and a trash can. It keeps Projects,
  Photos, iPod, Stickies, Soapbox, Terminal and System Preferences; other
  apps show up there while they're open.
- Now playing: ♫ in the menu bar while a song is on, with a card (cover,
  ⏮ ⏯ ⏭); the browser's media keys work through the Media Session API.
  One sound switch (menu bar speaker, System Preferences › Sound)
  governs interface sounds and music alike; pressing Play turns it on.
- The desktop picture changes (within its collection) each time the
  visitor leaves the tab and comes back; System Preferences can turn it
  off. Dynamic › Now Playing shows the playing song's cover, blurred.
- Window manager: drag, resize from five edges, z-order, minimize into
  the Dock with a Genie effect (SVG displacement map; Safari, phones and
  reduced motion get a plain shrink), zoom, windows that grow from the
  icon that opened them, and windows that can be thrown and bounce off
  the screen edges.
- Exposé: F9, the bottom-left hot corner or View → Exposé.
- Screensavers: Photos (Ken Burns), Starfield or Clock, after the idle
  time set in System Preferences (two minutes by default).
- Place: the visitor's city, coordinates and time zone come from their
  IP address through `api/geo.ts` (Vercel's `x-vercel-ip-*` headers; no
  prompt, nothing stored). They can pick another city instead.
- Sky: the wallpaper follows the light and weather at that place
  (Open-Meteo), with rain, storms, snow and fog drawn behind the windows
  and the temperature (°F or °C by country) in the menu bar. The menu
  bar clock uses the place's time zone; its tooltip shows San Jose time.
- Presence: the menu bar counts who's on the desktop and lists where
  they are ("🇯🇵 Tokyo"); other visitors' cursors carry the same label.
- Spotlight (⌘K); ⌥W / ⌥M / ⌥T close, minimize and open a terminal
  (browsers reserve ⌘W and ⌘T); ⌥Tab switches windows.
- Sounds: synthesized with Web Audio, off by default; the menu bar
  speaker or the Sound pane turns them on.
- Desktop icons can be dragged anywhere; "Clean Up Icons" on the desktop
  menu puts them back.
- Boot screen once per session; light and dark appearance.
- Desktop pictures include ryOS's photo collections and tiles.
- Look (2026-09-25 polish round, PRs #31–#39): Aqua pinstripes on
  windows, menus and bars; brushed metal for Finder, Browser, Photos and
  Calculator; an accent colour sampled from the desktop picture (or a
  fixed one); background windows go grey; glossy Aqua buttons, pop-ups,
  checkboxes, radios and sliders; an optional Glass material; Tiger
  drawers (Photos Info, Finder Get Info); six screen savers (Photos,
  Flurry, Soapbox, Starfield, Clock, Bounce); desktop pictures in
  collections, including solid colours, patterns and a dynamic sky.

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
- **Dashboard:** the visitor's clock (dark at night), calendar and
  weather with a five-day forecast (flip it with "i" to pick a city);
  "Jincheng's time" in San Jose with the offset and a guess at what
  Jincheng is doing; recent GitHub activity and a sticky note.
- **System Preferences:** Desktop & Screen Saver, Appearance (Automatic,
  Light, Dark, Follow the sun) and Date, Time & Place.
- **Soapbox:** Jincheng's own notes and rants, posted from a Telegram
  bot (`supabase/functions/soapbox-bot`); visitors react with one emoji
  per post, and the newest one shows on the Dashboard. Terminal:
  `soapbox`, `weather [city]`.
- **iPod:** a fifth-generation iPod with a working click wheel (drag,
  scroll or arrow keys): Music (Cover Flow, Albums, Artists, Songs),
  Extras (Karaoke, Brick, Music Quiz), Settings (shuffle, repeat,
  artwork or video on Now Playing, backlight, white / black / U2 theme),
  album pages, a dimming backlight, scrolling titles and a scrollbar.
  YouTube's title bar, logo and pause screen never show (cropped, and
  covered with the artwork until the video plays). A Controls menu
  appears in the menu bar while the iPod or Karaoke is in front.
- **Karaoke:** the song's video fills the window and its lyrics fill in
  time, line by line, with a song picker, seeking and a per-song lyrics
  timing nudge. Shares the iPod's library and "now playing": whichever
  app was used last plays, and the other picks up at the same second.
  Instrumentals get a listening view (cover over its own blur, album,
  what's next). The library (`src/data/songs.json`) has 15 songs and
  Ryuichi Sakamoto's BTTB -20th Anniversary Edition- (18 tracks);
  lyrics come from lrclib.net, or NetEase via `/api/lyrics`.
- **Finder:** Macintosh HD opens a Finder over Applications, Applets,
  Documents, Pictures and Projects, with icon and list views.
- **Applet Store:** Get / Open / Remove for the applets, which install
  into Finder's Applets folder and Spotlight: **Minesweeper** (installed
  by default), **Tile Game** (a 4 × 4 sliding puzzle cut from a random
  photo) and **Calculator**. The store and Tile Game icons are drawn in
  SVG in `icons.tsx`.

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

0. **Merge the 2026-09-25 stack, bottom up:** #19 (visitor location) →
   #20 (visitor cities in presence) → #21 (System Preferences) → #22
   (Soapbox) → #23 (⌥Tab) → #24 (sounds) → #25 (Soapbox on the
   Dashboard) → #26 (draggable icons) → #27 (Minesweeper). Then **set Soapbox up**: follow
   `supabase/functions/soapbox-bot/README.md` (migration, @BotFather,
   secrets, deploy with `--no-verify-jwt`, `setWebhook`). Until then the
   app is empty.

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
5. **Songs.** Ten of the starter songs remain (timing carried over from
   ryOS's values, unchecked by ear), plus 寧夏, Kiss & Tell, 寫信給你,
   心動 and 三個人的晚餐 (lyrics from NetEase) and BTTB. 三個人的晚餐
   uses the official MV, which is ten seconds shorter than the album cut,
   so its timing may need an `offset`. Chrome defers YouTube playback in
   background tabs, so a song started in a hidden tab waits until the tab
   is shown. `/api/lyrics` only runs on Vercel; under `astro dev` those
   songs show the listening view.
6. **Possible next work:** phone polish; persisting windows across
   reloads; automated tests for the window manager; the Chinese site and
   the AI assistant later; an ocra review-replay app once ocra's redesign
   is done.
7. **Visual parity with ryOS** is largely done (see Look above). Left
   on purpose: multiple OS themes (System 7, XP, 98), video wallpapers.
8. **Still missing compared with ryOS:** Soapbox photos (Telegram
   images into Supabase Storage); a Finder-style file browser over the
   projects; more Dashboard widgets (e.g. a world clock of where
   visitors are). Deliberately skipped: ryOS's Videos app, emulators, a virtual file system, multiple themes and AI
   chat. Listen to the sounds once (#24); they were checked by
   instrumentation, not by ear.
