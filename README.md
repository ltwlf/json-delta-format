# JSON Delta

**JSON Delta** is a language-agnostic format for representing **atomic changes to JSON documents**.

It defines three operations — `add`, `remove`, and `replace` — addressed by [JSONPath](https://www.rfc-editor.org/rfc/rfc9535)-based paths that support **identity-based array element selection**. This is the key capability that sets JSON Delta apart from existing standards like [RFC 6902 (JSON Patch)](https://www.rfc-editor.org/rfc/rfc6902) and [RFC 7396 (JSON Merge Patch)](https://www.rfc-editor.org/rfc/rfc7396).

Rather than storing full JSON snapshots, systems can record a sequence of structured change operations and reconstruct state at any point in time. A JSON document can therefore be treated as a **materialized view over an ordered sequence of delta operations**.

---

## Quick Example

Source document:

```json
{
  "user": { "name": "Alice", "email": "alice@example.com" },
  "items": [
    { "id": "1", "name": "Widget", "price": 10 },
    { "id": "2", "name": "Gadget", "price": 20 }
  ]
}
```

Target document:

```json
{
  "user": { "name": "Anna", "email": "alice@example.com", "role": "admin" },
  "items": [
    { "id": "1", "name": "Widget Pro", "price": 15 },
    { "id": "3", "name": "Thingamajig", "price": 40 }
  ]
}
```

JSON Delta:

```json
{
  "format": "json-delta",
  "version": 1,
  "operations": [
    { "op": "replace", "path": "$.user.name", "value": "Anna", "oldValue": "Alice" },
    { "op": "add",     "path": "$.user.role", "value": "admin" },
    { "op": "replace", "path": "$.items[?(@.id=='1')].name",  "value": "Widget Pro", "oldValue": "Widget" },
    { "op": "replace", "path": "$.items[?(@.id=='1')].price", "value": 15, "oldValue": 10 },
    { "op": "remove",  "path": "$.items[?(@.id=='2')]",
      "oldValue": { "id": "2", "name": "Gadget", "price": 20 } },
    { "op": "add",     "path": "$.items[?(@.id=='3')]",
      "value": { "id": "3", "name": "Thingamajig", "price": 40 } }
  ]
}
```

Note how array elements are addressed by **key-based identity** (`@.id=='1'`) rather than by positional index. This makes deltas resilient to reordering and meaningful across systems that don't share array positions.

---

## Why JSON Delta?

### vs. RFC 6902 (JSON Patch)

[RFC 6902](https://www.rfc-editor.org/rfc/rfc6902) uses [RFC 6901 JSON Pointer](https://www.rfc-editor.org/rfc/rfc6901) for paths, which addresses array elements **only by index** (`/items/0`). If an array is reordered or an element is inserted, every index-based path may become invalid.

JSON Delta uses JSONPath filter expressions (`$.items[?(@.id=='1')]`) to address array elements by a **stable identity key**. This is the fundamental capability gap that JSON Delta fills.

### vs. RFC 7396 (JSON Merge Patch)

[RFC 7396](https://www.rfc-editor.org/rfc/rfc7396) merges changes by overlaying partial documents. It cannot represent `null` as a value (it means "delete"), cannot express array element changes atomically, and cannot produce reversible changesets.

JSON Delta represents each change as a discrete, atomic operation with an explicit path. It supports `null` as a first-class value and enables reversibility through optional `oldValue` fields.

### Summary

| Capability | RFC 6902 | RFC 7396 | JSON Delta |
|---|---|---|---|
| Atomic operations | Yes | No | Yes |
| Array identity by key | No | No | **Yes** |
| Reversible | No | No | **Yes** (with `oldValue`) |
| `null` as value | Yes | No | Yes |
| Self-identifying document | No | No | **Yes** (`format` field) |
| Extensible | No | No | **Yes** (`x_` properties) |

---

## Operations

| Operation | Description | Required fields | Optional fields |
|---|---|---|---|
| `add` | Add a value at a path that does not exist | `op`, `path`, `value` | |
| `remove` | Remove a value at a path that exists | `op`, `path` | `oldValue` |
| `replace` | Replace the value at a path with a new value | `op`, `path`, `value` | `oldValue` |

- **`value`** always means the new value (present on `add` and `replace`).
- **`oldValue`** always means the previous value (present on `remove` and `replace` for reversible deltas).
- A delta is **reversible** when every `replace` and `remove` operation includes `oldValue`.

---

## Delta Document Structure

```json
{
  "format": "json-delta",
  "version": 1,
  "operations": [ ... ]
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `format` | string | Yes | Must be `"json-delta"`. Self-identifying discriminator. |
| `version` | integer | Yes | Format version. Currently `1`. |
| `operations` | array | Yes | Ordered array of operation objects. |

Additional properties are allowed and preserved. Properties prefixed with `x_` are reserved for extensions and will never conflict with future spec versions.

---

## JSON Delta Path

JSON Delta uses a constrained subset of JSONPath ([RFC 9535](https://www.rfc-editor.org/rfc/rfc9535)) with six productions:

| Production | Example | Description |
|---|---|---|
| Root | `$` | The entire document |
| Dot property | `$.user.name` | Object property access |
| Bracket property | `$['a.b']` | Property with special characters |
| Array index | `$.items[0]` | Zero-based positional index |
| Key filter | `$.items[?(@.id==42)]` | Identity by object property |
| Value filter | `$.tags[?(@=='urgent')]` | Identity by primitive value |

**Canonical form:** Dot notation for safe identifiers (`[a-zA-Z_][a-zA-Z0-9_]*`), bracket notation for everything else. Array indices always use brackets.

**Filter literals:** String (`'val'`), number (`42`), boolean (`true`/`false`), null. Comparison uses JSON value equality — type and value must match.

---

## Design Goals

JSON Delta is designed to be:

- **Deterministic** — The same inputs produce the same logical delta.
- **Atomic** — Each operation targets exactly one location.
- **Language-agnostic** — Suitable for TypeScript, Python, .NET, and beyond.
- **Replay-friendly** — Ordered deltas reconstruct document state over time.
- **Self-identifying** — The `format` field enables detection without inspecting internals.
- **Extension-friendly** — Unknown properties are preserved; `x_`-prefixed properties are future-safe.

## Non-Goals

JSON Delta intentionally does **not** define:

- A diff algorithm (how to compute deltas)
- A comparison strategy (how to detect semantic changes)
- Conflict resolution or merge semantics
- Transport or storage protocols

---

## Use Cases

- **Change tracking** — Record what changed, when, and (via extensions) by whom.
- **Replayable state history** — Reconstruct any past document state from an initial snapshot plus ordered deltas.
- **State synchronization** — Exchange minimal change descriptions between systems.
- **Audit trails** — Immutable, structured records of every modification.
- **AI agent state tracking** — Inspect and replay agent execution step by step.
- **Time-travel debugging** — Navigate forward and backward through state evolution.

---

## Extensibility

JSON Delta supports extensions through additional properties on both the envelope and individual operations:

```json
{
  "format": "json-delta",
  "version": 1,
  "operations": [
    {
      "op": "replace",
      "path": "$.summary",
      "value": "Revenue declined 15%...",
      "oldValue": "Revenue grew strongly...",
      "x_comparison": { "strategy": "semantic-embedding", "similarity": 0.42 }
    }
  ],
  "x_agent": "financial-analyst-v2",
  "x_workflow_step": 3
}
```

- Properties prefixed with `x_` are guaranteed to never conflict with future spec versions.
- Conformant implementations **must** preserve unknown properties and **must not** reject them.

---

## Specification

The full draft specification is at [spec/v0.md](spec/v0.md).

Conformance test fixtures are in [fixtures/](fixtures/).

---

## Implementations

- **[json-diff-ts](https://github.com/ltwlf/json-diff-ts)** — TypeScript library for computing, applying, and reverting JSON changesets. The originating implementation from which JSON Delta was derived.
- **[json-delta-py](https://github.com/ltwlf/json-delta-py)** — Python library implementing the full JSON Delta v0 specification. Level 2 (Reversible) conformance.

---

## Specification Status

**Draft v0** — JSON Delta is a draft specification. The core format is stable, but details may evolve before a v1 release. Feedback and contributions are welcome.

---

## License

MIT
