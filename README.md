# onchain-payments-research

Learning-in-public research on on-chain stablecoin payments and agent payments (x402, AP2, ACP, ERC-8004), starting from Base.

公开学习记录：链上稳定币支付与 Agent 支付研究，从 Base 开始。

**Status / 状态:** Week 1 of a 16-week plan. Everything here is work in progress. Numbers marked "unverified" have not been cross-checked yet.
第 1 周 / 共 16 周。所有内容均为进行中，标注 "unverified" 的数字尚未交叉核对。

---

## English

### Goal
Learn to pull on-chain data myself, verify it, and turn it into research other people can reproduce.

### Working rules
- Every number states its source, time window and definition.
- A script is trusted only after at least 3 sampled transactions have been checked against a block explorer.
- Mistakes are logged, not deleted (see `notes/`).

### Repository layout
| Path | What it holds |
|---|---|
| `notes/` | Written notes: glossary, transaction field notes, classification tables |
| `data/` | Data files. `raw/` is never edited by hand. `processed/` is derived from `raw/` |
| `notebooks/` | Jupyter notebooks and the code that produces the results |

More folders will be added only when needed.

### Setup
Requires Python 3 and an RPC endpoint for Base mainnet (chain ID 8453) from any provider.

```bash
git clone https://github.com/<your-username>/onchain-payments-research.git
cd onchain-payments-research
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env               # Windows: copy .env.example .env
# edit .env and set BASE_RPC_URL
jupyter lab
```

Never commit `.env`. Do not print the RPC URL in a notebook, because notebook outputs are saved in the file.

### Data sources
- Base mainnet via JSON-RPC (`BASE_RPC_URL`)
- BaseScan (https://basescan.org) for manual verification

### Known limits
- Free-tier RPC plans may restrict the block range of `eth_getLogs`, so early work uses small samples.
- Small samples are not representative. Each result states its sample range.

### Disclaimer
Educational research only. Nothing here is investment advice.

---

## 中文

### 目标
学会自己取得链上数据、核对数据，并把结果整理成别人可以复现的研究。

### 工作规则
- 每个数字都写明来源、时间窗口和口径。
- 脚本至少用 3 笔抽样交易与区块浏览器核对后，才视为可信。
- 错误会记录下来，不会直接删除（见 `notes/`）。

### 目录结构
| 路径 | 内容 |
|---|---|
| `notes/` | 文字笔记：术语表、交易字段说明、分类表 |
| `data/` | 数据文件。`raw/` 不手工修改，`processed/` 由 `raw/` 生成 |
| `notebooks/` | Jupyter 笔记本，以及产生结果的代码 |

需要时才会增加新目录。

### 环境设置
需要 Python 3，以及任一服务商提供的 Base 主网（链 ID 8453）RPC 端点。步骤见上方英文部分的命令。

请勿提交 `.env`。不要在笔记本里打印 RPC URL，因为笔记本会把输出一起保存。

### 数据来源
- 通过 JSON-RPC 读取 Base 主网（`BASE_RPC_URL`）
- 用 BaseScan（https://basescan.org）做人工核对

### 已知限制
- 免费 RPC 方案可能限制 `eth_getLogs` 的区块范围，所以早期只使用小样本。
- 小样本不具代表性，每项结果都会注明样本范围。

### 免责声明
仅为学习与研究，不构成任何投资建议。
