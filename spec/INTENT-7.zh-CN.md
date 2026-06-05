# INTENT-7：通用意图规范

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![Version](https://img.shields.io/badge/Version-0.6.0--draft-orange.svg)]() [![Status](https://img.shields.io/badge/Status-RFC%20Draft-yellow.svg)]() [![Org](https://img.shields.io/badge/Org-CommonIntents--144-darkgray.svg)](https://github.com/CommonIntents)

**版本**：0.6.0-draft
**状态**：征求意见（RFC）
**编者**：Jasonmilk 与 Cellrix 开源社区
**许可证**：Apache 2.0

---

## 1. 摘要

通用意图规范（CommonIntents-144 Specification，INTENT-7）是**面向AI的结构化意图描述语言（SIDL）**——一个定义意图JSON Schema的语法标准。

它只定义“AI想要执行的业务动作”的语义标准，通过本地静态映射层，可无损翻译为CLI命令、GUI交互、TUI终端组件操作、CSS视觉反馈。上层AI永远使用统一接口，下层任意交互载体自由切换。

INTENT-7 是与 UI 无关的声明式协议，将“能做什么”（意图）与“如何触发”（绑定与映射）彻底解耦，为人类用户与 AI 代理双方创建一个标准化的语义层。




## 2. 哲学基础

INTENT-7 是结构化意图描述语言（SIDL）。它定义意图的 JSON Schema 结构——意图类别、动作、目标、参数。它是句法标准，不是语义承诺。

语义对齐由下游可视化共识层（如 Cellrix）和编排层（如 Anaphase）协作完成。

INTENT-7 的目标：
- **稳定**：Schema 变更需明确版本号
- **可验证**：任何标准 JSON Schema 验证器均可校验句法有效性
- **透明**：没有隐藏语义，没有隐含假设

## 3. 约定与术语

本文档中的关键词“必须”（MUST）、“禁止”（MUST NOT）、“要求”（REQUIRED）、“应当”（SHALL）、“不应”（SHALL NOT）、“建议”（SHOULD）、“不建议”（SHOULD NOT）、“推荐”（RECOMMENDED）、“可选”（MAY）和“可选的”（OPTIONAL），均按 RFC 2119 的描述进行解释。

- **界面（Interface）**：暴露意图的交互表面（终端、Web、移动端、VR）。
- **意图（Intent）**：一个离散的、可调用的操作，由 id、描述、可选参数及安全约束定义。
- **代理（Agent）**：调用意图的任何客户端——通过 UI 的人类，或通过 API 的 AI。
- **绑定（Binding）**：一份声明，表明某个意图锚定在呈现层中的特定元素上。
- **载体（Backend）**：执行环境的具体类型——CLI、GUI、TUI、CSS、Web、移动端。
- **人在回路（HITL）**：一种安全机制，要求在执行高风险意图之前获得人类的显式确认。


## 4. 设计原则

1. **语义纯粹，传输无关，载体无关**：INTENT-7 是永恒的意图语言，不涉及传输、加密、身份或执行载体
2. **核心极简，可选扩展**：基础词汇保持尽可能小的集合，自定义数据通过 x- 前缀字段添加
3. **声明式激活，按需而动**：高级特性显式声明才激活，不声明即零开销
4. **最大包容，最小排他**：只定义“什么是对的”，不规定“怎么做”
5. **零依赖就绪，渐进式增强**：仅需 HTTP 服务和 JSON 解析即可基本兼容
6. **正交性**：意图独立于呈现而存在。一个界面可以暴露 50 个意图，但仅将其中 5 个渲染为可见元素；被授权的代理可以访问全部 50 个


## 5. 协议栈中的位置

INTENT-7 是 INTENT-7/CAPABILITY-13 协议族的**灵魂层**。它在四层协议栈中的位置：

```
┌─────────────────────────────────────────┐
│              INTENT-7  ← 本协议               │
│  通用意图与控制协议                      │
│  · 纯粹的意图语义标准                    │
│  · 传输无关，加密无关，载体无关          │
└─────────────────────────────────────────┘
                    ▲ 语义绑定
┌─────────────────────────────────────────┐
│              BIND-19                         │
│  INTENT-7/传输绑定协议                        │
└─────────────────────────────────────────┘
                    ▲ 当前绑定到
┌─────────────────────────────────────────┐
│              INTENT-7-SECURE                        │
│  安全意图与控制协议                      │
└─────────────────────────────────────────┘
                    ▲ 承载于
┌─────────────────────────────────────────┐
│              CAPABILITY-13                         │
│  能力认证协议                            │
└─────────────────────────────────────────┘
```

- **INTENT-7 定义** “AI 想做什么”——意图语义标准
- **CAPABILITY-13 定义** “AI 能做什么，以什么条件，在什么时限内”——通过 Manifest 声明应用支持哪些 INTENT-7 意图
- **BIND-19 定义** “意图如何传输”——格式协商与完整性保障
- **INTENT-7-SECURE 定义** “谁在说话，信道是否安全”——mTLS 身份验证


## 6. 意图注册表

一份 INTENT-7 文档是表示一个意图注册表（Intent Registry）的 JSON 对象。它必须（MUST）使用 UTF-8 编码。

### 6.1. 根 Schema

```json
{
  "$schema": "https://cin7.cellrix.org/schema/v0.6/cin7.json",
  "cis_version": "0.6",
  "interface_id": "string（可选）",
  "intents": [...]
}
```

- `$schema` 可选（MAY）用于对注册表本身进行自动化校验。
- `cis_version` 声明注册表所遵循的规范版本。
- `interface_id` 是来源界面的可选标识符。

### 6.2. 意图对象

| 字段 | 类型 | 必需 | 描述 |
|---|---|---|---|
| id | 字符串 | 是 | 全局唯一标识符（推荐 snake_case） |
| name | 字符串 | 是 | 短人类可读名称 |
| description | 字符串 | 是 | 适合 LLM 消费的详细说明，必须描述该操作的副作用 |
| parameters | 对象 | 否 | 一份 JSON Schema（Draft 2020‑12），定义预期载荷 |
| security | 对象 | 否 | 执行约束（参见 §7） |
| bindings | 数组 | 否 | 呈现层中的元素锚点（参见 §8.1） |

### 6.3. 核心意图集

| 意图 | 描述 | 适用载体 |
|:---|:---|:---|
| `Search` | 搜索指定内容 | 全部 |
| `Click` | 点击指定组件 | GUI, TUI, Web |
| `Input` | 向指定区域输入文本 | GUI, TUI, Web |
| `ListDir` | 列出目录内容 | CLI, GUI |
| `Navigate` | 导航到指定位置 | 全部 |
| `Read` | 读取指定内容 | 全部 |
| `Confirm` | 确认操作 | GUI, TUI, Web |
| `Cancel` | 取消操作 | GUI, TUI, Web |
| `Select` | 从列表中选择 | GUI, TUI, Web |
| `Execute` | 执行指定命令 | CLI |


## 7. 安全约束

`security` 对象定义某个意图的信任要求。如果省略，该意图被视为 `risk_level: low`，且不需要人类确认。

| 字段 | 类型 | 描述 |
|---|---|---|
| risk_level | 字符串 | 取值：`low`、`medium`、`high`、`critical` |
| requires_hitl | 布尔值 | 若为 true，运行时必须暂停执行，直到人类显式批准 |
| required_scopes | 字符串数组 | 调用者必须拥有的权限范围列表 |

**HITL 流程**：当调用一个 `requires_hitl: true` 的意图时，实现必须：
- 立即暂停该操作
- 向人类用户呈现确认提示
- 等待显式批准或拒绝（或超时）
- 仅在批准后恢复执行

**注意**：HITL 的异步队列、决策状态机、审批签名标准化等运行时实现细节，参见 **CAPABILITY-13 协议**（能力认证协议）。INTENT-7 只定义意图级别的安全约束声明，不定义 HITL 的运行时实现机制。



## 9. 参数 Schema（JSON Schema 集成）

INTENT-7 采用 JSON Schema Draft 2020‑12 进行参数定义。`parameters` 字段必须（MUST）是一个合法的 JSON Schema 对象。实现必须（MUST）在执行前对任何传入载荷按此 Schema 进行校验。

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

**空值规则**：如果意图未定义 `parameters` 字段，调用请求中不得（MUST NOT）包含 `parameters` 键。当意图未声明参数时，传入 `{}` 或 `null` 应当（SHALL）被拒绝，视为无效请求。


## 10. 执行生命周期

所有合规运行时必须（MUST）实现以下生命周期：

- **发现（Discovery）**——代理读取意图注册表。不产生任何副作用。
- **载荷构建（Payload Construction）**——代理构建与意图的 parameters Schema 匹配的 JSON 载荷。
- **调度（Dispatch）**——代理发送包含 intent_id 以及（如适用）parameters 的调用请求。
- **Schema 校验（Schema Validation）**——运行时按 JSON Schema 校验载荷。无效载荷被立即拒绝（快速失败）。
- **安全拦截（Security Interception）**——若 requires_hitl 为 true，执行被暂停，状态变为 PENDING_APPROVAL，并触发人类确认流程。
- **执行与响应（Execution & Response）**——经过校验的载荷被传递给业务逻辑。返回确定性响应。


## 11. 调用请求与响应

### 11.1. 通用调用请求

```json
{
  "intent_id": "reboot_server",
  "parameters": {
    "instance_id": "i-abc123",
    "force": false
  }
}
```

如果意图未声明 parameters，则不得包含 parameters 键：

```json
{ "intent_id": "logout" }
```

### 11.2. 通用调用响应

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

- status：必须（MUST）为 `"completed"`（任务同步完成）或 `"accepted"`（任务仍在进行中）。
- job_id：当 status 为 `"accepted"` 时为必需（REQUIRED）。

**如果要求 HITL 且未被预先批准**：

```json
{
  "success": false,
  "intent_id": "reboot_server",
  "error": "confirmation_required",
  "message": "此操作需要人类确认。"
}
```


## 12. 可扩展性

自定义字段可选（MAY）添加到任何 INTENT-7 对象。为避免冲突，所有扩展字段必须（MUST）以 `x-` 作为前缀（例如 `x-telemetry-id`）。实现必须（MUST）静默忽略未知的 x- 字段——这保证了前向兼容性。


## 13. 与呈现协议及 CAPABILITY-13 的关系

INTENT-7 完全独立于任何呈现技术。INTENT-7 注册表可由以下形式暴露：
- 终端应用（例如基于 Cellrix 的 TUI）
- Web 应用（通过 REST API）
- 移动应用
- 语音助手

呈现层负责渲染 UI 元素并将用户输入映射到已绑定的意图，这超出 INTENT-7 的范围。

INTENT-7 与 CAPABILITY-13 的关系：INTENT-7 定义“AI 想做什么”，CAPABILITY-13 定义“AI 能做什么，以什么条件，在什么时限内”。CAPABILITY-13 的 Manifest 声明应用支持哪些 INTENT-7 意图及每个意图的安全约束。INTENT-7 意图是 Manifest 中 actions 数组的 name 字段的语义来源。

**INTENT-7 与 AI 代码生成**：AI 拥有两个独立的能力系统——INTENT-7 + 静态映射（执行操作）和预训练知识（生成代码）。两者不套娃，服务于不同场景，走不同路径。


## 14. 合规性

一个实现若满足以下条件，即符合 INTENT-7 v0.6.0：
- 能够解析并校验意图注册表（§6）
- 按提供的 JSON Schema 校验调用参数（§9）
- 对要求 HITL 的意图执行 HITL（§7）
- 按 §11.2 所述的格式返回响应
- 当意图未声明参数时，拒绝包含 parameters 的调用请求（§9）
- 静默忽略未知的 x- 字段（§12）

参考实现提供了一套合规性测试套件。


## 15. 参考实现

INTENT-7 的参考实现作为 Cellrix 项目的一部分维护。它演示了：
- 用于解析和校验 INTENT-7 注册表的 Python 库
- 用于意图调用的 HTTP 端点
- 完整的 HITL 操作拦截器
- 一套合规性测试套件

INTENT-7 是一份独立的规范。任何实现本文档所述契约的 UI 框架，均为合法的 INTENT-7 运行时。

**相关协议与实现**：

| 协议 | 仓库 | 职责 |
|:---|:---|:---|
| INTENT-7 | CommonIntents-144/INTENT-7 | 通用意图语义标准（本协议） |
| CAPABILITY-13 | CommonIntents-144/CAPABILITY-13 | 能力认证与 HITL 决策 |
| BIND-19 | CommonIntents-144/BIND-19 | 传输绑定、格式协商 |
| INTENT-7-SECURE | CommonIntents-144/INTENT-7-SECURE | mTLS 安全传输 |

**平台无关性**：INTENT-7 协议规范本身通过内容寻址标识符（CID）发布。本白皮书的权威版本以 CID 作为唯一判定依据，不受具体网址链接限制。

| 参考实现 | 仓库 | 角色 |
|:---|:---|:---|
| Cellrix | Jasonmilk/Cellrix | INTENT-7/CAPABILITY-13 的参考实现与试验场 |


## 16. 版本管理

本规范遵循语义化版本。意图结构、调用契约或安全模型的破坏性变更将导致主版本号递增。新的可选字段、绑定类型或澄清说明将导致次版本号递增。


## 版本状态

本规范尚处于初期草案阶段（v0.6.0），现阶段核心工作是依托实际落地参考实现，完成协议的验证与优化完善。目前暂不启动社区正式治理工作，待规范迭代完善、趋近 v1.0 正式版后，将启动标准 RFC 正式制定流程。

---

*本文档依据 Apache 2.0 许可证发布。欢迎通过 CommonIntents-144 组织仓库贡献。*
