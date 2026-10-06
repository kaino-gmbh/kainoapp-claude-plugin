---
name: attach-design
description: Give an existing kainoapp app on kainoapp.com the Claude Design it should have had from the start, when it was created without its design or with an incomplete one. The design replaces the app's design as if it had come with the creation; the planned work packages and their order stay, nothing new is planned. Use when the user, in English or German, says the design is missing or did not arrive, e.g. "attach the design to my app", "the design did not come through", "Design nachreichen", "Hänge das Design an das Projekt an", "Das Design fehlt im Projekt". A design that changes a running app is new-phase.
---

# Attach the design to an existing app

You give an existing kainoapp app the design of this conversation as its design, as if it
had come with the creation. kainoapp replaces the app's design, which every planned work
package reads when it is built. No work package is
added and nothing is planned again. Answer in the user's language.

The tools come from the `kainoapp` connector. If they are missing, ask the user to
connect it under the plugin's Connectors tab and stop.

The design is attached only once `project-attach-design` has answered `replaced`. Until
then, never tell the user that it was attached, and never make up a state: every fact
comes from a tool answer. If a call fails or was not made, say so.

## 1. Choose the app

Call `project-list` and let the user pick the app (show the display name and the short
name). If the user already named it, confirm the match.

If the user wants to CHANGE what the app does or looks like, this is not the right skill:
point to `/kainoapp:new-phase`.

## 2. Collect the design files

The same files as for a new app, each exactly as it stands in the design, in full:

- every prototype screen as a root-level `*.html`, and the design's `*.dc.html` when it
  has one; a design that is a `*.dc.html` alone is complete as it is;
- every stylesheet and script the pages load (`support.js` of a `*.dc.html` included),
  under the paths the pages reference;
- `uploads/<Name>_Design_Tokens.json` and `uploads/<Name>_Design_System.md` when the
  design has them;
- only `html`, `css`, `js`, `json`, `md`, `txt`, `svg`; at most 40 files, 5 MB per file,
  8 MB together.

Never write a page of your own around a design file, never shorten, summarise or rewrite
a file. Images and fonts cannot be sent; tell the user which ones the design uses.

## 3. Send the design

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
2. Pack the design files of section 2 from that directory as they stand, with their
   paths: `rm -f /tmp/design.zip && cd <directory> && zip -qr /tmp/design.zip . -i '*.html' '*.css' '*.js' '*.json'
   '*.md' '*.txt' '*.svg'` (without `zip`, use Python's `zipfile`).
3. Upload it: `curl -sS -F "file=@/tmp/design.zip" "<url>"`. The answer lists the files
   and bytes kainoapp received under `files`.

Never print a design file. An upload that fails with a network error: run the check
above again. A `url` older than 30 minutes or an expired upload: call `design-upload-url`
again.

## 4. Ask for the go

Summarise: the app, the number of files and bytes the last upload answer lists,
and that the design replaces the app's current design for every work package not yet
built. Call nothing until the user clearly says yes.

## 5. Attach it

Call `project-attach-design` with `project` and `upload`.

- `replaced`: the answer's `next` says what happens to the app; report it in one or two
  sentences. When `next` says that the same call tries again ("derselbe Aufruf versucht
  es erneut"), the design is not yet taken over for the builds: call
  `project-attach-design` again with a new upload of the same files.
- `project_not_found` or a validation error: show it and fix it with the user.
