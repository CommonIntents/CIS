# CIS: Common Intents Specification

**Version:** 0.6.0-Draft
**Status:** Request for Comments (RFC)
**Editors:** Jasonmilk & Cellrix Open Source Community
**License:** MIT

## 1. Abstract

The Common Intents Specification (CIS) is a UI-agnostic, declarative protocol that describes the interactive capabilities of a software system. It decouples **what can be done** (Intent) from **how it is triggered** (Binding), creating a standardized semantic layer for both human users and AI agents.

CIS is designed natively for the post-AGI era. It enables LLMs, autonomous agents, and assistive technologies to discover, comprehend, and invoke actions with deterministic safety — no OCR, no heuristic parsing of UI elements, no brittle assumptions.

## 2. Conventions and Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

- **Interface**: An interactive surface (terminal, web, mobile, VR) that exposes Intents.
- **Intent**: A discrete, invocable operation defined by an `id`, a description, optional parameters, and security constraints.
- **Agent**: Any client — human via UI, or AI via API — that invokes Intents.
- **Binding**: A declaration that an Intent is anchored to a specific element in the presentation layer.
- **HITL (Human-in-the-Loop)**: A security mechanism requiring explicit human confirmation before a high‑risk Intent is executed.

## 3. Design Goals

1. **Orthogonality**: Intents exist independently of presentation. An interface may expose 50 Intents but only render 5 as visible elements; an authorized Agent may access all 50.
2. **Strict Contracts**: Every Intent carries a machine‑verifiable parameters schema (JSON Schema Draft 2020‑12). No fuzzy natural‑language contracts.
3. **Agent‑First**: Descriptions are written for LLM consumption — they MUST include expected side effects.
4. **Minimal Core, Extensible**: The base vocabulary is the smallest possible set. Custom data can be added via `x-` prefixed fields.
5. **Idempotent Discovery**: Reading the Intent registry MUST NOT cause side effects.

## 4. The Intent Registry

A CIS document is a JSON object representing an **Intent Registry**. It MUST be encoded in UTF-8.

### 4.1. Root Schema

```json
{
  "$schema": "https://cis.cellrix.org/schema/v0.6/cis.json",
  "cis_version": "0.6",
  "interface_id": "string (optional)",
  "intents": [ ... ]
}
```

- `$schema` MAY be used for automated validation of the registry itself.
- `cis_version` declares the specification version the registry adheres to.
- `interface_id` is an optional identifier for the source interface.

### 4.2. The Intent Object

| Field | Type | Required | Description |
|:---|:---|:---|:---|
| `id` | string | Yes | Globally unique identifier (snake_case RECOMMENDED). |
| `name` | string | Yes | Short human‑readable name. |
| `description` | string | Yes | Detailed explanation suitable for LLM consumption. MUST describe the action's side effects. |
| `parameters` | object | No | A JSON Schema (Draft 2020‑12) defining the expected payload. Omit if the Intent takes no arguments. |
| `security` | object | No | Execution constraints (see §4.3). |
| `bindings` | array | No | Element anchors in the presentation layer (see §4.4). |

### 4.3. Security Constraints

The `security` object defines the trust requirements for an Intent. If omitted, the Intent is considered `risk_level: low` and does not require human confirmation.

| Field | Type | Description |
|:---|:---|:---|
| `risk_level` | string | One of `low`, `medium`, `high`, `critical`. |
| `requires_hitl` | boolean | If `true`, the runtime MUST suspend execution until a human explicitly approves. |
| `required_scopes` | array of strings | A list of capability scopes (e.g., `admin:write`) that the caller must possess. |

**HITL Flow**: When an Intent with `requires_hitl: true` is invoked, the implementation MUST:

1. Immediately suspend the action.
2. Present a confirmation prompt to the human user, including the Intent `name` and `description`.
3. Wait for explicit approval or denial (or timeout).
4. Resume execution only on approval; otherwise return an appropriate error.

When an Agent receives a `confirmation_required` error, it SHOULD either poll the execution status (if the transport layer supports it) or wait for an asynchronous event from the implementation signaling approval or denial.

### 4.4. Bindings

Bindings declare that an Intent is anchored to a specific element in the presentation layer. A single Intent MAY have multiple bindings.

**Binding Object**:

| Field | Type | Required | Description |
|:---|:---|:---|:---|
| `type` | string | Yes | MUST be `"ui_element"`. |
| `target_id` | string | Yes | Identifier of the element in the presentation layer. |

The presentation layer is fully responsible for how a bound element is triggered — keyboard, mouse, touch, voice, or any other input modality. CIS does not define input modalities.

```json
"bindings": [
  { "type": "ui_element", "target_id": "restart_btn" }
]
```

### 4.5. Parameters Schema (JSON Schema Integration)

CIS adopts **JSON Schema Draft 2020‑12** for parameter definition. The `parameters` field MUST be a valid JSON Schema object. Implementations MUST validate any incoming payload against this schema before execution.

```json
"parameters": {
  "type": "object",
  "properties": {
    "instance_id": { "type": "string" },
    "force": { "type": "boolean", "default": false }
  },
  "required": ["instance_id"]
}
```

If validation fails, the implementation MUST reject the invocation with a descriptive error.

**Nullability Rule**: If an Intent does not define a `parameters` field, the invocation request **MUST NOT** contain a `parameters` key. Passing `{}` or `null` when no parameters are declared SHALL be rejected as an invalid request.

## 5. Execution Lifecycle

All conformant runtimes MUST implement the following lifecycle:

1. **Discovery** — Agent reads the Intent Registry. No side effects occur.
2. **Payload Construction** — Agent builds a JSON payload matching the Intent's `parameters` schema.
3. **Dispatch** — Agent sends an invocation request containing `intent_id` and, if applicable, `parameters`.
4. **Schema Validation** — Runtime validates the payload against the JSON Schema. Invalid payloads are rejected immediately (fail‑fast).
5. **Security Interception** — If `requires_hitl` is `true`, execution is suspended, the state becomes `PENDING_APPROVAL`, and the human confirmation flow is triggered.
6. **Execution & Response** — The validated payload is delivered to business logic. A deterministic response is returned.

## 6. Invocation Request and Response

### 6.1. Generic Invocation Request

```json
{
  "intent_id": "reboot_server",
  "parameters": { "instance_id": "i-abc123", "force": false }
}
```

If the Intent declares no `parameters`, the `parameters` key MUST be absent:

```json
{
  "intent_id": "logout"
}
```

### 6.2. Generic Invocation Response

**Synchronous completion**:
```json
{
  "success": true,
  "intent_id": "reboot_server",
  "status": "completed",
  "result": "Reboot initiated."
}
```

**Asynchronous acceptance** (for long‑running tasks):
```json
{
  "success": true,
  "intent_id": "reboot_server",
  "status": "accepted",
  "job_id": "job_9876xyz",
  "result": "Reboot task has been submitted and is running."
}
```

- `status`: MUST be `"completed"` if the task finished synchronously, or `"accepted"` if it is still in progress.
- `job_id`: REQUIRED when `status` is `"accepted"`. The Agent MAY use this identifier to query task progress (implementation‑defined).

If HITL is required and was not pre‑approved:
```json
{
  "success": false,
  "intent_id": "reboot_server",
  "error": "confirmation_required",
  "message": "This action requires human confirmation."
}
```

## 7. Extensibility

Custom fields MAY be added to any CIS object. To avoid collisions, all extension fields MUST be prefixed with `x-` (e.g., `x-telemetry-id`). Implementations MUST silently ignore unknown `x-` fields — this guarantees forward compatibility.

## 8. Relationship to Presentation Protocols

CIS is completely independent of any presentation technology. A CIS registry can be exposed by:

- A terminal application (e.g., a Cellrix‑based TUI)
- A web application (via REST API)
- A mobile app
- A voice assistant

The presentation layer is responsible for rendering UI elements and mapping user input to bound Intents. This is **out of scope for CIS**. A presentation protocol (such as Cellrix) MAY provide tools to auto‑generate Bindings from its layout, but this is not required for CIS compliance.

## 9. Conformance

An implementation conforms to CIS v0.6.0 if it:

1. Can parse and validate an Intent Registry (§4).
2. Validates invocation parameters against the provided JSON Schema (§4.5).
3. Enforces HITL for Intents that require it (§4.3).
4. Returns responses in the format described in §6.2.
5. Rejects invocation requests containing `parameters` when the Intent declares none (§4.5).
6. Silently ignores unknown `x-` fields (§7).

A conformance test suite is provided by the reference implementation.

## 10. Reference Implementation

The reference implementation of CIS is maintained as part of the **[Cellrix project](https://github.com/Jasonmilk/Cellrix)** . It demonstrates:

- A Python library for parsing and validating CIS registries.
- An HTTP endpoint for intent invocation.
- A full HITL Action Interceptor.
- A conformance test suite.

CIS is a standalone specification. Any UI framework that implements the contract described in this document is a valid CIS runtime.

## 11. Versioning

This specification follows Semantic Versioning. Breaking changes to the Intent structure, invocation contract, or security model will result in a major version increment. New optional fields, binding types, or clarifications will increment the minor version.

---

*This document is published under the MIT License. Contributions are welcome via the [Cellrix repository](https://github.com/Jasonmilk/Cellrix).*
