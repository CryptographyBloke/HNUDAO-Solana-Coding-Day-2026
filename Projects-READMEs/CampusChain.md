# 校园链 · 链上校园信誉体系 (CampusChain)

> 参赛队伍：瑟瑟发抖吧 | 提交人：李康硕
> 项目仓库：https://github.com/likangshuo888/codingday888

## 项目简介

**校园链（CampusChain）** 是一套面向高校场景的链上校园信誉与行为存证系统。项目基于 Python 从零实现了完整的区块链底层逻辑，包括账户模型、PoS（权益证明）共识算法、P2P 节点网络通信、默克尔树（Merkle Tree）数据校验、交易池管理与 RSA 数字签名机制。

系统将校园内的多类日常行为（如志愿服务、图书借阅、二手物品流转、社团活动、竞赛获奖、经费申请等）转化为链上不可篡改的凭据记录，并提供直观的学生端与平台管理控制台，助力高校构建透明可信的数字校园信用生态。

## 核心特性

- **自主实现区块链核心**：
  - **账户与钱包体系**：基于 2048 位 RSA 密钥对生成公私钥并完成交易签名与验签 (`Wallet.py`, `AccountModel.py`)。
  - **PoS 权益证明共识机制**：依据持币/权益权重与历史区块哈希动态计算抽奖哈希（Lot Hash）决定记账出块节点 (`ProofOfStake.py`, `Lot.py`)。
  - **Merkle Tree 存证校验**：自动构建交易列表哈希树并计算 Merkle 根，奇数节点自动复制补齐，确保存证防篡改 (`MerkleTree.py`)。
  - **P2P 网络与节点发现**：基于 Socket 通信与后台线程实现节点状态广播与对等节点自动发现 (`SocketCommunication.py`, `PeerDiscoveryHandler.py`, `Node.py`)。
- **校园多维行为链上存证**：
  - 定义专有存证类型：`BOOK`（图书）、`ITEM`/`BUY`（二手流转）、`VOLUNTEER`/`VOL_TASK`/`VOL_JOIN`（志愿活动）、`CLUB`（社团活动）、`COMPETITION`（竞赛获奖）、`FUND_APPLY`/`FUND_GRANT`（经费审批）。
  - 支持仅上链存证防伪，不产生无序代币投机。
- **双端 Web 交互控制台**：
  - **学生端** (`student.html`)：学生查看信誉凭据、发起志愿/活动申请。
  - **平台控制台** (`dashboard.html`)：管理员进行行为审核、出块监测与节点状态查看。

## 快速上手

### 1. 运行节点与 API
```bash
python Main.py <节点IP> <P2P端口> <API端口>
# 示例：
python Main.py 127.0.0.1 8000 5000
```

### 2. 访问界面
- 学生端控制台：`http://localhost:5000/app`
- 平台管理后台：`http://localhost:5000/`（默认管理口令：`campus-admin`）
