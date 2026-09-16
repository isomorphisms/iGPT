# AGENTS.md

## Repository purpose

iGPT is an alternative frontend and local-state experiment for the OpenAI developer API. The near-term work is specification, local storage, synchronization, and interface architecture. Do not assume this repository is a client for undocumented consumer ChatGPT internals.

## Read first

1. `vendor/openai/UPSTREAM` — exact upstream revision and expected Git blob id.
2. `vendor/openai/openapi.yaml` — mirrored OpenAI OpenAPI 3.1 specification. This is the normative API-shape source and is generated; never edit it by hand.
3. `docs/openai-api-notes.md` — machine-oriented review of the API and iGPT implications. This is commentary, not the contract.
4. `docs/local-state-v1.md` — current executable local-state stub and its deliberately unresolved boundaries.
5. `source/igpt.grease` — first executable client slice.
6. `vendor/openai/LICENSE` — license accompanying the mirrored OpenAI specification.

If the notes and the OpenAPI mirror disagree about an endpoint, parameter, request, response, or schema, the pinned OpenAPI mirror wins. If current upstream behavior is relevant, update the pin and mirror before rewriting assumptions around an old snapshot.

## Source updates

- Do not hand-edit `vendor/openai/openapi.yaml`.
- Change `vendor/openai/UPSTREAM` to an intentional upstream commit and corresponding blob id.
- Let `.github/workflows/mirror-openai-openapi.yml` fetch and verify the exact pinned file.
- After a source update, review `docs/openai-api-notes.md` for semantic drift.
- Preserve upstream provenance and license.

## Implementation language

The first executable iGPT client is Grease. Use the current Oils/YSH-derived Grease line from `isomorphisms/grease`; do not substitute Bash, POSIX shell, Python, or another interpreter and report that as Grease execution.

Before changing assumptions about Grease semantics, read `isomorphisms/grease/docs/CURRENT-GREASE.md`. The current Grease-readable boolean spellings include `⟦ ... ⟧`, `∧`, `∨`, and `¬` on the inherited Oils/YSH paths.

This Grease-first decision does **not** make the local record format Grease-specific and does not settle the eventual whole-program language. Idriç may later own typed protocol/state components once real usage establishes which distinctions and invariants deserve types. Preserve a language-neutral local representation so that transition does not require gratuitous data migration.

Do not introduce a generated SDK or a large dependency/build framework merely because one is convenient.

## Local-first invariant

The intended user experience is that already-seen history can render from local storage without waiting for a full network reload. Treat local readable conversation/event state as durable application data and remote API state as synchronizable protocol state. Do not make API-side retention the only copy of user-visible history.

For the current stub, `send` means: persist the user event locally, then put a record in the local outbox. Network submission belongs behind the `sync` boundary and is not implemented yet.

## API boundary

- Use documented OpenAI developer API surfaces.
- Do not infer access to personal ChatGPT history, ChatGPT memory, consumer subscription state, or internal ChatGPT endpoints.
- Keep API credentials out of source, fixtures, logs, transcripts, and generated artifacts.
- Keep remote ids alongside local ids rather than using remote ids as the only local identity.
- Preserve streaming, retry, interruption, tool-call, and error distinctions in durable state once those paths are implemented.
- Do not claim network/API acceptance from the current local-only stub.

## Storage and indexing

- `docs/local-state-v1.md` documents the current stub, not a permanent format guarantee.
- Prefer stable local records/fragments plus derived indexes over repeated copies of full message bodies.
- Keep readable UTF-8 available locally.
- `IGPT_HOME` is the explicit data-root override and may point at SD/removable storage.
- Compression is an empirical design choice, not a premise. If tested, use LZ4 as a latency baseline and zstd as a density comparison; benchmark on the actual target device and storage.
- If compression is adopted for interactive data, prefer independently decodable blocks over a monolithic archive.
- Do not choose SQLite, a filesystem-only final layout, or another database merely because the stub currently uses files. Decide from measured/specifed access patterns.

## Change discipline

- Keep vendor source, our observations, implementation policy, and executable evidence visibly separate.
- Mark unresolved choices as unresolved rather than manufacturing a decision.
- Prefer small branches and reviewable changes.
- Claims about current API behavior should name the upstream revision or a dated official source.
- Acceptance evidence must state what was actually exercised; do not promote desktop/emulator/cloud evidence into physical-device evidence.
- A Bash or generic shell run is not a Grease receipt. A Grease receipt must identify and exercise the current Grease/Oils/YSH implementation.
- Whenever giving the human a script or command block, assume `$PWD` is arbitrary. Resolve repository and file paths from the script's own location, an explicit project location, or a discovered repository root, and perform any required `cd` inside the script. Never require the human to `cd` first or rely on relative paths against their current working directory.
