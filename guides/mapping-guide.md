# Mapping Table Writing Guide

**File Path**: `CommonIntents/CIS/guides/mapping-guide.md`  
**Version**: v0.1.0  
**Status**: Draft
**Maintainer**: CommonIntents Protocol Family

---

### 1. Positioning and Boundaries

This guide is an **implementation reference for CIS (Common Intent Standard)** , not an independent protocol. It provides writing conventions for static mapping tables, helping developers systematically translate CIS intents into concrete software actions.

| Type | Responsibility |
|:---|:---|
| **CIS Protocol** | Defines the interaction semantics of intents (intent types, fields, required/optional) |
| **This Guide** | Defines how to map intents to executable actions (implementation pattern) |

**Core Principle**: Protocols define only interaction semantic standards. This guide provides only recommended implementation approaches. Any implementation that follows CIS semantics is fully CIS-compatible, even if it does not use the mapping table format described here.

---

### 2. Design Principles

- **Single Source of Truth**: The mapping table uses JSON as the sole canonical format—human-readable, directly editable, and trackable via Git.
- **Strict Validation**: Every mapping table must pass JSON Schema validation to ensure field completeness and type correctness.
- **Unambiguous Matching**: Entry matching proceeds in order. The first entry that satisfies all conditions takes effect immediately. No multi-entry merging occurs.
- **Open for Extension**: Implementation-specific extension parameters are allowed through the `custom` field. They must not affect core matching logic.
- **Semantic Versioning**: The mapping table carries its own version number, independent of the CIS protocol version, following semantic versioning rules.

---

### 3. Mapping Table Structure

#### 3.1 Top-Level Structure

```json
{
  "version": "0.1.0",
  "updated": "2026-05-22T00:00:00Z",
  "entries": []
}
```

| Field | Type | Required | Description |
|:---|:---|:---|:---|
| `version` | string | Yes | Version of this guide that the mapping table follows (SemVer) |
| `updated` | string | Yes | Date of last modification (ISO 8601) |
| `entries` | array | Yes | List of mapping entries, ordered by descending priority |

#### 3.2 Mapping Entry

```json
{
  "id": "map.snapshot.view",
  "description": "Handle agent.snapshot intent",
  "match": {
    "intent": "agent.snapshot",
    "format": "json",
    "ciss_required": "optional",
    "cap_required": ["read"]
  },
  "action": {
    "type": "render.static",
    "target": "snapshot_view",
    "params": {}
  }
}
```

**match fields** (all conditions must be satisfied simultaneously, AND logic):

| Field | Type | Required | Description |
|:---|:---|:---|:---|
| `intent` | string | Yes | CIS intent type, e.g. `agent.snapshot` |
| `format` | string | No | CIB format type (`json`/`cbor`/`binary`). Omit to accept any format |
| `ciss_required` | string | No | Security requirement: `required` / `optional` / `ignore`. Default: `ignore` |
| `cap_required` | array of string | No | CAP capability list, e.g. `["read"]`. Empty array means no capability required |
| `custom` | object | No | Extension point for implementation-specific match conditions. Unrecognized keys should be ignored |

**action fields**:

| Field | Type | Required | Description |
|:---|:---|:---|:---|
| `type` | string | Yes | Action type (see §4) |
| `target` | string | Yes | Action target: component name, function name, or resource address |
| `params` | object | No | Action parameters. Structure depends on action type |

#### 3.3 Matching Priority

The order of entries in the `entries` array defines priority. The **first** entry whose `match` conditions are all satisfied executes immediately. Subsequent entries are ignored. Therefore:

- More specific entries (e.g. `agent.snapshot`) should be placed first.
- Wildcard or default fallback entries should be placed last.

---

### 4. Action Types (Software Scope)

This guide currently covers only software interface scenarios. Hardware action types are reserved for future extension.

| Action Type | Description | Example `target` | Example `params` |
|:---|:---|:---|:---|
| `render.static` | Render a static component | `text_panel`, `list_view` | `{"template": "snapshot.yaml"}` |
| `render.dynamic` | Render a dynamic component (data extracted from intent payload) | `chart_view` | `{"data_path": "payload.values"}` |
| `ui.action` | Trigger a UI state change (e.g. navigation, modal) | `navigate.to_home` | `{"transition": "fade"}` |
| `call.service` | Call a backend service endpoint | `fetch_user_list` | `{"method": "GET", "path": "/users"}` |
| `custom` | Custom action | Implementation-defined | Implementation-defined |

Implementations may extend action types as needed, but cross-implementation interoperability is not guaranteed.

---

### 5. Extension Mechanism

`match.custom` and `action.params` are open extension points. Implementations may add their own key-value pairs for specific scenarios (e.g. performance weighting, debug flags). However, extension fields:

- Must not affect core matching logic.
- Must be ignored by parsers that do not recognize them.
- Have semantics that are the sole responsibility of the implementer and fall outside the scope of this guide.

---

### 6. Version Evolution

Mapping tables use Semantic Versioning (SemVer 2.0.0):
- **MAJOR**: Incompatible entry structure changes (e.g. removing fields, changing field types).
- **MINOR**: Backward-compatible additions (e.g. new action types, new optional match fields).
- **PATCH**: Documentation corrections, entry description updates.

When CIS protocol upgrades cause changes in intent types or fields, the mapping table should be versioned and re-validated accordingly.

---

### 7. Validation

After writing a mapping table, validate its structure using the companion `mapping-schema.json`. Example command (using the `jsonschema` tool):

```bash
jsonschema -i map.json cis/spec/schema/mapping-schema.json
```

Only tables that pass validation are considered valid. Adding a validation step to CI workflows is recommended.

---

### 8. Relationship to the Protocol Family

| Protocol | How This Guide References It |
|:---|:---|
| **CIS** | `match.intent` uses intent types defined by CIS |
| **CIB** | `match.format` uses format types defined by CIB |
| **CISS** | `match.ciss_required` defines the security level required for this intent |
| **CAP** | `match.cap_required` defines the capabilities required to execute this intent |

This guide defines only the **structure** of mapping tables. It does not modify the semantics of any protocol. It is a signpost for implementing CIS, not a part of CIS itself.

---

### 9. Example: Minimal Working Mapping Table

```json
{
  "version": "0.1.0",
  "updated": "2026-05-22T00:00:00Z",
  "entries": [
    {
      "id": "map.snapshot",
      "description": "Handle agent.snapshot intent, render static snapshot view",
      "match": {
        "intent": "agent.snapshot",
        "format": "json",
        "ciss_required": "optional"
      },
      "action": {
        "type": "render.static",
        "target": "snapshot_view"
      }
    },
    {
      "id": "map.fallback",
      "description": "Default handler for all unmatched intents",
      "match": {
        "intent": "*"
      },
      "action": {
        "type": "render.static",
        "target": "unknown_intent_view"
      }
    }
  ]
}
```

---

*This guide evolves alongside the CIS protocol. Issues for discussion and improvement are welcome.*
