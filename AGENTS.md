# AGENTS.md

## Repository purpose

iGPT is an alternative frontend and local-state experiment for the OpenAI developer API. The near-term work is specification, storage, synchronization, and interface architecture. Do not assume this repository is a client for undocumented consumer ChatGPT internals.

## Read first

1. `vendor/openai/UPSTREAM` — exact upstream revision and expected Git blob id.
2. `vendor/openai/openapi.yaml` — mirrored OpenAI OpenAPI 3.1 specification. This is the normative API-shape source and is generated; never edit it by hand.
3. `docs/openai-api-notes.md` — machine-oriented review of the API and iGPT implications. This is commentary, not the contract.
4. `vendor/openai/LICENSE` — license accompanying the mirrored OpenAI specification.

If the notes and the OpenAPI mirror disagree about an endpoint, parameter, request, response, or schema, the pinned OpenAPI mirror wins. If current upstream behavior is relevant, update the pin and mirror before rewriting assumptions around an old snapshot.

## Source updates

- Do not hand-edit `vendor/openai/openapi.yaml`.
- Change `vendor/openai/UPSTREAM` to an intentional upstream commit and corresponding blob id.
- Let `.github/workflows/mirror-openai-openapi.yml` fetch and verify the exact pinned file.
- After a source update, review `docs/openai-api-notes.md` for semantic drift.
- Preserve upstream provenance and license.

## Implementation language

Implementation language is deliberately unresolved between Grease and Idriç. Do not introduce a language choice, build system, dependency framework, or generated SDK merely because one is convenient. Documentation, protocol fixtures, storage experiments, and language-neutral acceptance cases may proceed before that decision.

## Local-first invariant

The intended user experience is that already-seen history can render from local storage without waiting for a full network reload. Treat local readable conversation/event state as durable application data and remote API state as synchronizable protocol state. Do not make API-side retention the only copy of user-visible history.

## API boundary

- Use documented OpenAI developer API surfaces.
- Do not infer access to personal ChatGPT history, ChatGPT memory, consumer subscription state, or internal ChatGPT endpoints.
- Keep API credentials out of source, fixtures, logs, transcripts, and generated artifacts.
- Keep remote ids alongside local ids rather than using remote ids as the only local identity.
- Preserve streaming, retry, interruption, tool-call, and error distinctions in durable state.

## Storage and indexing

- Prefer stable local records/fragments plus derived indexes over repeated copies of full message bodies.
- Keep readable UTF-8 available locally.
- Compression is an empirical design choice, not a premise. If tested, use LZ4 as a latency baseline and zstd as a density comparison; benchmark on the actual target device and storage.
- If compression is adopted for interactive data, prefer independently decodable blocks over a monolithic archive.
- Do not choose SQLite, a filesystem-only layout, or another database until the access patterns are measured or specified.

## Change discipline

- Keep vendor source, our observations, and implementation policy visibly separate.
- Mark unresolved choices as unresolved rather than manufacturing a decision.
- Prefer small branches and reviewable changes.
- Claims about current API behavior should name the upstream revision or a dated official source.
- Acceptance evidence must state what was actually exercised; do not promote desktop/emulator/cloud evidence into physical-device evidence.
