---
slug: hot-skill-adaptation
name: hot-skill-adaptation
displayName: 热门 Skill 改造指南
version: 1.0.0
license: MIT
description: >-
  指导把外部热门 AI Skill（Vercel、Cloudflare、Netlify、Anthropic 官方技能库及社区高分技能）
  改造为 EdgeOne Makers 生态版本，覆盖选品评估、改造决策、平台概念映射重写、中文语境增强、
  元数据规范、验收与发布全流程。当用户要「改造 skill」「移植 skill」「把 vercel / cloudflare /
  netlify 的 skill 换成 EdgeOne 版本」「适配热门 skill」「汉化并增强 skill」
  或 "port / adapt a skill to EdgeOne Makers" 时使用。
  不适用于：从零编写全新 skill（无需改造流程，直接按 Skill 规范编写）、
  审计或评估已有 skill（使用 skill-optimizer）。
metadata:
  author: edgeone
  version: "1.0.0"
---

# 热门 Skill 改造指南

把一个外部热门 skill 改造成 EdgeOne Makers 生态版本。改造对象是"经过市场验证的流程与方法论"，
不是平台命令的翻译练习。产出物 = 保留源 skill 的流程精华 + EdgeOne 平台正确性 + 中文语境增强。

**改造前必读三份本地参照**：

| 材料 | 位置 | 用途 |
|---|---|---|
| 改造样板案例 | `hot-skills/frontend-design-zh/` | 唯一已验收的完整改造实例，本文所有规范均有对应 |
| 平台映射表 | 本 skill 的 [references/platform-mapping.md](references/platform-mapping.md) | 重写平台绑定内容时的唯一事实来源 |
| 团队主 skill | `edgeone-makers-tools/`（1 入口 + 11 模块） | 判定"组合不复制"边界与选品重叠 |

---

## 一、改造四原则

违反任何一条即返工：

1. **换骨架不换灵魂**：源 skill 的决策树、检查顺序、错误处理流是它"好用"的原因，保留结构；
   只替换平台绑定的命令、路径、配置和服务。禁止把决策树摊平成知识罗列。
2. **组合不复制**：EdgeOne 部署、存储、函数等平台能力，一律引导触发 `edgeone-makers-tools`
   对应模块，**不在本 skill 内复制任何平台操作知识**（命令参数、登录流程、输出解析等）。
   知识双源必然漂移，这是团队已知的一号问题。
3. **本地化不直译**：中文语境增强是加"中文世界才有的判断"（字体子集化、中文排版规范、
   反模板化清单），不是把英文段落机翻。宁缺毋滥，没有真实增强点就不硬加。
4. **归因与合规**：保留上游 license 兼容声明，写 `NOTICE` 注明来源仓库与改造范围；
   上游是 MIT/Apache 才可直接改造，遇 Copyleft 或无 license 先停下问负责人。

---

## 二、改造流程（按序执行，不跳步）

### Step 1 选品评估

从来源池挑选候选，先评估再动手：

| 来源池 | 本仓库路径 |
|---|---|
| Vercel 官方 | `vercel-skill-master/.agents/skills/`（deploy-to-vercel、react-best-practices、composition-patterns 等） |
| Cloudflare 官方 | `cf-skill-master/.agents/skills/`（agents-sdk、wrangler、workers-best-practices、web-perf 等） |
| Netlify 官方 | `context-and-tools/skills/`（deploy、functions、blobs、edge-functions 等） |
| Anthropic / 社区 | `agent-skills/skills/`、`hot-skills/` 新增候选 |

评估四问，任一为"否"则换下一个候选：

1. **通用价值**：脱离原平台后，skill 的核心方法论对 EdgeOne 用户仍然成立吗？
2. **重叠检查**：与 `edgeone-makers-tools` 现有 11 模块（frameworks / agents / migration /
   deploy / edge-functions / cloud-functions / storage / middleware / cli / recipes /
   env-adaption）是否大面积重叠？重叠则放弃，或改为向主 skill 补充模块而不是另立 skill。
3. **增强空间**：中文语境或 EdgeOne 能力有没有真实增益点（见样板：中文排版 + 设计令牌脚本
   + EdgeOne 预览引导三重增强）？
4. **热度与需求**：是否高频使用场景、有用户请求或对标平台官方维护？

**产出：改造立项卡**（一张表）——源 skill 名称与来源、目标用户场景、四问结论、
预期增强点清单、初版改造决策表（Step 3 的空表）。

### Step 2 研读源 skill

通读源 `SKILL.md` 及其 `references/`、`scripts/`、`resources/`，盘点 **五类平台绑定点**，
逐项登记位置：

| 绑定点 | 识别方法 | 例（deploy-to-vercel） |
|---|---|---|
| CLI 命令与参数 | 代码块中的 shell 命令 | `vercel deploy -y --no-wait` |
| 配置文件与路径 | 文件路径字面量 | `.vercel/project.json` |
| 平台专有服务 | 概念名词 | Vercel KV / Blob / Fluid |
| 宿主环境段落 | 面向特定 Agent 宿主的说明 | claude.ai sandbox 的 `/mnt/skills/` 路径 |
| 平台输出解析 | 对命令输出格式的依赖 | `vercel ls --format json` 的 url 字段 |

同时记录源 skill 的**流程骨架**：分几步、每步的判定条件、哪些环节是它做得特别好的。

### Step 3 填改造决策表

对每个绑定点和每个章节做四选一裁决，写进立项卡：

| 裁决 | 适用 | 样板案例对应 |
|---|---|---|
| **保留** | 与平台无关的方法论 | 视觉原则、"先出方向卡再写代码"两遍流程 |
| **替换** | 有 EdgeOne 等价物的平台绑定内容 | 预览发布 → 引导 `edgeone-makers-tools`（第八节） |
| **增强** | 中文语境 / EdgeOne 独有增益 | 中文排版、`design-tokens.json` + 令牌生成脚本 |
| **删除** | 无对应能力且无替代价值 | claude.ai / Codex sandbox 宿主专属段落 |

删除和替换必须写理由；"无对应能力"本身就是重要信息，登记到 Step 4 的差距清单。

### Step 4 平台映射重写

按 [references/platform-mapping.md](references/platform-mapping.md) 重写所有"替换"项。
该文件是平台事实的唯一来源，**不要凭记忆或猜测写 EdgeOne 命令**；映射表没有的概念按三种方式处理：

1. **删段**：能力与 skill 目标弱相关 → 直接删，不留"Vercel 有我们没有"的说明。
2. **通用方案**：能力可用 Web 标准或用户自建替代 → 改写为通用做法，去掉平台绑定。
3. **差距登记**：能力是 skill 主线且 EdgeOne 确实缺失 → 在映射表"差距清单"追加一行，
   并同步到团队差距分析文档（`tx-docs/prepare/AI_Native_体系建设现状与差距分析.md`），
   这是向平台侧提需求的输入。

### Step 5 中文语境增强

按立项卡的增强点清单逐项落地，常见维度（不要求全做）：

- **排版**：中文字体族与 fallback、子集化与 `font-display`、正文行高 1.7–1.9、行长 25–40 字
- **文案规范**：界面文案的中文写法（CTA 明确动作、同一动作同名、错误指路不道歉）
- **反模板化**：针对中文 AI 生成内容的"一眼假"清单（见样板第五节）
- **双语触发词**：`description` 里中英触发词并列（"做个页面 / 改好看点 / design a page"）
- **示例本地化**：示例内容、占位文本用真实中文语境，不机翻

每个增强点必须能回答"这在英文原版里为什么没有"——答不出来说明是凑数，删掉。

### Step 6 目录与元数据规范

产物放在 `hot-skills/<skill-name>/`，目录结构：

```
<skill-name>/
├── SKILL.md          # 必须：frontmatter + 正文（精简，细节下沉 references/）
├── references/       # 可选：详细参考文档，按需加载
├── scripts/          # 可选：纯 Node 零依赖脚本（.mjs），支持 --help
├── assets/           # 可选：模板等输出用文件
├── LICENSE.txt       # 必须：与上游 license 兼容
└── NOTICE            # 必须：上游来源、原 license、改造范围说明
```

frontmatter 采用**双字段合并格式**（SkillHub 与 npx skills / Claude Code 各读各的）：

```yaml
---
slug: <skill-name>            # SkillHub 必需
name: <skill-name>            # CodeBuddy / Claude 必需
displayName: <中文名>
version: 1.0.0
license: MIT                  # 须与上游兼容
description: >-               # 触发描述：三要素（做什么/何时用/不适用）+ 中英触发词
metadata:
  author: edgeone
  version: "1.0.0"
---
```

`description` 写法：第三人称陈述"何时应使用本 skill"，明确列出触发词与**不适用边界**
（防止与其他 skill 抢触发），参考本文件自身的 description。

### Step 7 验收（四道门，全过才可发布）

| 门 | 做法 | 工具 |
|---|---|---|
| 1. 触发校验 | 构造正反例测试集，验证 description 触发准确、不误触 | `skill-optimizer` 触发校验模式 |
| 2. 真实任务走查 | 至少 1 个端到端场景在真实宿主跑通（如样板的"一句话 brief → 方向卡 → 代码"） | 人工 + 宿主 Agent |
| 3. 双源检查 | `rg -in "edgeone (pages )?(deploy|login)|EDGEONE_DEPLOY_URL" <产物目录>` 命中数必须为 0（平台操作知识只允许出现在映射表和主 skill 中） | rg |
| 4. 脚本执行 | `node scripts/*.mjs --help` 逐个跑通；零第三方依赖 | node |

另跑一遍 `skill-optimizer` 的质量安全审计（8 维度 42 项），🔴 Error 项必须清零。

### Step 8 发布与登记

1. 提交到 `hot-skills/` 仓库，commit message 注明源 skill 与改造要点。
2. 发布到 SkillHub：参照 `edgeone-makers-tools/BRANCH.md` 的 `main` / `skillhub`
   双分支模式（`npx skills add` 走 main，SkillHub / CodeBuddy 市场走 skillhub 分支）。
3. 在团队能力清单中登记：skill 名、负责人、版本、来源上游、成熟度（首版标"已验收未长期运行"）。
4. 上游 skill 更新时复查：关注上游仓库的新版本，评估是否同步改造（登记时记下上游版本号）。

---

## 三、参考案例：frontend-design-zh 改造要点

源：Anthropic `web-design-guidelines` / 社区 frontend-design 系（通用 UI 设计方法论）
产物：`hot-skills/frontend-design-zh`（`distinctive-ui-design-zh`，界面设计交付 · 中文增强版）

| 改造动作 | 内容 |
|---|---|
| 保留 | 设计工作室角色设定、"先方向卡后代码"两遍流程、视觉原则、可访问性走查 |
| 替换 | 发布预览 → 引导安装/触发 `edgeone-makers-tools`（`npx skills add TencentEdgeOne/edgeone-makers-tools`），不复制部署知识 |
| 增强 | 中文排版（字体子集化/行高/行长）、中文文案规范、中文反模板化清单、`design-tokens.json` + `tokens-to-css.mjs` 令牌体系、`contrast.mjs` 对比度检查 |
| 删除 | 无（该源 skill 平台绑定弱） |
| 合规 | `NOTICE` 注明上游来源与改造范围 |

遇到拿不准的裁决时，先回到这个案例找对应做法。
