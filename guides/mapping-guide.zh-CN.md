# 映射表编写指南 (Mapping Guide)

**文件路径**：`CommonIntents/CIS/guides/mapping-guide.md`  
**版本**：v0.1.0  
**状态**：草案
**维护者**：CommonIntents 协议族

---

### 1. 定位与边界

本指南是 **CIS（通用意图标准）的实现参考**，而非独立协议。它提供静态映射表的编写规范，帮助开发者将 CIS 意图系统地转换为具体的软件动作。

| 类型 | 职责 |
|:---|:---|
| **CIS 协议** | 定义意图的交互语义（意图类型、字段、必填/可选） |
| **本指南** | 定义如何将意图映射为可执行的动作（实现模式） |

**核心原则**：协议只定义交互语义标准，本指南只提供推荐的实现方式。任何遵循 CIS 语义的实现，即使不使用本指南的映射表格式，也完全兼容 CIS。

---

### 2. 设计原则

- **单一规范源**：映射表以 JSON 文件为唯一规范格式，人类可读、可直接编辑、可通过 Git 追踪变更。
- **严格验证**：映射表必须通过 JSON Schema 验证，确保字段完整性和类型正确。
- **无歧义匹配**：条目匹配按顺序进行，第一个满足所有条件的条目立即生效，不进行多条目合并。
- **扩展开放**：允许通过 `custom` 字段添加特定实现的扩展参数，不影响核心匹配逻辑。
- **版本语义化**：映射表自身带版本号，与 CIS 协议版本独立，遵循语义化版本规则。

---

### 3. 映射表结构

#### 3.1 顶层结构

```json
{
  "version": "0.1.0",
  "updated": "2026-05-22T00:00:00Z",
  "entries": []
}
```

| 字段 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `version` | string | 是 | 本映射表遵循的指南版本（语义化版本） |
| `updated` | string | 是 | 最后更新日期 (ISO 8601) |
| `entries` | array | 是 | 映射条目列表，按优先级降序排列 |

#### 3.2 映射条目

```json
{
  "id": "map.snapshot.view",
  "description": "处理 agent.snapshot 意图",
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

**match 字段**（所有条件必须同时满足，AND 逻辑）：

| 字段 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `intent` | string | 是 | CIS 意图类型，如 `agent.snapshot` |
| `format` | string | 否 | CIB 格式类型 (`json`/`cbor`/`binary`)，省略表示接受任意格式 |
| `ciss_required` | string | 否 | 安全要求：`required` / `optional` / `ignore`，默认 `ignore` |
| `cap_required` | array of string | 否 | CAP 能力列表，如 `["read"]`，空数组表示无需能力 |
| `custom` | object | 否 | 实现扩展字段，未识别的 key 应被忽略 |

**action 字段**：

| 字段 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `type` | string | 是 | 动作类型（见 §4） |
| `target` | string | 是 | 动作目标：组件名、函数名、资源地址 |
| `params` | object | 否 | 动作参数，结构由动作类型决定 |

#### 3.3 匹配优先级

条目在 `entries` 数组中的顺序定义优先级。**第一个** `match` 全部满足的条目立即执行，后续条目被忽略。因此：
- 更具体的条目（如 `agent.snapshot`）应放在前面。
- 通配条目或默认回退条目应放在最后。

---

### 4. 动作类型 (软件场景)

本指南目前仅覆盖软件界面场景。硬件动作类型留作未来扩展。

| 动作类型 | 说明 | 示例 target | 示例 params |
|:---|:---|:---|:---|
| `render.static` | 渲染静态组件 | `text_panel`, `list_view` | `{"template": "snapshot.yaml"}` |
| `render.dynamic` | 渲染动态组件（从意图载荷提取数据） | `chart_view` | `{"data_path": "payload.values"}` |
| `ui.action` | 触发 UI 状态变更（如导航、弹窗） | `navigate.to_home` | `{"transition": "fade"}` |
| `call.service` | 调用后端服务接口 | `fetch_user_list` | `{"method": "GET", "path": "/users"}` |
| `custom` | 自定义动作 | 由实现定义 | 由实现定义 |

实现者可按需扩展动作类型，但不保证跨实现的互操作性。

---

### 5. 扩展机制

`match.custom` 和 `action.params` 是开放扩展点。实现者可添加自己的键值对，用于特定场景（如性能权重、调试标记）。但扩展字段：

- 不应影响核心匹配逻辑。
- 解析器应忽略不认识的扩展字段。
- 扩展语义由实现者自行负责，不在本指南范围内。

---

### 6. 版本演进

映射表使用语义化版本（SemVer 2.0.0）：
- **MAJOR**：不兼容的条目结构变更（如删除字段、修改字段类型）。
- **MINOR**：向后兼容的新增（如新增动作类型、新增可选匹配字段）。
- **PATCH**：文档修正、条目描述更新。

当 CIS 协议升级导致意图类型或字段变化时，映射表也应相应更新版本并重新验证。

---

### 7. 验证

映射表编写完成后，应使用配套的 `mapping-schema.json` 进行结构验证。示例命令（使用 `jsonschema` 工具）：

```bash
jsonschema -i map.json cis/spec/schema/mapping-schema.json
```

只有通过验证的映射表才是有效的。建议在 CI 流程中加入验证步骤。

---

### 8. 与协议族的关系

| 协议 | 本指南如何引用 |
|:---|:---|
| **CIS** | `match.intent` 使用 CIS 定义的意图类型 |
| **CIB** | `match.format` 使用 CIB 定义的格式类型 |
| **CISS** | `match.ciss_required` 定义该意图的安全级别要求 |
| **CAP** | `match.cap_required` 定义执行该意图所需的能力 |

本指南只提供映射表的**结构**定义，不修改任何协议的语义。它是实现 CIS 的路标，而非 CIS 的一部分。

---

### 9. 示例：最小可工作映射表

```json
{
  "version": "0.1.0",
  "updated": "2026-05-22T00:00:00Z",
  "entries": [
    {
      "id": "map.snapshot",
      "description": "处理 agent.snapshot 意图，渲染静态快照视图",
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
      "description": "所有未匹配意图的默认处理",
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

*本指南随 CIS 协议演进。欢迎提交 issue 讨论扩展与改进。*

---

请审核这份指南，通过后我立即提供 `mapping-schema.json`。
