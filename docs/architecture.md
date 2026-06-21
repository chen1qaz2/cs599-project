# Agentic RAG 企业知识库自治运营智能体架构说明

## 1. 总体架构

项目采用前后端分离与后端多模块架构，核心目标是把企业知识库从“被动问答系统”升级为“可自主评估、诊断和改进的 Agentic RAG 系统”。

```mermaid
flowchart LR
    User[用户 / 管理员] --> Frontend[React 前端]
    Frontend --> API[Spring Boot API]
    API --> Rag[RAG 问答管道]
    API --> Agent[知识库运营 Agent]
    API --> Admin[管理端服务]

    Rag --> Rewrite[Query Rewrite]
    Rag --> Retrieve[多通道检索]
    Rag --> Prompt[Prompt 组装]
    Rag --> LLM[LLM 模型路由]
    Rag --> Fallback[无文档兜底回答]

    Agent --> Planner[Planner]
    Planner --> Executor[Executor]
    Executor --> Tools[Tool Registry]
    Tools --> Benchmark[BenchmarkTool]
    Tools --> Gap[知识缺口分析]
    Tools --> Freshness[新鲜度检查]
    Tools --> Sensitive[敏感信息检测]
    Tools --> Quality[Chunk 质量检查]

    Retrieve --> Vector[(pgvector / Milvus)]
    API --> DB[(PostgreSQL)]
    API --> Redis[(Redis)]
    API --> MQ[(RocketMQ)]
    Rag --> MCP[MCP Server]
```

## 2. 模块职责

- `src/bootstrap`: 主应用模块，包含 RAG 问答、Agent 编排、知识库管理、文档摄取、管理端接口和用户认证。
- `src/infra-ai`: AI 基础设施模块，封装 Chat、Embedding、Rerank、模型路由、健康检查和失败降级。
- `src/framework`: 通用工程能力模块，包含统一返回体、异常处理、Trace、Web 配置、Redis、RocketMQ、分布式 ID 等。
- `src/mcp-server`: MCP 工具服务端示例，为主应用提供天气、工单、销售查询等外部工具能力。
- `src/frontend`: React 管理端和问答端，包含聊天、知识库、摄取流水线、Trace、Agent 运营等页面。
- `src/resources`: 数据库初始化脚本、升级脚本、Docker Compose 文件和示例知识库文档。

## 3. Agent 运行流程

```mermaid
sequenceDiagram
    participant U as 管理员
    participant A as KnowledgeOpsAgentService
    participant P as Planner
    participant E as Executor
    participant T as Tool Registry
    participant R as Report

    U->>A: 发起知识库运营任务
    A->>P: 根据目标生成 AgentPlan
    P-->>A: 返回步骤列表
    A->>E: 执行计划
    loop 每个步骤
        E->>T: 根据 toolName 查找工具
        T-->>E: 返回 AgentTool
        E->>E: 注入上下文并执行
        E->>A: 记录步骤状态与结果
    end
    E->>R: 汇总工具输出
    R-->>A: 生成运营报告
    A-->>U: 返回运行记录、步骤明细、改进建议
```

## 4. RAG 问答流程

```mermaid
flowchart TD
    Q[用户问题] --> Memory[加载会话记忆]
    Memory --> Rewrite[问题改写与子问题拆分]
    Rewrite --> Intent[意图识别]
    Intent --> Route{是否需要检索}
    Route -->|系统闲聊| SystemAnswer[直接回答]
    Route -->|知识库 / 工具| Retrieval[多通道检索与 MCP 调用]
    Retrieval --> HasContext{是否有上下文}
    HasContext -->|有| RagPrompt[RAG Prompt 组装]
    HasContext -->|无| FallbackPrompt[兜底 Prompt 组装]
    RagPrompt --> Stream[LLM 流式回答]
    FallbackPrompt --> Stream
    SystemAnswer --> Stream
    Stream --> Trace[记录 Trace 与消息]
```

## 5. 数据流设计

- 文档摄取数据流: 文档源进入 `Fetcher`，经 `Parser` 解析、`Chunker` 切分、`Enhancer/Enricher` 增强，最后由 `Indexer` 写入向量库和关系库。
- 问答数据流: 用户问题经改写、意图识别、多通道检索、Rerank 和 Prompt 组装后发送给 LLM，回答过程通过 SSE 推送到前端。
- Agent 运营数据流: Agent 根据计划调用工具，工具读取知识库、检索结果、评测集和运行指标，生成步骤记录、风险项、知识缺口和改进建议。
- 可观测数据流: 每次问答和 Agent 执行都会记录 Trace、步骤状态、耗时、命中文档和工具输出，供管理端复盘。
