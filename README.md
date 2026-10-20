# 🐼👨‍🍳 Panda Chef · Scoot Wang

**AI 研发提效工程师** —— 企业级 Java 微服务 × 多 Agent 编排 × 规范输出

写企业级 Java 微服务，也把多个 AI Agent 编排成可复现的研发流水线，再把做法写成团队能照着执行的规范文档。
用 AI 从 0 到 1 做过桌面应用，也在持续运营一个有真实用户的 AI 科普公众号。

`Java` `Spring Cloud` `TypeScript` `Electron` `Python` `AI Agents` `MCP` `Docker`

📮 **scootwang@163.com**

---

## 🚀 Vibe Coding：我用 AI 亲手做出来的东西

> Vibe coding = 用自然语言驱动 AI 写代码。下面这些不是教程里的 demo，是我每天在跑、还在持续维护的工程。

### 1️⃣ IDEA-Air —— 从 0 到 1 的 AI 原生 IDE ⭐

**[github.com/panda-chef/IDEA-Air](https://github.com/panda-chef/IDEA-Air)** ｜ Electron · React 19 · TypeScript · 8500 行 · MIT

**问题**：AI 写代码已经便宜到几乎不算成本，瓶颈转移到了它改完之后——看懂改了什么、把项目跑起来、确认没问题再提交。为了这几个动作去开一套 IntelliJ IDEA（空闲常驻 600MB–1.5GB、启动 10–20 秒、索引一跑 CPU 就喘）不划算。

**做法**：把整条链路收进一个窗口 —— 打开项目 → 让 agent 改 → 看双栏 diff → 底部终端跑起来 → 提交。

**已经跑通并验证的部分**：

- Monaco 编辑器、多标签、脏标记、快捷键保存
- **非 UTF-8（如 GBK）文件自动识别并标记只读** —— 宁可只读，也不损坏原文件
- 底部控制台：Agent 会话 / Git 操作 / 文件变更 三类日志，外加各进程内存、CPU、会话数
- **安全边界**：渲染进程 `sandbox: true` + `contextIsolation: true`，所有文件访问经 preload 暴露的 API 走主进程，路径在主进程做工作区白名单校验
- **Agent 会话层**：Claude Code（官方 Agent SDK）+ Codex（`app-server`）双后端；工具调用需要用户拍板时把请求抛到界面，而不是替用户决定；会话支持从历史复活

**技术栈**：Electron · React 19 · TypeScript · Monaco · xterm.js + node-pty（ConPTY）· electron-vite

**踩过的坑**：`ELECTRON_RUN_AS_NODE` 环境变量会让 Electron 以 Node 模式启动，窗口根本不创建——排查了半天才发现不是代码问题。现在主进程 spawn 子进程时会主动剔除这个变量，避免传染给 agent CLI。

---

### 2️⃣ agent-loop —— 把「会用 AI」变成「编排 AI」 ⭐

**[github.com/panda-chef/agent-loop](https://github.com/panda-chef/agent-loop)** ｜ Python · 600 行 · MIT

**问题**：一次会话里让 AI 改代码，改完自己看、自己跑、自己判断能不能提交。这样做有三个硬伤——不可复现（换个环境就崩）、不可审计（会话一关，试错过程全没了）、自己审自己（写和审是同一个模型，盲区完全重合）。

**做法**：把链路拆成 5 个独立进程节点，节点之间只通过一份 JSON 状态通信。

```
develop ──→ verify ──pass──→ push ──→ review ──approve──→ merge
  ↑            │                        │
  └──fail──────┘        └────reject──────┘
 (Claude Code)  (构建门禁)      (Codex 独立审)      (人工闸门)
```

**四个我认为最要紧的设计决策**：

| 决策 | 为什么 |
|---|---|
| **制作者与检查者必须是不同模型** | 同一个模型写和审，盲区完全重合，等于没审。所以写代码用 Claude Code、审查用 Codex，并明确告诉审查方「你的职责是质疑，不是认可」 |
| **每个节点落 checkpoint，支持断点续跑** | Agent 任务动辄跑几十分钟，一次网络抖动就从头再来是不可接受的 |
| **条件边带重试上限** | 回边是必要的（Agent 一次写不对很正常），但没有上限的循环是账单黑洞 |
| **合并闸门默认关闭** | 前面出错代价是重跑，合并出错代价是污染主干。风险不对称的地方，就不该对称地自动化 |

还有一条：`verify` 节点跑的是**真实构建命令**，认的是退出码。一件事算不算做完了，由可执行命令决定，不由模型自评决定。

---

### 3️⃣ 内容生产管线 —— 51 篇日更背后的工程

公众号「硅基饲料」持续日更，背后的流水线是自建的：**cron 定时触发 → 抓当日热点 → AI 写稿 → 合规自查 → 生成封面 → 调用微信草稿 API → 同步 GitHub 内容库 → 静态站点自动部署**。

工程上值得说的几处：

- **1165 行 Python、零第三方依赖**对接微信公众号 API：access_token 生命周期管理、图片素材 multipart 上传、草稿 CRUD、已发布内容读取
- **自建合规检查器**：分类敏感词 + 白名单，写稿后强制自查，不通过不放行到发布环节
- **封面生成降级链**：PIL 可用时渲染完整版，不可用时回退纯 Python PNG 写入，保证流程不中断
- **踩过的坑**：微信 API 的 IP 白名单会随网络出口变化静默失效（真实出口 IP 只藏在错误码 40164 的报文里）；草稿正文里的 `<a>` 标签会被全部剥离，唯一可点的外链只有「阅读原文」字段

---

### 4️⃣ panda-hotpot —— 内容库的基建

**[github.com/panda-chef/panda-hotpot](https://github.com/panda-chef/panda-hotpot)** ｜ 51 道菜 · 1370 行工具链 · 48 个单测

每篇文章落成一个「菜品」目录：完整导语 + 参考资料 + 元信息。围绕它做了一套工具链：

- **分层设计**：`dishlib`（解析）→ `sitegen`（渲染）→ `build_menu`（编排），职责单一、可单测
- **CI 三重校验**：菜单覆盖完整性、相对链接可达性、元信息完整性；外加 48 个单元测试
- **GitHub Actions + Pages**：每次推送自动校验并部署静态站点
- **踩过的坑**：Windows 不允许目录名以点结尾——GitHub 上建得成，本地 `git clone` 直接报 `invalid path`。修完顺手把这条规则固化进目录名生成函数

---

## 💼 能力概览

**企业级 Java 微服务**
Java 8 / Spring Boot / Spring Cloud / MyBatis-Plus；多模块与多仓库拆分体系；Maven 私服与本地仓库治理；面向百万行级代码库的模块化改造。

**AI Agent 编排与落地**
自建 DAG 编排器（状态机 + 条件边 + checkpoint + 制作者-检查者分离）；打通四套 Agent 引擎（Claude Code / Codex / OpenCode / Hermes）；MCP 生态集成（项目管理、代码图谱、数据库、对象存储、SSH 等十余个服务）。

**流程规范输出**
把「人和 AI 各干什么、按什么顺序干、每步留下什么」写成可执行规范：8 阶段研发流程 + 3T 透明法则（Think / Trace / Testify）。这是我认为市场上最稀缺的一环——从个人技巧变成组织资产的那一步。

**遗留系统逆向与故障定位**
面对无文档老系统，能画出调用链、把缺陷定位到参数级，并产出修复方案与回归验证。

**全栈可见度**
后端 + 管理端前端 + 小程序 + 容器化部署 + 第三方平台对接，能独立判断问题出在哪一层。

**数据与可视化**
从 SQL 清洗、批量导入导出，到指标体系建模与驾驶舱设计，能把业务口径翻译成数据模型。

---

## 🧰 技术栈

```
后端    Java 8 · Spring Boot · Spring Cloud · MyBatis-Plus · MySQL · Redis · RabbitMQ
语言    Python（工具链/脚本） · Node.js · Bash
AI 工程 Claude Code · Codex · MCP · Agent 编排 · 上下文工程
运维    Docker · Linux(openEuler) · Nginx · Git/Gitea · CI/CD
数据    SQL 建模 · 指标口径 · FineBI
```

---

## 📮 联系我

- 📧 **scootwang@163.com**
- 🤖 开源作品：[agent-loop](https://github.com/panda-chef/agent-loop)
- 🍲 内容仓库：[panda-hotpot](https://github.com/panda-chef/panda-hotpot) · [在线菜单](https://panda-chef.github.io/panda-hotpot/)
- 📢 公众号：**硅基饲料**（ID: `AI_Feed`）—— AI 热点与深度科普

---

*把前沿技术，做成能跑的东西。*
