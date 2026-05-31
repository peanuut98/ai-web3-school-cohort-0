# Week 2 Module D｜Agent Wallet / Permission / Safe Execution 笔记与设计

## 1. 作业主题

本次作业围绕 **Agent 发起链上动作时的权限控制与安全执行** 展开。

在 AI × Web3 场景中，Agent 不能像普通脚本一样随意操作钱包，因为链上动作一旦执行，通常不可撤回。如果 Agent 拥有过高权限，可能会造成资产损失、错误授权、重复交易或被恶意利用。

因此，Agent Wallet 的核心不是“让 Agent 自动帮我操作一切”，而是设计一套可限制、可审计、可撤销、可恢复的执行机制。

---

## 2. 场景设定：DeFi Portfolio Assistant

我设计的场景是一个 **DeFi Portfolio Assistant**。

它是一个帮助用户管理 DeFi 仓位的 Agent，主要功能包括：

- 查询钱包资产余额
- 检查 DeFi 仓位状态
- 发现收益率变化
- 提醒用户是否需要调整仓位
- 在低风险范围内执行小额操作
- 对高风险操作发起人工确认请求

它不能直接无限制操作用户钱包，也不能绕过用户确认完成大额转账、授权或合约交互。

---

## 3. Agent 发起链上动作的执行流程图

```text
用户设定 Agent Wallet 权限策略
        ↓
Agent 读取用户任务目标
        ↓
Agent 分析当前钱包状态和链上数据
        ↓
Agent 生成链上动作建议
        ↓
系统检查权限策略
        ↓
判断动作风险等级
        ↓
┌──────────────────────────────┐
│ 低风险动作                    │
│ 例如：查询余额、读取合约状态     │
│ 可以自动执行                  │
└──────────────────────────────┘
        ↓
自动调用只读接口或低风险合约方法
        ↓
记录日志
        ↓
返回结果给用户


┌──────────────────────────────┐
│ 中风险动作                    │
│ 例如：小额 swap、小额 deposit   │
│ 在预算和白名单内可自动执行      │
└──────────────────────────────┘
        ↓
检查预算、合约白名单、token 白名单、频率限制
        ↓
如果符合策略
        ↓
Agent 发起交易请求
        ↓
Smart Account / Safe / Policy 模块检查
        ↓
执行交易
        ↓
记录交易哈希和执行结果
        ↓
返回结果给用户


┌──────────────────────────────┐
│ 高风险动作                    │
│ 例如：大额转账、approve、提款   │
│ 必须人工确认                  │
└──────────────────────────────┘
        ↓
Agent 生成交易草稿
        ↓
展示交易金额、合约地址、操作类型、风险提示
        ↓
用户人工确认
        ↓
钱包签名
        ↓
链上执行
        ↓
记录日志
        ↓
返回结果给用户
```

---

## 4. 哪些步骤可以自动化，哪些必须人工确认

| 步骤 | 是否可以自动化 | 原因 |
|---|---|---|
| 查询钱包余额 | 可以自动化 | 只读操作，不改变资产状态 |
| 查询 DeFi 仓位 | 可以自动化 | 只读操作，风险较低 |
| 分析收益率变化 | 可以自动化 | 只是信息处理 |
| 生成操作建议 | 可以自动化 | 不直接触发链上资产变化 |
| 小额 swap | 条件自动化 | 必须限制金额、频率、合约和 token |
| 小额 deposit | 条件自动化 | 必须在白名单协议和预算范围内 |
| approve 授权 | 必须人工确认 | 授权风险高，可能导致资产被转走 |
| 大额 transfer | 必须人工确认 | 直接影响资产所有权 |
| withdraw 提款 | 必须人工确认 | 涉及资产流出和仓位变化 |
| 修改权限策略 | 必须人工确认 | 权限本身决定 Agent 能做什么 |
| 撤销 Agent 权限 | 必须人工确认 | 属于安全管理动作 |
| 重试失败交易 | 条件自动化 | 低风险可重试，高风险必须确认 |

---

## 5. Agent Wallet 权限策略设计

### 5.1 预算限制

Agent 每日、每周、每次操作都必须有预算上限。

```text
单次交易上限：20 USDC
每日交易上限：100 USDC
每周交易上限：300 USDC
单个协议资金上限：200 USDC
Gas 费用每日上限：10 USDC
```

设计原因：

- 防止 Agent 因错误判断造成大额损失
- 防止重复交易耗尽资金
- 防止恶意合约诱导 Agent 连续操作
- 给用户保留最终资产控制权

---

### 5.2 可调用合约

Agent 只能调用白名单中的合约。

```text
允许调用：
- Uniswap Router
- Aave Pool
- Compound Market
- Safe Module
- 指定价格预言机合约

禁止调用：
- 未知合约
- 未验证合约
- 新部署但无审计记录的合约
- 与任务无关的 NFT 合约
- 混币器或高风险合约
```

设计原因：

- 降低与恶意合约交互的风险
- 避免 Agent 被 prompt injection 引导调用错误合约
- 限制 Agent 的行动范围

---

### 5.3 可执行动作

Agent 的动作需要分级管理。

#### 可以自动执行的动作

```text
- 查询余额
- 查询交易状态
- 查询 DeFi 仓位
- 查询收益率
- 生成风险报告
- 读取合约状态
```

#### 条件自动执行的动作

```text
- 小额 swap
- 小额 deposit
- 小额 repay
- 小额 claim rewards
```

这些动作必须同时满足：

```text
金额低于预算上限
合约在白名单中
token 在白名单中
没有超过每日频率限制
slippage 在用户设定范围内
```

#### 必须人工确认的动作

```text
- approve 授权
- transfer 转账
- withdraw 提款
- bridge 跨链
- 修改权限策略
- 添加新合约白名单
- 提高预算额度
- 调整多签规则
```

设计原因：

- 高风险动作会直接改变资产控制权
- 授权类操作可能造成长期风险
- 跨链和提款失败后恢复难度更高

---

### 5.4 人工确认阈值

人工确认阈值用于判断什么时候必须暂停自动执行。

```text
单次操作金额 > 20 USDC：必须人工确认
每日累计金额 > 100 USDC：必须人工确认
首次调用新合约：必须人工确认
approve 任意金额：必须人工确认
withdraw 任意金额：必须人工确认
跨链操作任意金额：必须人工确认
slippage > 1%：必须人工确认
Gas 费用异常升高：必须人工确认
连续失败 2 次：必须人工确认
```

这样设计的重点是：Agent 可以执行低风险重复动作，但不能在关键资产动作上替代人。

---

### 5.5 撤销方式

Agent Wallet 必须支持随时撤销权限。

撤销方式包括：

```text
1. 用户在钱包前端点击“暂停 Agent”
2. 用户撤销 Safe Module 权限
3. 用户删除 Agent 的 session key
4. 用户取消 token approval
5. 多签成员共同移除 Agent 权限
6. 系统检测异常后自动冻结 Agent 执行权限
```

撤销后，Agent 应该只能保留只读权限，不能继续发起交易。

---

### 5.6 日志记录

所有 Agent 发起的链上动作都必须记录日志。

日志内容包括：

```text
task_id：任务编号
agent_id：Agent 身份
timestamp：执行时间
action_type：动作类型
contract_address：调用合约地址
function_name：调用函数
token：涉及资产
amount：金额
risk_level：风险等级
approval_status：是否人工确认
tx_hash：交易哈希
execution_status：执行状态
failure_reason：失败原因
```

日志的作用：

- 方便用户复盘 Agent 行为
- 方便定位失败原因
- 方便判断责任归属
- 方便后续进行审计
- 方便建立 Agent reputation

---

### 5.7 失败处理

失败处理必须提前写入策略，而不是等失败后再临时决定。

#### 失败类型一：交易失败

```text
原因：
- Gas 不足
- 合约调用失败
- slippage 超限
- 网络拥堵

处理：
- 记录失败原因
- 不自动无限重试
- 低风险动作最多重试 1 次
- 高风险动作失败后必须等待用户确认
```

#### 失败类型二：金额异常

```text
原因：
- 报价变化
- 路由异常
- 合约返回结果异常

处理：
- 立即停止执行
- 提醒用户检查金额
- 不继续签名或提交交易
```

#### 失败类型三：权限不足

```text
原因：
- Agent 超出预算
- 调用了非白名单合约
- 尝试执行被禁止动作

处理：
- 拒绝执行
- 记录日志
- 提醒用户是否需要修改权限策略
```

#### 失败类型四：Agent 判断错误

```text
原因：
- 数据源错误
- 模型理解错误
- prompt injection
- 外部 API 返回错误

处理：
- 暂停自动执行
- 要求人工复核
- 更新策略或回滚任务
```

---

## 6. 完整权限策略示例

```json
{
  "agent_name": "DeFi Portfolio Assistant",
  "maintainer": "Example AI Web3 Team",
  "wallet_type": "smart_account",
  "budget": {
    "single_transaction_limit": "20 USDC",
    "daily_limit": "100 USDC",
    "weekly_limit": "300 USDC",
    "daily_gas_limit": "10 USDC"
  },
  "allowed_contracts": [
    "Uniswap Router",
    "Aave Pool",
    "Compound Market",
    "Chainlink Price Feed"
  ],
  "allowed_actions": [
    "read_balance",
    "read_position",
    "swap_small_amount",
    "deposit_small_amount",
    "claim_rewards"
  ],
  "human_confirmation_required": [
    "approve",
    "transfer",
    "withdraw",
    "bridge",
    "change_policy",
    "add_new_contract",
    "increase_budget"
  ],
  "risk_rules": {
    "max_slippage": "1%",
    "max_retries": 1,
    "new_contract_requires_confirmation": true,
    "approve_always_requires_confirmation": true
  },
  "revocation": {
    "pause_agent": true,
    "remove_session_key": true,
    "revoke_safe_module": true,
    "cancel_token_approval": true
  },
  "logging": {
    "record_task_id": true,
    "record_agent_id": true,
    "record_contract": true,
    "record_action": true,
    "record_amount": true,
    "record_tx_hash": true,
    "record_failure_reason": true
  }
}
```

---

## 7. ERC-4337 为什么重要

ERC-4337 的核心意义是让钱包从普通 EOA 钱包升级为更灵活的 **智能账户**。

传统 EOA 钱包的问题是：

```text
一个私钥控制一个账户。
只要私钥泄露，资产就可能全部丢失。
账户本身缺少复杂权限管理能力。
```

ERC-4337 解决的问题包括：

- 支持智能账户逻辑
- 支持批量交易
- 支持 gas 代付
- 支持 session key
- 支持更复杂的权限控制
- 支持账户恢复机制
- 支持更适合 Agent 的自动化操作

在 Agent Wallet 场景中，ERC-4337 很重要，因为它可以让用户不是简单地把私钥交给 Agent，而是给 Agent 一个有限权限的执行环境。

它解决的主要风险是：

```text
避免 Agent 直接控制完整私钥。
避免所有操作都依赖同一个 EOA。
让权限可以被限制、拆分和恢复。
```

---

## 8. Safe 为什么重要

Safe 是一种常见的智能合约钱包和多签管理工具。

它的重要性在于：Safe 可以把资产控制从“单个私钥”变成“多方确认 + 策略控制”。

Safe 可以解决的问题包括：

- 多签管理
- 团队资金安全
- 权限模块管理
- 交易审批流程
- 执行记录追踪
- 高风险交易拦截

在 Agent Wallet 场景中，Safe 可以作为 Agent 执行链上动作的安全底座。

例如：

```text
低风险动作：Agent 可以在策略范围内执行
高风险动作：必须进入 Safe 多签确认
异常动作：Safe guard / module 直接拦截
```

Safe 解决的主要风险是：

```text
避免单点私钥失控。
避免 Agent 或单个成员独立转走资产。
让团队资产管理更可控、可审计。
```

---

## 9. Guard / Policy 机制为什么重要

Guard / Policy 机制可以理解为 Agent Wallet 的“安全规则层”。

它的作用不是让 Agent 更聪明，而是限制 Agent 即使犯错也不能做太危险的事情。

Guard / Policy 可以检查：

- 调用的合约是否在白名单中
- 操作金额是否超过预算
- 交易频率是否异常
- token 是否被允许
- 是否涉及 approve
- 是否需要人工确认
- 是否超过 slippage 限制
- 是否出现异常重复交易

示例：

```text
如果 Agent 想调用非白名单合约：
Policy 拒绝执行。

如果 Agent 想 approve 无限额度：
Policy 要求人工确认。

如果 Agent 一天内连续执行 10 次 swap：
Policy 暂停 Agent 权限。

如果 Gas 费用异常升高：
Policy 暂停交易并通知用户。
```

Guard / Policy 解决的主要风险是：

```text
防止 Agent 越权执行。
防止恶意合约诱导交易。
防止 prompt injection 影响链上操作。
防止错误自动化造成连续损失。
```

---

## 10. ERC-4337、Safe、Guard / Policy 的关系

这三者可以理解为不同层级的安全设计。

| 机制 | 主要作用 | 解决的风险 |
|---|---|---|
| ERC-4337 | 提供智能账户能力 | 避免 EOA 权限过于单一 |
| Safe | 提供多签和资产管理底座 | 避免单点控制和团队资金失控 |
| Guard / Policy | 提供执行前规则检查 | 避免 Agent 越权和错误执行 |

简单来说：

```text
ERC-4337 让钱包变得可编程。
Safe 让资产管理变得可协作。
Guard / Policy 让 Agent 执行变得可限制。
```

如果没有 ERC-4337，Agent 很难获得灵活、可控的账户权限。

如果没有 Safe，团队资金容易依赖单个私钥，缺少审批流程。

如果没有 Guard / Policy，Agent 即使接入智能钱包，也可能因为错误判断、恶意输入或系统漏洞执行危险操作。

---

## 11. 我的思考

我认为 Agent Wallet 的关键不是“让 Agent 完全自动化”，而是明确区分：

```text
哪些动作可以自动化？
哪些动作必须暂停？
哪些动作必须由人确认？
失败之后如何恢复？
```

在 Web2 的自动化任务中，错误通常可以回滚，例如重新生成文件、重新调用 API、重新提交表单。

但在 Web3 中，链上交易一旦确认，往往无法撤销。因此 Agent 参与链上动作时，必须更加重视权限控制。

一个好的 Agent Wallet 应该满足以下条件：

- Agent 只能在有限预算内执行
- Agent 只能调用白名单合约
- Agent 只能执行低风险动作
- 高风险动作必须人工确认
- 所有动作都必须有日志
- 权限必须可以随时撤销
- 失败后必须有明确处理方式

这样设计之后，Agent 才不是一个危险的“自动签名机器”，而是一个受限制、可验证、可审计的链上执行助手。

---

## 12. 总结

本次 Module D 的核心结论是：

Agent 发起链上动作时，最重要的问题不是“能不能执行”，而是“能在什么范围内执行”。

因此，一个合理的 Agent Wallet 权限策略至少需要包含：

- 预算限制
- 可调用合约
- 可执行动作
- 人工确认阈值
- 撤销方式
- 日志记录
- 失败处理

ERC-4337、Safe 和 Guard / Policy 分别从账户抽象、多签管理和策略控制三个层面降低风险。

最终目标是让 Agent 可以参与链上任务，但不能越权控制用户资产。

---

## 13. 敏感信息声明

本作业不包含任何：

- 私钥
- 助记词
- API Key
- token
- .env 文件
- 真实资金账户信息
- 真实钱包敏感授权信息

文中所有金额、合约名称和策略均为学习示例，不代表真实资金操作建议。
