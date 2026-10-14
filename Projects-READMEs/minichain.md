# minichain

用 **Node.js 内置模块从零实现的一条区块链**，零第三方依赖。

涵盖真实区块链的核心要素：哈希链、Merkle 树、secp256k1 数字签名、地址派生、PoW 工作量证明、内存池、防重放、防篡改校验、JSON 持久化，以及一个只读 + 挖矿的 HTTP 接口。

> 适合用来理解「一笔交易从签名到上链再到防止双花」的完整链路，不适合用于生产。

---

## 快速开始

需要 Node.js 18+（开发环境为 v24），无需 `npm install`。

启动后访问 `http://localhost:3000` 打开交互式网络控制台，可查看链状态、管理本地钱包、发送签名交易并挖矿。

```bash
node src/index.js wallet        # 创建钱包，拿到地址
node src/index.js mine <地址>    # 挖矿，获得 50 枚区块奖励
node src/index.js send <收款地址> 30
node src/index.js mine <地址>    # 把内存池里的交易打包进区块
node src/index.js wallets       # 查看所有地址及余额
node src/index.js verify        # 全链校验
```

完整演示：

```bash
node src/index.js init
node src/index.js wallet                                   # alice
node src/index.js wallet                                   # bob
node src/index.js mine <alice地址>                          # alice 余额 50
node src/index.js mine <alice地址>                          # alice 余额 100
node src/index.js send <bob地址> 30 --from <alice地址>       # 签名后进内存池
node src/index.js mine <alice地址>                          # 上链，alice 120 / bob 30
node src/index.js verify                                   # chain is valid
```

---

## 命令

| 命令 | 说明 |
| --- | --- |
| `init` | 清空链，回到创世区块 |
| `wallet` | 生成新钱包并保存到 `data/wallets/` |
| `wallets` | 列出所有钱包地址与余额 |
| `balance <address>` | 查询某个地址余额 |
| `mine [minerAddress]` | 挖下一个区块；传地址即获得 50 枚奖励 |
| `send <to> <amount> [--from <address>]` | 签名一笔转账并放入内存池 |
| `mempool` | 查看待打包交易 |
| `chain` | 打印整条链的 JSON |
| `verify` | 校验全链（哈希、签名、工作量、防重放） |
| `serve [port]` | 启动 HTTP 节点，默认 3000 |

---

## HTTP 接口

```bash
node src/index.js serve 3000
```

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| `GET` | `/chain` | 返回整条链与校验结果 |
| `GET` | `/balance?address=0x..` | 查询余额 |
| `GET` | `/mempool` | 返回内存池 |
| `POST` | `/mine` | Body 可传 `{ "miner": "0x..", "transactions": [...] }` |
| `GET` | `/api/overview` | 控制台数据：区块、钱包、交易、链校验与指纹 |
| `POST` | `/api/wallets` | 创建并保存本地钱包 |
| `POST` | `/api/transactions` | Body: `{ "from": "0x..", "to": "0x..", "amount": 10 }`，由本地钱包签名并进入内存池 |
| `POST` | `/api/mine` | 打包所有有效待处理交易并挖矿；Body: `{ "miner": "0x.." }` |

```bash
curl "http://127.0.0.1:3000/balance?address=0x..."
curl -X POST http://127.0.0.1:3000/mine -H 'Content-Type: application/json' -d '{"miner":"0x..."}'
```

---

## 代码结构

```
src/
  crypto.js       哈希、Merkle 树、secp256k1 签名与验签、地址派生
  transaction.js  交易：签名载荷、交易 ID、防篡改校验
  block.js        区块：区块头、Merkle 根、PoW 挖矿
  blockchain.js   链：追加校验、余额计算、全链校验
  mempool.js      内存池：去重、剔除付不起的交易
  wallet.js       钱包：密钥对、构造并签名交易
  store.js        持久化：链 / 内存池 / 钱包的 JSON 读写
  server.js       HTTP 节点
  index.js        命令行入口
test/
  minichain.test.js
```

---

## 关键设计

**签名绑定发送方。** 交易里同时存 `publicKey` 和 `from`（地址），校验时会重新推导 `addressFromPublicKey(publicKey)` 并与 `from` 比对，两者不一致直接判定无效。签名覆盖 `payload()` 的全部字段，改金额、改收款人都验签失败。

**Merkle 根提交交易。** 区块头只提交 `merkleRoot`，交易再多区块头也是固定大小；节点只需比对根哈希即可确认交易集未被替换。

**PoW。** 不断递增 `nonce` 直到区块头哈希满足 N 位前导零（难度 N）。区块哈希在验证时会被重新计算，任何字段被改动都会导致校验失败。

**防重放。** 交易 ID = `hash(payload)`。`addBlock` 会检查全链已有交易 ID，重复的交易无法让链变长。

**余额即重放。** 不维护额外状态表，`getBalance` 通过重放全部区块得出，天然与链上数据一致。

**确定性序列化。** 哈希前先递归排序对象键，避免 JavaScript 属性顺序影响计算结果。

---

## 测试

```bash
node --test "test/*.test.js"
```

20 个用例覆盖：哈希确定性、Merkle 树、签名验签与三种攻击路径（错钥签名、改金额、伪造发送方）、创世区块、区块头篡改、交易重放、错误 `previousHash` 的分叉、内存池去重与余额校验、JSON 往返后仍然有效。

---

## 可以继续做的

- **UTXO 模型**：把账户余额换成未花费输出集合，更贴近比特币
- **P2PKH 地址**：用 `hash160(pubkey)` + base58check 替代当前的简化地址
- **难度调整**：按目标出块时间动态调整前导零位数
- **节点间同步**：`/peers` 广播区块 + 最长链规则解决分叉
- **持久化换成 LevelDB**：避免每次全量读写 `chain.json`
- **智能合约**：栈式 VM 或简单的存储槽模型
