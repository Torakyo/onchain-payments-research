# W1_D1_Blockchain_Transaction_Basics

---

## 一、 学到的核心概念

1. **EOA（Externally-Owned Account，外部拥有账户）**：由私钥控制的普通用户账户，拥有 ETH 余额，可以主动发起交易。
2. **Contract Account（合约账户）**：部署在以太坊网络上的智能合约代码，没有私钥，只能在接收到交易/消息调用时被动触发执行逻辑。
3. **Nonce（账户交易计数器）**：针对 EOA 账户递增的序号，用于标识该账户发起的交易总数，防止重放攻击并确保交易按顺序处理。
4. **Transaction Hash（交易哈希）**：交易数据经签名编码后通过密码学哈希函数生成 64 位十六进制字符串，是交易在链上的唯一识别码。
5. **Gas Limit & Gas Used**：Gas Limit 是用户允许一笔交易消耗的最大 Gas 数量上限；Gas Used 是 EVM 实际执行计算和存储操作所消耗的 Gas 量。
6. **Function Selector & Calldata**：Calldata 是交易中的 `input` 字段。前 4 字节为函数选择器（由函数签名的 Keccak-256 哈希截取得出），后接按 32 字节补齐的编码参数。
7. **Transaction Pool / Mempool（交易池）**：未打包交易的暂存区，所有广播到以太坊网络但尚未被验证者写入区块的 Pending 交易均保存在此。
8. **Execution Payload（执行载荷/区块体）**：区块中包含具体交易列表（`transactions`）、状态根（`state_root`）、收据根（`receipts_root`）等核心执行数据的对象。
9. **Transaction Receipt（交易回执）**：交易被 EVM 执行后生成的凭证记录，包含交易执行状态（成功/失败）、实际 Gas 消耗（Gas Used）及触发的事件日志（Event Logs）。
10. **Block Finalization（区块终局性）**：在 Proof-of-Stake 共识下，区块经过验证者的 Attestation 确认，由 Justified 升级为 Finalized 后，交易状态达到不可逆转的最终确定性。

---

## 二、 重点知识与验收标准：Transaction、Block、Receipt 三者关系

**【验收标准自我诊断】不看资料解释三者关系**：

* **Transaction（交易）** 是用户表达“意图”的入参指令，包含了发送方签名、接收方、转账金额以及调用的合约数据（Calldata）。
* **Block（区块）** 是保存交易与状态变更的容器。验证者将交易池中的多笔 Transaction 批量收集、打包并按顺序执行，统一写入链上更新全局状态（State）。
* **Receipt（交易回执）** 是交易被 EVM 真实执行后产生的“结果账单与凭证”。它不存储在交易体内部，而是在区块生成时由节点执行完交易后输出，记录该笔交易最终是成功还是失败、消耗了多少 Gas 以及抛出的合约事件日志（Event Logs）。

**关系小结**：Transaction 是**因（执行输入）**，Block 是**载体（批量组织与共识）**，Receipt 是**果（执行输出与凭证）**。区块头中分别通过 `transactions_root` 和 `receipts_root` 构建默克尔树对交易列表与回执列表进行密码学校验。

---

## 三、 交易生命周期图 (Transaction Lifecycle)

```text
 [1. 构建与签名]
  EOA 私钥签名 Transaction (包含 Nonce, Gas, To, Value, Calldata)
         │
         ▼
 [2. 广播与入池]
  通过 RPC 节点广播至网络 ───► 进入验证者节点的 Mempool (Transaction Pool / Pending)
                                     │
                                     ▼
                            [3. 打包与 EVM 执行]
                            验证者根据 Priority Fee 挑选交易打包进 Block
                            EVM 按顺序逐笔执行，更新全局状态 (State Root)
                                     │
                                     ▼
                            [4. 产出回执与出块]
                            生成 Transaction Receipt (记录 Gas Used, Status, Logs)
                            构建 Execution Payload 并在 Slot 内广播区块
                                     │
                                     ▼
                            [5. 共识与终局性]
                            其他验证者节点重执行验证 ──► Justified ──► Finalized

```

---

## 四、 3 笔观察样本的初步观察记录

### 样本 1：标准 ETH 原生代币转账

* **交易哈希**：`0xeba2df809e7a612a0a0d444ccfa5c839624bdc00dd29e3340d46df3870f8a30e`
* **观察要点**：
* **交易类型**：Type 0 / Legacy 或 Type 2 普通 ETH 转账。
* **Calldata (Input Data)**：`0xabcd` 或为空（`0x`）。
* **Gas Limit**：固定为 21,000 单位。
* **交易效果**：仅修改发送方与接收方的 ETH 余额（Wei 账户余额变动），不触发复杂的智能合约逻辑。



### 样本 2：ERC-20 代币转账 (USDC Transfer)

* **交易哈希**：`0xd0dcbe007569fcfa1902dae0ab8b4e078efe42e231786312289b1eee5590f6a1`
* **观察要点**：
* **交互合约 (`To`)**：`0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` (USDC Proxy Contract)。
* **Function Selector**：`0xa9059cbb`，对应签名 `transfer(address,uint256)`。
* **参数解析**：前 32 字节去除填充零后为接收方地址 `0x4f6742badb049791cd9a37ea913f2bac38d01279`；后 32 字节十六进制 `0x3b0559f4` 换算十进制为 990,206,452，按 USDC 6 位小数转换为 990.206452 个 USDC。
* **Receipt Logs**：回执中触发了 `Transfer(address,address,uint256)` 事件日志。


### 样本 3：智能合约部署交易 (Contract Creation)

* **交易哈希**：`0x97d99bc7729211111a21b12c933c949d4f31684f1d6954ff477d0477538ff017`
* **观察要点**：
* **接收地址 (`To`)**：为空（`null` 或未指定）。
* **Input Data**：包含完整的初始化代码与字节码（Compiled Bytecode）。
* **Receipt 结果**：交易回执中会包含新生成的合约地址 (`contractAddress`)，该地址通过创建者地址和发送时的 Nonce 值由 `CREATE` 指令算出。

---

## 五、 仍然不理解的 3 个深层问题

1. **问题 1：Nonce 跳号后的交易处理机制**
* *疑问*：如果账户当前的链上 Nonce 是 2，用户误操作依次广播了 Nonce=2 和 Nonce=4 的交易，但 Nonce=3 的交易丢包未发，交易池（Mempool）会如何处理 Nonce=4 的交易？它会在池中静默等待多长时间？

2. **问题 2：Transaction Receipt 中的 Logs Bloom 检索原理**
* *疑问*：执行回执中包含 `logs_bloom` 过滤器。轻节点或以太坊客户端是如何利用布隆过滤器在不遍历整个区块回执的情况下，实现毫秒级快速匹配特定合约和事件话题（Topics）的？

3. **问题 3：EIP-1559 机制下动态 Base Fee 与网络拥堵控制**
* *疑问*：当区块 Gas 使用量连续超过 30M Target 值时，Base Fee 会以每区块最高 12.5% 的幅度递增。若遇到极端热点 NFT 铸造或市场恐慌，这种指数级增长是如何从博弈论和经济学约束上阻止网络死锁的？
