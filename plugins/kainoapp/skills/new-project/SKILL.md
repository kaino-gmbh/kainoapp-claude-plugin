---
name: new-project
description: Create a new kainoapp app on kainoapp.com from the Claude Design in this conversation. Use when the user, in English or German, wants the design or its current state turned into a running app, or asks for "a kainoapp" (an app on kainoapp.com), e.g. "make this an app", "build this design", "new project in kainoapp", "make a kainoapp from this", "Baue mir eine kainoapp aus dem aktuellen Stand", "Mach daraus eine App", "Neues Projekt aus diesem Design". For an app that already runs, use new-phase.
---

# New app from a Claude Design

You turn the design in this conversation into a new kainoapp app. kainoapp sets the app
up at the address `https://<name>.kainoapp.com`, then plans the work. This costs real
money (the setup and the build runs), so the user's explicit go comes before the one write
call. Answer in the user's language.

The tools come from the `kainoapp` connector. If they are missing, ask the user to
connect it under the plugin's Connectors tab and stop.

The app exists only once `project-create-from-design` has answered with its name. Until
then, never tell the user that an app was built, created, set up or deployed, and never
make up a name, an address or a state: a prototype in this conversation is a design, not
an app. Every fact about the app comes from a tool answer. If a call fails or was not
made, say so.

## 1. Find the design

Use the Claude Design artifact of this conversation (its prototype screens, design
system and tokens). If the conversation has none, ask the user to open their design here
first and stop.

## 2. Make sure the app is new

Call `project-list` and show the user the projects whose name or display name could be
this app. Ask whether one of them is it. If yes, stop: when that app was created without
this design or with an incomplete one, point to `/kainoapp:attach-design`; when the
design changes the app, point to `/kainoapp:new-phase`.

## 3. Propose the app

Propose and let the user adjust:

- `name`: the short name, 3 to 22 characters, lowercase letters, digits and hyphens, no
  leading or trailing hyphen, not one of `s`, `www`, `api`, `mail`, `admin`, `status`,
  not ending in `-s`, `-staging` or `-production`. kainoapp appends a hyphen and five
  random characters, so `rekruta` becomes for example `rekruta-k3x9p`.
- `display_name`: the app's name as users will see it.
- `description`: the brief the planning reads. Who uses the app, what they do with it,
  the screens and the data, what is out of scope. Plain prose, no marketing.

## 4. Collect the design files

List the design's text files, each with its relative path. The planning, the theme
takeover and every build read these files as the design, and nothing else of the
conversation reaches them:

- every prototype screen as a root-level `*.html`, and the design's `*.dc.html` when it
  has one; a design that is a `*.dc.html` alone is complete as it is;
- every stylesheet and script the pages load (`support.js` of a `*.dc.html` included),
  under the paths the pages reference. kainoapp refuses the call when a page loads or
  links a text file that is not among the files, and names it;
- never write a page of your own around a design file: send the design file itself;
- `uploads/<Name>_Design_Tokens.json` and `uploads/<Name>_Design_System.md` when the
  design has them;
- only `html`, `css`, `js`, `json`, `md`, `txt`, `svg`; at most 40 files, 5 MB per file,
  8 MB together.

Each file goes exactly as it stands in the design, in full. Do not shorten,
summarise, reformat or rewrite a file, and do not drop parts of it: icons stay as they
are (inline `<svg>` stays inline, an icon file goes along as `*.svg`).

What cannot be transferred (images, fonts, a file over the limits) goes into the
`description` as a last paragraph starting with `Nicht übertragen:`, naming each item and
where it appears, so the planning knows the gap. Mention it to the user as well.

## 5. Send the design

Send every design file with `design-upload`, one file per call, before the create call.
It creates nothing and costs nothing. A design file never goes into the create call
itself: a call that long ends inside its arguments and reaches kainoapp empty.

- The first call omits `upload`; its answer names the upload id. Pass it in every further
  call.
- A file of at most 20 000 characters goes as part 1 of 1 (`part` and `parts` may be
  left out). A longer file goes in parts: decide `parts` first (the length divided by
  20 000, rounded up), then send part 1, 2, ... with the same `parts`, each the exact next
  slice of the file. kainoapp joins them without separator, so cut anywhere, but leave
  nothing out and add nothing.
- Never leave a file out because it is long, and never send a shortened or rewritten
  version instead.
- After the last file, read the answer: `complete` is true and no file lists `missing`
  parts. Send a missing part again; a part sent again replaces itself.
- `upload_not_found`: the upload expired (24 hours after its last part). Start over
  without `upload` and send every file again.
- `too_large`: the file or the whole design breaks the limits. Name it under
  `Nicht übertragen:` and tell the user.

## 6. Ask for the go

Summarise in a few lines: the short name, the display name, the brief, the number of
files and bytes the last `design-upload` answer lists, and that the call creates the
infrastructure and starts the planning, which costs money. Call nothing until the user
clearly says yes.

## 7. Create the app

Call `project-create-from-design` with `name`, `display_name`, `description` and `upload`.
Never put a design file into this call.

- Success: report the final short name from the answer (with its random part) and the
  address `https://<final name>.kainoapp.com`. Say that the setup runs in the background,
  the user is notified when the app is reachable, and can ask here for the state at any
  time.
- `project_exists`: the answer names the existing app. Do not create anything; offer
  `/kainoapp:new-phase` for that app.
- `empty_arguments`: the call was too long and arrived empty. The design goes through
  `design-upload` only; send the call again with `upload`.
- A validation error: show it, fix the named field with the user, and ask for the go again
  only if the change is material. A missing part or a missing referenced file: send it
  with `design-upload` and call again.

## 8. Report the state

When the user asks how far the app is, call `project-status` with the final short name
and answer in a few lines:

- `setup.state`: `running` or `queued` (the app is being set up), `completed` (it is set up) or
  `failed` (the operator has been notified; the answer carries no error text by design).
  Report the state only: never ask for or name the steps of the setup.
- `urls.production` once it is set: the app is reachable there. It is the only address
  to report.
- `planning.state`: `running` (the work packages are being planned), `waiting_quota`
  (planning resumes at `planning.retry_at`), `done`, `failed` (the operator has been
  notified) or `pending` (no automatic planning run is recorded; for an older app the
  work packages under `phases` were planned by hand).
- `phases`: each work package with its features per status.
