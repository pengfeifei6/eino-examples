# Eino Examples 详细学习指南

这是一份面向初学者的 Eino Examples 环境配置、项目结构与学习实践指南，重点回答三个问题：

1. 如何准备开发环境并成功运行第一个示例？
2. 每个目录分别解决什么问题，它们之间是什么关系？
3. 应该按什么顺序阅读、运行和修改代码，才能逐步掌握 Eino？

如果只想查找某个示例，可以阅读 [README.zh_CN.md](./README.zh_CN.md) 或 [COOKBOOK.md](./COOKBOOK.md)。如果是第一次接触 Eino，建议从本文第一节开始。

## 一、项目与环境配置

### 1.1 项目简介

[Eino](https://github.com/cloudwego/eino) 是一个使用 Go 编写的大模型应用开发框架。本仓库是它的示例集合，覆盖以下几层能力：

```text
基础数据结构与组件
        ↓
Prompt / ChatModel / Tool / Retriever 等 Component
        ↓
Chain / Graph / Workflow 等 Compose 编排
        ↓
ReAct、RAG、Multi-Agent 等 Flow
        ↓
Agent、Runner、Session、Middleware 等 ADK 应用能力
        ↓
调试、可视化、可观测和 Web 服务
```

这个仓库不是一个单一应用。大多数子目录都是彼此独立的示例，可能使用不同的模型、环境变量和外部服务。运行某个示例前，应先查看该目录下的 `README.md` 和 `main.go`。

### 1.2 基础软件要求

| 软件或服务 | 是否必需 | 建议版本或用途 |
|---|---:|---|
| Go | 是 | 根模块声明 `go 1.24.7`；若希望覆盖所有子模块，建议使用 Go 1.25.5 或更高版本 |
| Git | 是 | 克隆仓库、查看改动 |
| 大模型 API Key | 大多数示例需要 | OpenAI/OpenAI 兼容接口、火山方舟 Ark，或示例指定的其他模型 |
| Ollama | 可选 | 本地运行 `quickstart/chat`，避免调用云端模型 |
| Docker / Docker Compose | 可选 | 启动 Redis、运行完整 RAG 应用等 |
| Redis | 可选 | RAG、向量检索或记忆相关示例 |
| Python / uv | 可选 | Deer-Go、Excel Agent 以及部分 MCP 工具 |

先确认本地工具：

```bash
go version
git --version
go env GOTOOLCHAIN GOPROXY
```

如果 Go 版本低于模块要求，可以升级 Go；也可以保持 `GOTOOLCHAIN=auto`，让新版 Go 命令在需要时自动选择工具链。

### 1.3 获取代码与安装依赖

```bash
git clone https://github.com/cloudwego/eino-examples.git
cd eino-examples
go mod download
```

根目录是主 Go module，但仓库内还有几个独立 module：

| 子模块 | Go 版本声明 | 说明 |
|---|---:|---|
| `adk/intro/http-sse-service` | 1.24.9 | ADK HTTP SSE 服务 |
| `flow/agent/deer-go` | 1.25.5 | Deer-Go 完整应用 |
| `flow/agent/manus` | 1.24 | Manus Agent |
| `quickstart/chatwitheino` | 1.24.9 | 11 章渐进式 Agent 教程 |
| `quickstart/eino_assistant` | 1.23.0，指定 1.24.1 toolchain | 完整 RAG 助手 |

Go 命令在根 module 中执行时不会自动遍历这些嵌套 module。学习某个子模块时，需要进入对应目录再下载依赖或运行测试，例如：

```bash
cd quickstart/chatwitheino
go mod download
go test ./...
```

当前仓库快照中，个别独立 module 的 `go.sum` 没有覆盖源码实际导入的全部依赖。如果看到 `missing go.sum entry` 或 `updates to go.mod needed`，请在该子模块内执行 `go mod tidy` 后再运行。这个命令可能更新该子模块的 `go.mod` 和 `go.sum`，因此最好在自己的学习分支中执行并检查差异。

不建议刚开始就执行 `go run ./...`：仓库中有许多独立的 `main` 包，而且不同示例依赖不同配置。应该对具体目录使用 `go run ./path/to/example`。

### 1.4 配置大模型

#### 方式 A：OpenAI 或 OpenAI 兼容服务

不同年代的示例分别使用了 `OPENAI_MODEL`、`OPENAI_MODEL_NAME` 或 `OPENAI_MODEL_ID`。为了减少切换示例时的配置成本，可以同时设置三个变量：

```bash
export OPENAI_API_KEY="your-api-key"
export OPENAI_BASE_URL="https://api.openai.com/v1"  # 官方接口也可不设置
export OPENAI_MODEL="your-model-name"
export OPENAI_MODEL_NAME="$OPENAI_MODEL"
export OPENAI_MODEL_ID="$OPENAI_MODEL"
export OPENAI_BY_AZURE="false"
export MODEL_TYPE="openai"
```

如果使用 DeepSeek 等 OpenAI 兼容服务，将 `OPENAI_BASE_URL` 和模型名替换成服务商给出的值即可。是否完整兼容 Tool Calling、流式输出和 Agentic Message，取决于服务商与所选模型。

常见用法差异如下：

| 示例 | 读取的模型变量 |
|---|---|
| `adk/helloworld`、`adk/common/model` | `OPENAI_MODEL` |
| `quickstart/chat`、部分 `compose`/`flow` 示例 | `OPENAI_MODEL_NAME` |
| `adk/agentic/*` | `OPENAI_MODEL_ID` |
| `quickstart/chatwitheino` | `OPENAI_MODEL` |

#### 方式 B：火山方舟 Ark

```bash
export MODEL_TYPE="ark"
export ARK_API_KEY="your-api-key"
export ARK_MODEL="your-endpoint-or-model-id"
export ARK_MODEL_ID="$ARK_MODEL"
export ARK_MODEL_NAME="$ARK_MODEL"
export ARK_BASE_URL=""                       # 仅在需要自定义地址时填写
```

完整 RAG 示例还可能要求：

```bash
export ARK_CHAT_MODEL="your-chat-model"
export ARK_EMBEDDING_MODEL="your-embedding-model"
```

#### 方式 C：本地 Ollama

`quickstart/chat` 内置了 Ollama 分支，默认连接 `http://localhost:11434` 并使用代码中写定的 `llama2`：

```bash
ollama pull llama2
ollama serve

# 新开一个终端
export EINO_CHAT_PROVIDER="ollama"
go run ./quickstart/chat
```

如需换模型或地址，请修改 `quickstart/chat/ollama.go` 中的 `Model` 和 `BaseURL`。

#### 使用本地 `.env`

大部分根目录示例只调用 `os.Getenv`，不会自动读取 `.env`。可以在仓库根目录创建已被 `.gitignore` 忽略的 `.env`，然后在当前 shell 中加载：

```bash
set -a
source .env
set +a
```

不要把真实 Key 写入 README、提交到 Git，或放进供他人复制的 `.example.env`。一旦密钥进入提交历史或公开日志，应立即在服务商控制台撤销并重新生成。

### 1.5 可选外部服务

按需配置，不需要一次全部安装：

| 能力 | 常见配置 | 使用位置 |
|---|---|---|
| CozeLoop 可观测 | `COZELOOP_API_TOKEN`、`COZELOOP_WORKSPACE_ID` | Compose、Flow、ADK 中的 Trace 示例 |
| APMPlus | `APMPLUS_APP_KEY`、可选 `APMPLUS_REGION` | Eino Assistant、Deer-Go |
| Redis | `REDIS_ADDR`，或示例内默认地址 | RAG、Retriever、Memory |
| Langfuse | `LANGFUSE_HOST`、`LANGFUSE_PUBLIC_KEY`、`LANGFUSE_SECRET_KEY` | Manus 等可观测示例 |
| VikingDB | `VIKING_DB_HOST/REGION/AK/SK` | 向量检索示例 |

只有同时设置 CozeLoop Token 和 Workspace ID 时，相关示例才会注册 Trace Handler；不需要可观测功能时通常可以留空。

### 1.6 第一次运行

建议先运行不调用真实大模型的示例，验证 Go 环境和依赖：

```bash
# Prompt 模板渲染
go run ./components/prompt/chat_prompt

# 仅包含 Lambda 的简单 Workflow
go run ./compose/workflow/1_simple

# 使用 mock ChatModel 的 Graph，演示 Invoke 与 Stream
go run ./compose/graph/simple
```

模型环境变量配置完成后，再运行最小 Agent：

```bash
go run ./adk/helloworld
```

这个示例的关键执行链是：

```text
读取环境变量 → 创建 ChatModel → 创建 ChatModelAgent
              → 创建 Runner → 输入 Message → 遍历流式 Event
```

接着可以运行包含模板、普通生成和流式生成的 Chat 示例：

```bash
go run ./quickstart/chat
```

注意：这个目录读取的是 `OPENAI_MODEL_NAME`，不是 `OPENAI_MODEL`。

### 1.7 编译与测试建议

先验证正在学习的单个包，再扩大范围：

```bash
# 查看包是否能被 Go 正确识别
go list ./adk/helloworld

# 独立 module 的测试需要先进入该目录
cd quickstart/chatwitheino
go mod tidy
go test ./msgops
cd ../..

# 测试根 module；嵌套 module 需要分别进入目录测试
go test ./...

# 显示更完整的测试日志
go test -v ./path/to/package
```

部分示例的测试或运行过程会访问模型、Redis、浏览器或远程服务。出现失败时，先区分它是编译错误、配置缺失，还是外部服务不可用。

## 二、项目目录

### 2.1 顶层目录

```text
eino-examples/
├── adk/          # Agent 开发套件：Agent、Runner、Session、Middleware、多 Agent
├── components/   # 可复用基础组件：Model、Prompt、Tool、Retriever、Document、Lambda
├── compose/      # 确定性编排：Chain、Graph、Workflow、Batch
├── flow/         # 已封装的 Agent 流程：ReAct、Multi-Agent、Manus、Deer-Go
├── quickstart/   # 从最小对话到完整 Web/RAG 应用的入门项目
├── devops/       # 调试与 Mermaid 可视化
├── internal/     # 仅供本仓库复用的日志和指针辅助代码
├── COOKBOOK.md   # 更细的示例导航
├── go.mod        # 根 module 依赖与 Go 版本
└── README.zh_CN.md
```

可以按照下面的方式理解它们的职责：

| 目录 | 主要问题 | 学习后应该掌握 |
|---|---|---|
| `components` | 一个模型、工具或检索器如何接入 Eino？ | 组件接口、消息结构、配置与调用方式 |
| `compose` | 多个组件如何组成稳定流程？ | Chain、Graph、Workflow、类型映射、流式处理 |
| `flow` | 常见 Agent 模式如何实现？ | ReAct、记忆、计划执行、多 Agent 协作 |
| `adk` | 如何构建可运行、可恢复、可扩展的 Agent 应用？ | Agent、Runner、Session、中断恢复、中间件、HITL |
| `quickstart` | 如何把这些能力连成一个产品原型？ | 从 Console 到 Web、RAG、Skill 和 UI 事件流 |
| `devops` | 如何观察和调试执行过程？ | Graph 可视化、节点调试、Trace |

### 2.2 `components`：基础组件

- `model`：ChatModel 的 A/B 路由、HTTP Transport 日志等。
- `prompt`：消息模板、变量填充、历史消息占位。
- `tool`：JSON Schema Tool、MCP Tool 以及 Tool Middleware。
- `retriever`：多查询检索、按条件选择 Retriever。
- `document`：文本、自定义和扩展文档解析器。
- `lambda`：把普通 Go 函数适配成可编排节点。

阅读组件时重点观察三件事：构造函数需要什么配置、输入输出类型是什么、它支持 `Invoke` 还是同时支持 `Stream`。

### 2.3 `compose`：编排层

- `chain`：线性执行，适合固定的“预处理 → Prompt → Model → 后处理”。
- `graph`：用节点和边表达分支、循环、状态和中断。
- `workflow`：同时描述控制依赖和字段级数据映射。
- `batch`：批量任务、并发控制、中断和恢复。

三个核心概念：

1. **构建**：添加节点、边、分支和字段映射。
2. **编译**：检查图结构和类型，得到可执行对象。
3. **运行**：使用 `Invoke`、`Stream`、`Collect` 或 `Transform` 处理不同数据形态。

### 2.4 `flow`：常用 Agent 模式

- `agent/react`：模型思考、选择工具、读取结果并继续推理的 ReAct 循环。
- `agent/multiagent`：Host、Plan-Execute 等多 Agent 协作方式。
- `agent/manus`：参考 OpenManus 的通用 Agent，依赖独立 module 和更多外部工具。
- `agent/deer-go`：研究型团队工作流，是复杂完整应用，不适合作为第一个示例。

### 2.5 `adk`：Agent Development Kit

- `helloworld`：最小的 `ChatModelAgent + Runner`。
- `intro`：ChatModel Agent、自定义 Agent、Session、Transfer、Workflow、SSE 服务。
- `human-in-the-loop`：审批、修改、反馈、追问、Supervisor、Plan-Execute-Replan 等八组模式。
- `middlewares`：Skill 与动态工具检索。
- `multiagent`：Supervisor、分层 Supervisor、Deep Agent、项目管理和 Excel Agent。
- `agentic`：AgenticModel、研究助手、输出截断重试。
- `cancel`：取消、退出和恢复。
- `common`：多个 ADK 示例共享的 Model、Tool、Store、Trace 和打印逻辑。

### 2.6 `quickstart`：完整入门路线

- `chat`：最小 ChatModel 示例，包含 Prompt、Generate 和 Stream。
- `todoagent`：用 Agent 管理 Todo 的小型案例。
- `chatwitheino`：11 章渐进教程，是最适合系统学习的主线。
- `eino_assistant`：带知识索引、Redis、Agent 服务和 Web UI 的完整 RAG 应用。

### 2.7 `devops` 与 `internal`

- `devops/debug` 展示 Chain 和 Graph 的调试能力。
- `devops/visualize` 把编排结构输出为 Mermaid 图。
- `internal/logs` 和 `internal/gptr` 是仓库内部辅助包，不属于 Eino 对外 API。

## 三、推荐学习路线

### 3.1 学习方法

我会对每个示例重复下面这个闭环，而不是只阅读代码：

1. **先运行**：记录输入、输出、需要的环境变量和外部服务。
2. **再画执行链**：找出数据从哪里进入，经过哪些组件，在哪里结束。
3. **阅读关键接口**：重点看构造、编译、调用和错误处理，不一开始追所有依赖源码。
4. **做一个小修改**：替换 Prompt、增加节点、添加 Tool 或制造一次错误。
5. **写下结论**：用自己的话解释该示例解决的问题，以及何时不该用它。

建议在自己的分支学习：

```bash
git switch -c study/eino-examples
```

每完成一个阶段提交一次，方便比较自己的改动和原始实现。

### 3.2 阶段 0：补齐 Go 基础

开始前至少应熟悉：

- `context.Context` 的取消与超时；
- interface、泛型和函数类型；
- goroutine、channel 与流式消费；
- error 处理和 `defer`；
- Go module 与环境变量。

不需要先成为 Go 专家，但如果看不懂泛型输入输出或 channel，后面的 Compose 和 Stream 会比较吃力。

### 3.3 阶段 1：Message、Prompt 与 Lambda  11

按顺序运行：

```bash
go run ./components/prompt/chat_prompt
go run ./compose/workflow/1_simple
```

阅读目标：

- 理解 `schema.Message` 的 Role、Content 和 Tool Call 信息；
- 理解 Prompt Template 如何把 `map[string]any` 渲染成消息列表；
- 理解 Lambda 如何把普通 Go 函数变成编排节点；
- 区分数据输入输出类型和模型本身。

练习：在 Prompt 中添加一个变量；把 Workflow 的输入从 `int` 改成自定义 struct，并在 Lambda 中生成字符串。

### 3.4 阶段 2：ChatModel 与流式输出

```bash
go run ./quickstart/chat
```

建议依次阅读：

1. `quickstart/chat/main.go`：整体调用顺序；
2. `template.go`：如何构造消息；
3. `openai.go` 或 `ollama.go`：模型初始化；
4. `generate.go`：普通响应；
5. `stream.go`：循环接收流式 chunk 和处理 EOF。

练习：更换 system prompt；给调用增加超时；比较 `Generate` 与 `Stream` 的错误出现时机。

### 3.5 阶段 3：Chain、Graph 与 Workflow

推荐顺序：

```bash
go run ./compose/workflow/1_simple
go run ./compose/graph/simple
go run ./compose/chain
```

前两个示例可以帮助理解编排本身；`compose/chain` 会调用真实模型，并加入分支、并行节点和可选 Trace。

这一阶段要能回答：

- 为什么要先 `Compile` 再运行？
- Chain 和 Graph 分别适合什么场景？
- `START`、`END`、节点和边分别代表什么？
- `Invoke` 和 `Stream` 的输入输出有何差异？
- 状态、分支、中断和字段映射应该放在哪一层？

随后阅读 `compose/workflow/2_field_mapping` 到 `6_stream_field_map`，再看 `compose/graph/state`、`tool_call_once` 和 `react_with_interrupt`。

### 3.6 阶段 4：最小 ADK Agent

```bash
go run ./adk/helloworld
```

阅读顺序：模型初始化 → `adk.NewChatModelAgent` → `adk.NewRunner` → `runner.Run` → 遍历 Event。

练习：

- 修改 Agent 的 `Name`、`Description` 和 `Instruction`；
- 输入两条消息，观察上下文；
- 打印不同类型的 Event；
- 故意使用错误 Key，观察错误从哪里返回。

完成后阅读 `adk/intro`，顺序建议为：

```text
chatmodel → session → custom → transfer
          → workflow/sequential → parallel → loop
          → agent_with_summarization → http-sse-service
```

### 3.7 阶段 5：沿 ChatWithEino 完成系统教程

进入独立 module：

```bash
cd quickstart/chatwitheino
go mod tidy
go run ./cmd/ch01 -- "用一句话解释 Eino 的 Component 设计"
```

按照章节前后依赖学习：

| 章节 | 主题 | 文档 |
|---:|---|---|
| 1 | ChatModel 与 AgenticMessage | [ch01](./quickstart/chatwitheino/docs/ch01_chatmodel_agent_console.md) |
| 2 | Agent 与 Runner | [ch02](./quickstart/chatwitheino/docs/ch02_chatmodel_agent_runner_console.md) |
| 3 | Memory 与 Session | [ch03](./quickstart/chatwitheino/docs/ch03_memory_session_jsonl.md) |
| 4 | Tool 与文件系统 | [ch04](./quickstart/chatwitheino/docs/ch04_tool_backend_filesystem.md) |
| 5 | Middleware | [ch05](./quickstart/chatwitheino/docs/ch05_middleware.md) |
| 6 | Callback 与 Trace | [ch06](./quickstart/chatwitheino/docs/ch06_callback.md) |
| 7 | Interrupt / Resume | [ch07](./quickstart/chatwitheino/docs/ch07_interrupt_resume.md) |
| 8 | Graph Tool | [ch08](./quickstart/chatwitheino/docs/ch08_graph_tool.md) |
| 9 | Skill | [ch09](./quickstart/chatwitheino/docs/ch09_skill.md) |
| 10 | A2UI 与流式 UI | [ch10](./quickstart/chatwitheino/docs/ch10_a2ui.md) |
| 11 | TurnLoop 生命周期 | [ch11](./quickstart/chatwitheino/docs/ch11_turnloop.md) |

每学完一章，先比较它和上一章新增了哪些文件、对象和状态，再继续下一章。最终可在该目录运行 Web 版：

```bash
HOST=127.0.0.1 PORT=8080 go run .
```

默认只应绑定本机地址。示例服务没有完整的生产认证、限流和租户隔离，不要直接暴露到公网。

### 3.8 阶段 6：Tool、RAG 与人机协作

推荐顺序：

1. `components/tool/jsonschema`：理解 Tool Schema 和参数；
2. `flow/agent/react`：理解模型如何循环调用工具；
3. `components/retriever` 与 `components/document`：理解文档处理和检索；
4. `quickstart/eino_assistant`：把索引、Retriever、Agent、Redis 和 Web 服务串起来；
5. `adk/human-in-the-loop/1_approval` 到 `8_supervisor-plan-execute`：理解中断、审批、编辑和恢复。

学习 Human-in-the-Loop 时，要特别关注两份状态：业务状态和 Agent/Graph 执行状态。恢复操作必须知道从哪个 checkpoint 继续，以及用户补充的数据进入哪个节点。

### 3.9 阶段 7：多 Agent、可观测与综合项目

最后再进入：

- `adk/multiagent`：Supervisor、Plan-Execute-Replan、Deep Agent；
- `flow/agent/multiagent`：Host 和 Plan-Execute；
- `devops/debug`、`devops/visualize`：调试与拓扑可视化；
- `flow/agent/manus`、`flow/agent/deer-go`：综合案例。

此时不要只追求“能运行”，还要检查超时、取消、重试、Token 消耗、Trace、工具权限、密钥管理和失败恢复。

## 四、我的学习记录模板

可以复制下面的模板，为每个示例建立学习笔记：

```markdown
## 示例：compose/graph/simple

- 日期：
- 目标：
- 启动命令：
- 必需环境变量：
- 输入 / 输出：
- 核心对象：
- 执行链：
- Invoke / Stream 行为：
- 我做的修改：
- 遇到的错误与原因：
- 这个示例适合解决：
- 这个示例不适合解决：
- 下一步：
```

完整学习打卡项：

- [ ] 我能解释 `schema.Message` 和 Tool Call 的基本结构。
- [ ] 我能独立创建 ChatModel，并完成普通和流式调用。
- [ ] 我能用 Prompt Template 构造多条消息。
- [ ] 我能解释 Component 的输入输出类型。
- [ ] 我能分别构建一个 Chain、Graph 和 Workflow。
- [ ] 我能给 Graph 添加分支、状态或中断恢复。
- [ ] 我能创建 Tool，并让 ReAct Agent 调用它。
- [ ] 我能使用 Runner、Session 和 Middleware。
- [ ] 我能实现一次审批或补充信息后继续执行。
- [ ] 我能为执行过程添加日志、Trace 和超时取消。
- [ ] 我能把示例中的模型、工具和存储替换成自己的实现。
- [ ] 我能说明从示例代码进入生产环境还缺少哪些安全与可靠性能力。

## 五、常见问题

### 5.1 `go: go.mod requires go >= ...`

本地 Go 版本低于当前 module 的要求。升级 Go，或确认 `go env GOTOOLCHAIN` 为 `auto`。注意 Deer-Go 要求的版本高于根 module。

### 5.2 配置了模型名但程序仍报空值

先查看该示例源码究竟读取 `OPENAI_MODEL`、`OPENAI_MODEL_NAME` 还是 `OPENAI_MODEL_ID`：

```bash
rg 'os.Getenv' path/to/example
```

这个仓库中的环境变量尚未完全统一，不能假定一套最小变量适用于所有示例。

### 5.3 返回 401、403 或 model not found

依次检查 API Key、Base URL、模型名/Endpoint ID、账户权限和区域。不要打印完整 Key；日志中只保留前后少量字符用于辨认。

### 5.4 在根目录找不到子模块的包

包含独立 `go.mod` 的目录属于另一个 module。先 `cd` 到该目录，再执行 `go mod download`、`go run` 或 `go test`。

如果随后出现 `missing go.sum entry` 或 `updates to go.mod needed`，在该子模块运行 `go mod tidy`。执行后应检查依赖文件差异，避免无意中提交与学习无关的版本升级。

### 5.5 Redis、MCP 或浏览器工具连接失败

先确认示例所需服务已启动，再用服务自己的客户端验证连通性。例如先执行 `redis-cli ping`，不要一开始就在 Agent 的多层调用中排查。

### 5.6 流式输出中途报错

流的创建成功不代表整个请求成功。错误可能在后续 `Recv` 时出现，因此需要一直处理到 `io.EOF`，并确保关闭 StreamReader。

### 5.7 示例可以运行，但不适合直接上线

示例主要用于演示框架能力。生产化通常还需要：认证授权、输入校验、工具权限隔离、敏感信息过滤、限流、超时、重试、幂等、持久化、审计、指标、成本控制和安全测试。

## 六、延伸阅读

- [中文项目索引](./README.zh_CN.md)
- [更细的示例索引](./COOKBOOK.md)
- [ChatWithEino 教程入口](./quickstart/chatwitheino/docs/README.md)
- [Eino 框架](https://github.com/cloudwego/eino)
- [Eino 扩展组件](https://github.com/cloudwego/eino-ext)
- [Eino 中文官方文档](https://www.cloudwego.io/zh/docs/eino/)
- [贡献指南](./CONTRIBUTING.md)

## 七、安全说明

不要提交真实 API Key。发现项目本身的潜在安全问题时，请通过字节跳动[安全中心](https://security.bytedance.com/src)或[漏洞报告邮箱](mailto:sec@bytedance.com)私下报告，不要创建公开 Issue。

本项目依据 [Apache-2.0 许可证](./LICENSE-APACHE) 授权。
