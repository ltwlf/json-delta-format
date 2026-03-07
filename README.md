# JSON Delta

**JSON Delta** is a draft, language-agnostic format for representing **atomic changes to JSON documents**.

It is inspired by the capabilities of [`json-diff-ts`](https://github.com/ltwlf/json-diff-ts), a TypeScript library that can:

- calculate differences between JSON objects
- apply and revert changesets
- identify array items by key instead of only by index
- represent atomic changes with JSONPath-based paths

Rather than repeatedly storing full snapshots, systems can record a sequence of structured change operations and reconstruct state over time.

A JSON document can therefore be treated as a **materialized view over an ordered sequence of delta operations**.

---

## Why JSON Delta?

Many systems store full JSON snapshots even when only a few fields change.

A deterministic delta format can serve as a foundation for:

- change tracking
- replayable state history
- version reconstruction
- audit history
- time-travel debugging
- state synchronization
- AI agent state tracking and replay

---

## What Exists Today

The current reference capability comes from **`json-diff-ts`**.

According to its public README, `json-diff-ts` already supports:

- **Key-based array identification** for comparing array elements using keys instead of indices
- **JSONPath support** for targeting specific parts of JSON documents
- **Atomic changesets** via `atomizeChangeset` / `unatomizeChangeset`
- **Apply and revert** operations via `applyChangeset` / `revertChangeset`
- **Type change handling** and **path skipping**
- Use in browsers, Node.js, and modern TypeScript projects

This repository is intended to define a cleaner, implementation-independent format on top of those ideas.

---

## Example

Original document:

```json
{
  "user": {
    "name": "Alice",
    "email": "alice@example.com"
  }
}
```

Updated document:

```json
{
  "user": {
    "name": "Anna",
    "email": "alice@example.com",
    "role": "admin"
  }
}
```

Possible delta operations:

```json
[
  { "op": "replace", "path": "$.user.name", "value": "Anna" },
  { "op": "add", "path": "$.user.role", "value": "admin" }
]
```

This example reflects the kind of JSONPath-addressable atomic change model that `json-diff-ts` already exposes.

---

## Design Goals

JSON Delta aims to be:

- **Deterministic**  
  The same inputs should produce the same logical delta.

- **Atomic**  
  Changes should be representable as granular operations.

- **Language-agnostic**  
  The format should be suitable for implementations across TypeScript, Python, .NET, and other ecosystems.

- **Replay-friendly**  
  Ordered deltas should be usable to reconstruct document state over time.

- **Practical for real systems**  
  Especially systems involving state sync, audit trails, and AI agent workflows.

---

## JSON Delta for AI Systems

Modern AI applications often maintain structured state that changes step by step during workflows.

A delta-oriented model can help with:

- replaying agent execution
- inspecting state evolution
- synchronizing state between UI, backend, and agents
- tracking which operations changed specific fields

JSON Delta is therefore intended to be useful for **AI agent state tracking and debugging**, while remaining general-purpose.

---

## Specification Status

⚠️ **Draft Specification**

JSON Delta is currently a draft concept and not yet a finalized standard.

At the moment:

- the mature implementation is **`json-diff-ts`**
- this format repository is intended to extract and stabilize the core ideas into a cleaner cross-language spec
- naming and operation semantics may still evolve before a v1 release

---

## Reference Implementation

Current implementation source of truth:

- **`json-diff-ts`** — TypeScript library for diffing, applying, reverting, and atomizing JSON changesets

---

## Repository Goals

This repository is intended to contain:

- the format draft
- operation definitions
- examples
- conformance fixtures
- guidance for future implementations in other languages

---

## License

MIT
