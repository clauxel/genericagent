# GenericAgent Launch 开发者文档

**语言:** [English](README.md) | **简体中文**

> 这套文档面向当前 GenericAgent Launch 仓库：React/Vite 前端、Node.js 控制面、下单支付、远程部署、工作区 console、analytics、管理后台和运维脚本。

最后按本地仓库结构核对时间：2026-04-22。

在线体验 GenericAgent：<https://www.genericagent.org/>

## 有用性结论

旧版文档主要在讲上游 Python 版 GenericAgent runtime，会把开发者带到 `mykey.py`、`agentmain.py`、`launch.pyw` 这类当前仓库并不负责的入口。

现在这套文档改成服务真实开发工作流：

- 怎么本地跑起来；
- 哪些 Node/Vite 入口真的存在；
- `.env.*` 和 `genericagent.config.json` 到底怎么生效；
- checkout、支付、订单、部署、console、admin、analytics、model proxy 分别在哪改；
- PostgreSQL、memory driver、mock deployment、SSH deployment、router mode 的边界是什么；
- 改价格、模型、渠道、页面文案、API、部署脚本时应该从哪里下手；
- 改完用什么命令验证。

## 当前仓库负责什么

GenericAgent Launch 不是上游 agent runtime 源码镜像，而是围绕 GenericAgent 工作区的 hosted launch / checkout / provisioning / console 控制面。

它负责：

- 营销页、方案页、价格页、对比页、FAQ、法律页和 SEO；
- launch checkout、支付确认、订单状态和部署队列；
- 工作区 console 入口、升级、停止、卸载和重新部署；
- 通过 mock 或 SSH runtime 交付远程工作区；
- PostgreSQL 应用状态，以及测试用 memory driver；
- model proxy、analytics、管理员功能和运行时元数据。

## 推荐先读

| 文档 | 最适合谁 |
| --- | --- |
| [guide/installation.md](guide/installation.md) | 想干净搭好本地环境的人 |
| [guide/quickstart.md](guide/quickstart.md) | 想最快跑通 app 和第一条 checkout 流的人 |
| [guide/configuration.md](guide/configuration.md) | 想弄清 env、配置优先级、支付、数据库、部署凭据的人 |
| [guide/deployment.md](guide/deployment.md) | 想部署到生产、接 router、走 SSH provisioning 的人 |
| [guide/troubleshooting.md](guide/troubleshooting.md) | 遇到启动、数据库、支付、部署、console 问题的人 |
| [developer-usefulness-audit.md](developer-usefulness-audit.md) | 想看这套文档为什么达到 10 分有用的人 |
| [developer/architecture.md](developer/architecture.md) | 想读源码、改模块、做二开的开发者 |
| [features/agent-loop.md](features/agent-loop.md) | 想理解从下单到工作区可用的完整生命周期 |
| [features/frontends.md](features/frontends.md) | 想改前端、checkout、console、admin、SEO 的人 |
| [features/layered-memory.md](features/layered-memory.md) | 想理解环境、配置、数据库、mock/远程状态怎么保存的人 |
| [features/reflection.md](features/reflection.md) | 想理解 mock/SSH/router/lifecycle 自动化的人 |
| [reference/runtime-modes.md](reference/runtime-modes.md) | 想查 `npm` scripts 和运行入口的人 |
| [reference/config.md](reference/config.md) | 想快速查配置和环境变量的人 |
| [reference/prompt-recipes.md](reference/prompt-recipes.md) | 想用 AI 辅助读库和改库的人 |

## 5 分钟本地启动

安装依赖：

```bash
npm install
```

准备部署配置：

```bash
cp genericagent.config.example.json genericagent.config.json
```

创建 `.env.development`。如果只是本地开发 UI 和 API，可以先用 memory driver 与模拟部署：

```bash
PORT=5175
APP_ORIGIN=http://localhost:5175
MULTICA_POSTGRES_DRIVER=memory
MULTICA_POSTGRES_MEMORY_ID=genericagent-dev
MULTICA_ALLOW_SIMULATED_DEPLOYMENT=true
MULTICA_TOKEN_SECRET=replace-with-a-long-dev-token-secret
MULTICA_CONFIG_SECRET=replace-with-a-long-dev-config-secret
PAYMENT_PROVIDER=creem
CREEM_ENV=test
API_TEST_KEY=
```

启动：

```bash
npm run dev
```

访问 `.env.development` 里的 `PORT`。如果没有设置 `PORT`，`server.mjs` 默认使用 `5175`。

提交前验证：

```bash
npm run build
npm test
```

## 最重要的文件

| 路径 | 用途 |
| --- | --- |
| `server.mjs` | Node 主入口、env 加载、数据库初始化、API 挂载、Vite 集成、console proxy 和 SSR fallback |
| `src/App.tsx` | React 主应用和路由渲染 |
| `src/content/` | 页面文案、FAQ、法律页、产品叙事 |
| `src/lib/api.ts` | 浏览器侧 API client |
| `server-lib/api/` | auth、catalog、orders、webhooks、analytics、admin、model proxy 路由 |
| `server-lib/app-database.mjs` | PostgreSQL / memory driver、schema 创建和迁移 |
| `server-lib/deployment-config.mjs` | 部署配置解析与标准化 |
| `server-lib/deployment-runtime.mjs` | mock、SSH、router-aware 部署、升级、停止、卸载 |
| `shared/catalog.mjs` | 套餐、模型、渠道、价格、折扣的前后端共享目录 |
| `scripts/` | 部署、打包、数据库、dev tunnel、环境重置脚本 |
| `deploy/` | 生产 env、systemd、router、nginx、Vercel 示例 |
| `test/` | Node 测试套件 |

## 开发任务从哪里开始

| 任务 | 先看 | 再看 |
| --- | --- | --- |
| 改套餐、价格、模型、渠道 | `shared/catalog.mjs` | `src/content/catalog.ts`、checkout 测试 |
| 改页面文案 | `src/content/site-content.tsx` | `src/lib/seo.ts` |
| 新增或修改 API | `server-lib/api/index.mjs` 和对应 route module | `test/` |
| 改 checkout | `server-lib/api/order-routes.mjs` | `server-lib/payment-helpers.mjs`、`api/launch-checkout.mjs` |
| 改支付或 webhook | `server-lib/payment-helpers.mjs`、`server-lib/api/webhook-routes.mjs` | 支付 sandbox 配置 |
| 改部署逻辑 | `server-lib/deployment-runtime.mjs` | `server-lib/deployment-config.mjs`、`scripts/deploy-order-to-real-server.mjs` |
| 改生产拓扑 | `deploy/`、`server-router.mjs` | `server-lib/multica-instance-router.mjs` |

## 现实提醒

- 推荐配置文件是 `genericagent.config.json`，旧的 `multica.config.json` 仍兼容。
- 推荐显式 env 变量是 `GENERICAGENT_ENV_PATH`，旧的 `MULTICA_ENV_PATH` 仍兼容。
- 很多运行时变量仍叫 `MULTICA_*`，这是迁移兼容层，不是文档写错。
- 本地开发可以用 `MULTICA_POSTGRES_DRIVER=memory`，生产应使用 PostgreSQL。
- mock deployment 只适合开发；SSH deployment 会真的写远程工作区、systemd unit、router record 和 env 文件。
- 不要把真实 API key、root 密码、支付密钥写进公开文档或示例。
