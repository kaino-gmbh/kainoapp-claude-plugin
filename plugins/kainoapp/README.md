# kainoapp plugin

Turn a design you made with Claude Design into a running web app, and extend the app
later with new designs, without leaving your Claude conversation.

## What it does

- `/kainoapp:new-project` takes the design in your conversation, checks that the app does
  not exist yet, agrees the app's name and brief with you, and after your explicit go
  creates the app at `https://<name>.kainoapp.com`: repository, server, address and a
  plan of the work.
- `/kainoapp:new-phase` lets you pick one of your apps, shows where the built app departs
  from its design, designs the change with you, and after your go sends it as a new work
  package that is planned into features.
- `/kainoapp:attach-design` gives an app that was created without its design, or with
  an incomplete one, the design of your conversation as if it had come with the creation:
  the planned work packages keep their order and build with it.

You can also just describe what you want, in English or German ("make this design an
app", "Baue mir eine kainoapp aus dem aktuellen Stand"); Claude picks the matching
skill.

## Connecting

After installing the plugin, connect the `kainoapp` connector in the plugin's
Connectors tab and sign in. Every write call asks for your explicit go in the chat first,
because it creates infrastructure or starts planning runs that are billed.

## What data it sends

The connector sends to `https://kainoapp.com/mcp/features`: the app name, display name and brief you agreed
on, and the text files of your design (prototype HTML, CSS, JavaScript, design tokens
and design system notes). Images and fonts are not sent. Reads return your apps' names,
their design state and the features built so far.
