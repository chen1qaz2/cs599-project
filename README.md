# Agentic RAG 企业知识库自治运营智能体

## 项目简介
本项目是一个面向企业知识库的 Agentic RAG 系统，用自治智能体完成文档检索问答、兜底回答、知识缺口分析、Benchmark 评估和知识库运营报告生成，解决传统 RAG 只能被动问答、缺少持续评估与自治运营能力的问题。

## 方向
Agentic AI 原生开发

## 技术栈
- AI IDE: Trae CN
- LLM: DeepSeek API（课程部署目标，可通过模型路由层扩展接入）；当前代码也支持 Bailian、SiliconFlow、AIHubMix、Ollama 等模型提供方
- Agent 编排: 自研 Planner / Executor / Tool Registry（参考 LangGraph 的状态机与工具编排思想）
- 后端框架: Spring Boot 3.5、MyBatis-Plus、Sa-Token
- 前端框架: React 18、Vite、TypeScript、Tailwind CSS、Zustand
- RAG 能力: Query Rewrite、多通道检索、Rerank、Prompt 组装、对话记忆、SSE 流式输出、无文档兜底回答
- Agent 工具: 知识库画像、文档新鲜度检查、敏感信息检测、BenchmarkTool、知识缺口分析、Chunk 质量检查、覆盖率评估
- 向量与存储: PostgreSQL / pgvector、Milvus、Redis、RustFS / S3
- 消息队列: RocketMQ
- MCP: MCP Server / Client，用于天气、工单、销售等外部工具调用
- 容器: Docker、Docker Compose
- 构建工具: Maven Wrapper、npm

## 目录结构
仓库根目录按课程提交要求重建，源代码统一放在 `src/` 下，报告与架构说明放在 `docs/` 下。

```text
.
├── docs/
│   ├── architecture.md             # 项目架构说明
│   └── CS599_大作业报告.pdf          # 最终报告，当前尚未提交
├── src/
│   ├── bootstrap/                  # 主后端应用：RAG 问答、Agent、知识库、摄取流水线、管理端接口
│   │   └── src/main/java/com/nageoffer/ai/ragent/
│   │       ├── agent/              # 企业知识库自治运营 Agent：规划器、执行器、工具、运行记录和报告
│   │       ├── rag/                # RAG 主链路：检索、改写、Prompt、MCP、记忆、Trace、SSE 流式聊天
│   │       ├── knowledge/          # 知识库、文档、Chunk、定时刷新和文档处理任务管理
│   │       ├── ingestion/          # 文档摄取流水线：Fetcher、Parser、Chunker、Enhancer、Indexer
│   │       ├── core/               # 文档解析、文本切分、向量 Chunk 等底层能力
│   │       ├── admin/              # 管理端数据看板与系统运营统计
│   │       └── user/               # 登录认证、用户管理和权限上下文
│   ├── infra-ai/                   # LLM、Embedding、Rerank、模型路由、健康检查和降级
│   ├── framework/                  # 通用返回体、异常、Trace、幂等、分布式 ID、Web 配置
│   ├── mcp-server/                 # MCP 工具服务端示例：天气、工单、销售查询工具
│   ├── frontend/                   # React 前端：聊天页、管理端、知识库、Trace、Agent 运营页面
│   ├── resources/                  # 数据库脚本、Docker Compose、示例知识库文档
│   ├── assets/                     # 项目截图和架构图素材
│   ├── scripts/                    # 测试和演示脚本
│   ├── pom.xml                     # Maven 多模块入口
│   └── mvnw / mvnw.cmd             # Maven Wrapper
├── README.md
├── .gitignore
└── LICENSE
```

## 环境搭建

### 1. 依赖安装
后端需要 JDK 17，前端需要 Node.js 18+，本地服务建议使用 Docker / Docker Compose 启动。所有后端构建命令都从 `src/` 目录执行。

```bash
cd src

# 后端依赖与打包
./mvnw -pl bootstrap -am -DskipTests package

# 前端依赖
cd frontend
npm install
```

Windows PowerShell 可使用：

```powershell
cd src
.\mvnw.cmd -pl bootstrap -am -DskipTests package

cd frontend
npm install
```

### 2. 环境变量配置
不要在代码或配置文件中硬编码 API Key。按实际使用的模型提供方设置环境变量，至少配置一个可用的 Chat 模型和一个 Embedding 模型。

```bash
# 示例：课程部署目标，可按模型路由层接入 DeepSeek API
export DEEPSEEK_API_KEY="your-deepseek-api-key"

# 项目默认配置中可直接使用的模型提供方
export BAILIAN_API_KEY="your-bailian-api-key"
export SILICONFLOW_API_KEY="your-siliconflow-api-key"
export AIHUBMIX_API_KEY="your-aihubmix-api-key"
```

Windows PowerShell：

```powershell
$env:DEEPSEEK_API_KEY="your-deepseek-api-key"
$env:BAILIAN_API_KEY="your-bailian-api-key"
$env:SILICONFLOW_API_KEY="your-siliconflow-api-key"
$env:AIHUBMIX_API_KEY="your-aihubmix-api-key"
```

前端环境变量请参考 `src/frontend/.env.example`，真实 `.env` 文件不要提交到仓库。

本地基础依赖默认地址如下，可在 `src/bootstrap/src/main/resources/application.yaml` 中调整：

- PostgreSQL: `127.0.0.1:5432`
- Redis: `127.0.0.1:6379`
- RocketMQ: `127.0.0.1:9876`
- MCP Server: `http://localhost:9099`
- 后端服务端口: `9090`
- 前端服务端口: `5173`

数据库初始化脚本位于：

```text
src/resources/database/schema_pg.sql
src/resources/database/init_data_pg.sql
```

### 3. 启动步骤
启动基础中间件：

```bash
cd src

# RocketMQ
docker compose -f resources/docker/rocketmq-stack-5.2.0.compose.yaml up -d

# 如使用 Milvus 向量库，可启动 Milvus；默认也支持 pgvector 路线
docker compose -f resources/docker/milvus-stack-2.6.6.compose.yaml up -d
```

启动后端：

```bash
cd src
./mvnw -pl bootstrap -am -DskipTests package
java -jar bootstrap/target/bootstrap-0.0.1-SNAPSHOT.jar
```

启动 MCP Server（可选，用于外部工具调用）：

```bash
cd src
./mvnw -pl mcp-server -am -DskipTests package
java -jar mcp-server/target/mcp-server-0.0.1-SNAPSHOT.jar
```

启动前端：

```bash
cd src/frontend
npm run dev -- --host 127.0.0.1 --port 5173
```

访问地址：

- 前端: `http://127.0.0.1:5173/`
- 后端: `http://localhost:9090/api/ragent`

## 项目状态
- [x] Proposal
- [x] MVP
- [ ] Final
