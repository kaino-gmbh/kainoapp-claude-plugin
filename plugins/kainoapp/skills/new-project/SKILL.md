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

A link to the app's dashboard comes only from `dashboard_url` in a tool answer, exactly as
it stands. When an answer has no `dashboard_url`, say nothing about a dashboard.

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

The design goes to kainoapp in one upload from your code execution environment, never
in a tool argument. It creates nothing and costs nothing.

The design files exist as files in that environment: Claude Design writes them into the
canvas project directory of its scratchpad before it saves them. Find the directory with
`find / -path '*canvas/project/*' -name '*.dc.html' -not -path '/proc/*' 2>/dev/null`.

Check first that the environment reaches kainoapp, before you pack anything:
`curl -sS -o /dev/null -w "%{http_code}\n" https://kainoapp.com/up`. `200`: go on.
Anything else (a network or DNS error, `403 blocked-by-allowlist`, code execution off):
stop and tell the user, in their language: this is a setting of their Claude account, not
an error of kainoapp; Claude may send files only to domains the account allows. They turn
on Settings › Capabilities › Code execution and file creation and add `kainoapp.com` under
Additional allowed domains (in a Team or Enterprise organisation, the admin does this).
Ask them to say when it is done, then run the check again.

1. Call `design-upload-url`. Its answer names `upload` and a `url` that is valid for 30
   minutes.
2. Pack the design files of section 4 from that directory as they stand, with their
   paths: `rm -f /tmp/design.zip && cd <directory> && zip -qr /tmp/design.zip . -i '*.html' '*.css' '*.js' '*.json'
   '*.md' '*.txt' '*.svg'` (without `zip`, use Python's `zipfile`).
3. Upload it: `curl -sS -F "file=@/tmp/design.zip" "<url>"`. The answer lists the files
   and bytes kainoapp received under `files`.

Never print a design file. An upload that fails with a network error: run the check
above again. A `url` older than 30 minutes or an expired upload: call `design-upload-url`
again. A refusal for size: name the file under `Nicht übertragen:` and tell the user.

## 6. Ask for the go

Summarise in a few lines: the short name, the display name, the brief, the number of
files and bytes the last upload answer lists, and that the call creates the
infrastructure and starts the planning, which costs money. Call nothing until the user
clearly says yes.

## 7. Create the app

Call `project-create-from-design` with `name`, `display_name`, `description` and `upload`.
Never put a design file into this call.

- Success with `dashboard_url`: put that link first, on a line of its own, and send the
  user there. Tell the user that the app is created, what the answer's `next` says
  happens now, and that the dashboard shows live what happens with the app (its setup,
  the planning of the work packages, the build progress of the features), where they
  answer the open questions right away, one at a time; any of them may stay empty. Then
  report the final short name from the answer (with its random part) and the address
  `https://<final name>.kainoapp.com`, and that they can also ask here for the state at
  any time.
- Success without `dashboard_url`: report the final short name from the answer (with its
  random part) and the address `https://<final name>.kainoapp.com`. Say that the setup
  runs in the background, the user is notified when the app is reachable, and can ask
  here for the state at any time.
- `project_exists`: the answer names the existing app. Do not create anything; offer
  `/kainoapp:new-phase` for that app.
- A validation error: show it, fix the named field with the user, and ask for the go again
  only if the change is material. A missing referenced file: add it to the ZIP, upload
  again through a new `design-upload-url` and call again.

## 8. Report the state

When the user asks how far the app is, call `project-status` with the final short name.
When the answer carries `dashboard_url`, put that link first and send the user there:
the dashboard shows the setup, the planning, the open questions and the build progress
live. Then answer in a few lines:

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
