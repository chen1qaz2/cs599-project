# KnowledgeOps Agent：企业知识库自治运营智能体

## 项目简介

KnowledgeOps Agent 是一个面向企业知识库场景的 Agentic RAG 系统，围绕“文档入库、知识问答、Agent 评估、知识优化”构建闭环，让企业知识库从被动资料库升级为可诊断、可评估、可持续优化的知识运营平台。

本项目对应课程方向为 **Agentic AI 原生开发**。系统不仅支持基于企业文档的 RAG 问答，还提供数据通道、意图管理、链路追踪、Benchmark 评测、知识缺口分析和 Agent 自治运营报告，用于解决传统知识库“能存文档但难以评估质量、能回答问题但难以定位缺口”的问题。

## 核心能力

- **文档摄取 Pipeline**：支持本地文件、URL、飞书文档、S3/RustFS 等来源，经过 Fetcher、Parser、Enhancer、Chunker、Enricher、Indexer 节点，将原始文档转化为可检索、可引用、可评估的知识片段。
- **Agentic RAG 问答**：支持问题改写、会话记忆、意图识别、多通道检索、Rerank 重排、Prompt 组装、SSE 流式输出和低置信兜底回答。
- **混合检索与混合存储**：检索侧包含全局向量检索和意图定向检索；存储侧使用 PostgreSQL 保存业务数据、Run/Step/Trace 等结构化记录，并通过 pgvector 或 Milvus 支撑向量检索。
- **知识库自治运营 Agent**：通过 Planner、Executor、Tool Registry、Reporter 组织工具链，对知识库进行画像、检索测试、Benchmark、知识缺口分析、Chunk 质量检查、覆盖率评估和敏感信息检测。
- **可观测与可复盘**：RAG 问答记录 Trace，Agent 任务记录 Run 和 Step，每一步工具调用都保留输入、输出、状态、耗时和异常信息，便于定位问题和复盘结果。
- **Benchmark 评测体系**：支持单条 Query 评测和批量问题集评测，记录 SSE 流式响应、命中文档、命中 Chunk、首 token 时间、Hit@K、Recall、MRR、RAGAS 类指标和失败样例。

## 技术栈

- **AI IDE**：Trae CN、Codex
- **LLM 接入**：DeepSeek API（课程目标），同时保留 Bailian、SiliconFlow、AIHubMix、Ollama 等模型提供方配置
- **Agent 编排**：自研 Planner / Executor / Tool Registry / Reporter，借鉴 LangGraph 的状态流转和工具节点思想
- **后端**：Spring Boot 3.5、MyBatis-Plus、Sa-Token、Maven 多模块工程
- **前端**：React 18、Vite、TypeScript、Tailwind CSS、Zustand、Radix UI、Recharts
- **RAG 能力**：Query Rewrite、Memory、Intent、Multi-channel Retrieval、Rerank、Prompt、Fallback、Trace
- **数据与基础设施**：PostgreSQL / pgvector、Milvus、Redis、RocketMQ、RustFS / S3
- **工具协议**：MCP Server / Client
- **容器化**：Docker、Docker Compose

## 系统架构

系统采用前后端分离和后端多模块架构。前端负责问答、知识库管理、数据通道、意图管理、Trace 和 Agent 运营页面；后端负责用户认证、知识库管理、文档摄取、RAG 问答、Agent 工具编排和运行记录持久化；AI 基础设施层负责模型路由、Embedding、Rerank 和模型健康检查。

核心闭环如下：

```text
文档入库 -> 知识片段生成 -> RAG 问答消费 -> Agent 评估诊断 -> 知识补充与检索策略优化
```

Agent 自治运营链路如下：

```text
Run Request -> Planner -> Executor -> AgentTool -> Step Record -> Reporter -> KnowledgeOps Report
```

RAG 问答链路如下：

```text
User Query -> Query Rewrite / Memory -> Intent -> Multi-channel Retrieval -> Dedup / Rerank -> Prompt -> LLM -> Answer / Fallback -> Trace
```

## 目录结构

```text
cs599-project/
├── docs/                    # 项目文档
│   ├── CS599_大作业报告.pdf   # 最终课程报告
│   └── architecture.md       # 架构说明
├── src/                     # 项目源代码
│   ├── bootstrap/           # 主业务应用：RAG、Agent、知识库、数据通道、管理端接口
│   ├── infra-ai/            # LLM、Embedding、Rerank、模型路由和健康检查
│   ├── framework/           # 通用工程能力：异常、上下文、Trace、Redis、MQ、幂等
│   ├── mcp-server/          # MCP 工具服务示例
│   ├── frontend/            # React 前端：问答、Dashboard、知识库、Trace、Agent 运营
│   ├── resources/           # 数据库脚本、Docker Compose、示例知识库
│   ├── assets/              # 截图和架构图素材
│   ├── scripts/             # 测试和演示脚本
│   ├── pom.xml              # Maven 多模块入口
│   └── mvnw / mvnw.cmd      # Maven Wrapper
├── README.md
├── .gitignore
└── LICENSE
```

后端核心包职责：

```text
src/bootstrap/src/main/java/com/nageoffer/ai/ragent/
├── agent/       # KnowledgeOps Agent：Run、Step、Planner、Executor、Tool、Reporter
├── rag/         # RAG 主链路：改写、意图、记忆、检索、Prompt、Trace、SSE 问答
├── ingestion/   # 数据通道：Fetcher、Parser、Enhancer、Chunker、Enricher、Indexer
├── knowledge/   # 知识库、文档、Chunk、文档处理任务
├── admin/       # Dashboard 和管理端统计
├── core/        # 文档解析、文本切分、向量 Chunk 等基础能力
└── user/        # 登录认证、用户和权限上下文
```

## 环境搭建

### 1. 基础依赖

建议环境：

- JDK 17+
- Node.js 18+
- Docker / Docker Compose
- PostgreSQL 15+，需要支持 pgvector
- Redis
- RocketMQ
- Milvus（可选，默认也可使用 pgvector）

### 2. 环境变量

不要在代码或配置文件中硬编码 API Key。按实际使用的模型提供方配置环境变量，至少需要一个可用 Chat 模型和一个 Embedding 模型。

PowerShell 示例：

```powershell
$env:DEEPSEEK_API_KEY="your-deepseek-api-key"
$env:BAILIAN_API_KEY="your-bailian-api-key"
$env:SILICONFLOW_API_KEY="your-siliconflow-api-key"
$env:AIHUBMIX_API_KEY="your-aihubmix-api-key"
```

Bash 示例：

```bash
export DEEPSEEK_API_KEY="your-deepseek-api-key"
export BAILIAN_API_KEY="your-bailian-api-key"
export SILICONFLOW_API_KEY="your-siliconflow-api-key"
export AIHUBMIX_API_KEY="your-aihubmix-api-key"
```

前端环境变量参考：

```text
src/frontend/.env.example
```

后端关键配置文件：

```text
src/bootstrap/src/main/resources/application.yaml
```

默认端口：

- 后端：`http://localhost:9090/api/ragent`
- 前端：`http://127.0.0.1:5173`
- MCP Server：`http://localhost:9099`
- PostgreSQL：`127.0.0.1:5432`
- Redis：`127.0.0.1:6379`
- RocketMQ：`127.0.0.1:9876`
- Milvus：`http://localhost:19530`

### 3. 初始化数据库

数据库脚本位于：

```text
src/resources/database/schema_pg.sql
src/resources/database/init_data_pg.sql
```

默认初始化账号：

```text
username: admin
password: admin
```

### 4. 启动基础服务

```bash
cd src

# RocketMQ
docker compose -f resources/docker/rocketmq-stack-5.2.0.compose.yaml up -d

# Milvus，可选；若使用 pgvector 路线，可不启动
docker compose -f resources/docker/milvus-stack-2.6.6.compose.yaml up -d
```

PostgreSQL、Redis 和 RustFS/S3 可按本机环境启动，也可以根据课程演示环境替换为已有服务。连接参数在 `application.yaml` 中调整。

### 5. 启动后端

Windows PowerShell：

```powershell
cd src
.\mvnw.cmd -pl bootstrap -am -DskipTests package
java -jar bootstrap\target\bootstrap-0.0.1-SNAPSHOT.jar
```

Bash：

```bash
cd src
./mvnw -pl bootstrap -am -DskipTests package
java -jar bootstrap/target/bootstrap-0.0.1-SNAPSHOT.jar
```

### 6. 启动 MCP Server（可选）

```bash
cd src
./mvnw -pl mcp-server -am -DskipTests package
java -jar mcp-server/target/mcp-server-0.0.1-SNAPSHOT.jar
```

### 7. 启动前端

```bash
cd src/frontend
npm install
npm run dev -- --host 127.0.0.1 --port 5173
```

浏览器访问：

```text
http://127.0.0.1:5173/login
```

## 主要 API

Agent 自治运营以 `Run` 为核心资源。一次 `Run` 表示一次完整的知识库运营任务，任务中的每一步工具调用记录为 `Step`。

```text
POST /agent/knowledge-ops/runs
GET  /agent/knowledge-ops/runs
GET  /agent/knowledge-ops/runs/{runId}
GET  /agent/knowledge-ops/runs/{runId}/steps
```

示例请求：

```json
{
  "kbId": "kb_001",
  "task": "评估该知识库是否能够支撑新人入职制度问答",
  "scenario": "benchmark",
  "topK": 8,
  "benchmarkQuestions": [
    "新人试用期多久？",
    "入职材料需要哪些？"
  ]
}
```

## 测试与评估

项目采用“功能测试 + Agent 行为评估 + Benchmark 评测”的方式验证效果。

功能测试覆盖：

- 系统启动与登录
- 知识库、文档和 Chunk 管理
- 数据通道任务执行
- 意图管理与问题路由
- RAG 问答和兜底回答
- Trace 链路追踪
- Agent Run、Step 和报告展示

评测流程覆盖：

- 单条 Query 的 SSE 流式回答采集
- `/rag/eval` 元数据抽取
- 命中文档、命中 Chunk、意图字段、MCP 调用信息记录
- 批量问题集 Benchmark
- Hit@K、Recall、MRR、首 token 时间、整体延迟等自建指标
- faithfulness、relevancy、correctness、precision、recall 等 RAGAS 类指标
- 失败样例、CSV 和 Markdown 报告输出

## 项目状态

- [x] Proposal
- [x] MVP
- [x] Final Report
- [x] Agentic RAG 核心链路
- [x] BenchmarkTool 与知识缺口分析
- [x] 兜底回答机制
- [x] 前后端联调启动

## 课程提交说明

最终报告位于：

```text
docs/CS599_大作业报告.pdf
```

报告主题为 **KnowledgeOps Agent：企业知识库自治运营智能体**，方向为 **Agentic AI 原生开发**。

## License

本仓库使用 Apache License 2.0。
