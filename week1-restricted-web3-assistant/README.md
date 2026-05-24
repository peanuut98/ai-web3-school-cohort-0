# Restricted Web3 Transaction Assistant

> 受限型 Web3 交易检查与记录助手 · Week 1 设计文档
>
> AI 只做解释 / 规划 / 检查 / 文档整理，**所有钱包连接、签名、授权、转账、部署、写入合约的动作一律由用户本人在钱包中人工确认**。本文档不含任何真实自动交易代码。

---

## 1. 项目要解决的问题

Web3 初学者在第一次做链上操作时，最常踩的坑不是「概念太难」，而是**信息密度太高 + 错一步就不可逆**。具体表现：

- **签名弹窗看不懂**。MetaMask / Rabby 弹一段 hex 或 typed data，新手常常下意识点 Confirm，但很多钓鱼是把「授权对方转走我所有资产」伪装成普通签名。
- **交易字段看不懂**。Etherscan / BaseScan 上 From / To / Value / Gas / Input Data 一堆字段，新手不知道哪些必须看、哪些可以先跳过。
- **测试网和主网混淆**。RPC 切换后没注意，可能把测试操作发到主网，或反之。
- **「钱包说成功」≠「链上成功」**。新手常以钱包 UI 的「Confirmed」为准，没有去区块浏览器二次验证。
- **敏感信息泄露**。在调试过程中可能把私钥 / 助记词 / API Key / `.env` 截图发给客服 / 贴到聊天 / 提交到 GitHub。这一类错误是**一次性致命的**，没有补救机会。

本 workflow 的目标分三段：

- **操作前**：让用户理解风险，得到一份「这次操作我必须人工确认什么」的清单。
- **操作中**：让用户在钱包弹窗里能逐字段判断要不要点 Confirm。
- **操作后**：让用户在区块浏览器上验证结果，并把记录整理成可复盘的文档（不含敏感信息）。

整套流程的设计前提：**AI 永远不接触私钥、不替用户签名、不绕过任何人工确认。**

---

## 2. Workflow 总览

```
用户提出 Web3 操作需求
        ↓
AI 拆解操作步骤，并生成风险检查清单
        ↓
用户检查网络、地址、金额、合约参数
        ↓
用户在钱包中人工确认连接 / 签名 / 授权 / 交易
        ↓
链上交易被广播
        ↓
用户复制交易哈希或合约地址
        ↓
AI 指导用户使用区块浏览器验证结果
        ↓
AI 帮助用户整理 GitHub / Notion 提交记录
        ↓
用户检查文档中无敏感信息后提交
```

注意每一个「钱包确认」节点都是**用户独占**的。AI 只在前后两端工作 — 进钱包前帮你列清单，出钱包后帮你验证 + 写记录。

---

## 3. 用户输入是什么

### ✅ 可以输入

**示例一（操作前）：**
> 我准备在 Base Sepolia 测试网上部署一个 HelloWorld 合约，请帮我检查步骤，并告诉我哪些地方必须人工确认。

**示例二（操作后）：**
> 我已经完成了一笔测试网交易，交易哈希是 0x123...abc，请帮我整理记录，并告诉我如何验证它是否成功。

其他可以输入的内容：

- 钱包地址（公开信息）
- 交易哈希、合约地址、区块号
- 区块浏览器链接
- 报错截图（前提：截图里没有助记词输入框 / 私钥栏 / API Key）
- 操作目标的自然语言描述（"我想 swap 一点测试网 USDC"）

### ❌ 绝对不要输入

- **私钥**（任何形态：64 位 hex、JSON keystore 文件、Show Private Key 截图）
- **助记词 / Seed Phrase**（12 / 24 个英文单词，不论是文本、截图、还是「分两次粘贴一半」）
- **API Key / Secret / Token**（Alchemy / Infura / OpenSea 等）
- **`.env` 文件、`secrets.json`、`keystore.json` 这类配置文件的完整内容**
- **真实资产截图中包含助记词 / 私钥 / 二维码恢复码的部分**
- **可以倒推出私钥的任何信息**（比如「我助记词第 3 个单词是 X」）

> 凡是要求你输入助记词 / 私钥的「客服 / AI 助手 / 验证页面」，**100% 是诈骗**。包括 Claude。

---

## 4. AI / Agent 可以做什么

| 能力 | 具体做什么 |
| --- | --- |
| **解释** | 解释交易哈希、Gas、合约地址、签名、授权、Nonce、Block、Status 等概念 |
| **规划** | 把一个链上操作（如「部署合约 + 验证」）拆解成有序的步骤清单 |
| **检查** | 检查用户提供的网络名称是否一致、地址格式是否合法（0x + 40 hex）、交易记录是否完整 |
| **生成草稿** | 生成 README、交易记录、风险提示文案、验证步骤 checklist |
| **结果验证指导** | 指导用户在 BaseScan / Etherscan 查看 Status / Block / Gas Used / Contract Created |
| **风险提醒** | 在每一步操作前提醒：不要把私钥、助记词、API Key、token 或 `.env` 文件粘到对话里或提交到 GitHub |

---

## 5. AI / Agent 不能做什么

| 限制 | 含义 |
| --- | --- |
| **不能读取、保存、请求私钥或助记词** | 即使用户主动给，也必须拒绝并提示「立刻轮换 / 视为已泄露」 |
| **不能读取、保存 API Key、token 或 `.env` 文件** | 包括「帮我看看 .env 写得对不对」这种请求 — 让用户先脱敏再发 |
| **不能代替用户点击钱包确认** | 钱包弹窗内的 Confirm / Reject 只能由用户本人鼠标 / 触屏点击 |
| **不能绕过用户完成人工签名** | 不调用任何 `signMessage` / `signTypedData` / `eth_sign` 类接口 |
| **不能自动转账、授权、Swap、Mint、Stake** | 不调用任何 ERC-20 transfer / approve / Uniswap swap / NFT mint 类的 write function |
| **不能自动部署合约或调用 Write Function** | Remix / Hardhat / Foundry 的部署命令必须由用户本人在自己的终端 / 浏览器里执行 |

实现层面：本助手**不连接任何钱包、不持有任何 RPC 写入凭证、不安装任何能签名的 SDK**。一切「写入链上」的能力都被刻意排除在系统之外。

---

## 6. Web3 工具与链上步骤

### 工具表

| 工具 | 在本 workflow 中的角色 |
| --- | --- |
| **MetaMask / Rabby Wallet** | 用户连接钱包、切换网络、人工确认签名的入口；AI 不与之直接交互 |
| **Remix** | 用户编写、编译和部署测试合约的浏览器 IDE；部署按钮由用户本人点击 |
| **Base Sepolia Faucet** | 用户领取测试网 ETH 的水龙头（如官方 Base 文档中链接） |
| **BaseScan / Etherscan** | 用户查询交易哈希、合约地址、交易状态的区块浏览器（只读） |
| **GitHub** | 提交 workflow 设计文档、学习记录、复盘笔记的版本仓库 |

### 链上步骤（以「部署测试合约」为例）

1. 用户准备操作目标，例如「在 Base Sepolia 部署一个 HelloWorld 合约」。
2. AI 生成操作前检查清单（网络 / 余额 / Solidity 版本 / 编译产物 / 验证方案）。
3. 用户打开钱包并**人工确认当前网络是 Base Sepolia**（不是主网，不是 Ethereum Sepolia）。
4. 用户在 Remix 或 DApp 中发起操作（点击 Deploy）。
5. 钱包弹窗出现，显示 To（合约创建时为空）/ Value / 预估 Gas fee / 数据 hex。
6. 用户**人工检查并确认**网络、Gas 量级、调用数据是否符合预期，然后点 Confirm。
7. 交易被广播到 Base Sepolia 节点池，等待打包。
8. 用户从钱包 / Remix 复制 **Tx Hash**。
9. 用户在 https://sepolia.basescan.org 搜 Tx Hash，验证 Status = Success、Block 已确认、Contract Created 字段出现 Contract Address。
10. AI 帮助整理记录文档：把 Tx Hash、Contract Address、部署时间、Gas 用量贴到 README，并提醒用户提交前再扫一遍是否泄露敏感信息。

---

## 7. 必须人工确认的步骤

下面这些动作 AI **只能解释字段含义和提供检查清单**，无论用户怎么要求都不能代为执行：

- **连接钱包**必须人工确认（每次连接都暴露你的地址，授权范围由你决定）
- **签名消息**必须人工确认（包括 Sign-In with Ethereum、permit、setApprovalForAll 等所有 typed data）
- **授权代币**必须人工确认（approve / increaseAllowance — 这是最常见的钓鱼入口）
- **转账**必须人工确认（ETH 转账、ERC-20 transfer、ERC-721 transferFrom）
- **合约部署或写入**必须人工确认（Remix 的 Deploy、合约 Write Function 调用）
- **修改权限设置**必须人工确认（owner / admin / 角色变更、proxy 升级）

AI 在这些步骤之前的角色：解释「这个签名实际授权了什么」、「这个 approve 的额度是不是 unlimited」、「这个合约是不是 verified」。但**点 Confirm 这一下，永远是用户**。

---

## 8. 输入和输出示例

### 示例一：部署合约前检查

**用户输入：**
> 我准备在 Base Sepolia 测试网上部署一个 HelloWorld 合约，请帮我检查步骤，并告诉我哪些地方必须人工确认。

**AI 输出：**

操作前检查清单：

1. **网络确认**
   - 钱包顶部网络应为 **Base Sepolia**（不是 Base Mainnet，不是 Ethereum Sepolia）
   - Chain ID = 84532
2. **余额确认**
   - From 地址至少有 ~0.01 Base Sepolia ETH（部署合约通常 Gas 远小于此，留余量）
   - 如不足，从官方 faucet 领取
3. **代码确认**
   - HelloWorld.sol 已在 Remix 编译通过，无 warning 标红
   - Solidity 版本与 pragma 一致
4. **部署参数确认**
   - 选择正确的 Environment：**Injected Provider - Rabby / MetaMask**（不是 Remix VM）
   - 选择正确的 Account（你刚才在钱包里切换的那个测试地址）
   - constructor 参数（如果有）核对一遍

🔴 **必须人工确认的步骤**：

- ✋ 钱包弹窗出现时，必须你本人查看：网络是否 Base Sepolia / Gas fee 是否合理 / Data 是否对应你的合约 bytecode
- ✋ 点 Confirm 这一下只能你本人点
- ✋ 部署完成后**自己**复制 Tx Hash + Contract Address，AI 不替你抓

⚠️ **不要做**：
- 不要把 Remix 里的 .env / 私钥导入功能贴出来给我看
- 不要把助记词输入到任何「合约部署助手」类网页

下一步告诉我 Tx Hash，我帮你看上链结果。

---

### 示例二：交易完成后记录

**用户输入：**
> 我已经完成了一笔测试网交易，交易哈希是 0x123...abc，请帮我整理记录，并告诉我如何验证它是否成功。

**AI 输出：**

验证步骤（你本人在浏览器里做）：

1. 打开 https://sepolia.basescan.org
2. 搜索框粘贴 `0x123...abc`
3. 检查以下字段：
   - **Status**：应为 ✅ Success（如果是 Pending，等几秒；Failed 则交易失败但 Gas 仍扣除）
   - **Block**：应有具体区块号，且页面显示「N Block Confirmations」
   - **Timestamp**：和你点 Confirm 的时间相差几秒至几十秒
   - **From**：你本人的测试地址
   - **To**：你期望的接收地址或合约地址
   - **Value**：你设置的金额（测试网 ETH，无价值）
   - **Gas Used / Txn Fee**：普通转账约 21000 gas；合约调用更多
   - 如果是合约部署：应看到「Contract Created: 0x...」字段

记录草稿（你确认无敏感信息后再提交）：

```markdown
## 测试网交易记录

- 网络：Base Sepolia
- Tx Hash：0x123...abc
- 类型：（ETH 转账 / 合约部署 / Token approve）
- From：0x...（你的测试地址）
- To：0x...
- Value：x.xxx Base Sepolia ETH
- Block：xxxxxx
- Status：Success
- Gas Used：xxxxx
- Txn Fee：x.xxx ETH
- BaseScan：https://sepolia.basescan.org/tx/0x123...abc
- 备注：（自己写一句这笔交易的目的）
```

⚠️ 提交 GitHub 前自查：
- 这份记录里**只有公开信息**（地址、Tx Hash、合约地址都是公开账本数据）
- 没有 `.env` 文件
- 没有助记词 / 私钥 / API Key 的任何片段
- 截图（如果有）里没有钱包的「Show Private Key」「Reveal Seed Phrase」面板
- 截图里 Rabby / MetaMask 的余额、网络、地址栏没有问题再贴

---

## 9. 如何验证执行结果

链上验证清单（**用户在区块浏览器上执行**，AI 只指导）：

- [ ] **Status** 是 Success（不是 Pending 也不是 Failed）
- [ ] **Block** 有确认数（建议 ≥ 1 block confirmation 再认为成功；主网建议 ≥ 12）
- [ ] **From** 是你预期的发送地址
- [ ] **To** 是你预期的接收地址 / 合约地址
- [ ] **Value** 数额和位数正确（特别注意 wei 视图下的 0 数量）
- [ ] **Gas Used** 在合理范围（普通转账 ~21000；合约调用按合约复杂度）
- [ ] **Txn Fee** 量级合理，没有异常高
- [ ] 如果是合约部署：页面出现 **Contract Created: 0x...**，记下这个 Contract Address
- [ ] 如果是合约调用：检查 **Input Data** 解码后的方法名是否符合预期（比如确实是 `transfer(...)` 而不是 `setApprovalForAll(...)`)

GitHub 提交前自查（**用户在本地执行**）：

- [ ] 文件里**没有**任何 64 位 hex 的疑似私钥
- [ ] 文件里**没有** 12 / 24 个英文单词的助记词块
- [ ] 文件里**没有** API Key / Secret / Token 字符串
- [ ] **没有** `.env`、`secrets.json`、`keystore.json` 被 staged（`git status` 检查）
- [ ] `.gitignore` 已包含上述敏感文件类型
- [ ] 截图里没有钱包敏感面板

---

## 10. 风险和限制

1. **AI 可能解释错误**。本助手生成的概念解释、字段含义、检查清单都基于训练时的知识，可能与最新协议规范、特定 L2 的细节有出入。**用户应把 AI 的输出当作起点，不是终点** — 关键操作前对照官方文档。
2. **AI 不能判断所有合约是否安全**。即使一个合约在 BaseScan 上 verified，也不代表它没有后门、没有可升级 admin、没有 rug 风险。AI 可以提示风险维度，但不能给「绝对安全」的结论。
3. **用户可能泄露敏感信息**。即使本 workflow 反复提示，用户在调试压力下仍可能粘贴 `.env` 或私钥到对话框。**一旦发生，必须立刻视为泄露 — 把资产转走 + 不再使用这组凭据 + 轮换所有相关 API Key**。AI 的提示是软约束，不是硬阻断。
4. **测试网和主网可能混淆**。Chain ID 切错、RPC 配错、Faucet 误导都可能导致测试操作进了主网（或反之）。AI 能基于用户描述提示，但不能强制校验钱包当前实际网络 — 这一步永远是用户自己看钱包顶栏。
5. **自动化能力有限**。本助手**故意不实现**任何自动签名 / 自动授权 / 自动转账 / 自动部署。这意味着即使在「批量小额测试」这类场景下，每一笔仍需要用户人工确认。这是设计取舍，不是缺陷 — 用安全换效率。

---

## 11. 总结

Restricted Web3 Transaction Assistant 的定位很明确：

- **AI 是解释助手** — 把签名弹窗、交易字段、合约调用解码成用户能读懂的语言。
- **AI 是流程助手** — 把「部署一个合约」拆成 10 步清单，每步说清楚谁做、做什么、怎么验证。
- **AI 是检查助手** — 在用户点 Confirm 前帮 Ta 列出该看的字段；在用户提交 GitHub 前帮 Ta 扫一遍敏感信息。
- **AI 是文档助手** — 把交易记录整理成结构化的、可以复盘的、不含敏感信息的笔记。

**但 AI 不是资产控制者。** 所有涉及钱包连接、签名、授权、转账、合约部署、合约写入的动作，都必须由用户本人在钱包中人工确认。

这条边界不是因为 AI 做不到 — 技术上它做得到 — 而是因为**做到了就意味着用户失去最后一道防线**。Web3 的核心承诺是「用户拥有自己的资产」，而拥有的代价就是「最后一击必须自己点」。本助手把这条边界设计在系统层面：不持有私钥、不连接钱包、不调用任何 write function。

宁可少一点自动化，也要保住这条边界。
