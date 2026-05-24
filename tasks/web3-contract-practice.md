# 测试网最小智能合约部署记录
## 一、作业目标
本次作业的目标是在测试网上部署或调用一个最小智能合约，并理解合约地址、读取函数、写入函数、钱包确认、交易确认和区块浏览器验证之间的关系。
我选择的完成方式是：使用 Remix IDE 和 Rabby Wallet，在 Base Sepolia 测试网上部署一个最小智能合约。
由于前期 Remix 与 Rabby 钱包连接时多次触发部署确认，我在 Base Sepolia 测试网上产生了三笔 Contract Creation 交易。
三笔均为相同 SimpleMessage 合约的部署记录。作业中我选取最新的一笔部署交易作为提交对象。
---
## 二、使用工具
| 项目 | 内容 |
|---|---|
| 开发工具 | Remix IDE |
| 钱包 | Rabby Wallet |
| 测试网络 | Base Sepolia Testnet |
| 区块浏览器 | Base Sepolia Basescan |
---
## 三、合约代码
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
contract SimpleMessage {
    string public message;
    constructor(string memory _message) {
        message = _message;
    }
    function setMessage(string memory _newMessage) public {
        message = _newMessage;
    }
}

⸻

四、部署信息

项目	内容
合约名称	SimpleMessage
合约地址	0xf4ebf4354fecd286f4e7dda94fd7b33eeecc42fe
部署钱包地址	0xf4EbF4354FECD286F4e7ddA94FD7b33eeECC42FE
部署交易哈希	0xd1b5bbf0eaf1141d1fbc68d075f23b5562bd8e39cd1ba6f64520fc972dd6dbad
区块高度	41912964
区块浏览器链接	[【填写 Basescan 合约地址链接】](https://sepolia.basescan.org/address/0xf4ebf4354fecd286f4e7dda94fd7b33eeecc42fe)

⸻

五、部署过程说明

我在 Remix IDE 中新建了 SimpleMessage.sol 文件，并使用 Solidity 0.8.20 版本进行编译。

编译成功后，我将 Remix 的运行环境切换为浏览器钱包插件，并连接 Rabby Wallet。钱包网络选择为 Base Sepolia Testnet。

部署合约时，我在 constructor 参数中输入：

"Hello Web3"

随后点击 Deploy，并在 Rabby Wallet 中人工确认交易。交易确认后，合约成功部署到 Base Sepolia 测试网，并在区块浏览器中生成了对应的合约地址和部署交易记录。

⸻

六、读取结果

合约中的 message 是一个 public 变量，因此可以直接读取。

读取函数：

message()

读取结果：

Hello Web3

该操作属于读取链上状态，不会修改区块链数据，因此不需要钱包签名，也不需要支付 gas。

⸻

七、写入函数说明

合约中包含一个写入函数：

function setMessage(string memory _newMessage) public {
    message = _newMessage;
}

该函数可以修改链上保存的 message 内容。

示例写入内容：

"My first testnet contract"

由于该函数会改变链上状态，所以调用时需要在钱包中人工确认交易，并支付少量测试网 gas。

⸻

八、人工确认步骤

本次操作中，需要人工确认的步骤包括：

1. 部署智能合约时，需要在 Rabby Wallet 中确认交易。
2. 调用 setMessage() 写入函数时，需要在 Rabby Wallet 中确认交易。
3. 读取 message() 函数时不需要钱包确认，因为读取操作不会改变链上状态。

⸻

九、补充说明

在操作过程中，由于 Remix 与 Rabby Wallet 连接时多次触发确认请求，我的地址下出现了多笔 Contract Creation 交易。这些交易都是在 Base Sepolia 测试网上进行的合约部署记录。

本次作业最终选择其中一笔成功部署交易作为提交对象。

最终提交合约地址：
0xf4ebf4354fecd286f4e7dda94fd7b33eeecc42fe

最终提交区块浏览器链接：

(https://sepolia.basescan.org/address/0xf4ebf4354fecd286f4e7dda94fd7b33eeecc42fe)

⸻

十、安全说明

本次提交内容只包含公开的测试网信息，包括合约地址、交易哈希、区块浏览器链接和函数调用说明。

未提交任何私钥、助记词、API Key、token、.env 文件或其他敏感信息。

你现在只要把 `【填写...】` 这些地方替换成你 Basescan 上看到的真实信息就可以。
