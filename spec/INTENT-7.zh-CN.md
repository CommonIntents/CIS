# INTENT-7：结构化意图描述语言（SIDL）规范（v1.0.0-RFC-4）

## 1. 引言与设计目标
本规范定义了 **INTENT-7**，即 **CommonIntents-144（CI-144）** 协议族中的结构化意图描述语言（SIDL）。

INTENT-7 严格定位为**纯应用/语义层载荷协议**，与底层传输成帧、多路复用、滑动窗口流控、链路加密保护完全解耦且透明（上述能力全部交由 **BIND-19** 与 **INTENT-7-SECURE** 负责）。

INTENT-7 的核心目标：
- 为AI原生意图声明提供高确定性的结构化 schema
- 原生集成 W3C 追踪上下文标准，实现认知-物理因果全链路可追溯
- 在执行运行时的螺旋执行记录（HXR）与长期记忆 L3 情景节点之间建立零拷贝数据契约

---

## 2. W3C 追踪上下文与元数据结构
为保证每一次认知决策与后续物理动作，都能跨进程、跨网络追溯到根因，所有 INTENT-7 载荷必须包含标准化元数据头部。

### 2.1 元数据结构
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

#### 字段说明
- **`protocol_version`**：字符串，INTENT-7 规范的语义化版本号（通过 BIND-19 握手协商确定）
- **`traceparent`（W3C 标准）**：128 位全局唯一追踪 ID + 64 位跨度 ID，作为认知审计的统一「神经信号纤维」
- **`autonomy_level`**：枚举值，声明当前交互的权限边界：
  - `AGENT`：代理模式，只读型瞬态交互
  - `OPEN`：开放模式，读写协作模式
  - `SURVIVAL`：生存模式，全自主生存循环（最低能耗运行）
- **`impasse_depth`**：整数（0-5），记录潜意识五级阻滞模型中的当前升级状态

---

## 3. 核心意图动词（ACTION）与结构定义
INTENT-7 将所有语义执行诉求限定为一组精简、无重叠的动词集合。每个动词入队前都必须经过 JSON Schema 严格校验。

### 3.1 FETCH（读取语义记忆）
向长期知识库请求特定概念或规则，支持关键的**摘要优先两级性能优化**。

```json
{
  "action": "FETCH",
  "params": {
    "node_id": "algo_quick_sort",
    "mode": "summary"
  }
}
```

*   `mode == "summary"`：仅返回轻量的 id、标题、摘要字段，节省上下文窗口空间
*   `mode == "full"`：仅当大模型分析摘要后明确请求时，才返回完整正文 `full_content`

### 3.2 WRITE_NODE（提交记忆/知识）
将新沉淀的结论或观测结果追加写入记忆有向无环图。

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

### 3.3 TENTACLE（调用无状态执行工具）
推荐调用沙箱化的外部命令行工具，必须声明申请的 CAPABILITY-13 逻辑权限，供沙箱强制执行。

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
让出执行权，关闭当前认知循环。
*   `FINISH`：成功达成根目标，返回最终事务报告
*   `CANCEL`：因深度阻滞或安全违规中止当前执行树，附带详细诊断日志

---

## 4. 螺旋执行记录（HXR）与 L3 节点对齐
为实现**极致复用**，避免高成本、非确定性的运行时格式转换，记录状态机完整执行轨迹的**螺旋执行记录（HXR）**，其结构设计与标准 L3 情景节点中的 `content` JSON 载荷**完全一致**。

### 4.1 HXR 载荷结构规范
活跃会话写入 L3 记忆有向无环图时，HXR 载荷直接嵌入 `content` 区块：

```json
{
  "session_id": "sess_20260411_001",
  "step_id": "step_003",
  "ts": "2026-04-11T14:23:00.123Z",
  "action": "FETCH",
  "target_graph": "knowledge_base",
  "node_id": "algo_quick_sort",
  "parent_ids": ["fact_001", "fact_002"],
  "intent": "获取快速排序公理定义",
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

## 5. 与 BIND-19 的传输绑定
所有 INTENT-7 载荷必须作为 BIND-19 数据帧（帧类型 0x01）的载荷进行封装传输：

- **帧类型**：必须为 `0x01`（数据帧），标识为标准语义载荷
- **通道 ID**：由 Anaphase 调度器显式分配；`0x00` 通道保留用于控制信令与关键查询
- **标志位**：
  - 超过 64MB 的分片载荷必须设置 `FIN`（`0x01`）标志，由帧层完成数据包重组
  - 若 INTENT-7 动作需要物理授权，必须设置 `CON`（`0x02`）标志，触发异步 CAPABILITY-13 挂起流程
- **序列号**：按通道 ID 单调递增，遵循 BIND-19 规范定义

---

## 6. 错误处理
意图层的 schema 校验或载荷执行失败，通过 BIND-19 错误帧（`0x06`）传递，载荷中包含以下语义错误码之一：

| 错误码十进制 | 十六进制值 | 错误名称 | 说明 |
|:---|:---|:---|:---|
| `1600` | `0x0640` | `INVALID_PAYLOAD_SCHEMA` | JSON 载荷未通过对应动词定义的 schema 校验 |
| `1610` | `0x064A` | `UNKNOWN_INTENT_VERB` | 执行器无法识别指定的 action 动作动词 |
| `1620` | `0x0654` | `TRACE_CONTEXT_CORRUPTED` | W3C traceparent 缺失、格式错误或哈希链路无效 |
