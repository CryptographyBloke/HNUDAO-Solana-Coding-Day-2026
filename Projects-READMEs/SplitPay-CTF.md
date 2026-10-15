# 基于智能合约的聚餐AA收款系统与安全挑战 (SplitPay CTF)

> 参赛队伍：鸢尾花  
> 项目仓库：https://github.com/Hugh102200/demo  

## 项目简介

**基于智能合约的聚餐AA收款系统与安全挑战 (SplitPay CTF)** 是一个将日常聚餐 AA 制费用分摊与以太坊智能合约安全演练（CTF）相结合的项目。

项目通过 Solidity 实现了去中心化聚餐 AA 记账与充值提现机制，并在设计中还原了以太坊历史上著名的经典安全隐患——**重入漏洞（Reentrancy Vulnerability）**。项目同时提供了漏洞合约、自动化攻击利用合约以及遵循最佳安全实践（Checks-Effects-Interactions 模式）的加固版安全合约，构建了一个完整的智能合约攻防教学与演练闭环。

---

## 核心架构与合约设计

项目包含三大核心智能合约（开发与调试基于 Remix IDE 与 Ethers.js）：

### 1. 存在漏洞的收款合约 (`SplitPayCTF.sol`)
- **功能**：组织者与参与者可通过 `deposit()` 存入 ETH 记录 AA 账户余额，并通过 `withdraw()` 提取余额。
- **重入缺陷**：在 `withdraw()` 函数中，合约在未扣减用户余额（`balances[msg.sender]`）的情况下，直接通过底层的 `.call{value: amount}("")` 向调用者转账：
  ```solidity
  // 漏洞点：先外部调用转账，后更新本地状态
  (bool success, ) = msg.sender.call{value: amount}("");
  require(success, "Transfer failed");
  balances[msg.sender] = 0;
  ```
- **通关验证**：设置了 `isSolved()` 状态判定，当合约资金池被完全清空时，返回通关 Flag：`flag{aa_pay_reentrancy_hacked}`。

### 2. 重入攻击合约 (`Attacker.sol`)
- **攻击链路**：
  1. 调用 `target.deposit{value: msg.value}()` 先存入一笔合法代币建立初始余额；
  2. 随后调用 `target.withdraw()` 触发目标合约向攻击合约转账；
  3. 攻击合约在 `receive()` 回调函数中检测到资金到账时，在目标合约更新余额前再次递归调用 `target.withdraw()`；
  4. 循环往复，直至抽干目标合约账户中的所有以太坊资产。

### 3. 加固安全合约 (`SplitPaySafe.sol`)
- **安全修复**：严格遵循以太坊官方推荐的 **Checks-Effects-Interactions（检查-生效-交互）** 编程范式：
  ```solidity
  // 修复：先清零用户余额，杜绝递归重入可能
  balances[msg.sender] = 0;
  (bool success, ) = msg.sender.call{value: amount}("");
  require(success, "Transfer failed");
  ```
  在向不可信外部地址发起任何以太币转账之前，先行将本地状态置零，从根本上免疫重入攻击。

---

## 快速使用与演练

1. **部署环境**：推荐使用 Remix IDE（项目已内置 `remix.config.json` 与测试脚本）。
2. **复现攻击**：
   - 部署 `SplitPayCTF.sol`，模拟多个用户存入 AA 聚餐费用。
   - 部署 `Attacker.sol` 并传入 `SplitPayCTF` 的合约地址。
   - 调用 `Attacker.attack{value: 1 ether}()`，触发重入利用，清空收款合约资金池。
   - 调用 `SplitPayCTF.isSolved()`，验证漏洞触发成功。
3. **加固验证**：部署 `SplitPaySafe.sol` 重复上述流程，验证攻击合约由于状态已提前清零而无法发起重入。
