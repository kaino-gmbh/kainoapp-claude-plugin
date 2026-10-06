---
name: new-phase
description: Add a new work package to an existing kainoapp app on kainoapp.com from a Claude Design. Use when the user, in English or German, wants to change or extend an app that already runs, e.g. "new design for my app", "next phase", "add these screens to my app", "Neues Arbeitspaket für meine App", "Bau das in meine bestehende kainoapp ein", "Erweitere meine App um diese Screens". For an app that does not exist yet, use new-project.
---

# New work package for an existing app

You compare an existing kainoapp app with its design, design the change here, and send it
as a new work package. kainoapp then plans the features from it. This starts a planning
run that costs money, so the user's explicit go comes before the one write call. Answer
in the user's language.

The tools come from the `kainoapp` connector. If they are missing, ask the user to
connect it under the plugin's Connectors tab and stop.

The work package exists only once `phase-create-from-design` has answered. Until then,
never tell the user that it was created, planned or built, and never make up its state:
every fact about the app comes from a tool answer. If a call fails or was not made, say
so.

## 1. Choose the app

Call `project-list` and let the user pick the app (show the display name and the short
name). If the user already named it, confirm the match. The short name is the `project`
argument of every later call.

## 2. Read where the app stands

Call `project-get-design` with the `project`. It returns the last imported design, the
theme and component sources of the built app, and the features built since that design.
Summarise for the user where the built app departs from the design. Read further
`design_files` or `components` only when the summary needs them.

The answer may also carry `snapshot`: the current state of the running app, its pages as
they look now. Without `snapshot`, skip the rest of this step and say nothing about it.
Otherwise ask the user whether the current state shall come into the design. When
`snapshot.state` is `pending`, say roughly how long it takes: `estimated_seconds` in
minutes, rounded.

- No: go on without it.
- Yes and `ready`: fetch it now (below).
- Yes and `pending`: go on with section 3 while it is produced. After each of your own
  replies, call `project-get-design` with `project` and `snapshot_only: true` (the answer
  holds only `project` and `snapshot`) and fetch once it is `ready`. When the user is done
  with the design before then: up to 10 times, run `sleep 30` in your code execution
  environment and then the same `snapshot_only` call; stop at any state other than
  `pending`.
- `unavailable`, or still `pending` after that: tell the user in one sentence that the
  current state is not available now, and go on without it.

Fetch: run the reachability check of section 5 first; when it fails, tell the user what
section 5 says. Then, with the `url` of the `ready` answer:
`rm -rf /tmp/ist-stand /tmp/ist-stand.zip && curl -sSf -o /tmp/ist-stand.zip "<url>" && mkdir /tmp/ist-stand && cd /tmp/ist-stand && unzip -q /tmp/ist-stand.zip`
(without `unzip`, use Python's `zipfile`). The `url` is valid for 30 minutes: a `403` or
`404`, or a `url` older than that, gets one new `snapshot_only` call and one more try with
its `url`.

Read `manifest.json`. Add every entry of its `pages` to the Claude Design canvas as its
own page under `ist-stand/` in the canvas project directory, from its `html` file
unchanged, named by its `title`; keep the archive's paths below `ist-stand/`
(`ist-stand/pages/…`, `ist-stand/assets/…`), so each page finds its assets. Its
`screenshot` is how the page renders in the app: the comparison for the canvas page.
When `theme_css` is set, that file is the app's current theme: start the design system
of the change from it. Never edit a page under `ist-stand/`; the changed screens are new
prototypes beside them.

A design file never loads or links anything under `ist-stand/`: no stylesheet, script,
page or link there. Copy what the design needs, such as the tokens of
`ist-stand/theme.css`, into the design's own files.

## 3. Design the change

Work with the user on the Claude Design in this conversation: the design system and the
prototypes of the affected screens. Keep what the app already has unless the user
changes it.

## 4. Prepare the work package

- `name`: a short, concrete title as it would appear on an offer, for example
  "Rechnungen im Kundenportal": at most 60 characters, no colon, no list, no technical
  term. The tool rejects a title that is not offer-ready and names the rule; then shorten
  or rephrase it.
- `description`: the brief of the CHANGE, with the headings Ziel, Umfang, Baut auf,
  Abgrenzung, Offene Punkte. Per screen: what changes versus the running app.
- The design files, the same rules as for a new app: root-level `*.html`
  prototypes and the design's `*.dc.html` when it has one (a `*.dc.html` alone is
  complete), every stylesheet and script the pages load (`support.js` included), the
  tokens and design system files under `uploads/` when present; only `html`, `css`, `js`,
  `json`, `md`, `txt`, `svg`; at most 40 files, 5 MB per file, 8 MB together. kainoapp
  refuses the call when a page loads or links a text file that is not among the files,
  and when it loads or links anything under `ist-stand/`. Never write a page of your own
  around a design file.
- Each file goes exactly as it stands in the design, in full: the feature
  planning and the builds read these files and nothing else of the conversation. Do not
  shorten, summarise, reformat or rewrite a file, and do not drop parts of it; icons stay
  as they are (inline `<svg>` stays inline, an icon file goes along as `*.svg`).
- What cannot be transferred (images, fonts, a file over the limits) goes under
  `Offene Punkte` in the `description` as `Nicht übertragen:`, naming each item and where
  it appears. Mention it to the user as well.

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
   paths: `rm -f /tmp/design.zip && cd <directory> && zip -qr /tmp/design.zip . -x 'ist-stand/*' -i '*.html' '*.css' '*.js' '*.json'
   '*.md' '*.txt' '*.svg'` (without `zip`, use Python's `zipfile` and leave out
   `ist-stand/`). `-x 'ist-stand/*'` keeps the current state of the app out of the upload.
3. Upload it: `curl -sS -F "file=@/tmp/design.zip" "<url>"`. The answer lists the files
   and bytes kainoapp received under `files`.

Never print a design file. An upload that fails with a network error: run the check
above again. A `url` older than 30 minutes or an expired upload: call `design-upload-url`
again. A refusal for size: name the file under `Offene Punkte` as `Nicht übertragen:` and
tell the user.

## 6. Ask for the go

Summarise the app, the title, the brief and the number of files and bytes the last
upload answer lists, and that the call starts the feature planning, which costs
money. Call nothing until the user clearly says yes.

## 7. Create the work package

Call `phase-create-from-design` with `project`, `name`, `description` and `upload`. Never
put a design file into this call.

- `planning` is `started`: the work package exists and the features are being planned;
  the user is notified when they are ready. Asked for the state later, call
  `project-status` with the `project` and report the work package with its features per
  status.
- `planning` is anything else (`not_started` while the app is still being set up): the
  work package and the design are stored, the planning has not started. Tell the user
  what the answer's `next` says.
- A validation error: show it, fix the named field with the user, and ask for the go again
  only if the change is material. A missing referenced file: add it to the ZIP, upload
  again through a new `design-upload-url` and call again. A reference that points under
  `ist-stand/`: remove it from the prototype (copy what the page needs into the design's
  own files), never add the `ist-stand/` file to the ZIP; then upload again and call
  again.
