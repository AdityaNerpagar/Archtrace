# align-architecture

When your code and your architecture diagram disagree, this skill walks through each
disagreement with you and lets **you** decide: fix the code, or change the diagram.

## What it does

1. Runs `archtrace check`, which finds dependencies your
   [LikeC4](https://likec4.dev) architecture model doesn't allow: imports between code
   folders (JavaScript/TypeScript) and network connections between services
   (docker-compose and Kubernetes configs).
2. Investigates each violation and classifies it: a shortcut, a new real need, leftover
   code, or a misplaced file.
3. Asks you, one violation at a time, to choose **fix the code**, **change the model**,
   or **decide later**, with a recommendation and the rough size of the change.
4. Makes code fixes one at a time, re-running your tests and the check after each.
5. Edits the model only where you said so.
6. Records every decision and its reason in `ALIGNMENT.md`.

It never decides on its own, and never adds arrows just to make the check pass.

## How to use

Ask your agent: *"Align the code with the architecture."*

## Requirements

- The free [archtrace](https://www.npmjs.com/package/archtrace) CLI (Node.js 20 or
  newer): `npm install -g archtrace`. The skill checks for it and asks before anything is
  installed.
- A LikeC4 architecture model in the project (default `architecture/`). Create one with
  `archtrace init`, or from your docs with the **draft-architecture** skill.
- Optional: [Emerge](https://github.com/glato/emerge), for a view of how the code is
  really structured.

## Files

- `SKILL.md`: the instructions your agent follows.
- `references/likec4-reference.md`: the LikeC4 syntax used for model edits.

License: MIT.
