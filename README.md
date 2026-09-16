# iGPT

iGPT is an alternative frontend and local-state experiment for the OpenAI developer API, aimed at keeping already-seen chat state locally available instead of making every view depend on reloading remote history.

The first executable slice is being stubbed in Grease, the current Oils/YSH-derived shell line. The local record format is deliberately language-neutral so later Idriç components can strengthen the model without owning the existing data by accident.

## First executable slice

`source/igpt.grease` currently provides a local-only command surface:

- `where` — print the resolved local data root;
- `new` — create a local conversation;
- `list` — list local conversations;
- `append` — append a local event;
- `send` — persist a user event, then put it in the local outbox;
- `show` — replay locally stored history without network access;
- `pending` — list pending outbox records;
- `sync` — explicit API boundary, intentionally unimplemented in this stub.

The data root is selected by `IGPT_HOME`, then `XDG_DATA_HOME`, then `$HOME/.local/share/igpt`. Pointing `IGPT_HOME` at SD/removable storage is supported by the current design; automatic device storage selection is not decided yet.

The current local layout and its limitations are documented in `docs/local-state-v1.md`. The acceptance fixture is `tests/local-first.grease` and must be run with the current Grease runtime, not Bash.

## API sources

- `vendor/openai/openapi.yaml` — exact pinned mirror of OpenAI's OpenAPI 3.1 specification; generated, not hand-edited.
- `vendor/openai/UPSTREAM` — upstream commit and expected Git blob id.
- `docs/openai-api-notes.md` — second-pass machine-oriented notes and iGPT design implications.
- `AGENTS.md` — instructions for agents working in this repository.

The full OpenAPI mirror is authoritative for endpoint and schema details. The notes deliberately separate API facts from iGPT design choices and unresolved decisions.

## Current boundary

This branch does not yet contact OpenAI, read an API key, stream a response, execute tools, import ChatGPT history, choose a final database/index, or claim crash-safe concurrent writes. The stub exists to establish the local-first write/replay boundary before those pieces are added.
