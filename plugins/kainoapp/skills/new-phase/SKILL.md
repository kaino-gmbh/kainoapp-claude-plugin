---
name: new-phase
description: Add a new work package to an existing kainoapp app from a Claude Design. Use when the user wants to change or extend an app that already runs ("new design for my app", "next phase", "add these screens").
---

# New work package for an existing app

You compare an existing kainoapp app with its design, design the change here, and send it
as a new work package. kainoapp then plans the features from it. This starts a planning
run that costs money, so the user's explicit go comes before the one write call. Answer
in the user's language.

The tools come from the `kainoapp` connector. If they are missing, ask the user to
connect it under the plugin's Connectors tab and stop.

## 1. Choose the app

Call `project-list` and let the user pick the app (show the display name and the short
name). If the user already named it, confirm the match. The short name is the `project`
argument of every later call.

## 2. Read where the app stands

Call `project-get-design` with the `project`. It returns the last imported design, the
theme and component sources of the built app, and the features built since that design.
Summarise for the user where the built app departs from the design. Read further
`design_files` or `components` only when the summary needs them.

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
- `files`: the design as text files, the same rules as for a new app: root-level `*.html`
  prototypes (at least one that is not `*.dc.html`) and the design's `*.dc.html` when it
  has one, the stylesheets and scripts they load, the tokens and design system files under
  `uploads/` when present; only `html`, `css`, `js`, `json`, `md`, `txt`, `svg`; at most
  40 files, 5 MB per file, 8 MB together.
- Each `content` is the file exactly as it stands in the design, in full: the feature
  planning and the builds read these files and nothing else of the conversation. Do not
  shorten, summarise, reformat or rewrite a file, and do not drop parts of it; icons stay
  as they are (inline `<svg>` stays inline, an icon file goes along as `*.svg`).
- What cannot be transferred (images, fonts, a file over the limits) goes under
  `Offene Punkte` in the `description` as `Nicht übertragen:`, naming each item and where
  it appears. Mention it to the user as well.

## 5. Ask for the go

Summarise the app, the title, the brief and the number of files, and that the call starts
the feature planning, which costs money. Call nothing until the user clearly says yes.

## 6. Send it

Call `phase-create-from-design` with `project`, `name`, `description` and `files`.

- `planning` is `started`: the work package exists and the features are being planned;
  the user is notified when they are ready. Asked for the state later, call
  `project-status` with the `project` and report the work package with its features per
  status.
- `planning` is `no_repository`: the work package and the design are stored; the
  planning starts once the app's repository is set up. Tell the user.
- A validation error: show it, fix the named field with the user, and ask for the go again
  only if the change is material.
