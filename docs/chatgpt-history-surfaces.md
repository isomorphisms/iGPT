---
schema_version: 1
kind: source_boundary_notes
subject: ChatGPT history and conversation surfaces
verified_date: 2026-09-16
source_registry: vendor/openai/chatgpt-history-sources.tsv
---

# ChatGPT history and conversation surfaces

This file separates four things that are easy to confuse. They are not interchangeable APIs.

## 1. OpenAI developer Conversations API

Authority: `vendor/openai/openapi.yaml`, pinned by `vendor/openai/UPSTREAM`.

The developer API has conversation objects used with developer-platform responses and items. These are not the same thing as the conversations in a consumer ChatGPT account sidebar.

At the checked API revision:

- conversations can be created and retrieved;
- conversation items can be added, listed, retrieved, and deleted according to the OpenAPI contract;
- a conversation update changes `metadata`;
- the documented conversation object has an id, creation time, metadata, and object type;
- there is no documented ChatGPT-sidebar `title` field or operation that means "rename this personal ChatGPT chat";
- do not infer that a developer conversation id can address a consumer ChatGPT conversation.

A local iGPT title may be stored independently. If iGPT chooses to duplicate a title into developer-conversation metadata, that remains iGPT metadata and does not rename a ChatGPT sidebar entry.

## 2. Personal ChatGPT data export

Official source: <https://help.openai.com/en/articles/7260999>

OpenAI documents a supported export path for eligible personal ChatGPT accounts through Settings / Data controls or the Privacy Portal. The downloaded ZIP includes chat history and other eligible account data.

Treat this as an import/reference source for iGPT, not as a live synchronization or mutation API. An export may let iGPT recover old conversation text and metadata for local indexing, but it does not provide a supported mechanism for changing a chat title back in ChatGPT.

The exact export schema should be learned from real exports and fixtures rather than assumed from the help article. Preserve the original export unchanged when importing; derive iGPT records separately.

## 3. ChatGPT Enterprise/Edu Compliance Platform

Official source: <https://help.openai.com/en/articles/9261474>

The Compliance Platform is an administrative workspace surface for eligible ChatGPT Enterprise and Edu workspaces. It uses workspace-scoped administrative access and permissions and exposes compliance logs / state for governance, audit, eDiscovery, DLP, and related workflows.

It is not a personal ChatGPT-history API and must not be used as evidence that a personal account can enumerate or mutate its sidebar chats through the developer API.

If iGPT later supports managed-workspace ingestion, keep that adapter separate from personal export ingestion and from developer Conversations API synchronization.

## 4. ChatGPT product-internal history interfaces

The ChatGPT web and mobile products necessarily communicate with product backend services to display and modify product state. Those implementation details are not a documented public contract for iGPT.

As of the verification date above, the official sources checked for this repository do not provide a supported public personal-account API with a contract such as:

```text
list my ChatGPT sidebar chats
fetch a personal ChatGPT chat by sidebar id
rename a personal ChatGPT chat
archive a personal ChatGPT chat
```

Do not vendor guessed endpoints, copied browser requests, session cookies, or reverse-engineered request shapes as if they were an OpenAI API specification. If experimentation with product-internal behavior is ever performed, keep it in a clearly marked research area, never make it the default sync path, and never promote observed behavior into a supported contract without an official source.

## Consequence for chat retitling

There are two distinct retitling problems:

1. **iGPT-owned conversation title** — iGPT can define and persist this locally. This is under our control.
2. **Existing ChatGPT sidebar title** — no supported developer-API write path is documented for this. A browser/UI workflow may be able to apply a generated title, but that is UI automation, not developer-API synchronization.

A useful future `igpt retitle` command should therefore operate on local iGPT metadata first, retain the previous title in an audit receipt, and treat any ChatGPT-UI rename adapter as a separate optional boundary.

## Source update rule

- Developer API shapes come from the pinned OpenAPI mirror, not from this prose.
- Official non-OpenAPI ChatGPT sources are indexed in `vendor/openai/chatgpt-history-sources.tsv` with a verification date.
- Recheck those sources before making new claims about personal-history, export, or Compliance access.
- If OpenAI publishes a supported personal ChatGPT history API later, add it as a new source class rather than silently reinterpreting one of the existing classes.
