# INTENT-7: Structured Intent Description Language (SIDL) Specification (v1.0.0-RFC-4)

## 1. Introduction and Objectives

This specification defines **INTENT-7**, the Structured Intent Description Language (SIDL) within the **CommonIntents-144 (CI-144)** suite.

INTENT-7 operates strictly as a **pure application/semantic layer payload protocol**. It is entirely decoupled from, and transparent to, the underlying transport framing, multiplexing, sliding-window flow control, or cryptographic link protections (which are fully delegated to **BIND-19** and **INTENT-7-SECURE**).

The core objectives of INTENT-7 are:
- To provide a highly deterministic, structured schema for AI-native intent declaration.
- To natively integrate W3C Trace Context standards for cognitive-physical causality tracing.
- To establish a zero-copy data contract between the execution runtime's Helix Execution Record (HXR) and the long-term memory's L3 episodic nodes.

---

## 2. W3C Trace Context and Metadata Schema

To guarantee that every cognitive decision and subsequent physical action can be traced back to its root cause across process and network boundaries, every INTENT-7 payload MUST include a standardized metadata header.

### 2.1 Metadata Schema

```json
{
  "metadata": {
    "protocol_version": "1.0",
    "traceparent": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
    "timestamp": 1782376405,
    "autonomy_level": "AGENT",
    "impasse_depth": 0
  }
}
```

#### Fields Description
- **`protocol_version`**: String. Defines the SemVer version of the INTENT-7 specification (negotiated via BIND-19 Handshake).
- **`traceparent` (W3C Standard)**: A 128-bit globally unique `Trace ID` followed by a 64-bit `Span ID`, providing the unified "neural signal fiber" for cognitive auditing.
- **`autonomy_level`**: Enum. Declares current relationship boundaries:
  - `AGENT`: Read-only transient interactions.
  - `OPEN`: Read-write collaborative mode.
  - `SURVIVAL`: Fully autonomous survival loop (minimum power consumption).
- **`impasse_depth`**: Integer (0-5). Tracks the current escalation state in the five-stage impasse model of the subconscious.

---

## 3. Core Intent Verbs (ACTIONs) and Schemas

INTENT-7 limits all semantic execution claims to a compact, non-overlapping set of verbs (ACTIONs). Each verb is strictly validated against its JSON Schema before enqueueing.

### 3.1 FETCH (Retrieve Semantic Memory)
Requests specific concepts or rules from the long-term knowledge base. Supports the critical **two-stage summary-first** performance optimization.

```json
{
  "action": "FETCH",
  "params": {
    "node_id": "algo_quick_sort",
    "mode": "summary"
  }
}
```

*   `mode == "summary"`: Returns only the lightweight `id`, `title`, and `summary` columns to save context window space.
*   `mode == "full"`: Returns the full text in `full_content` only if the LLM explicitly requests it after analyzing the summary.

### 3.2 WRITE_NODE (Commit Memory / Knowledge)
Appends a newly crystallized conclusion or observation into the memory DAG.

```json
{
  "action": "WRITE_NODE",
  "params": {
    "node_id": "hyp_user_preference",
    "type": "L3",
    "parent_ids": ["fact_001", "fact_002"],
    "content": { ... }
  }
}
```

### 3.3 TENTACLE (Invoke Stateless Execution)
Recommends the invocation of a sandboxed external CLI tool. Must declare the requested CAPABILITY-13 logical permissions for sandboxed enforcement.

```json
{
  "action": "TENTACLE",
  "params": {
    "tool_name": "targeted_scraper",
    "arguments": {
      "platform": "weibo",
      "keywords": ["hot_topic_x"]
    },
    "required_permissions": ["network:outbound", "filesystem:write"]
  }
}
```

### 3.4 FINISH / CANCEL
Yields execution and closes the current cognitive loop.
*   `FINISH`: Successfully resolves the root goal, returning the final transaction report.
*   `CANCEL`: Aborts the active execution tree due to a deep impasse or safety violation, passing a detailed diagnostics log.

---

## 4. Helix Execution Record (HXR) and L3 Node Alignment

To achieve **extreme reuse** and avoid costly, non-deterministic runtime translations, the schema of the **Helix Execution Record (HXR)**—which logs the precise execution trajectory of the FSM loop—is designed to be **completely identical** to the `content: JSON` payload inside a standard L3 episodic node.

### 4.1 HXR Payload Schema Specification

When an active session writes to the L3 memory DAG, the HXR payload is embedded directly inside the `content` block:

```json
{
  "session_id": "sess_20260411_001",
  "step_id": "step_003",
  "ts": "2026-04-11T14:23:00.123Z",
  "action": "FETCH",
  "target_graph": "knowledge_base",
  "node_id": "algo_quick_sort",
  "parent_ids": ["fact_001", "fact_002"],
  "intent": "Retrieve axiomatic definition of quicksort",
  "confidence": 0.85,
  "handler": "semantic_retrieval",
  "method": "inference",
  "tokens_used": 800,
  "duration_ms": 3200,
  "success": true,
  "error": null,
  "gene_lock_check": "passed",
  "user_authorized": false,
  "budget": {
    "tokens_remaining": 6200,
    "wallclock_remaining": 280,
    "api_calls_remaining": 45
  }
}
```

---

## 5. Transport Binding to BIND-19

All INTENT-7 payloads MUST be bound and transmitted as the payload of BIND-19 Data Frames (FrameType `0x01`):

- **Frame Type**: MUST be `0x01` (Data Frame) to indicate a standard semantic payload.
- **Channel ID**: Explicitly assigned by Anaphase's scheduler. Channel `0x00` is reserved for control and critical queries.
- **Flags**: 
  - Fragmented payloads over 64MB MUST use the `FIN` (`0x01`) flag to reassemble packets at the framing layer.
  - If the INTENT-7 action requires physical authorization, the `CON` (`0x02`) flag MUST be set to trigger asynchronous CAPABILITY-13 suspension.
- **Sequence ID**: Monotonically assigned per `Channel ID` as specified in BIND-19.

---

## 6. Error Handling

Intent-level schema or payload execution failures are propagated using BIND-19 Error Frames (`0x06`) containing one of the following semantic error codes in the payload:

| Error Code | Hexadecimal | Name | Description |
|:---|:---|:---|:---|
| `1600` | `0x0640` | `INVALID_PAYLOAD_SCHEMA` | The JSON payload failed schema validation against the verb definition. |
| `1610` | `0x064A` | `UNKNOWN_INTENT_VERB` | The specified `action` is not recognized by the executor. |
| `1620` | `0x0654` | `TRACE_CONTEXT_CORRUPTED` | The W3C `traceparent` is missing, malformed, or has an invalid hash chain. |

