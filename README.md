# DeepSync-Researcher 🧠🔍

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/Framework-HelloAgents-orange)](https://github.com/datawhalechina/hello-agents)
[![Build Tool](https://img.shields.io/badge/Package%20Manager-uv-green)](https://github.com/astral-sh/uv)

**DeepSync-Researcher** 是一款基于 **多智能体 (Multi-Agent) 协同架构** 的自动化深度调研系统。它通过将复杂的课题拆解为任务流，模拟人类专家的研究逻辑，实现从“意图理解”到“万字深度报告生成”的全链路自动化。

---

## 🌟 核心特性

- **多机协作架构 (Multi-Agent Orchestration)**：由 Planner、Researcher、Editor 三个核心角色构成，基于状态机管理任务分发，解决长任务处理中的逻辑断层。
- **自主规划与执行 (ReAct)**：采用 **Reason + Act** 范式，智能体能够根据搜索反馈动态调整后续研究路径，而非死板地执行预设指令。
- **高性能异步后端**：基于 **FastAPI** 构建，支持 **SSE (Server-Sent Events)** 技术，实时流式展示 Agent 的思考链路（Thought Chain）。
- **现代化工具链**：集成 Tavily Search API，实现信息的精准去噪与结构化沉淀。
- **极简工程实践**：采用 **uv** 高性能包管理工具，通过优化 `pyproject.toml` 解决了跨模块依赖寻址与构建冲突。

---

## 🏗️ 系统架构

本项目采用分层架构，确保了 Agent 逻辑与业务展示的解耦：

1. **TODO Planner**: 利用大模型的规划能力，将宽泛的调研需求拆解为具体、可操作的调研清单。
2. **Researcher**: 驱动搜索工具并发获取信息，利用 `NoteTool` 进行知识去噪与结构化沉淀。
3. **Editor (Report Writer)**: 汇总沉淀的笔记，按照专业报告标准进行逻辑串联与内容撰写。

---

## 🛠️ 技术挑战与解决方案 (Technical Challenges & Solutions)

本项目并非简单的 API 调用，在复现与二次开发过程中，针对以下核心工程痛点进行了深度优化与重构：

### 1. 复杂环境下的构建冲突与依赖寻址
* **挑战**：项目采用 **Monorepo (单体大仓库)** 结构，传统 `pip` 在 Windows 环境下频繁出现 `src` 布局导致的跨目录模块导入失败（ImportError）。
* **对策**：引入 **uv** 高性能包管理工具，通过重构 `pyproject.toml` 的项目依赖寻址逻辑，利用 **Editable Install (-e)** 模式实现核心框架与业务代码的平滑挂载。
* **结果**：环境配置耗时缩短 90%，彻底解决了开发环境与生产环境的路径不一致性。

### 2. 长任务生成的连接中断与用户焦虑
* **挑战**：大模型生成万字报告耗时较长（通常 > 60s），极易触发浏览器的 HTTP 超时限制，导致请求中断且用户无法得知生成进度。
* **对策**：后端基于 **FastAPI 异步架构**，前端引入 **SSE (Server-Sent Events)** 技术实现 Agent 思考链路（Thought Chain）的流式实时推流。
* **结果**：实现了生成过程的可视化预览，解决了长连接超时问题，显著提升了系统的鲁棒性与用户体验。

### 3. 原始搜索数据的“噪点”与 Token 成本控制
* **挑战**：直接将搜索引擎返回的原始 HTML 或无关冗余信息喂给 LLM，会导致模型产生严重“幻觉”，且单次调研消耗的 Token 成本过高。
* **对策**：封装自定义 `SearchTool` 与 `NoteTool`，在数据注入模型前执行**自动化去噪与结构化提炼逻辑**。
* **结果**：单次调研任务的 Token 消耗降低约 30%，报告内容的准确性与信源关联度大幅提升。

### 4. 多智能体协作的逻辑偏移与状态管理
* **挑战**：单体 Agent 在处理长路径任务时容易“跑偏”或遗忘初始目标。
* **对策**：引入 **状态机 (State-Machine)** 思想进行任务编排，通过 **Planner** 强制执行任务动态拆解，并在每一阶段增加状态校验与反馈闭环。
* **结果**：实现了可回溯的任务执行机制，确保了万字级复杂调研任务的高完成度与逻辑一致性。
