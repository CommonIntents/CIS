# CIS：通用意图规范

**版本：** 0.6.0-草案  
**状态：** 征求意见（RFC）  
**编者：** Jasonmilk 与 Cellrix 开源社区  
**许可证：** MIT

## 1. 摘要

通用意图规范（Common Intents Specification，CIS）是一套与 UI 无关的声明式协议，用于描述任意软件系统的交互能力。它将 **“能做什么”** （意图）与 **“如何触发”** （绑定）彻底解耦，为人类用户与 AI 代理双方创建一个标准化的语义层。

CIS 专为后 AGI 时代而设计。它使大语言模型、自主代理以及辅助技术能够以确定性的安全方式发现、理解并调用操作——无需 OCR、无需对 UI 元素做启发式解析、无需脆弱的假设。

## 2. 约定与术语

本文档中的关键词“必须”（MUST）、“禁止”（MUST NOT）、“要求”（REQUIRED）、“应当”（SHALL）、“不应”（SHALL NOT）、“建议”（SHOULD）、“不建议”（SHOULD NOT）、“推荐”（RECOMMENDED）、“可选”（MAY）和“可选的”（OPTIONAL），均按 [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) 的描述进行解释。

- **界面（Interface）**：暴露意图的交互表面（终端、Web、移动端、VR）。
- **意图（Intent）**：一个离散的、可调用的操作，由 `id`、描述、可选参数及安全约束定义。
- **代理（Agent）**：调用意图的任何客户端——通过 UI 的人类，或通过 API 的 AI。
- **绑定（Binding）**：一份声明，表明某个意图锚定在呈现层中的特定元素上。
- **人在回路（HITL）**：一种安全机制，要求在执行高风险意图之前获得人类的显式确认。

## 3. 设计目标

1. **正交性**：意图独立于呈现而存在。一个界面可以暴露 50 个意图，但仅将其中 5 个渲染为可见元素；被授权的代理可以访问全部 50 个。
2. **严格契约**：每个意图携带一个机器可验证的参数模式（JSON Schema Draft 2020‑12）。不存在模糊的自然语言契约。
3. **代理优先**：描述面向 LLM 的消费而编写——必须包含预期的副作用。
4. **最小核心，可扩展**：基础词汇保持尽可能小的集合。自定义数据可通过 `x-` 前缀字段添加。
5. **幂等发现**：读取意图注册表不得产生任何副作用。

## 4. 意图注册表

一份 CIS 文档是表示一个**意图注册表（Intent Registry）** 的 JSON 对象。它必须（MUST）使用 UTF-8 编码。

### 4.1. 根 Schema

```json
{
  "$schema": "https://cis.cellrix.org/schema/v0.6/cis.json",
  "cis_version": "0.6",
  "interface_id": "string (可选)",
  "intents": [ ... ]
}
```

- `$schema` 可选（MAY）用于对注册表本身进行自动化校验。
- `cis_version` 声明注册表所遵循的规范版本。
- `interface_id` 是来源界面的可选标识符。

### 4.2. 意图对象

| 字段 | 类型 | 必需 | 描述 |
|:---|:---|:---|:---|
| `id` | 字符串 | 是 | 全局唯一标识符（推荐（RECOMMENDED） snake_case）。 |
| `name` | 字符串 | 是 | 短人类可读名称。 |
| `description` | 字符串 | 是 | 适合 LLM 消费的详细说明。必须（MUST）描述该操作的副作用。 |
| `parameters` | 对象 | 否 | 一份 JSON Schema（Draft 2020‑12），定义预期载荷。如果意图不接受任何参数，则省略此字段。 |
| `security` | 对象 | 否 | 执行约束（参见 §4.3）。 |
| `bindings` | 数组 | 否 | 呈现层中的元素锚点（参见 §4.4）。 |

### 4.3. 安全约束

`security` 对象定义某个意图的信任要求。如果省略，该意图被视为 `risk_level: low`，且不需要人类确认。

| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `risk_level` | 字符串 | 取值：`low`、`medium`、`high`、`critical`。 |
| `requires_hitl` | 布尔值 | 若为 `true`，运行时必须（MUST）暂停执行，直到人类显式批准。 |
| `required_scopes` | 字符串数组 | 调用者必须拥有的权限范围列表（例如 `admin:write`）。 |

**HITL 流程**：当调用一个 `requires_hitl: true` 的意图时，实现必须（MUST）：

1. 立即暂停该操作。
2. 向人类用户呈现确认提示，包括意图的 `name` 和 `description`。
3. 等待显式批准或拒绝（或超时）。
4. 仅在批准后恢复执行；否则返回适当的错误。

当代理收到 `confirmation_required` 错误时，它建议（SHOULD）轮询执行状态（如果传输层支持），或等待来自实现的异步事件以通知批准或拒绝。

### 4.4. 绑定

绑定声明某个意图锚定在呈现层中的特定元素上。单个意图可（MAY）拥有多个绑定。

**绑定对象**：

| 字段 | 类型 | 必需 | 描述 |
|:---|:---|:---|:---|
| `type` | 字符串 | 是 | 必须（MUST）为 `"ui_element"`。 |
| `target_id` | 字符串 | 是 | 呈现层中元素的标识符。 |

呈现层全权负责一个绑定元素如何被触发——键盘、鼠标、触摸、语音或任何其他输入模态。CIS 不定义输入模态。

```json
"bindings": [
  { "type": "ui_element", "target_id": "restart_btn" }
]
```

### 4.5. 参数 Schema（JSON Schema 集成）

CIS 采用 **JSON Schema Draft 2020‑12** 进行参数定义。`parameters` 字段必须（MUST）是一个合法的 JSON Schema 对象。实现必须（MUST）在执行前对任何传入载荷按此 Schema 进行校验。

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

如果校验失败，实现必须（MUST）以描述性错误拒绝该调用。

**空值规则**：如果意图未定义 `parameters` 字段，调用请求中**不得（MUST NOT）** 包含 `parameters` 键。当意图未声明参数时，传入 `{}` 或 `null` 应当（SHALL）被拒绝，视为无效请求。

## 5. 执行生命周期

所有合规运行时必须（MUST）实现以下生命周期：

1. **发现（Discovery）** —— 代理读取意图注册表。不产生任何副作用。
2. **载荷构建（Payload Construction）** —— 代理构建与意图的 `parameters` Schema 匹配的 JSON 载荷。
3. **调度（Dispatch）** —— 代理发送包含 `intent_id` 以及（如适用）`parameters` 的调用请求。
4. **Schema 校验（Schema Validation）** —— 运行时按 JSON Schema 校验载荷。无效载荷被立即拒绝（快速失败）。
5. **安全拦截（Security Interception）** —— 若 `requires_hitl` 为 `true`，执行被暂停，状态变为 `PENDING_APPROVAL`，并触发人类确认流程。
6. **执行与响应（Execution & Response）** —— 经过校验的载荷被传递给业务逻辑。返回确定性响应。

## 6. 调用请求与响应

### 6.1. 通用调用请求

```json
{
  "intent_id": "reboot_server",
  "parameters": { "instance_id": "i-abc123", "force": false }
}
```

如果意图未声明 `parameters`，则不得包含 `parameters` 键：

```json
{
  "intent_id": "logout"
}
```

### 6.2. 通用调用响应

**同步完成**：
```json
{
  "success": true,
  "intent_id": "reboot_server",
  "status": "completed",
  "result": "重启已启动。"
}
```

**异步受理**（适用于长时间运行的任务）：
```json
{
  "success": true,
  "intent_id": "reboot_server",
  "status": "accepted",
  "job_id": "job_9876xyz",
  "result": "重启任务已提交，正在运行中。"
}
```

- `status`：必须（MUST）为 `"completed"`（任务同步完成）或 `"accepted"`（任务仍在进行中）。
- `job_id`：当 `status` 为 `"accepted"` 时为必需（REQUIRED）。代理可选（MAY）使用此标识符查询任务进度（实现自定义）。

如果要求 HITL 且未被预先批准：
```json
{
  "success": false,
  "intent_id": "reboot_server",
  "error": "confirmation_required",
  "message": "此操作需要人类确认。"
}
```

## 7. 可扩展性

自定义字段可选（MAY）添加到任何 CIS 对象。为避免冲突，所有扩展字段必须（MUST）以 `x-` 作为前缀（例如 `x-telemetry-id`）。实现必须（MUST）静默忽略未知的 `x-` 字段——这保证了前向兼容性。

## 8. 与呈现协议的关系

CIS 完全独立于任何呈现技术。CIS 注册表可由以下形式暴露：

- 终端应用（例如基于 Cellrix 的 TUI）
- Web 应用（通过 REST API）
- 移动应用
- 语音助手

呈现层负责渲染 UI 元素并将用户输入映射到已绑定的意图。这**超出 CIS 的范围**。一个呈现协议（如 Cellrix）可选（MAY）提供工具，从自身布局自动生成绑定，但这并非 CIS 合规性的要求。

## 9. 合规性

一个实现若满足以下条件，即符合 CIS v0.6.0：

1. 能够解析并校验意图注册表（§4）。
2. 按提供的 JSON Schema 校验调用参数（§4.5）。
3. 对要求 HITL 的意图执行 HITL（§4.3）。
4. 按 §6.2 所述的格式返回响应。
5. 当意图未声明参数时，拒绝包含 `parameters` 的调用请求（§4.5）。
6. 静默忽略未知的 `x-` 字段（§7）。

参考实现提供了一套合规性测试套件。

## 10. 参考实现

CIS 的参考实现作为 **[Cellrix 项目](https://github.com/Jasonmilk/Cellrix)** 的一部分维护。它演示了：

- 用于解析和校验 CIS 注册表的 Python 库。
- 用于意图调用的 HTTP 端点。
- 完整的 HITL 操作拦截器。
- 一套合规性测试套件。

CIS 是一份独立的规范。任何实现本文档所述契约的 UI 框架，均为合法的 CIS 运行时。

## 11. 版本管理

本规范遵循语义化版本。意图结构、调用契约或安全模型的破坏性变更将导致主版本号递增。新的可选字段、绑定类型或澄清说明将导致次版本号递增。

---

*本文档依据 MIT 许可证发布。欢迎通过 [Cellrix 仓库](https://github.com/Jasonmilk/Cellrix) 贡献。*
