---
name: draft-architecture
description: Use when the user asks to draft, create, or update an architecture diagram or model from their project's documentation, for example "draft the architecture from the docs", "turn our design docs into a C4 diagram", "make an architecture model we can check the code against", or "update the architecture model from the README". Produces a simple LikeC4 (C4-style) model with cited sources and a list of documentation gaps. Works in any coding agent that can read files and run shell commands.
---

# Draft the architecture model from documentation

You turn a project's documentation into a simple LikeC4 architecture model. Humans will
review and correct it, and the code can later be checked against it, so it must be
accurate, simple, and easy to edit.

The LikeC4 syntax you need is in `references/likec4-reference.md` next to this file.
Read it before writing the model.

## Requirements

- **Node.js 20 or newer**, to validate the model with LikeC4. Check with `node --version`.
- **Optional: the `archtrace` command-line tool**, to check the code against the model.
  Check with `archtrace --version`. Without it, this skill still drafts and validates the
  model; it just skips the final check.

Don't install anything without asking. If something is missing, tell the user what it is
for and the command to install it, and continue with what works.

## Rules

- **Read documentation, not source code**: README files, `docs/`, ADRs, design docs, API
  specs (OpenAPI, AsyncAPI, protobuf), and architecture notes in config files such as
  docker-compose or Kubernetes manifests. Listing folder *names* is allowed in step 6;
  reading what's inside source files is not.
- **Don't invent anything.** Every element and arrow must come from the docs. Record where
  with a `source` metadata entry, e.g. `source 'docs/design.md#payments'`.
- **When the docs are unclear, missing, or contradict each other, don't guess.** Write it
  in `GAPS.md` instead.
- **Keep it simple.** At most about 10 elements per view. Plain-language names,
  one-sentence descriptions, short verb labels on arrows (`'sends orders to'`). Leave out
  libraries, logging and utilities unless the docs treat them as architecture.
- **Arrows mean ALLOWED dependencies.** Anything not drawn counts as a violation when the
  code is checked.
- **Project contents stay on the machine.** Don't send files or excerpts to any website
  or service.

## Steps

1. **Find the model folder.** Use the folder the user named. Otherwise use `architecture/`.
   If it already contains `.c4` files (for example from `archtrace init`), you are
   *updating* that model: keep existing ids, `code` and `service` entries, and anything
   marked approved, and add to it rather than rewriting it.

2. **List the documentation** you found and summarize in one paragraph what it says about
   the system. If there is almost no documentation, say so and ask whether to continue
   with a minimal model, or to start from the folder structure with `archtrace init`
   instead (if installed).

3. **Propose the subsystems**: name, purpose, and the doc each comes from. Show the list
   and **wait for the user to confirm** before writing files.

4. **Write the model** in the model folder, following `references/likec4-reference.md`:
   - `spec.c4`: a few element kinds, e.g. `person`, `system`, `container`, `component`,
     `database`, `queue`.
   - `model.c4` for a small system, or one file per subsystem for a large one.
   - `views.c4`: a top-level view of everything, one view per subsystem, and component
     views only where the docs give enough detail. Add `dynamic` views for the 3-5 most
     important flows that cross subsystems, if the docs describe them.
   - Give each subsystem `metadata { status 'draft' }`.
   - Give every element a stable camelCase id and a `source` entry.

5. **Validate.** If `likec4` isn't installed in the project, ask the user before running
   `npx likec4@1.59.4 validate <model folder>`, which downloads LikeC4 from npm the first
   time. Fix every error it reports and validate again until it passes.

6. **Map elements to code and services.** List the project's folder names (not file
   contents) and propose a `code` entry for each element, e.g. `code 'src/payments'`. If
   there are docker-compose or Kubernetes files, also propose a `service` entry with the
   service name that runs each element, e.g. `service 'payments-api'`. Show the mapping
   and **wait for confirmation**, then add the entries. Leave out any you can't map
   confidently and say which.

7. **Write `GAPS.md`** in the model folder, one section per subsystem: missing information,
   contradictions between documents, things that look outdated, and any connection
   between subsystems that isn't described on both sides.

8. **Check the code (only if archtrace is installed).** Run
   `archtrace check --model <model folder>`. Don't change code or remove arrows to make it
   pass. Report the violations as findings for the user to decide on: fix the code, or
   change the model. If archtrace isn't installed, say that it can check the code against
   this model, and that it installs with `npm install -g archtrace`.

9. **Finish with a short summary**: number of elements per level, the main gaps, the
   check result if any, and how to view the diagrams:
   `npx likec4@1.59.4 start <model folder>`.
