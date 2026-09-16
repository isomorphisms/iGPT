---
schema_version: 1
kind: api_review_notes
subject: OpenAI API
verified_date: 2026-09-16
upstream_commit: ac89e26b5fa142cf9e0c4be4e19c1858607835c1
normative_spec: vendor/openai/openapi.yaml
source_pin: vendor/openai/UPSTREAM
implementation_language: grease-first
---

# OpenAI API notes

These notes are a machine-oriented second pass over the API surface. They are not the API contract. When these notes disagree with `vendor/openai/openapi.yaml`, the mirrored OpenAPI specification wins.

## Source boundary

- Upstream specification repository: `openai/openai-openapi`.
- Pinned revision: `ac89e26b5fa142cf9e0c4be4e19c1858607835c1`.
- Upstream describes the document as OpenAPI 3.1 and as the machine-readable description of endpoints, authentication, parameters, request schemas, and response schemas.
- `vendor/openai/openapi.yaml` is generated from that exact pin. Do not edit it by hand.
- `vendor/openai/UPSTREAM` records both the upstream commit and expected Git blob id.
- This repository concerns the developer API. Do not assume undocumented access to consumer ChatGPT conversations, ChatGPT memory, subscription state, or internal ChatGPT endpoints.

## Frontend-relevant surface

### Responses

- `POST /responses` is the central general model-response operation.
- Inputs can include text and images; outputs can include text or structured JSON.
- Responses can invoke application-defined tools and supported hosted tools.
- A response may be associated with a conversation. Conversation items are supplied as context and response input/output items can be added to that conversation.
- Response storage is configurable. Current reference behavior says `store` defaults to true when omitted and stored response data is retained for at least 30 days, subject to documented retention exceptions.
- Responses support streaming. The ordinary HTTP streaming representation uses server-sent events.
- `GET /responses/{response_id}` retrieves a stored response.
- State can also be chained with `previous_response_id`; instructions from the previous response are not automatically carried forward merely because that identifier is supplied.

### Long-running state and compaction

- The API has response compaction support for carrying long-running work with a smaller model-context footprint.
- Compaction output is designed for continuation rather than human inspection; treat opaque/encrypted compacted material as protocol state, not as the local transcript.
- The current upstream revision includes a `response.compaction.compacting` streaming event.
- iGPT should retain its own readable local history regardless of whether API-side compaction is used.

### Conversations and server state

- API-side conversations are useful remote state but are not a substitute for iGPT's local corpus.
- Local records should retain stable identifiers for remote conversations, responses, messages/items, tool calls, and stream sequence positions when those identifiers exist.
- Do not require a complete remote-history fetch before rendering locally available history.

### Agents and sessions

- The current specification includes agent/session API surface.
- At the pinned revision, agent-session updates can change model, reasoning effort, and service tier for subsequent turns.
- Keep agent/session protocol objects distinct from iGPT's own durable conversation/index model.

### Live / realtime

- The API has live/realtime session surfaces in addition to ordinary request/response HTTP operations.
- Do not make the first iGPT storage model depend on a specific transport. Persist semantic events/items so an HTTP/SSE path and a live bidirectional path can share local history where their semantics overlap.

### Other API families

The specification, not this list, is exhaustive. Current public API surface also includes model discovery and specialized families such as audio/speech/transcription/translation, images, video, embeddings, moderation, files/uploads, batches, fine-tuning, realtime, and administrative organization/project resources. Legacy compatibility endpoints also exist. Do not infer endpoint availability from this summary; inspect the pinned OpenAPI contract.

## Authentication and trust boundary

- Standard API operations use an OpenAI API key; administrative operations have separate administrative credentials and scopes.
- Credentials are secrets and never belong in version control, logs, transcript fragments, crash reports, fixtures, or the mirrored specification.
- Before choosing the Android authentication architecture, distinguish a personal local client from a distributable client. Do not silently bake a shared project credential into an APK.

## Local-first implications for iGPT

### Authority

For user-visible history, prefer this authority order:

1. local immutable event/message record;
2. local indexes and derived views;
3. remote OpenAI object identifiers and synchronization metadata;
4. API-side retrieval as reconciliation, not as the only readable copy.

This lets an existing conversation paint immediately from local storage while the network fetches only new or missing state.

### Write path

A useful invariant for a new event is:

1. accept local user input;
2. persist the local record and ordering information;
3. place remote work in an explicit local outbox;
4. submit the API request;
5. persist streamed deltas/events or the completed response with remote identifiers;
6. update indexes asynchronously or incrementally;
7. reconcile remote state without blocking display of already-local material.

The first Grease stub implements steps 1-3 only. `sync` is an explicit unimplemented boundary. Exact crash-consistency and concurrent-writer semantics remain undecided and should be specified from evidence before they are claimed.

### Text representation

- Keep readable UTF-8 text available without requiring API access.
- Compression is not yet a repository requirement.
- If compression is introduced, benchmark on target hardware and storage rather than asserting a universally fastest codec.
- LZ4 is the latency-oriented baseline; zstd is the density-oriented comparison point. Independently decodable blocks are preferable to a monolithic archive for interactive random access.
- Derived indexes should refer to stable local records/fragments rather than duplicate complete message bodies unless a measured query path warrants duplication.

### Streaming

- Preserve ordering and remote event identifiers/sequence numbers where the API supplies them.
- The renderer should be able to display a partially received assistant result without rewriting the entire conversation file.
- Completion, interruption, retry, tool-call, and error states must remain distinguishable after restart.

### Synchronization

Track at least:

- local conversation id;
- optional remote conversation id;
- local item id;
- optional remote item/response id;
- parent/order relation;
- creation time when known;
- local persistence state;
- remote submission state;
- remote completion state;
- last observed stream/event position when applicable;
- model/configuration metadata needed to reproduce or understand the request.

Do not equate remote deletion, expiration, or inaccessible state with permission to destroy the local record unless an explicit local deletion policy says so.

## Implementation-language boundary

- The first executable client slice is Grease, using the current Oils/YSH-derived Grease line.
- The on-disk state must remain language-neutral.
- Idriç remains a later option for typed protocol/state components after real usage shows which distinctions and invariants need to be encoded.
- Do not reinterpret the Grease-first decision as a permanent prohibition on Idriç or as permission to make the storage format depend on Grease syntax.

## Things deliberately not decided yet

- Which later components, if any, should move from Grease to Idriç.
- Exact final on-disk record format beyond the documented local-state stub.
- SQLite versus filesystem-first metadata/index structures.
- Compression codec and block size.
- Whether iGPT talks directly to the API from a personal device or through an intermediary service.
- How much API-side conversation state to use versus resending locally selected context.
- Tool execution architecture.
- Import strategy for pre-existing ChatGPT exports.
- Automatic SD-card placement policy.

## Machine-reading rules

- Treat `vendor/openai/openapi.yaml` as normative for API shapes.
- Treat this file as design commentary and extracted operational constraints.
- Treat `docs/local-state-v1.md` as the current executable stub contract, not as a permanent storage promise.
- Preserve statements under `Things deliberately not decided yet` as unresolved; do not choose defaults merely to complete a task.
- Any future API claim should carry either an upstream spec revision or a dated official-documentation verification.
- When updating the upstream pin, regenerate the mirror first, then review this file for drift.
