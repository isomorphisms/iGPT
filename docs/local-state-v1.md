---
schema_version: 1
kind: local_state_contract
implementation: grease-first
status: stub
---

# iGPT local state v1 stub

This document describes the first executable local-state slice. It is deliberately smaller than the eventual storage/index design.

## Goal

Already-seen conversation text must be replayable from local storage without contacting OpenAI. A new user message is persisted locally before it is eligible for remote submission.

## Data root

The command resolves the local data root in this order:

1. `IGPT_HOME`;
2. `$XDG_DATA_HOME/igpt`;
3. `$HOME/.local/share/igpt`.

`IGPT_HOME` is intentionally an ordinary path so a device can point iGPT at removable/SD storage without changing application semantics.

## Layout

```text
$IGPT_HOME/
  conversations/
    conversation.XXXXXXXX/
      conversation.meta
      next_sequence
      events/
        event.00000001.meta
        event.00000001.txt
        event.00000002.meta
        event.00000002.txt
  outbox/
    conversation.XXXXXXXX.event.00000001.pending
```

The `.txt` files are the readable UTF-8 bodies. Metadata and queue records are line-oriented `key=value` text. The metadata is intentionally restricted to values that do not require an escaping scheme in this stub.

## Event write order

For a local event, the current stub does this:

1. choose the next unused sequence number;
2. write the UTF-8 body to a temporary sibling file;
3. rename the body into place;
4. write event metadata to a temporary sibling file;
5. rename metadata into place;
6. advance the sequence hint.

`send` then writes a pending outbox record only after the user event exists locally.

This gives the intended local-first ordering but is **not yet a full crash-consistency or concurrency protocol**. There is no multi-process lock, directory `fsync`, transaction journal, remote retry state machine, or recovery sweep in this stub.

## Remote boundary

`sync` exists only as an explicit boundary and returns failure because network synchronization has not been implemented yet. No API key is read and no OpenAI request is made by this slice.

A later synchronizer should consume pending outbox records, preserve remote identifiers beside local identifiers, and change remote state without rewriting readable local history.

## Grease / Idriç boundary

The first executable implementation is Grease. The on-disk representation is not intended to be Grease-specific. A later Idriç component should be able to read the same records and strengthen the model around event kinds, tool calls, stream states, synchronization, and invariants without requiring a migration merely because the implementation language changes.

## Deliberately unresolved

- final record encoding;
- multi-process locking and crash recovery;
- SQLite versus filesystem/index structures;
- compression and block size;
- remote conversation usage versus locally selected context;
- API credential placement for Android/distribution;
- tool execution;
- ChatGPT export import;
- automatic SD-card placement policy.

Do not treat this v1 stub layout as a permanent storage format merely because code exists for it.
