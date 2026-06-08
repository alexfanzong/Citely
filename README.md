<a id="citely"></a>

# ⚖️ Citely

> 专家把 Web3 法律、合规、安全与风险研究报告签名上链；真人读者和 AI Agent 用 x402 按篇付费解锁全文，收入 100% 直达作者钱包。

[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=nextdotjs)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=000)](https://react.dev/)
[![x402](https://img.shields.io/badge/x402-pay--per--read-3B82F6)](https://x402.org/)
[![Base Sepolia](https://img.shields.io/badge/Base-Sepolia-0052FF)](https://docs.base.org/)
[![EAS](https://img.shields.io/badge/EAS-attestations-6B7280)](https://attest.org/)

**中文** | [English](#english)

## 目录

- [项目介绍](#项目介绍)
- [为什么做 Citely](#为什么做-citely)
- [演示闭环](#演示闭环)
- [核心功能](#核心功能)
- [技术栈](#技术栈)
- [本地运行](#本地运行)
- [Agent API](#agent-api)
- [项目结构](#项目结构)
- [路线图](#路线图)
- [已知边界](#已知边界)
- [团队成员](#团队成员)
- [许可](#许可)
- [English](#english)

## 🌐 项目介绍

Citely 是一个面向专业 Web3 合规内容的链上内容授权与付费阅读平台。律师、合规顾问、安全研究员、审计师、税务师或行业分析师可以把 Web3 风险报告发布为可验证内容：作者身份、内容哈希、定价和免责声明通过 EAS attestation 写到链上；读者或 AI Agent 再用 USDC 通过 x402 按篇付费获取全文。

它的目标不是做一个普通内容平台，而是跑通一个更小但更关键的闭环：

```text
作者签名上链 -> 报告进入目录 -> 真人和 Agent 同价付费 -> 全文解锁 -> 收益直达作者 -> 链上可核验
```

本次提交是 Citely 平台的 **hackathon MVP**：先在 Base Sepolia 测试网上跑通一篇报告的完整生命周期，证明作者授权、链上存证、真人付费、Agent 付费和收益可见这条主链路成立。演示规格见 [docs/HACKATHON.md](docs/HACKATHON.md)，3 分钟演示脚本见 [docs/DEMO.md](docs/DEMO.md)。

## 🧩 为什么做 Citely

高质量的 Web3 风险分析通常有三个问题：

- **出处不够可验证**：读者很难判断文章是否由真实专家发布、是否被改动、定价是否透明。
- **Agent 很难合法消费专业内容**：传统订阅和 API key 模式不适合临时、按需、低金额的 agentic reading。
- **平台抽成和对账复杂**：作者需要相信平台后台，读者需要相信平台没有隐藏加价。

Citely 用链上存证和 x402 把这些问题拆开：

- EAS 记录作者、内容哈希、价格和版本，降低冒名和篡改风险。
- x402 让真人和 AI Agent 用同一个价格、同一条付费路径解锁全文。
- USDC 直接结算到作者钱包，平台不托管资金，内容侧 v1.0 阶段 0 抽成。
- Cobo Agentic Wallet / pact 负责约束 Agent 的付款边界，例如花多少、付给谁、在哪条链。

## 🔁 演示闭环

1. **作者发布**：在 `/publish` 导入报告、设定价格，用钱包签名并提交 EAS attestation。
2. **目录展示**：报告进入 `/reports` 和首页目录，并展示 on-chain / EAS 徽章。
3. **真人付费**：读者先看预览，再用钱包通过 x402 支付测试 USDC 解锁全文。
4. **Agent 付费**：Agent 读取公开目录，访问付费接口，经历 `402 -> pay -> 200` 后拿到全文和 companion。
5. **收益可见**：付款记录聚合到首页 For Writers 的 Top Earning Authors 榜单。
6. **链上核验**：EAS attestation 可在 EAS Explorer 验证，x402 settlement 可在 Base Sepolia 区块浏览器核验。

## ✨ 核心功能

- **可验证发布**：`contentHash + author + price + version + disclaimer` 写入 EAS。
- **公开目录**：`GET /api/v1/articles` 免费返回文章元数据、价格、作者和付费阅读路径。
- **x402 付费阅读**：`GET /api/v1/articles/[slug]` 返回 402，付款后返回全文 markdown。
- **付费正文加密**：文章正文以 AES-256-GCM 加密存储在 `.enc` 文件中，仅服务端在付款后解密。
- **Agent companion**：付费响应附带术语表、法条地图、误区表等结构化 companion。
- **人类与 Agent 同价**：价格以链上 attestation 和服务端索引为准，不接受请求方覆盖。
- **作者收益榜**：本地 MVP 通过 `data/payment-log.json` 聚合付款记录。

## 🛠 技术栈

- **Framework**：Next.js 16 App Router, React 19, TypeScript
- **Styling**：Tailwind CSS v4
- **Payments**：x402, Coinbase CDP facilitator, USDC on Base Sepolia
- **Wallet / Chain**：wagmi, viem, ethers
- **Attestations**：Ethereum Attestation Service (EAS)
- **Content**：MDX metadata + encrypted paid body
- **Tests**：Vitest, Testing Library
- **Deployment**：Vercel

## 🚀 本地运行

### 前置要求

- Node.js 22 LTS
- pnpm
- 一个 Base Sepolia 作者钱包，带少量测试 ETH
- 一个非作者读者钱包，带测试 USDC
- Coinbase CDP x402 facilitator 凭证

### 安装

```bash
pnpm install
cp .env.local.example .env.local
```

然后填入 `.env.local`。注意：

- `CDP_API_KEY_ID`、`CDP_API_KEY_SECRET` 和 `CONTENT_ENC_KEY` 只应存在服务端环境。
- `DEMO_AUTHOR_PRIVATE_KEY` 只用于本地 seed 脚本，不能使用主网私钥。
- `NEXT_PUBLIC_*` 会进入前端包，不要放任何秘密值。

### 启动

```bash
pnpm dev
```

打开 `http://localhost:3000`。

### 常用脚本

```bash
pnpm test
pnpm build
pnpm register-schema
pnpm seed yaoqian-crypto-liability web3-illegal-employment
pnpm reset-demo
```

脚本说明：

- `register-schema`：注册 EAS schema，并把 schema UID 回填到环境变量。
- `seed <slug...>`：把 seed 文章上链并写入本地目录索引。
- `reset-demo`：清空本地付款记录，并回到干净演示开场。

## Agent API

### 免费发现

```http
GET /api/v1/articles
```

支持过滤：

```http
GET /api/v1/articles?q=employment
GET /api/v1/articles?tag=AML
GET /api/v1/articles?author=Lawson
```

响应只包含元数据，不包含正文：

```json
{
  "count": 1,
  "articles": [
    {
      "slug": "yaoqian-crypto-liability",
      "title": "...",
      "summary": "...",
      "author": "...",
      "price": "$0.30",
      "priceUSDC": "300000",
      "attestationUID": "0x...",
      "read": "/api/v1/articles/yaoqian-crypto-liability"
    }
  ]
}
```

### 付费阅读

```http
GET /api/v1/articles/{slug}
```

流程：

1. 第一次请求返回 `402 Payment Required`。
2. Agent 钱包按 x402 payment requirements 支付精确金额。
3. 带付款凭证重试。
4. 成功后返回全文、companion 和链上 citation。

完整 agent 操作说明见 [public/SKILL.md](public/SKILL.md)。

## 📁 项目结构

```text
app/                    Next.js pages and API routes
components/             Home, report, publish, and shared UI components
content/reports/         Public metadata and encrypted paid article bodies
content/companions/      Agent Mode companion files
data/                   MVP JSON indexes and payment logs
docs/                   Hackathon spec, demo script, mockups, implementation plans
lib/                    EAS, x402, reports, markdown, crypto, leaderboard logic
public/SKILL.md          Public operating guide for AI agents
scripts/                Schema registration, seeding, encryption, demo reset
```

## 🧭 路线图

Citely 的长期方向不是停在单篇 demo，而是成为一个多作者的合规内容授权网络。当前仓库先提交 hackathon MVP，后续路线按 PRD v2.0 拆成几个层次：

### 当前提交 · Hackathon MVP

- 跑通一篇报告的完整闭环：作者签名上链、报告进目录、真人 x402 付费、Agent x402 付费、作者收益上升。
- 使用 Base Sepolia 测试网、EAS attestation、x402、Cobo Agentic Wallet / pact、加密正文和 Agent companion。
- 用本地 JSON 文件承载目录索引和付款日志，优先服务演示，不把它伪装成生产级存储。

### v1.0 · Citely 平台化

- **作者白名单 / 审核制入网**：冷启动阶段限定实名律师、合规研究员和合规相关大 V；文章必须有可验证原始来源，避免普通自媒体和低质量快讯涌入。
- **/creators 作者授权页**：5 分钟提交身份、钱包地址、原文 URL 和单篇价格；平台人工审核后上架。
- **多作者内容库**：目标至少 6 篇授权深度内容、3 位实名律师作者；文章覆盖执法、政策、牌照、制裁等方向。
- **动态 payTo**：每篇文章的 x402 收款地址指向对应作者钱包，平台不托管内容侧资金，内容侧 v1.0 阶段 0 抽成。
- **生产化数据层**：从本地 JSON 迁移到 Postgres / KV，补齐 payments、entitlements、authors 表。
- **工具侧收入**：平台收入主要来自冻卡诊断、Web3 求职 offer 风险评估等工具 API，而不是内容分成。
- **Agent 发现入口**：维护 `/SKILL.md`、`/llms.txt`、`/openapi.json` 和 `/agents`，让外部 Agent 能稳定发现和调用。

### v1.5 · 作者与律师网络扩展

- 上线律师目录、律师 profile、咨询入口和律师 insights。
- 把实名律师作者自然转化为律师目录的种子供给。
- 扩展英文主要内容，服务海外 Agent 和 Web3 团队。
- 作者规模扩展到 20+，文章规模扩展到 50+。

### v1.6 · 账号、订阅与作者后台

- 引入用户账号和跨设备 entitlement，让已购文章真正支持跨设备重读。
- 上线作者后台 v2：更细的数据分析、多文章批量定价、推广工具、文章上下架和调价。
- 探索 x402 周费 newsletter，而不是传统 Stripe 月订阅。
- 做 AI 跨文综合报告：检索多篇授权内容，展示引用源，并为被引用作者设计分账路径。

### v2 · 更大的合规情报网络

- 专题模块、多链结算、移动端体验和 x402 Bazaar 深度集成。
- 扩展更多工具：合约 review、项目尽调、监管文件结构化等。
- 继续保留核心边界：作者授权、来源可证、付款边界清楚、AI 输出必须带出处。

## 🧱 已知边界

- 当前演示运行在 **Base Sepolia 测试网**，不应当视为主网资金流。
- Vercel serverless 文件系统不适合持久写入；线上发布索引和收益榜应迁移到 KV、Redis 或数据库。详见 [DEPLOY.md](DEPLOY.md)。
- Citely 提供风险教育和来源验证基础设施，不提供法律意见。
- AI Agent 的付款必须受钱包策略约束；不要让 Agent 获得无限授权、主网私钥或不受限支付能力。
- 公开仓库中不应提交 `.env.local`、私钥、助记词、CDP secret、真实资金账户信息或未加密的付费正文。

## 👥 团队成员

待补充。团队信息会按最终 hackathon 提交口径确认后更新。

## 📜 许可

许可证待确认。复用、分发或商用前请先联系维护者确认授权边界。

---

<a id="english"></a>

## English

> Citely is an on-chain content licensing and pay-per-read platform for expert Web3 risk intelligence. This repository contains the hackathon MVP that proves one end-to-end report lifecycle.

[中文](#citely) | **English**

## Table of Contents

- [About The Project](#about-the-project)
- [Why Citely](#why-citely)
- [Demo Loop](#demo-loop)
- [Features](#features)
- [Built With](#built-with)
- [Getting Started](#getting-started)
- [Agent API](#agent-api-1)
- [Project Structure](#project-structure)
- [Roadmap](#roadmap)
- [Known Limits](#known-limits)
- [Team](#team)
- [License](#license)

## 🌐 About The Project

Citely is an on-chain content licensing and pay-per-read platform for expert Web3 compliance content. Lawyers, compliance advisors, security researchers, auditors, tax professionals, and domain analysts can publish reports as verifiable content: authorship, content hash, price, and disclaimer are recorded through EAS attestations; human readers and AI agents pay USDC through x402 to unlock the full text.

The product is intentionally narrow:

```text
author signs -> report enters catalog -> human and agent pay the same price -> full text unlocks -> revenue goes to author -> proof is verifiable on-chain
```

This repository contains the **hackathon MVP** for the Citely platform. It runs on Base Sepolia first and proves the core loop: creator authorization, on-chain provenance, human payment, agent payment, and visible author revenue. See [docs/HACKATHON.md](docs/HACKATHON.md) for the demo-loop spec and [docs/DEMO.md](docs/DEMO.md) for the 3-minute demo script.

## 🧩 Why Citely

High-quality Web3 risk analysis has three recurring problems:

- **Provenance is hard to verify**: readers need stronger proof of author identity, content integrity, and pricing.
- **Agents need a native content payment path**: subscriptions and API keys are a poor fit for small, on-demand, agent-initiated reads.
- **Platform custody creates friction**: authors and readers both depend on platform accounting and payout promises.

Citely separates those concerns:

- EAS records authorship, content hash, price, and version.
- x402 gives humans and AI agents the same pay-per-read path.
- USDC settles directly to the author wallet; the v1.0 content layer takes no platform cut.
- Cobo Agentic Wallet / pact can constrain agent spending by amount, recipient, and chain.

## 🔁 Demo Loop

1. **Publish**: the author imports a report, sets a price, signs, and creates an EAS attestation.
2. **Catalog**: the report appears in `/reports` and the homepage catalog with an on-chain / EAS badge.
3. **Human unlock**: a reader previews the article and pays test USDC through x402 to unlock the full text.
4. **Agent unlock**: an agent reads the public catalog, calls the paid endpoint, follows `402 -> pay -> 200`, and receives the full text plus companion.
5. **Visible revenue**: local payment records aggregate into the homepage Top Earning Authors leaderboard.
6. **Verification**: attestations can be checked in an EAS explorer, and settlements can be checked on Base Sepolia.

## ✨ Features

- **Verifiable publishing**: `contentHash + author + price + version + disclaimer` are recorded through EAS.
- **Public catalog**: `GET /api/v1/articles` returns free metadata, price, author, and read path.
- **x402 paid reading**: `GET /api/v1/articles/[slug]` returns 402 first, then full markdown after payment.
- **Encrypted paid body**: report bodies are stored as AES-256-GCM `.enc` files and decrypted server-side after payment.
- **Agent companion**: paid responses include glossary, legal map, misconception table, and citation metadata.
- **Same price for humans and agents**: price is resolved from the server-side attestation index, not request parameters.
- **Author leaderboard**: the local MVP aggregates `data/payment-log.json`.

## 🛠 Built With

- Next.js 16 App Router, React 19, TypeScript
- Tailwind CSS v4
- x402, Coinbase CDP facilitator, USDC on Base Sepolia
- wagmi, viem, ethers
- Ethereum Attestation Service (EAS)
- MDX metadata + encrypted paid body
- Vitest and Testing Library
- Vercel

## 🚀 Getting Started

### Prerequisites

- Node.js 22 LTS
- pnpm
- A Base Sepolia author wallet with test ETH
- A non-author reader wallet with test USDC
- Coinbase CDP x402 facilitator credentials

### Installation

```bash
pnpm install
cp .env.local.example .env.local
```

Fill in `.env.local`.

- `CDP_API_KEY_ID`, `CDP_API_KEY_SECRET`, and `CONTENT_ENC_KEY` must stay server-side.
- `DEMO_AUTHOR_PRIVATE_KEY` is only for local seed scripts. Never use a mainnet key.
- `NEXT_PUBLIC_*` values are bundled into the browser, so they must never contain secrets.

### Run Locally

```bash
pnpm dev
```

Open `http://localhost:3000`.

### Useful Scripts

```bash
pnpm test
pnpm build
pnpm register-schema
pnpm seed yaoqian-crypto-liability web3-illegal-employment
pnpm reset-demo
```

## Agent API

### Free Discovery

```http
GET /api/v1/articles
```

Filters:

```http
GET /api/v1/articles?q=employment
GET /api/v1/articles?tag=AML
GET /api/v1/articles?author=Lawson
```

The response contains metadata only:

```json
{
  "count": 1,
  "articles": [
    {
      "slug": "yaoqian-crypto-liability",
      "title": "...",
      "summary": "...",
      "author": "...",
      "price": "$0.30",
      "priceUSDC": "300000",
      "attestationUID": "0x...",
      "read": "/api/v1/articles/yaoqian-crypto-liability"
    }
  ]
}
```

### Paid Reading

```http
GET /api/v1/articles/{slug}
```

Flow:

1. The first request returns `402 Payment Required`.
2. The agent wallet pays the exact x402 requirement.
3. The agent retries with the payment proof.
4. The successful response contains full markdown, companion, and on-chain citation metadata.

See [public/SKILL.md](public/SKILL.md) for the agent operating guide.

## 📁 Project Structure

```text
app/                    Next.js pages and API routes
components/             Home, report, publish, and shared UI components
content/reports/         Public metadata and encrypted paid article bodies
content/companions/      Agent Mode companion files
data/                   MVP JSON indexes and payment logs
docs/                   Hackathon spec, demo script, mockups, implementation plans
lib/                    EAS, x402, reports, markdown, crypto, leaderboard logic
public/SKILL.md          Public operating guide for AI agents
scripts/                Schema registration, seeding, encryption, demo reset
```

## 🧭 Roadmap

Citely is intended to grow from this single-report demo into a multi-author content rights network.

### Current Submission · Hackathon MVP

- Prove one full report lifecycle: author attestation, catalog listing, human x402 unlock, agent x402 unlock, and visible author revenue.
- Use Base Sepolia, EAS, x402, Cobo Agentic Wallet / pact, encrypted paid content, and Agent Mode companion data.
- Keep JSON files as demo storage only; do not present them as production infrastructure.

### v1.0 · Platformization

- **Author allowlist / review-based onboarding**: start with verified lawyers, compliance researchers, and established compliance creators.
- **/creators onboarding**: authors submit identity, wallet address, original URL, and per-article price; the platform reviews before listing.
- **Multi-author report library**: target at least 6 licensed deep-dive reports and 3 verified lawyer authors at launch.
- **Dynamic payTo**: each article routes x402 settlement to that author's wallet; the platform does not custody content-side funds.
- **Production data layer**: move indexes, payments, entitlements, and authors into Postgres / KV.
- **Tool-side monetization**: platform revenue comes from tools such as freeze diagnosis and Web3 job-offer risk assessment, not v1.0 content commissions.
- **Agent discovery**: maintain `/SKILL.md`, `/llms.txt`, `/openapi.json`, and `/agents`.

### v1.5 · Creator And Lawyer Network

- Launch lawyer directory, lawyer profiles, consultation entry points, and lawyer insights.
- Use verified lawyer authors as the first supply base for the directory.
- Expand English content for overseas agents and Web3 teams.
- Grow toward 20+ authors and 50+ reports.

### v1.6 · Accounts, Subscription, And Author Dashboard

- Add user accounts and cross-device entitlements for already purchased reports.
- Ship author dashboard v2 with deeper analytics, bulk pricing, promotion tools, delisting, and repricing.
- Explore x402 weekly newsletter access rather than Stripe monthly subscriptions.
- Explore AI cross-report synthesis with source attribution and author payout logic.

### v2 · Larger Compliance Intelligence Network

- Add topic modules, multi-chain settlement, mobile experience, and deeper x402 Bazaar integration.
- Expand tools such as contract review, project due diligence, and regulatory document structuring.
- Preserve the core boundary: licensed authors, verifiable provenance, constrained payment authority, and cited AI output.

## 🧱 Known Limits

- The current demo runs on **Base Sepolia testnet** and should not be treated as a mainnet payment flow.
- Vercel serverless file storage is not persistent; production attestation indexes and payment logs should move to KV, Redis, or a database. See [DEPLOY.md](DEPLOY.md).
- Citely provides risk education and provenance infrastructure, not legal advice.
- AI agent payments must be constrained by wallet policy. Do not grant unlimited approvals, mainnet private keys, or unbounded payment authority.
- Do not commit `.env.local`, private keys, seed phrases, CDP secrets, real-funds account data, or unencrypted paid article bodies.

## 👥 Team

To be added. Team details will be confirmed and updated according to the final hackathon submission.

## 📜 License

License to be confirmed. Please contact the maintainers before reuse, distribution, or commercial use.
