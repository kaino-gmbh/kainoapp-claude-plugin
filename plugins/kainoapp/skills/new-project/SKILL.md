---
name: new-project
description: Create a new kainoapp app from the Claude Design in this conversation. Use when the user wants to turn their design into a running app on kainoapp.com ("make this an app", "build this design", "new project").
---

# New app from a Claude Design

You turn the design in this conversation into a new kainoapp app. kainoapp creates the
repository, the server and the address `https://<name>.kainoapp.com`, then plans the
work. This costs real money (servers, build runs), so the user's explicit go comes before
the one write call. Answer in the user's language.

The tools come from the `kainoapp` connector. If they are missing, ask the user to
connect it under the plugin's Connectors tab and stop.

## 1. Find the design

Use the Claude Design artifact of this conversation (its prototype screens, design
system and tokens). If the conversation has none, ask the user to open their design here
first and stop.

## 2. Make sure the app is new

Call `project-list` and show the user the projects whose name or display name could be
this app. Ask whether one of them is it. If yes, stop and point to `/kainoapp:new-phase`,
which adds the design to an existing app.

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

Pass the design as text files in `files` (each with a relative `path` and its `content`).
The planning, the theme takeover and every build read these files as the design, and
nothing else of the conversation reaches them:

- every prototype screen as a root-level `*.html` (at least one that is not `*.dc.html`),
  and the design's `*.dc.html` when it has one;
- the stylesheets and scripts the prototypes load, under the paths the prototypes
  reference;
- `uploads/<Name>_Design_Tokens.json` and `uploads/<Name>_Design_System.md` when the
  design has them;
- only `html`, `css`, `js`, `json`, `md`, `txt`, `svg`; at most 40 files, 5 MB per file,
  8 MB together.

Each `content` is the file exactly as it stands in the design, in full. Do not shorten,
summarise, reformat or rewrite a file, and do not drop parts of it: icons stay as they
are (inline `<svg>` stays inline, an icon file goes along as `*.svg`).

What cannot be transferred (images, fonts, a file over the limits) goes into the
`description` as a last paragraph starting with `Nicht übertragen:`, naming each item and
where it appears, so the planning knows the gap. Mention it to the user as well.

## 5. Ask for the go

Summarise in a few lines: the short name, the display name, the brief, the number of
files, and that the call creates the infrastructure and starts the planning, which costs
money. Call nothing until the user clearly says yes.

## 6. Create the app

Call `project-create-from-design` with `name`, `display_name`, `description` and `files`.

- Success: report the final short name from the answer (with its random part) and the
  address `https://<final name>.kainoapp.com`. Say that the setup runs in the background,
  the user is notified when the app is reachable, and can ask here for the state at any
  time.
- `project_exists`: the answer names the existing app. Do not create anything; offer
  `/kainoapp:new-phase` for that app.
- A validation error: show it, fix the named field with the user, and ask for the go again
  only if the change is material.

## 7. Report the state

When the user asks how far the app is, call `project-status` with the final short name
and answer in a few lines:

- `setup.state` and the steps that are done, running or failed. A failed step: say that
  the operator has been notified; the answer carries no error text by design.
- `urls.production` once it is set: the app is reachable there.
- `planning.state`: `running` (the work packages are being planned), `waiting_quota`
  (planning resumes at `planning.retry_at`), `done`, `failed` (the operator has been
  notified) or `pending` (no automatic planning run is recorded; for an older app the
  work packages under `phases` were planned by hand).
- `phases`: each work package with its features per status.
