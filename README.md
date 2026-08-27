# INTENT-7 — Common Intent Specification [![Org](https://img.shields.io/badge/Org-CommonIntents--144-darkgray.svg)](https://github.com/CommonIntents)

**Structured Intent Description Language (SIDL).** INTENT-7 is a syntactic standard for AI-native intent expression. It defines a strict JSON Schema for intent messages — what fields they contain, how they are validated, and how they are routed.

INTENT-7 is a minimalist syntactic layer. It defines structure, not behavior. Adaptability to different backends is achieved by downstream components (orchestration, visualization), not by INTENT-7 itself.

INTENT-7 does NOT:
- Promise semantic alignment (that is the job of visualization consensus layers like Cellrix)
- Prove trust (that is a multi-dimensional collaboration across the ecosystem)
- Map intents to tool capabilities (that is the job of the orchestration layer like Anaphase)

## Protocol Stack
```
INTENT-7 ← You are here
  ↑
BIND-19 (transport binding — format negotiation & integrity)
  ↑
INTENT-7-SECURE (optional mTLS reference implementation)
  ↑
CAPABILITY-13 (consensus confirmation — what you see is what you sign)
```

## Read the Spec
- [INTENT-7 v1.0.0-RFC-4](spec/INTENT-7.md)
- [中文版](spec/INTENT-7.zh-CN.md)

## Related
| Protocol | Repository |
|:---|:---|
| CAPABILITY-13 | [CommonIntents/CAPABILITY-13](https://github.com/CommonIntents/CAPABILITY-13) |
| BIND-19 | [CommonIntents/BIND-19](https://github.com/CommonIntents/BIND-19) |
| INTENT-7-SECURE | [CommonIntents/INTENT-7-SECURE](https://github.com/CommonIntents/INTENT-7-SECURE) |

## License
Apache 2.0 — see [LICENSE](LICENSE).
