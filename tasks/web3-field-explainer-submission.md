# Web3 Transaction Field Explainer — Submission

> 一个最小可交互的网页 demo：输入一个 Web3 概念或交易字段，得到「一句话解释 / 具体例子 / 常见误区 / 安全提醒」四块输出。完全离线、不调用任何 API、单文件浏览器直接打开即可。

## 解决什么问题

我自己作为 AI/Web3 新手，最大的痛点不是「概念太难」，而是**Etherscan 上的字段太多、教程里术语堆叠**。比如 Day 3 在 Etherscan 上看一笔 tx，From / To / Value / Gas Used 这几个字段我能勉强读懂，但一遇到 Input Data、Internal Tx、Gas Limit 就懵。

这个 demo 想做的事很小：

- 给一个**只解释字段**的工具，不夹带任何「点这里领空投」「连接钱包」之类的链上交互。
- 每个解释固定四块结构（一句话 / 例子 / 误区 / 安全），强迫输出同时包含「直觉」和「避坑」 — 因为 Web3 的字段最容易被忽略的就是安全侧。
- 离线、零依赖，浏览器双击就能打开 — 让「学概念」这件事不要再启动一堆服务。

适合的人：第一次读 Etherscan 的人 / 刚装上 MetaMask 但搞不清字段的人 / 看签名弹窗时懒得查文档的人。

## 如何交互

1. 浏览器直接打开 `experiments/web3-field-explainer.html`，无需任何依赖、无需任何服务器。
2. 在输入框里手动输入一个概念，或点上方的预设按钮（共 10 个：Gas / Signature / Private Key / Smart Contract / Testnet / Block Explorer / From / To / Value / Gas Used）。
3. 点 **Explain** 按钮（或按回车）后，下方会显示四张卡片：
   - 一句话解释
   - 具体例子
   - 常见误区
   - 安全提醒（红色标题，强调一下）
4. 输入未匹配到的内容时，会提示没匹配并建议尝试预设词。

匹配是先严格匹配 alias，再 fallback 到子串匹配，所以输入「私钥」「private key」「PrivateKey」都能命中同一条。

## 输入输出示例

**示例 1：输入 `Gas`**

- 一句话解释：Gas 是你为让矿工 / 验证者执行你这笔交易而付的费用，单位是 ETH（以 Gwei 计价）。
- 具体例子：在 MetaMask 里发一笔 0.001 ETH 的转账，确认页除了金额还会多一个 ~0.0001 ETH 的 Gas fee — 这部分给的是网络，不是收款人。
- 常见误区：新手常以为 Gas = 平台抽成。其实 Gas 给的是出块的节点，不是某家公司。Gas 价格也不是固定的，链拥堵时会暴涨。
- 安全提醒：签名前一定看清 Gas fee 一栏。如果一笔小额转账要花几十刀 Gas，大概率是合约调用很复杂或网络异常 — 先取消，别确认。

**示例 2：输入 `signature`（或「签名」）**

- 一句话解释：签名是用私钥对一段数据生成的一段不可伪造的字符串，证明「这条消息确实是这个地址发的」。
- 具体例子：你在 OpenSea 登录时弹出的「Sign Message」窗口 — 它没有花一分钱 Gas，只是用你的私钥签了一段消息证明你拥有这个地址。
- 常见误区：「签名」≠「转账」。但有些钓鱼网站会让你签一段看起来无害的消息，里面其实是「授权对方转走你所有 NFT」的指令。签名也可以很贵。
- 安全提醒：永远看清签名内容。看不懂的 hex / typed data，不要签。尤其是涉及 setApprovalForAll、permit、increaseAllowance 这类关键词的。

**示例 3：输入 `quantum bridge`（不在词典里）**

- 输出：「没有匹配到「quantum bridge」。试试上面的预设按钮，或者输入：Gas / Signature / From / To / Value 等。」

## AI 辅助部分

这个 demo 用 Claude Code 协助完成，AI 负责的工作：

- 生成 HTML 骨架 + CSS（卡片样式、配色、移动端适配的简单版本）。
- 生成 JS 的整体结构（DATA 字典、normalize / lookup / render 函数、事件绑定）。
- 起草 10 个预设字段的「一句话 / 例子 / 误区 / 安全」四块文本初稿。
- 起草这份 submission.md 的结构。

也就是说，**所有可见的代码和文案的初稿都来自 AI**，我作为新手不需要从零写 JS。

## 人工修改 / 验证部分

我在 AI 给出的初稿基础上做了下面的检查和修改：

1. **逐条核对解释内容是否符合 Day 1–Day 3 学到的内容**
   - Gas / From / To / Value / Gas Used 这五个字段，对照 Day 1 在 Etherscan 上看的那笔交易，确认描述和我看到的一致。
   - Signature 那条特意检查了「签名 ≠ 转账，但签名也可能很贵」这一点 — 这是我希望任何新手都看到的。
2. **强化「安全提醒」一栏**
   - 私钥那条加了一句「凡是问你要私钥的，100% 是骗子。包括 Claude」 — 这是 AI 给的初稿没有的口吻。
   - Block Explorer 那条把「钓鱼站会假装 Etherscan」放在最显眼位置。
3. **测试网交互**
   - 在浏览器（Chrome）里实际打开了 HTML，点了全部 10 个预设按钮，确认每个都能正确渲染四张卡片。
   - 输入「私钥」「PRIVATE KEY」「private  key」（多空格）测大小写和空格归一化，都命中。
   - 输入「foo」确认 fallback 提示能正常显示。
   - 用 DevTools Network 面板确认页面加载后**没有发出任何外部请求**（除了 favicon 这种浏览器默认行为）。
4. **安全审查**
   - 全文 grep 了 `apiKey` / `API_KEY` / `secret` / `token` / `mnemonic` / `private` 这些关键字，确认源码里**没有任何**真实凭据 / 密钥 / .env 内容。
   - 用户输入通过 `escapeHtml` 转义后再插入 DOM，避免 XSS — 我特意输入 `<script>alert(1)</script>` 测试了一下，作为字符串显示而不是执行。

## 限制

- **解释来自一个固定的字典**，不是真正的 LLM 推理。输入未在字典里的词只会给「没匹配」提示，没办法理解相近概念。
- 只覆盖了 10 个最基础的字段 / 概念。一个新手很可能下一个就问「nonce 是什么」「ERC-20 transfer 的 input data 怎么读」，目前都没有。
- 解释内容是我作为新手 + AI 辅助写的，**不是权威定义**，可能在细节上有偏差（比如 Gas 计价单位的精确表述、合约可升级性的边界）。
- 没有做国际化，目前是中文为主、术语保留英文。
- 没有做暗色模式 / 字体大小调整 / 无障碍优化。

## 下一步改进

- **字典扩展**：补 nonce / Input Data / Internal Tx / ERC-20 / Approve / Allowance / Gas Limit / Base Fee / Priority Fee 这一组。
- **真正的「字段拼接」模式**：让用户粘贴一笔 Etherscan 交易的 URL 或者完整字段块，自动逐字段解释（仍然不调网络 — 用正则切分链接里的 hash + 字段映射就行）。
- **链接到原文**：每条解释下面挂上 Handbook 里对应章节的锚点链接，让用户能从「快速理解」跳到「完整原文」。
- **Day 3 我那笔 Sepolia 自转账的真实截图**：把它放到 README 里作为 demo 截图，让说明更具体。
- **可选：Web3 钓鱼词典**：在「安全提醒」那一栏专门做一个 sub-mode，列出当前流行的钓鱼模式（permit 钓鱼、blind signing、address poisoning 等），作为查询型工具。

## 文件清单

- `experiments/web3-field-explainer.html` — 单文件 demo，浏览器直接打开。
- `tasks/web3-field-explainer-submission.md` — 本说明文档。

无 `.env`、无外部依赖、无网络请求。
