# CIS — Common Intent Specification

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![Version](https://img.shields.io/badge/Version-0.6.0--draft-orange.svg)]() [![Status](https://img.shields.io/badge/Status-RFC%20Draft-yellow.svg)]() [![Org](https://img.shields.io/badge/Org-CommonIntents-darkgray.svg)](https://github.com/CommonIntents)

**The native language of AI.**

CIS is a device-independent intent protocol. It defines *what* AI wants to do — purely semantic, transport-agnostic, backend-agnostic. Through a local static mapping layer, a single CIS intent losslessly translates into CLI commands, GUI interactions, TUI operations, or CSS feedback.

CIS is the **soul** of the CIS/CAP protocol family.

```
AI says:  Search
          ↓   static mapping (zero Tokens)
CLI:      app search --query=...
GUI:      locate box → focus → type → click
TUI:      switch panel → focus → input → enter
```

## Protocol Stack

```
CIS  ← You are here
 ↑
CIB  (transport binding)
 ↑
CISS (mTLS security)
 ↑
CAP  (capability auth & HITL)
```

## Read the Spec

- [CIS v0.6.0-draft](spec/CIS.md)
- [中文版](spec/CIS.zh-CN.md)

## Related

| Protocol | Repository |
|----------|------------|
| CAP | [CommonIntents/CAP](https://github.com/CommonIntents/CAP) |
| CIB | [CommonIntents/CIB](https://github.com/CommonIntents/CIB) |
| CISS | [CommonIntents/CISS](https://github.com/CommonIntents/CISS) |

## License

Apache 2.0 — see [LICENSE](LICENSE).
