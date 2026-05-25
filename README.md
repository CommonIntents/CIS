# CIS — Common Intent Specification [![Org](https://img.shields.io/badge/Org-CommonIntents-darkgray.svg)](https://github.com/CommonIntents)

**Structured Intent Description Language (SIDL).** CIS is a syntactic standard for AI-native intent expression. It defines a strict JSON Schema for intent messages — what fields they contain, how they are validated, and how they are routed.

CIS is a minimalist syntactic layer. It defines structure, not behavior. Adaptability to different backends is achieved by downstream components (orchestration, visualization), not by CIS itself.

CIS does NOT:
- Promise semantic alignment (that is the job of visualization consensus layers like Cellrix)
- Prove trust (that is a multi-dimensional collaboration across the ecosystem)
- Map intents to tool capabilities (that is the job of the orchestration layer like Anaphase)

## Protocol Stack
```
CIS ← You are here
  ↑
CIB (transport binding — format negotiation & integrity)
  ↑
CISS (optional mTLS reference implementation)
  ↑
CAP (consensus confirmation — what you see is what you sign)
```

## Read the Spec
- [CIS v0.7.0-draft](spec/CIS.md)
- [中文版](spec/CIS.zh-CN.md)

## Related
| Protocol | Repository |
|:---|:---|
| CAP | [CommonIntents/CAP](https://github.com/CommonIntents/CAP) |
| CIB | [CommonIntents/CIB](https://github.com/CommonIntents/CIB) |
| CISS | [CommonIntents/CISS](https://github.com/CommonIntents/CISS) |

## License
Apache 2.0 — see [LICENSE](LICENSE).
