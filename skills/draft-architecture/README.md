# draft-architecture

Turns your project's documentation into a simple, editable architecture diagram: a
[LikeC4](https://likec4.dev) model (C4-style boxes and arrows) that you review, correct,
and approve.

## What it does

1. Reads only your docs (README, `docs/`, ADRs, API specs, deployment configs), not your
   source code.
2. Proposes the main parts of the system and waits for you to confirm them.
3. Writes the model, with every box and arrow citing the doc it came from.
4. Validates it with LikeC4 and fixes any errors.
5. Maps each box to its code folder and deployed service, after you confirm.
6. Lists what the docs don't answer in `GAPS.md`.
7. If the free `archtrace` CLI is installed, checks your code against the model.

Every arrow means "this dependency is allowed", so the model doubles as a set of rules
your code can be checked against.

## How to use

Ask your agent: *"Draft the architecture model from the docs."* By default it writes to
`architecture/`; name another folder if you like.

View the result: `npx likec4@1.59.4 start architecture`

## Requirements

- Node.js 20 or newer (for LikeC4 validation and viewing).
- Optional: [archtrace](https://www.npmjs.com/package/archtrace) to check code against
  the model: `npm install -g archtrace`. Pair this skill with **align-architecture** to
  resolve what the check finds.

The skill never installs anything without asking, and doesn't send your files anywhere.

## Files

- `SKILL.md`: the instructions your agent follows.
- `references/likec4-reference.md`: the LikeC4 syntax it uses, with a validated example.

License: MIT.
