# Agent Harness

自主 AI Agent 运行时与技能自进化框架。

这个项目实现了一套完整的 LLM Coding Agent Harness：模型负责推理，Harness 负责工具调用、会话持久化、上下文治理、安全审批、跨入口消息路由、执行环境抽象，以及基于评估反馈的技能优化。

## 核心能力

- Agent Loop：基于 OpenAI-compatible `tool_calls` 协议实现模型调用、工具执行、结果回填和多轮 continuation。
- Tool Registry：通过 `ToolEntry` 和 `ToolRegistry` 管理工具 schema、toolset 和 handler，支持低耦合工具扩展。
- Session Store：使用 SQLite 持久化 sessions/messages/tool calls，并通过 WAL 模式提升并发读写稳定性。
- Context Governance：实现粗粒度 token 估算、旧工具结果裁剪、中段摘要压缩和 system prompt 多来源组装。
- Memory & Skills：使用 `MEMORY.md`、`USER.md` 和 `skills/*/SKILL.md` 管理长期记忆与可复用技能。
- Permission System：在终端执行前检测递归删除、磁盘写入、危险 SQL、远程脚本管道执行等高风险命令。
- Subagent Delegation：为子任务创建隔离 messages、隔离 system prompt 和受限工具集，降低主上下文污染。
- Gateway Runtime：抽象 `MessageEvent`、`SessionSource`、`BasePlatformAdapter` 和 `GatewayRunner`，让不同入口复用同一套 Agent Loop。
- Execution Backends：抽象 Local、Docker、SSH 执行后端，统一命令执行、CWD 跟踪、环境快照和超时处理。
- MCP / Browser / Multimodal：提供 MCP 工具桥接、模拟浏览器自动化、视觉分析、TTS/STT 管道示例。
- Self-Evolution：把对话经验沉淀为 memory/skill，将轨迹转成训练数据，并通过评估反馈自动优化技能文件。

## 自我进化闭环

项目里的“自我进化”不是直接训练模型权重，而是让 Agent 的行为层持续变强：

```text
对话执行
  -> 工具调用与结果回写
  -> 后台审视用户偏好、错误恢复路径和复杂工作流
  -> 写入 MEMORY.md / USER.md / SKILL.md
  -> 下次对话按需注入记忆与技能
  -> 轨迹采集、评估、反馈驱动改写技能
  -> 约束通过后部署新版本
```

关键组件：

- `BackgroundReviewer`：分析对话中是否出现值得复用的复杂工作流。
- `convert_to_trajectory()`：将 OpenAI messages 转换为 ShareGPT-like trajectory。
- `SyntheticDatasetBuilder`：根据 skill 文本生成评估用例。
- `FitnessScore`：按 correctness、procedure_following、conciseness 计算复合分。
- `SkillOptimizer`：执行 feedback -> mutate -> evaluate -> select 循环。
- `ConstraintValidator`：检查技能大小、增长比例、非空和 Markdown 结构，防止劣化版本部署。
- `evolve_skill()`：完整的加载、生成数据、评估、优化、holdout 验证、备份和部署流程。

## 快速开始

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Windows PowerShell：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env
```

编辑 `.env`：

```env
OPENAI_API_KEY=sk-xxx
OPENAI_BASE_URL=https://openrouter.ai/api/v1
MODEL=anthropic/claude-sonnet-4
```

启动 CLI：

```bash
python agent_harness.py
```

运行内置测试：

```bash
python agent_harness.py --test
pytest -q
```

优化某个技能：

```bash
python agent_harness.py --evolve <skill_name>
```

## 目录结构

```text
Agent_Harness/
├── agent_harness.py          # 完整 Agent Harness 实现
├── docs/
│   └── ARCHITECTURE.md       # 架构说明
├── tests/
│   └── test_agent_harness.py # 核心行为测试
├── .env.example              # 环境变量模板
├── requirements.txt          # Python 依赖
└── README.md
```

## 配置说明

配置优先级：

```text
环境变量 > HERMES_HOME/config.yaml > DEFAULT_CONFIG
```

常用变量：

- `OPENAI_API_KEY`：模型服务 API Key。
- `OPENAI_BASE_URL`：OpenAI-compatible API 地址。
- `MODEL`：默认模型名称。
- `HERMES_HOME`：profile 目录，保存 memory、skills、allowlist、cache 等运行时状态。

## 适合展示的工程点

- 从零实现 LLM 工具调用闭环和工具注册分发。
- 用 SQLite 结构化管理对话和工具调用历史。
- 用 memory/skill 文件体系解决长会话知识沉淀问题。
- 用 Gateway 和 Adapter 模式复用同一套 Agent Loop。
- 用执行后端抽象隔离本地、容器和远程运行环境。
- 用 LLM-as-judge、约束门控和 feedback-driven mutation 实现技能自优化。

## License

MIT License. See [LICENSE](./LICENSE).
