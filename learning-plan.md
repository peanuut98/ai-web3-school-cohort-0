# Learning Plan — 8 Weeks

> 基于 [profile.md](./profile.md)。Week 1 Day 1 = 2026-05-18（周一）。
> 节奏：每天 2 小时 = 30min Handbook 阅读 + 60min 实验或笔记 + 30min 打卡和复盘。

## 📐 总体思路

四象限路径：
1. **Web3 基础打底**（Week 1–2）— 学员的最大短板
2. **Vibe Coding 武装**（Week 3）— 把 AI 工具变成「代码外骨骼」
3. **AI × Web3 Bridge**（Week 4–6）— 进入真正的交叉问题
4. **Hackathon 冲刺**（Week 7–8）— 把知识打包成原型

---

## Week 1 — Web3 基础 Part 1：账户与签名

| Day | Handbook 章节 | 实验 / 输出 |
| --- | --- | --- |
| Mon | [Network](https://aiweb3.school/zh/handbook/) — 区块、共识、L1/L2 | Daily note：用一段话向我妈解释「区块」 |
| Tue | Cryptography — 哈希、公私钥、签名 | 在 https://emn178.github.io/online-tools/sha256.html 算两个字符串的哈希，记录直觉 |
| Wed | Wallet — 钱包是身份和签名入口 | 装 MetaMask（如未装），切到 Sepolia 测试网 |
| Thu | Wallet 续 — Seed phrase / Keystore / 派生路径 | **不要**把助记词写进任何文件 |
| Fri | 复盘 + Handbook feedback Round 1 | 整理本周读 Handbook 时不清楚的 3 个点，写到 `handbook-feedback/` |
| Sat | Network 续 — RPC、Gas、Nonce | 用 Etherscan 翻一笔交易，把字段含义记下来 |
| Sun | 休息 or 缓冲 | （Optional）|

**Week 1 里程碑**：能用一句话解释「钱包做了什么」「一笔交易里有哪些字段」「私钥泄露的后果」。

## Week 2 — Web3 基础 Part 2：合约与账户抽象

| Day | Handbook 章节 | 实验 / 输出 |
| --- | --- | --- |
| Mon | Smart Contract — 部署、调用、状态 | 在 https://remix.ethereum.org 跑一个 Hello World 合约 |
| Tue | Smart Contract — 事件、ABI、storage vs memory | 用 Etherscan Read Contract 调一个公开合约的 view 函数 |
| Wed | Account Abstraction — 为什么 EOA 不够 | 读 [ERC-4337 一图流](https://www.erc4337.io) |
| Thu | AA 续 — Smart Account / Session Key / Paymaster | 想象一个 Agent 钱包的权限模型，画在 Excalidraw |
| Fri | DeFi 速览 — 资产、流动性、借贷 | （只看概念，不实操资产）|
| Sat | Indexing + Oracle 一览 | 列出 The Graph 和 Chainlink 各解决什么问题 |
| Sun | 周复盘 + 更新 profile 里的「能解释」清单 | |

**Week 2 里程碑**：能解释「为什么 Agent 需要 Smart Account」。

## Week 3 — Vibe Coding 武装周

| Day | 主题 | 输出 |
| --- | --- | --- |
| Mon | Handbook: Vibe Coding 章节 | 装 Claude Code（已装）+ Cursor，跑 Hello World |
| Tue | Handbook: MCP — 模型如何接工具 | 给 Claude Code 接一个最小 MCP（filesystem 就行）|
| Wed | Handbook: Frameworks 速览 | 列出 LangChain / LangGraph / Agents SDK 各自定位 |
| Thu | 实操：让 Claude Code 帮你跑一段 ethers.js | `experiments/` 下放一个能 `node` 执行的脚本，读 Sepolia 余额 |
| Fri | Handbook feedback Round 2 + 实验复盘 | |
| Sat | 选一个 WCB 任务用 Vibe Coding 做 | |
| Sun | 缓冲 | |

**Week 3 里程碑**：能用 Claude Code 跑一段读链脚本，看懂它生成的代码在干什么。

## Week 4 — AI × Web3 Bridge Part 1

聚焦：**Chain-aware Context** + **Web3 Tool Use** + **Agent Workflow**。
- 每天读一节 + 写一段产品角度的批注（你的强项）。
- Friday：选一个 Bridge 概念，写一篇 800 字的产品研究笔记，发到 `submissions/`。

## Week 5 — AI × Web3 Bridge Part 2

聚焦：**Agent Wallet** + **Machine Payment** + **Settlement & Escrow**。
- 中期产出：调研「现有 Agent 钱包项目对比」（Coinbase AgentKit / Crossmint / Privy / Safe）。

## Week 6 — AI × Web3 Bridge Part 3

聚焦：**Agent Identity** + **Agent Trust & Reputation** + **Verifiable AI** + **AI Security**。
- 周末：开始 Hackathon 选题。Open Track 优先，挑一个你愿意每天想 1 小时的问题。

## Week 7 — Hackathon Prep

- Mon–Wed：把选题用一页纸说清楚（问题、用户、最小路径、技术栈）。
- Thu–Sun：搭原型骨架，Claude Code 写代码，你审产品和流程。

## Week 8 — Hackathon Demo

- 完成 Demo + 录屏 + README。
- 提交到 WCB 平台 + cohort Demo Day。

---

## 🔁 每周固定动作

- **每周五**：Handbook feedback 至少 1 条到 `handbook-feedback/`
- **每周日**：更新 `profile.md` 里的「能解释」清单 + 在 `learning-plan.md` 末尾追加一段周复盘
- **每两周**：检查这个计划是否需要调整（不要硬扛错的计划）

## 📌 调整原则

> 计划是给现实让路的工具，不是用来执行的合同。每两周允许重写一次本文件，但**已完成的里程碑保留**（不要装作没做）。
