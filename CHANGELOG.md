# Changelog

## 0.1.7

- feat(citations): 引用集真相源上移到服务端——`run_end` payload 新增 `citations`（本轮实际参考的证据集，跨轮跨工具按 `target_id + marker` 去重），供宿主直接渲染「参考依据」，**不再依赖模型在正文照抄 `[K1]` 标记**（模型遵从是概率性的：同题两次一次 4 个标记、一次 0 个，旧做法会让出处整块消失）；新增公开函数 `aggregate_tool_citations`，作为引用集唯一装配点，`policy_query` 改为复用；标记只保留「句级定位挂不挂」的职责
- docs: README 补 `run_end.payload.citations` 契约与宿主最小消费示例（含多库 `library_id` 回填、旧版本无该字段的兼容处理）

## 0.1.6

- fix: agent 循环幻觉工具名熔断——模型连续 `ANGINEER_TOOL_HALLUCINATION_TURNS`（默认 2，0=关）轮调用的工具全部未注册时，直接转无工具直答收尾（reason=tool_hallucination_breaker），不再烧光轮次预算一题未答；轮内混入真实工具即清零计数、换段复位，不误伤正常重试
- feat: QA prompt v20 规则 18 图表化交付——数据题表格后附 angineer-chart/angineer-map 块，数值必须来自证据；title/categories/series.name 写纯文本、禁 LaTeX 与 $ 记号（echarts 不走公式引擎）
- fix(citations): Kxx 引用标记容忍括号内空格（渲染、过滤、守卫三层同修）；引用解析表默认覆盖全部 cite 标记——修答案引用第 6 条以后前端解析不到、回退显示 doc_id
- feat(retrieval): 新增文档级检索工具 doc_search/doc_read（枚举题车道）；doc_search 增时间维度（sort_by/date_after/date_before）、entity_contains 实体范围与唯一命中 authoritative 卡片；doc_read 增 section 按章节读、match 模式 AND→OR 回退；检索分组抽象（group_id/group_kind、目录谓词、matter 清单）
- fix(table): 表格 query 首次无行数值允许按指标名/报表名改写一次再查；doc 范围明确且候选无完整表时按该文档 canonical 表格块兜底（全量 token 去重打分）；表查询引擎修三缺陷族（LaTeX 目标列失配回落成对列、区间列错配、右半区只扫首列选错行）与 '<=' 含等号误判方程
- feat(sop): SOP 领域知识外置 domain pack（`ANGINEER_SOP_DOMAIN_PACK` 按名/按 JSON 路径启用，内置 harbor 随 wheel 打包）；精排 LLM 加内层超时闸（`ANGINEER_SOP_ROUTE_LLM_TIMEOUT` 默认 30s）；L3 点名引导与证据闸纳入成功步骤产出；run 按真值步骤图遍历执行（next_step_id/on_failure 跳转生效，被跳过步骤记 skipped）
- fix(classifier): 公式/指标定义问句不再判 L3（须有具体数值参数）；L3 无匹配 SOP 回退语义检索；L0 闲聊词表补「笑话」
- feat(agent): 实验臂（默认关）——枚举闭集守卫与结构化整答（P4/P4b/P5）、枚举预取目录模式（P0.5）；删除失败的 P2.1/P2.2 证据绑定实验（默认从未启用）；graph 抽取并发闸门与问答轮数/预算闸放开（Harvey LAB 实踩）；budget stopper docstring 纠偏（显式 threshold＝真实 token 线，仅 usage 路径）


## 0.1.5

- fix: **独立壳补六端口 HTTP 适配器**（`angineer_core.server.http_ports`）——HTTP 检索模式下工具结果曾缺 `citations`（前端引用为空）、phrase rerank 与条款号归一化被跳过、canonical 查表找不到册、calculator 一调用即报错（根因：壳无组装层、进程内端口全为 None；外部消费方同题对照实测）。壳启动时自动注册六端口 HTTP 等价物（normalize_query / relevant_citations / local_rerank / local_nodes_loader / table_blocks / engtool_registry → 检索服务 `/internal/normalize-query`·`/rerank`·`/relevant-citations`·`/doc-nodes`·`/table-blocks`·`/engtool-run`；配 `ANGINEER_DOCS_API_URL` 生效，`ANGINEER_HTTP_PORTS=0` 关闭）；网络失败一律降级为原行为、绝不发工具级异常（`table_blocks` 例外按显式失败上浮）。行为与进程内装配对齐（同一份 docs-core 实现）
- fix: 壳启动即配 `logging.basicConfig`——此前日志落 lastResort（stderr 裸格式），排障时误判「警告没落日志」
- feat: 领域包扩「参数规则」段 + classifier 注入式重构（随本批直通）——`domain_packs/harbor.json` 增 arg_rules 30 条 / arg_computes 7 算子 / 船型 6 表 / 停用词 21 / 清洗规则；新 `domain_arg_rules` 应用器（不做泛化 DSL）；classifier 改 `domain_args` 注入（**不启用领域包时默认零领域规则，行为逐字不变**）；Hnav 取值链（显式值＞乘潮 90~95% 档＞设计低水位，JTS 165 §4.5.2.4 口径）与桥区通航净空尺度输出映射修复；对拍 23/23 旧键全等
- chore: `angineer_core.server` 惰性导出（导入 `server.http_ports` 不再构造壳应用）；`DocsRetrievalClient` 增 normalize_query / rerank / relevant_citations / table_blocks / run_engtool 五方法
- 已知限制：`/internal/engtool-run` 的执行体 `angineer-engtools` 未随壳发布——需 calculator / conditional 的独立部署请从主仓库源码安装该包；未安装时调用按显式 503 降级（不再静默）

## 0.1.4

- feat: **独立宿主壳 `[server]` extra（chat-api）**——`angineer_core.server:app` 把 agent 编排以 SSE 服务形态直接暴露：`POST /api/chat/agent`（AgentEvent 帧、`frame_version: 1`、首帧 `route_debug`、流末 `[DONE]`）、`GET /health`；可选 `ANGINEER_API_TOKEN` 门（`X-API-Key` / `Bearer`）。SOP 发布面 / chat-history / admin 鉴权均不依赖，缺之照常启动；fastapi / uvicorn 只进 extra，纯库消费者不变重（与 AnGIneer 内部服务同源同行为）
- feat: `route_pre` 自内部服务迁入本包（`angineer_core.route_pre`）单点维护——`route_request` 派工单 / 赌博式预检 `fire_speculative_first_search` / `route_debug` 首帧；对 docs-core 的表题判别依赖转为可选降级（未装 docs-core 照常预检）；`base_contracts` 增 `normalize_library_ids` 引擎侧镜像（独立壳不依赖 docs-core 的等价实现）
- feat: **查表错配双闸 + 矩阵表读表判读通道 `table_read`**——表号形态（表5.4.8 / 表A.0.1）编号显式匹配全 miss 时显式报错（附候选 context 诊断），禁止拆词兜底把「找不到表」伪装成「行不匹配」（LNG 实踩）；`table_read` 对矩阵 / 双层表头表交单次 LLM 抽取并回填单元格引用（可审计），sop_runner / agent_tools 工具白名单同步
- feat: SOP 引导与证据三件——**L3 点名引导**（TF-IDF 粗筛 top-1≥0.10 时向复杂档提示词注入「疑似适用 SOP」，`ANGINEER_SOP_L3_HINT` 默认开）；**证据闸纳入 SOP 成功步骤产出**（修「仅调 SOP 未调检索」被 no_evidence 整答误杀；步骤全失败仍拒答）；**复杂档提示词版本机制** `ANGINEER_COMPLEX_PROMPT_VERSION`（预留回退位，当前仅注册 v5）
- feat: classifier 参数派生补「设计高 / 低水位」（SOP 受力控制路径 `${设计高水位}` 直接引用）

## 0.1.3

- feat: **SOP 领域知识外置为领域知识包（domain pack）**——`classifier.PARAM_DESCRIPTIONS` 不再硬编码疏浚/海港参数词典（默认空 dict，引擎零领域内容）；新增 `sop_domain.load_domain_pack_from_env()`，宿主经 `ANGINEER_SOP_DOMAIN_PACK`（内置包名如 `harbor`，或任意 JSON 文件路径）显式启用，调用期惰性加载、加载失败按空包降级告警；内置包 `domain_packs/harbor.json` 随 wheel 打包（setuptools `package-data`）。`IntentClassifier` 新增 `param_descriptions` 构造参数、`SopRunner` 新增 `domain_hints`（参数释义 / 工具输出别名 `output_aliases` / calculator 领域提示三字段独立注入），`SopRunnerAdapter` 自动接线；**不启用领域包时行为逐字不变**
- feat: `SopRunner.run` 真按步骤图遍历执行——步骤失败且 `on_failure` 指向未执行步骤时跳转（被跳过的中间步骤记 `skipped`），无可用跳转时回退原线性序；`next_step_id`/`on_failure` 自 P4 起是 SOP 契约字段，本版起被执行面真正消费
- feat: prompts——QA 系统提示词 v18（规则 2「写明出处」加适用前提：仅对象核对通过后适用，对象核不上禁止用《X》包装相邻片段，按规则 3 走「未覆盖 + 供参考」第三档）；`answer_eval` 新增 `evidence_support_prompt` v1（该拒未拒二段判定：区分有据未拒与真幻觉）与 `refusal_content_prompt` v1（未拒答案 vs gold 三档 correct/wrong/uncertain）；SOP 执行 calculator hint 升 v2（领域取值规则 K1/折减系数迁往领域包，提示词仅留通用推导规则）

## 0.1.2

- fix: 出处守卫书名号闸「标题核不到」降级为软信号——答案引用的论文/真题名在证据面（`doc_title`=文件名）核不到时，若「第X章/节/条」引号章节名或裸数字条款号在证据中可核（归一化命中／数字链 `X.Y+`）则放行、落入原有编号核对；标题与章节号双双核不到仍判编造。修 v0.2.92 书名号闸两晚把 30+ 题好答案整答换拒答（OpenRAG −2.6pp、24 题误杀）
- fix: 外部文献名引用降为剥标记（方案 A）——`find_unsupported_reference` 三态化：编造规范编号仍 hard 整答替换；「标题全核不到 + 章节不可信」降为 strip（`_strip_absent_citations` 摘句首《X》状语、悬空章节号补「⚠️出处待核」、正文事实保留，不再换成拒答）；`has_unsupported_reference` 语义收口为 hard-only，兼容旧调用点；新增结果码 `external_citation_stripped`（`final_outcome=model_answer_stripped`，`path_trace` 记痕）

## 0.1.1

- fix: **观测打点不再写进宿主工作目录**。`ops_metrics` 的落盘目录探测此前在找不到主仓库标记（同时含 `services/` 与 `apps/`）时回落到 `Path.cwd()`——于是 pip 装到 site-packages 的环境里，TTFT / 工具耗时 / 检索分段等 jsonl（内容含用户提问片段，如 `inspect_evidence` 记 `query[:120]`）会被写进宿主进程的工作目录。现在改为：找不到标记即 `ops_dir()` 返回空串、`record_event` 静默跳过；要落盘必须显式给 `ANGINEER_OPS_DIR`。**主仓库树内行为一字不变**（本机 repo 与生产镜像都是 `pip install -e`，`__file__` 在主仓库树内 → 默认仍是 `<仓库根>/data/ops`）
- chore: 新增 `tests/test_ops_metrics.py` 锁边界——树外默认不落盘且不碰 cwd / 显式给目录照常落盘 / 树内默认目录不变 / `ANGINEER_OPS_DISABLE` 优先级最高 / 上下文 `run_id` 随打点落库；另有一条「主仓库树内能命中标记」的用例（独立仓 / site-packages 环境自动跳过）
- docs: README 补两处口径——环境变量表增加 `ANGINEER_OPS_DIR` / `ANGINEER_OPS_DISABLE` 的默认值与优先级；并写明本库与其依赖 `angineer-ai-inference` 在 **import 时**各按 python-dotenv 规则加载一次 `.env`（`override=False`，已存在的环境变量不被覆盖），宿主据此可自控环境

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
