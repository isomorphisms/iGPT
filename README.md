# iGPT

iGPT is an alternative frontend and local-state experiment for the OpenAI developer API, aimed at keeping already-seen chat state locally available instead of making every view depend on reloading remote history.

The repository is being bootstrapped documentation-first. Implementation language is intentionally undecided between Grease and Idriç.

## API sources

- `vendor/openai/openapi.yaml` — exact pinned mirror of OpenAI's OpenAPI 3.1 specification; generated, not hand-edited.
- `vendor/openai/UPSTREAM` — upstream commit and expected Git blob id.
- `docs/openai-api-notes.md` — second-pass machine-oriented notes and iGPT design implications.
- `AGENTS.md` — instructions for agents working in this repository.

The full OpenAPI mirror is authoritative for endpoint and schema details. The notes deliberately separate API facts from iGPT design choices and unresolved decisions.
