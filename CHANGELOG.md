# Changelog

## 0.1.0（对外发布基线）

首个对外发布版本：AnGIneer 的问答编排内核（Agent Harness 引擎）从主仓库独立成包。
纯 Python 库，不含 HTTP 服务、不含知识库存储与检索配方——检索由消费方注入（端口注册或 `ANGINEER_DOCS_API_URL`），
LLM 调用由 `angineer-ai-inference` 承担。本版本为对外消费方的稳定基线，能力如下：

### 编排链路
- 意图分级 `IntentClassifier`：L0 闲聊 / L1 正文问答 / L2 条款表格定位 / L3 规范计算 / L4 复杂综合；
  LLM 判分失败时按关键词规则兜底，不把请求打成 500。
- 策略展开 `build_attempts`：按级别生成 Attempt 序列，前一级失败自动降级到下一级，全部失败才拒答收尾。
- 循环执行器 `run_agent_loop`：steer 中途注水、预算门（oldest-first 压缩 + 超限停止）、协作式取消、
  终答守卫统一收口；事件流 `run_start/turn_*/message_*/tool_*/note/answer/error` 供前端还原思考过程。
- 工具文本协议 `TextToolCallCodec`：模型用 ```tool_calls 围栏块发起调用，兼容一切 OpenAI 兼容端点
  （不依赖原生 function calling）；工具结果超预算时压缩成摘要并留 doc 指针，模型可凭指针回看原文
  （`knowledge_search` 支持 `doc_ids` 定向回看，激进压缩开关 `ANGINEER_EAGER_COMPRESS` 默认关）。

### 工具箱
- `knowledge_search` / `table_search` / `entity_search` / `knowledge_stats` 四件套（L1/L2 统一档，模型自选）；
- `calculator` 进 QA 档（2026-10-07）：数值题不再心算，工具清单含计算器时协议侧强制走工具；
- L3/L4 复杂档追加 `sop_execute`（SOP 步骤机 + 变量黑板）与 `conditional`（条件分支）；
- 多知识库作用域：`library_id` + `library_ids` 集合 + `doc_ids` 收窄，检索 memo 按库集合分桶不串；
- 检索 memo（赌博式预检）：请求进来即并行预跑一次 L1 检索，命中即把检索段移出关键路径（`ANGINEER_ROUTE_PARALLEL`）。

### 检索接入（端口注入 / HTTP 二选一）
- 端口注册表 `angineer_core.ports`：`register_local_nodes_loader` / `register_local_rerank` /
  `register_agent_search`（normalize_query / knowledge_local / table_local / entity_local /
  local_stats / engtool_registry / relevant_citations / table_blocks 八件套）；
  未注册的端口按降级语义（警告 + 空结果）处理，引擎不 import 任何具体检索实现包。
- HTTP 客户端 `docs_retrieval_client`：配 `ANGINEER_DOCS_API_URL` 即走 docs-api 契约
  （retrieve / entity-search / doc-nodes / graph-append-note 四端点），失败可回落进程内实现；
  `ANGINEER_DISABLE_LOCAL_FALLBACK=1` 时禁止回退，彻底消灭跨进程直读 SQLite。

### 证据守卫与提示词
- 终答守卫 `make_final_answer_guard`：无证据拒答；引用未检索到的规范编号或书名号标题替换为拒答话术
  （书名号为「全部核不到才拦」，部分核到的次级引用放行）；编造 cite 标记移除；有证据却整段拒答时定向重试一次。
- 上下文截断 `ANGINEER_CONTEXT_TOP_N`（默认 15）、证据上桌/软帽留痕（admission）、
  LLM 投影 V2（items[] 遥测外壳不进 prompt）。
- 提示词资产化（P5 注册表 + 版本号）：QA v17、COMPLEX v5、followup v3；
  答案形态注入口子 `answer_format`（评测侧穿线，默认 None 逐字节不变）。
- 流式首字存活线 `ANGINEER_FIRST_TOKEN_LIVENESS_S`：上游连接活着但不吐字时按超线收口，
  避免挂起型卡死；取消（停止按钮）在等待期同样可打断。

### 观测
- `ops_metrics` 事件埋点（含 `inspect_evidence` 回看观测）、`trace_collector`、
  `run_manifest` 运行清单、`stage_timings` 分段计时、`llm_errors` 故障降级留痕。

### 已知边界
- 依赖 docs-core 的集成用例（检索端口真实适配器）在未安装 docs-core 的环境自动跳过；
- 意图分类与 LLM 质量强依赖消费方自备的 `LLM_CONFIGS` 与端点，换模型后回答质量须重新评测。
