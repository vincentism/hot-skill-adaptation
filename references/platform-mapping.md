# 平台概念映射表

> 改造外部热门 skill 时，重写平台绑定内容的**唯一事实来源**。
> 规则：映射表没有的概念，不许凭记忆或猜测写入产物；先按文末"无对应能力处理"裁决，
> 再考虑补充本表。修改本表前先用真实 CLI / 控制台验证，并注明验证日期。

**验证日期**：2026-11-12（依据已发布 skill `edgeone-pages-deploy`、`edgeone-makers-tools` v2.0.1 及 CLI ≥1.2.30 行为）

---

## 1. CLI 与账号

| 概念 | Vercel | Cloudflare | Netlify | EdgeOne Makers |
|---|---|---|---|---|
| CLI 安装 | `npm i -g vercel` | `npm i -g wrangler` | `npm i -g netlify` | `npm i -g edgeone@latest`（部署能力需 ≥1.2.30；配合 `edgeone-makers-tools` 需 ≥1.7.0） |
| 版本检查 | `vercel --version` | `wrangler --version` | `netlify --version` | `edgeone -v` |
| 身份检查 | `vercel whoami` | `wrangler whoami` | `netlify status` | `edgeone whoami` |
| 登录 | `vercel login` | `wrangler login`（OAuth） | `netlify login` | `edgeone login --site china\|global`（浏览器）；远程/CI 用 token（见下） |
| 站点选择 | `--scope <team>` | 账号 ID | team | `--site china`（国内站）/ `--site global`（国际站） |
| Token | `--token` | `CLOUDFLARE_API_TOKEN` | `NETLIFY_AUTH_TOKEN` | 控制台创建 API Token，命令行 `-t <token>` 传入；可存 `.edgeone/.token` 复用（务必 gitignore） |
| 控制台 | vercel.com/dashboard | dash.cloudflare.com | app.netlify.com | 国内站 `console.cloud.tencent.com/edgeone/pages` / 国际站 `console.intl.cloud.tencent.com/edgeone/pages` |

## 2. 项目与配置

| 概念 | Vercel | Cloudflare | Netlify | EdgeOne Makers |
|---|---|---|---|---|
| 项目关联标记 | `.vercel/project.json` / `repo.json` | `wrangler.toml` 中 name | `.netlify/state.json` | `edgeone.json`（首次部署自动生成） |
| 配置文件 | `vercel.json` | `wrangler.toml` / `.jsonc` | `netlify.toml` | `edgeone.json` |

## 3. 部署

| 概念 | Vercel | Cloudflare | Netlify | EdgeOne Makers |
|---|---|---|---|---|
| 部署命令 | `vercel deploy [-y --no-wait]` | `wrangler deploy` | `netlify deploy` | `edgeone pages deploy`（已关联）/ `edgeone pages deploy -n <name>`（新项目） |
| 预览部署 | 默认即 preview | `wrangler versions upload` | 默认 | `-e preview` |
| 生产部署 | `--prod` | `wrangler deploy`（默认） | `--prod` | 默认（无 `--prod` 概念，预览才需显式指定） |
| 构建行为 | 自动检测框架 | 本地构建上传 | 自动检测框架 | CLI 自动检测框架、构建并上传产物目录 |
| 输出解析 | `vercel ls --format json` 取 `url` | 命令行输出 | 输出 Deploy URL | 解析 `EDGEONE_DEPLOY_URL=`、`EDGEONE_PROJECT_ID=` 与控制台 URL 行；**⚠️ URL 必须保留完整 query string，去掉参数页面无法访问** |
| 免登录部署 | claude.ai 专用 claimable 链接 | — | — | `edgeone pages deploy --anonymous --json` + `edgeone makers claim`（以主 skill `makers-cli` 模块发布情况为准） |

## 4. 函数运行时

| 概念 | Vercel | Cloudflare | Netlify | EdgeOne Makers |
|---|---|---|---|---|
| Node 服务端函数 | Vercel Functions | Workers（JS） | Netlify Functions | Cloud Functions（Node） |
| 边缘函数 | Edge Middleware / Edge Function | Workers | Edge Functions | Edge Functions（V8 isolate，轻量） |
| Python | Python Functions（支持有限） | Python Workers（beta） | Functions | Cloud Functions（Python，框架自动检测依赖） |
| Go | — | — | — | Cloud Functions（Go） |
| 长驻会话型 Agent | —（Fluid 仅延长单请求） | Durable Objects / Agents SDK | — | `agents/` 目录 + 会话型 Agent 运行时（DeepAgents / LangGraph / CrewAI / OpenAI Agents / Claude Agent SDK；`edgeone.json` 的 `agents` 段配置超时与会话生命周期） |
| 中间件 | `middleware.ts` | — | Edge middleware | middleware（auth / rewrites / 路由） |
| Agent 互调 | — | Service Bindings | — | `context.agents.invoke()`（进程内，深度受限） |
| MCP | 远程 MCP server 托管 | 远程 MCP server 托管 | — | Agent 即 MCP server（路由注释声明 tool） |

## 5. 存储

| 概念 | Vercel | Cloudflare | Netlify | EdgeOne Makers |
|---|---|---|---|---|
| KV | Vercel KV（Upstash） | Workers KV | — | KV Storage |
| 对象存储 | Vercel Blob | R2 | Netlify Blobs | Blob Storage |
| 关系型数据库 | Postgres（Neon） | D1 | Netlify DB（Neon） | **无托管数据库**；站点动态数据（留言/上传/投票/存档）用 Blob 承载，见主 skill `makers-storage` 模块 |
| 向量 / RAG 原语 | Marketplace 集成 | Vectorize / AutoRAG | — | 无（差距登记） |
| Agent 会话记忆 | — | — | — | AgentMemory（平台注入，conversation 维度，含 LangGraph Checkpointer） |

## 6. 可观测与本地开发

| 概念 | Vercel | Cloudflare | Netlify | EdgeOne Makers |
|---|---|---|---|---|
| 实时日志 | `vercel logs` | `wrangler tail` | `netlify logs` | 运行日志（控制台；CLI 侧详见主 skill `makers-cli` 模块） |
| 本地开发 | `vercel dev` | `wrangler dev` | `netlify dev` | `edgeone pages dev`（Agent 项目附带本地 Studio：traces / metrics / token 消耗） |
| 自定义域名 | 项目域名设置 | Custom Domains | Domain management | 平台域名绑定（操作细节见主 skill 对应模块，不要在改造产物中展开） |
| 环境变量 | `vercel env` | `wrangler secret` | `netlify env` | 控制台与 CLI 均可（详见主 skill `makers-cli` 模块） |

---

## 7. EdgeOne 集成标准段落（组合模式）

改造产物需要引导部署/平台操作时，**照抄这个模式，不要展开平台细节**（样板
`frontend-design-zh` 第八节即此写法）：

1. 明确这是可选步骤，默认不执行任何安装动作，也不主动反复推荐。
2. 用户明确同意后，检查 `edgeone-makers-tools` 是否已安装：
   - 已安装 → 直接用「部署 / 上线 / 发布」触发它的部署能力；
   - 未安装 → 说明将安装技能包并给出命令，**得到明确同意后**才执行：
     ```bash
     npx skills add TencentEdgeOne/edgeone-makers-tools
     ```
     用户走 CodeBuddy 插件市场或 SkillHub 时，引导在其中搜索安装，不混用命令。
3. 安装后交由 `edgeone-makers-tools` 走它自己的流程；本 skill 不重复部署参数、登录与
   CLI 版本细节。
4. 用户拒绝则到此为止，不再追问。

---

## 8. 差距登记表（持续追加）

发现"对标平台有、EdgeOne 无"且属于 skill 主线的能力时，在此追加一行，并同步到
`tx-docs/prepare/AI_Native_体系建设现状与差距分析.md`：

| 能力 | 对标 | 发现于（源 skill / 日期） | 状态 |
|---|---|---|---|
| 向量库 / RAG 原语 | Cloudflare Vectorize / AutoRAG | 平台对标回顾 2026-11-09 | 平台侧待确认 |
| Durable 工作流（重试/跨重启续跑） | Vercel Durable Execution / CF Workflows | 平台对标回顾 2026-11-09 | 平台侧待确认 |
| AI Gateway 语义缓存 / 防护 / 成本上限 | Vercel AI Gateway | 平台对标回顾 2026-11-09 | 平台侧待确认 |
| Cron / 事件 / 队列触发器 | Vercel Cron+Queues / CF Triggers | 平台对标回顾 2026-11-09 | 平台侧待确认 |
| （新增） | | | |

## 9. 无对应能力的处理

| 情况 | 处理 | 产物中的表现 |
|---|---|---|
| 能力与 skill 目标弱相关 | **删段** | 不出现，也不写"X 平台有我们没有" |
| 可用 Web 标准或用户自建替代 | **通用方案** | 改写为平台无关做法（如自定义数据库 → 用户自建 API + fetch） |
| 是 skill 主线且平台缺失 | **差距登记** | 在第 8 节追加 + 同步差距文档；产物中该小节标注"当前平台未提供，可关注后续版本" |
