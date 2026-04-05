# JSON‑Atom Format Conformance Fixtures

This directory contains test fixtures for verifying conformance with the JSON‑Atom Format specification.

## Fixture Format

Each fixture is a JSON file with the following structure:

```json
{
  "name": "fixture-name",
  "description": "What this fixture tests",
  "source": { ... },
  "target": { ... },
  "delta": {
    "format": "json-atom",
    "version": 1,
    "operations": [ ... ]
  },
  "computeHints": { ... },
  "level": 1
}
```

### Fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | Yes | Unique identifier for the fixture. |
| `description` | string | Yes | Human-readable description of what the fixture tests. |
| `source` | any | Yes | The source document (before changes). May be `null` for document creation tests. |
| `target` | any | Yes | The expected target document (after changes). May be `null` for document deletion tests. |
| `delta` | object | Yes | A valid JSON‑Atom Format document. |
| `computeHints` | object | No | Non-normative hints for diff implementations (e.g., array keys). |
| `level` | integer | Yes | Conformance level: `1` (Apply) or `2` (Reversible). |

### `computeHints`

The `computeHints` field provides non-normative metadata to help diff implementations produce the expected delta. It is NOT used during conformance verification — only during delta computation.

```json
{
  "computeHints": {
    "arrayKeys": {
      "items": "id",
      "users": "email"
    }
  }
}
```

- `arrayKeys`: A map from array property name to the identity key property. Tells diff implementations which property to use for key-based array comparison.

## Conformance Verification

### Level 1: Apply

For every fixture with `level >= 1`:

```
apply(fixture.source, fixture.delta) == fixture.target
```

Apply the delta to the source document. The result MUST equal the target document. Comparison uses JSON value equality (deep equality with strict type matching).

### Level 2: Reversible

For every fixture with `level == 2`:

```
apply(fixture.source, fixture.delta) == fixture.target
apply(fixture.target, inverse(fixture.delta)) == fixture.source
```

In addition to the Level 1 check, compute the inverse of the delta and apply it to the target document. The result MUST equal the source document.

Level 2 fixtures include `oldValue` on every `replace` and `remove` operation (and `value` on every `copy`), making the delta fully reversible.

## Fixture Index

| File | Level | Description |
| --- | --- | --- |
| `basic-replace.json` | 2 | Replace a top-level string property |
| `nested-replace.json` | 2 | Replace a deeply nested property |
| `property-add-remove.json` | 2 | Add and remove object properties |
| `type-change-replace.json` | 2 | Replace a value with a different type |
| `root-operations.json` | 1 | Create root document value |
| `index-array-operations.json` | 2 | Add, remove, and replace in index-based arrays |
| `value-filter-array.json` | 2 | Add and remove using value filters |
| `keyed-array-update.json` | 2 | Update, add, and remove in keyed arrays |
| `move-rename-property.json` | 2 | Move (rename) an object property |
| `move-keyed-array-element.json` | 2 | Move an element between keyed arrays |
| `copy-property.json` | 2 | Copy a property value to another location |
| `mixed-operations.json` | 2 | All five operation types in one delta |
