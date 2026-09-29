---
name: archive
description: "Move completed CC artifacts or retired docs to the canonical cold archive. Activates at session-end artifact cleanup, after a prompt finishes executing, or whenever a file is done and should leave the working set."
---

# Archive

One archive, flat, cold. Move done files in; never read from it to reason or reuse.

## Context
- Canonical archive: `Archive\` — ONE folder, flat. No range subfolders, no per-layer archives, no nested Archive folders.
- The archive is NOT semantically indexed (the indexer excludes `Archive/` by folder name). Placing a file here removes it from search automatically — that is the entire point of archiving.
- Retrieval is by direct name lookup only, by the agent that archived it. The filename is the sole carrier of provenance, so it must be self-locating.

## What counts as "done" (archive-eligible)
- `CC-PROMPT-*` — on execution complete.
- `CC-BUILD-LOG-*`, `CC-FEEDBACK-*`, `CC-FEEDBACK-PROCESSED-*` — always (post-execution by nature).
- Any doc with `status: COMPLETE` (or explicit Jordan confirmation). Active / in-progress files stay in the working set.

## Steps
1. Confirm done (above). If not done -> STOP, leave it in the working folder.
2. Rename to the self-locating convention: `S{session}-{verb}-{subject}-{type}.md`
   - `{type}`: `prompt` | `build-log` | `feedback` | `feedback-processed` | `doc`
   - verb-subject lowercased from the source filename.
3. Move to `Archive\` — flat, directly in Archive, no subfolders.
4. Verify: list the source directory; confirm NO `CC-*` files remain. If any remain, move them now — do not defer.

## Rules
- ONE archive only: `Archive\`. Never create another Archive folder, never per-project/per-layer, never range subfolders. If you find one, flag it — do not add to it.
- Flat inside Archive. No subfolders.
- Self-locating filename is mandatory — the path no longer carries provenance.
- Never archive an active / in-progress file.
- Never read from Archive to reason or reuse. If you need to reuse it, it shouldn't have been archived. Direct name lookup only, when you know exactly what you're fetching.
- Never add `Archive/` to the semantic index scope.
