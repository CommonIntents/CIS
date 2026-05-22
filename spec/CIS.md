# CIS: Common Intents Specification

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Version](https://img.shields.io/badge/Version-0.6.0--draft-orange.svg)]()
[![Status](https://img.shields.io/badge/Status-RFC%20Draft-yellow.svg)]()
[![Org](https://img.shields.io/badge/Org-CommonIntents-darkgray.svg)](https://github.com/CommonIntents)

**Version**: 0.6.0-draft
**Status**: Request for Comments (RFC)
**Editor**: Jasonmilk and the Cellrix Open Source Community
**License**: Apache 2.0

---

## 1. Abstract

The Common Intents Specification (CIS) is a **device-independent intent protocol for AI**.

It defines only the semantic standard of "what AI wants to execute as a business action." Through a local static mapping layer, it losslessly translates into CLI commands, GUI interactions, TUI terminal component operations, and CSS visual feedback. The upper-layer AI always uses a unified interface, while the lower-layer interaction backends can be freely switched.

CIS is a UI-independent declarative protocol that completely decouples "what can be done" (intents) from "how to trigger" (bindings and mappings), creating a standardized semantic layer for both human users and AI agents.

**CIS is the native language of AI.** The reasoning core of AI is non-linguistic — when it wants to "search," that intent itself has nothing to do with any specific grammar. CIS encodes this pure, pre-linguistic intent, allowing AI to understand nothing about the underlying backend. It only needs one language — CIS. The existence of all other languages is completely transparent to AI.

**CIS is digital water.** It is formless — poured into CLI, it becomes command parameters; poured into GUI, it becomes click events; poured into TUI, it becomes panel operations. It does not compete — it does not replace any existing infrastructure, merely providing an extremely thin, unified semantic layer on top of all infrastructure. It flows downward — using compile-time deterministic code translation, consuming zero AI Tokens, translated once and reused forever.


## 2. Philosophical Foundation

### 2.1 The Native Language of AI

The reasoning core of AI is non-linguistic. When a human thinks "bring that thing over," this intent, as it forms in the mind, is not Chinese, not English, not any language — it is a pure, pre-linguistic intent. The brain's language center then automatically translates it into the required form of expression.

CIS is the pre-linguistic intent layer for AI. AI does not need to choose among multiple languages, nor does it need to understand the syntax of the underlying backend. It only needs one language — CIS.

### 2.2 A World Without Syntax Errors

In the world of CIS, syntax errors do not exist. Only logical errors and philosophical errors exist.

| Error Type | Who Errs | Layer |
|:---|:---|:---|
| Syntax Error | Does not exist | CIS parser directly rejects illegal intents |
| Logical Error | AI | Reasoning layer — mistakes AI makes in its own reasoning space |
| Philosophical Error | Protocol designer | Protocol layer — structural design flaws |

CIS removes syntax from AI's list of responsibilities. AI does not need to worry about "did I say this correctly?" — it only needs to focus on "did I think this correctly?"

### 2.3 Core Beliefs

**Trust must be proven, not assumed.**

**Achieve maximum functionality with minimum force.**

**How far a protocol can go depends on how much it can embrace.**


## 3. Conventions and Terminology

The key words "MUST," "MUST NOT," "REQUIRED," "SHALL," "SHALL NOT," "SHOULD," "SHOULD NOT," "RECOMMENDED," "MAY," and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

- **Interface**: An interactive surface that exposes intents (terminal, web, mobile, VR).
- **Intent**: A discrete, invocable operation defined by an id, description, optional parameters, and security constraints.
- **Agent**: Any client that invokes an intent — a human through a UI, or an AI through an API.
- **Binding**: A declaration that a specific intent is anchored to a particular element in the presentation layer.
- **Static Mapping Table**: A set of purely local code, determined at compile-time or load-time, that losslessly translates CIS intents into native operations for each backend. Zero AI involvement, zero Token consumption.
- **Backend**: The specific type of execution environment — CLI, GUI, TUI, CSS, Web, Mobile.
- **Human-in-the-Loop (HITL)**: A security mechanism that requires explicit human confirmation before executing high-risk intents.


## 4. Design Principles

1. **Semantically Pure, Transport-Agnostic, Backend-Agnostic**: CIS is an eternal intent language, uninvolved with transport, encryption, identity, or execution backends.
2. **Minimal Core, Optional Extensions**: Keep the base vocabulary as small as possible; custom data is added through `x-` prefixed fields.
3. **Declarative Activation, On-Demand Execution**: Advanced features are activated only when explicitly declared; not declared means zero overhead.
4. **Maximum Embrace, Minimum Exclusion**: Define only "what is correct," not "how to do it."
5. **Zero-Dependency Ready, Progressive Enhancement**: Basic compatibility requires only HTTP services and JSON parsing.
6. **Orthogonality**: Intents exist independent of presentation. An interface can expose 50 intents but render only 5 of them as visible elements; an authorized agent can access all 50.


## 5. Position in the Protocol Stack

CIS is the **soul layer** of the CIS/CAP protocol family. Its position in the four-layer protocol stack:

```
┌─────────────────────────────────────────┐
│              CIS  ← This Protocol        │
│  Common Intent Specification             │
│  · Pure intent semantic standard         │
│  · Transport-agnostic, crypto-agnostic   │
│  · Backend-agnostic                      │
└─────────────────────────────────────────┘
                    ▲ Semantic Binding
┌─────────────────────────────────────────┐
│              CIB                         │
│  CIS/Transport Binding Protocol          │
└─────────────────────────────────────────┘
                    ▲ Currently Bound To
┌─────────────────────────────────────────┐
│              CISS                        │
│  Secure Intent & Control Protocol        │
└─────────────────────────────────────────┘
                    ▲ Carried By
┌─────────────────────────────────────────┐
│              CAP                         │
│  Capability Authentication Protocol      │
└─────────────────────────────────────────┘
```

- **CIS defines** "what AI wants to do" — intent semantic standards
- **CAP defines** "what AI can do, under what conditions, within what timeframe" — declaring which CIS intents an application supports through its Manifest
- **CIB defines** "how intents are transported" — format negotiation and integrity assurance
- **CISS defines** "who is speaking, and whether the channel is secure" — mTLS identity verification


## 6. Intent Registry

A CIS document is a JSON object representing an Intent Registry. It MUST be UTF-8 encoded.

### 6.1. Root Schema

```json
{
  "$schema": "https://cis.cellrix.org/schema/v0.6/cis.json",
  "cis_version": "0.6",
  "interface_id": "string (optional)",
  "intents": [...]
}
```

- `$schema` is OPTIONAL and MAY be used for automated validation of the registry itself.
- `cis_version` declares the specification version the registry follows.
- `interface_id` is an optional identifier for the originating interface.

### 6.2. Intent Object

| Field | Type | Required | Description |
|---|---|---|---|
| id | string | Yes | Globally unique identifier (snake_case recommended) |
| name | string | Yes | Short human-readable name |
| description | string | Yes | Detailed description suitable for LLM consumption, MUST describe the side effects of this operation |
| parameters | object | No | A JSON Schema (Draft 2020‑12) defining the expected payload |
| security | object | No | Execution constraints (see §7) |
| bindings | array | No | Element anchors in the presentation layer (see §8.1) |

### 6.3. Core Intent Set

| Intent | Description | Applicable Backends |
|:---|:---|:---|
| `Search` | Search for specified content | All |
| `Click` | Click a specified component | GUI, TUI, Web |
| `Input` | Input text into a specified area | GUI, TUI, Web |
| `ListDir` | List directory contents | CLI, GUI |
| `Navigate` | Navigate to a specified location | All |
| `Read` | Read specified content | All |
| `Confirm` | Confirm an operation | GUI, TUI, Web |
| `Cancel` | Cancel an operation | GUI, TUI, Web |
| `Select` | Select from a list | GUI, TUI, Web |
| `Execute` | Execute a specified command | CLI |


## 7. Security Constraints

The `security` object defines the trust requirements for an intent. If omitted, the intent is treated as `risk_level: low` and does not require human confirmation.

| Field | Type | Description |
|---|---|---|
| risk_level | string | Values: `low`, `medium`, `high`, `critical` |
| requires_hitl | boolean | If true, the runtime MUST pause execution until a human explicitly approves |
| required_scopes | array of strings | List of permission scopes the caller must possess |

**HITL Flow**: When invoking an intent with `requires_hitl: true`, the implementation MUST:
- Immediately suspend the operation
- Present a confirmation prompt to the human user
- Wait for explicit approval or rejection (or timeout)
- Resume execution only upon approval

**Note**: Runtime implementation details such as HITL asynchronous queues, decision state machines, and approval signature standardization are defined in the **CAP Protocol** (Capability Authentication Protocol). CIS defines only intent-level security constraint declarations, not HITL runtime implementation mechanisms.


## 8. Binding and Static Mapping Table

### 8.1. UI Binding

A binding declares that a specific intent is anchored to a particular element in the presentation layer. A single intent can have multiple bindings.

Binding Object:

| Field | Type | Required | Description |
|---|---|---|---|
| type | string | Yes | MUST be `"ui_element"` |
| target_id | string | Yes | Identifier of the element in the presentation layer |

The presentation layer is entirely responsible for how a bound element is triggered — keyboard, mouse, touch, voice, or any other input modality. CIS does not define input modalities.

### 8.2. Static Mapping Table

**The static mapping table is the key mechanism by which CIS achieves ultimate efficiency.**

It is a set of purely local code, determined at compile-time or load-time, that translates CIS intents into native operations for each backend. This translation process involves no AI reasoning and consumes no Tokens. Each backend maintains its own mapping table, completely decoupled from others.

```
AI always outputs only: Search
                ↓
        Static Mapping Table (Zero AI involvement, Zero Token consumption)
                ↓
    CLI:  app-cli search --query=...
    GUI:  Locate search box → Focus → Input → Click
    TUI:  Switch to search panel → Focus cursor → Input → Enter
    CSS:  Add .focus class to search box + highlight animation in results area
```

**Why does adding one more layer actually improve efficiency?**

Data flow path: `AI Reasoning → CIS Intent → Static Mapping → CLI Command → Execution`

Compared to AI directly generating CLI commands, the path adds a "CIS Intent → Static Mapping" layer. However, the computational cost of this layer (nanosecond-level local code matching) is far lower than the Token consumption required for AI to handle environment adaptation during reasoning. Extremely cheap local computation is exchanged for extremely expensive AI context overhead.

| Dimension | AI Directly Generates CLI | AI Generates CIS + Static Mapping |
|:---|:---|:---|
| Context AI must understand | OS, Shell, command syntax | None, AI only sees unified intents |
| Token Consumption | High (environment info occupies context) | Low (intent itself only) |
| Cross-platform migration | AI must re-reason | Same `ListDir` output, mapping table auto-translates |
| Local computation overhead | Zero | One match (< 1μs) |

**Maintenance of Static Mapping Tables**: The mapping relationship between each CIS intent and its native operation is precise, enumerable, and one-to-one. Similar backends can generate mapping code in batches through templates. Each backend's mapping table is maintained by the person most familiar with that backend, and all mapping tables evolve independently without interdependence.

### 8.3. Backend Mapping Completeness

CIS covers all mainstream interaction paradigms:

- **CLI (Command-Driven)**: CIS Intent → CLI command and parameters
- **GUI (Component-Driven)**: CIS Intent → Interface component location and simulated operation
- **TUI (Terminal GUI)**: CIS Intent → Terminal panel and cursor operation. TUI is essentially a component-driven interface in the terminal; its mapping logic is closer to GUI, not CLI.
- **CSS (Presentation-Driven)**: CIS Intent → Style feedback. Does not execute actions, only provides visual feedback.
- **Web / Mobile**: CIS Intent → DOM operations or native control operations


## 9. Parameter Schema (JSON Schema Integration)

CIS adopts JSON Schema Draft 2020‑12 for parameter definition. The `parameters` field MUST be a valid JSON Schema object. Implementations MUST validate any incoming payload against this Schema before execution.

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

If validation fails, the implementation MUST reject the call with a descriptive error.

**Null Rule**: If an intent does not define a `parameters` field, the invocation request MUST NOT contain a `parameters` key. Passing `{}` or `null` when the intent declares no parameters SHALL be rejected as an invalid request.


## 10. Execution Lifecycle

All conformant runtimes MUST implement the following lifecycle:

- **Discovery** — The agent reads the intent registry. No side effects are produced.
- **Payload Construction** — The agent constructs a JSON payload matching the intent's parameters Schema.
- **Dispatch** — The agent sends an invocation request containing the intent_id and, if applicable, parameters.
- **Schema Validation** — The runtime validates the payload against the JSON Schema. Invalid payloads are rejected immediately (fail-fast).
- **Security Interception** — If requires_hitl is true, execution is suspended, the status changes to PENDING_APPROVAL, and the human confirmation flow is triggered.
- **Execution & Response** — The validated payload is passed to business logic. A deterministic response is returned.


## 11. Invocation Request and Response

### 11.1. Generic Invocation Request

```json
{
  "intent_id": "reboot_server",
  "parameters": {
    "instance_id": "i-abc123",
    "force": false
  }
}
```

If the intent declares no parameters, the parameters key MUST NOT be included:

```json
{ "intent_id": "logout" }
```

### 11.2. Generic Invocation Response

**Synchronous Completion**:

```json
{
  "success": true,
  "intent_id": "reboot_server",
  "status": "completed",
  "result": "Reboot has been initiated."
}
```

**Asynchronous Acceptance** (for long-running tasks):

```json
{
  "success": true,
  "intent_id": "reboot_server",
  "status": "accepted",
  "job_id": "job_9876xyz",
  "result": "Reboot task has been submitted and is running."
}
```

- status: MUST be `"completed"` (task finished synchronously) or `"accepted"` (task still in progress).
- job_id: REQUIRED when status is `"accepted"`.

**If HITL is required and not pre-approved**:

```json
{
  "success": false,
  "intent_id": "reboot_server",
  "error": "confirmation_required",
  "message": "This operation requires human confirmation."
}
```


## 12. Extensibility

Custom fields MAY be added to any CIS object. To avoid conflicts, all extension fields MUST be prefixed with `x-` (e.g., `x-telemetry-id`). Implementations MUST silently ignore unknown x- fields — this guarantees forward compatibility.


## 13. Relationship with Presentation Protocols and CAP

CIS is completely independent of any presentation technology. A CIS registry can be exposed by:
- Terminal applications (e.g., Cellrix-based TUI)
- Web applications (via REST API)
- Mobile applications
- Voice assistants

The presentation layer is responsible for rendering UI elements and mapping user input to bound intents, which is beyond the scope of CIS.

**Relationship between CIS and CAP**: CIS defines "what AI wants to do"; CAP defines "what AI can do, under what conditions, within what timeframe." CAP's Manifest declares which CIS intents an application supports and the security constraints for each intent. CIS intents are the semantic source of the `name` field in the `actions` array of the Manifest.

**CIS and AI Code Generation**: AI possesses two independent capability systems — CIS + Static Mapping (for executing operations) and pre-trained knowledge (for generating code). These two are not nested; they serve different scenarios and follow different paths.


## 14. Conformance

An implementation conforms to CIS v0.6.0 if it:
- Can parse and validate an intent registry (§6)
- Validates invocation parameters against the provided JSON Schema (§9)
- Enforces HITL for intents that require it (§7)
- Returns responses in the format described in §11.2
- Rejects invocations containing `parameters` when the intent declares no parameters (§9)
- Silently ignores unknown x- fields (§12)

The reference implementation provides a conformance test suite.


## 15. Reference Implementation

The reference implementation of CIS is maintained as part of the Cellrix project. It demonstrates:
- A Python library for parsing and validating CIS registries
- HTTP endpoints for intent invocation
- A complete HITL operation interceptor
- A conformance test suite

CIS is an independent specification. Any UI framework that implements the contract described in this document is a valid CIS runtime.

**Platform Independence**: The CIS protocol specification itself is published via content addressing (CID). The authoritative version of this white paper is identified by its CID, not by its specific URL.

**Related Protocols and Implementations**:

| Protocol | Repository | Responsibility |
|:---|:---|:---|
| CIS | CommonIntents/CIS | Common Intent Semantic Standard (this protocol) |
| CAP | CommonIntents/CAP | Capability Authentication and HITL Decisions |
| CIB | CommonIntents/CIB | Transport Binding and Format Negotiation |
| CISS | CommonIntents/CISS | mTLS Secure Transport |

| Reference Implementation | Repository | Role |
|:---|:---|:---|
| Cellrix | Jasonmilk/Cellrix | Reference implementation and testbed for CIS/CAP |


## 16. Versioning

This specification follows Semantic Versioning. Breaking changes to intent structure, invocation contracts, or security models will result in a major version increment. New optional fields, binding types, or clarifications will result in a minor version increment.


## Version Status

This specification is currently in the early draft stage (v0.6.0). The core work at this stage is to complete protocol validation and refinement based on practical reference implementations. Formal community governance is not initiated at this time. Once the specification iterates and matures, approaching v1.0, the formal standards RFC process will commence.

---

*This document is released under the Apache 2.0 license. Contributions are welcome through the CommonIntents organization repository.*
