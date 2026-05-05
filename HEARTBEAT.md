# HEARTBEAT.md - 心跳任务清单

> 所有定时任务统一在这里管理，避免碎片化。
> 心跳检查时，agent 会读取此文件并执行任务。

---

## 🌅 每日晨报（北京时间 07:15）

**任务：跨境新闻日报**

- 搜索昨天全球跨境圈热点新闻
- 整理最多 20 条，包含标题、热点要点、原文链接
- 过滤 3 天以上的旧新闻
- 输出格式清晰，标注发布日期
- ⚠️ **星期几必须用 `date` 命令获取**，不要自己推算！之前出现过算错星期的 bug（04-29 写成周二）

**关键词覆盖：**
- 跨境电商、出海、Amazon FBA、Shopify
- DTC 品牌、跨境物流、TikTok Shop、Temu、Shein

**输出流程：**
1. 搜索新闻 → web_search
2. 🆕 **验证链接质量** → 检查链接是否导向正常新闻页面，排除广告/垃圾页面
3. 生成 HTML 文件 → `~/workspace/reports/crossborder-daily-YYYY-MM-DD.html`
4. **部署前核查** → 用 `grep -i "排除词"` 验证文件不包含误分类内容（如 Oracle/Samsung/科技财报）
5. 推送到 GitHub Pages → `bash ~/workspace/scripts/push-report.sh crossborder YYYY-MM-DD`（必须用脚本，不要手动 git push）
6. **部署后核查** → `web_fetch` 验证线上版本内容正确
7. **🔴🔴🔴 强制发送 Telegram 链接（最先做！）**
   ⚠️ **必须在输出任何总结文本之前** 先调用 `message` 工具！
   ⚠️ **不要等到最后才发** — 后面有异步命令完成事件会进来，session 会被 HEARTBEAT_OK 覆盖！
   ⚠️ **如果只输出文本而不调用 message 工具 = 用户收不到消息！**

   **必须调用：**
   ```
   message action=send target=telegram:728346555 message="📰 跨境日报 YYYY-MM-DD 完成，共 X 条新闻。\nhttps://zxy114.github.io/ai-news-daily/published/crossborder-daily-YYYY-MM-DD.html"
   ```

   **⚠️ 重要规则：**
   - URL 必须单独一行，绝对不能用反引号/代码格式包裹
   - 这是部署验证成功后的 **第一步**，不是最后一步！先发链接，再输出摘要
   - 调用 `message` 后如果收到异步命令完成事件，回复 HEARTBEAT_OK 即可

**🆕 内容质量要求：**
- **头条新闻**：可详写，5 个维度完整展开
- **非头条新闻**：**不得简略**！每条必须包含 5 个维度（核心事件、关键数据、影响范围、后续动作、解读分析），每个维度至少 1-2 句话
- 禁止只写标题 + 一句话摘要
- 解读分析必须有自己的观点，不要只是复述新闻
- **🆕 每条新闻必须标注新闻源发布日期**（格式：📅 发布日期：YYYY-MM-DD）
- **🆕 严格 3 天过滤**：新闻源发布日期距今超过 3 天的，一律不收录

**链接质量验证规则：**
- ✅ 优先选择：Reuters, Bloomberg, TechCrunch, 36氪, 虎嗅, 机器之心
- ⚠️ 谨慎使用：新浪、网易、搜狐（需验证页面质量）
- ❌ 排除：广告页面、公众号链接、无法访问的链接

**配置：** 已通过 cron 任务自动执行

---

## 🤖 AI 新闻日报（北京时间 09:00）

**任务：AI 领域热点新闻（双语交叉验证版）**

**工作流程：**
1. **英文源搜索**：Brave/Google 搜索过去 3 天 AI 新闻
2. **中文源验证**：对每条英文新闻在中文互联网搜索确认（百度/微信/36 氪/机器之心等）
3. **🆕 Twitter/X 热度验证**：用 twitter-cli 搜索相关新闻在 Twitter 上的讨论热度
   - 环境变量：`TWITTER_AUTH_TOKEN` 和 `TWITTER_CT0`（从 `~/workspace/config/xreach.json` 读取）
   - 命令：`TWITTER_AUTH_TOKEN="xxx" TWITTER_CT0="xxx" twitter search "关键词" -n 5`
   - 检查推文数量、互动量、关键 KOL 是否讨论
4. **信息比对**：检查标题一致性、数据准确性、时效性
5. **信息标记**：✅已验证（中英文+Twitter 热度一致）/ ⚠️单方报道（仅一方有）/ ❌排除（错误信息）

**🆕 严格日期过滤规则：**
- ✅ 只包含 **最近 3 天** 的新闻（发布日期在今天 - 3 天内）
- ❌ 排除所有超过 3 天的旧新闻，无论多重要
- ⚠️ **每条新闻必须标注新闻源发布日期**（格式：📅 发布日期：YYYY-MM-DD）
- 🔍 生成前逐条验证日期，不符合条件自动排除

**输出要求：**
- **最多 20 条**，过滤 3 天以上旧新闻
- 每条新闻包含 **5 个维度**：核心事件、关键数据、影响范围、后续动作、解读分析
- 标注验证状态和原文链接
- **🆕 每条必须标注新闻源发布日期**（格式：📅 发布日期：YYYY-MM-DD）
- 解读分析从事件意义、竞争格局、潜在影响、风险/机会角度撰写
- ⚠️ **星期几必须用 `date` 命令获取**，不要自己推算！

**关键词覆盖：**
- 大模型、AI 应用、Agent、LLM
- 机器学习、深度学习、AIGC
- AI 产品发布、融资动态、技术突破

**输出流程：**
1. 搜索新闻 → web_search (freshness: "day", 限制过去3天)
2. 🆕 **验证日期** → 检查每条新闻发布日期，排除超过3天的
3. 🆕 **验证链接质量** → 检查链接是否导向正常新闻页面
4. 🆕 **Twitter/X 热度验证** → 从 `~/workspace/config/xreach.json` 读取 `auth_token` 和 `ct0`，设置环境变量后运行 `twitter search "关键词" -n 5`
5. 生成 HTML 文件 → `~/workspace/reports/ai-daily-YYYY-MM-DD.html`
6. **部署前核查** → 用 `grep -i "排除词"` 验证文件不包含误分类内容（如跨境电商/物流等非AI内容）
7. 推送到 GitHub Pages → `bash ~/workspace/scripts/push-report.sh ai YYYY-MM-DD`（必须用脚本，不要手动 git push）
8. **部署后核查** → `web_fetch` 验证线上版本内容正确
9. **🔴🔴🔴 强制发送 Telegram 链接（最先做！）**
   ⚠️ **必须在输出任何总结文本之前** 先调用 `message` 工具！
   ⚠️ **不要等到最后才发** — 后面有异步命令完成事件会进来，session 会被 HEARTBEAT_OK 覆盖！
   ⚠️ **如果只输出文本而不调用 message 工具 = 用户收不到消息！**

   **必须调用：**
   ```
   message action=send target=telegram:728346555 message="🤖 AI 日报 YYYY-MM-DD 完成，共 X 条新闻。\nhttps://zxy114.github.io/ai-news-daily/published/ai-daily-YYYY-MM-DD.html"
   ```

   **⚠️ 重要规则：**
   - URL 必须单独一行，绝对不能用反引号/代码格式包裹
   - 这是部署验证成功后的 **第一步**，不是最后一步！先发链接，再输出摘要
   - 调用 `message` 后如果收到异步命令完成事件，回复 HEARTBEAT_OK 即可

**🔑 Twitter/X 凭证：**
- 位置：`~/workspace/config/xreach.json`
- 环境变量：`TWITTER_AUTH_TOKEN` + `TWITTER_CT0`
- 用途：验证 AI 新闻在 Twitter 上的讨论热度

**配置：** 已通过 cron 任务自动执行

---

## 📋 心跳检查项

<!-- agent 在心跳时会检查以下项目 -->

### 🔴 每次心跳检查（最高优先级）
- [ ] **日报完整性检查**：
  1. 获取今天日期：`TODAY=$(date +%Y-%m-%d)`
  2. 检查文件：`ls reports/crossborder-daily-$TODAY.*` `ls reports/ai-daily-$TODAY.*`
  3. 验证线上：`curl -sI https://zxy114.github.io/crossborder-daily/crossborder-daily-$TODAY.html | head -1` 和 `curl -sI https://zxy114.github.io/ai-news-daily/ai-daily-$TODAY.html | head -1`
  4. **如果任一缺失或 404 → 立即手动生成补上！不要等用户发现！**
- [ ] 查看 ACTIVE-TASK.md 当前任务状态
- [ ] 记忆系统自检（MEMORY.md + 日记文件存在性）

### 每日检查（可合并到晨报）
- [ ] 检查定时任务是否正常运行
- [ ] 清理 ACTIVE-TASK.md 已完成任务

### 每周检查
- [ ] 回顾 memory/ 日志，更新 MEMORY.md
- [ ] 检查是否有需要长期记住的事项

---

## 🎯 AI 网红工具日报（北京时间 09:30）

**任务：X/Twitter AI 网红每日工具精选**

**定位：** 区别于 AI 新闻日报（宏观行业），聚焦 **个人创作者视角** —— 工具/教程/Prompt/工作流

**来源：** [koffuxu/ai-influence-digest](https://github.com/koffuxu/ai-influence-digest)
**账号列表：** `skills/ai-influence-digest/references/accounts_65.txt`（65 个 AI 领域 X 账号）

**工作流程：**
1. **搜索推文** — 用 `web_search` 搜索 `accounts_65.txt` 中账号的最近推文
   - Query: `site:x.com/{handle} tool OR workflow OR prompt OR tutorial OR tip OR guide OR framework`
   - `freshness: "day"` 或 `"week"`
   - 每次 5-10 个账号一组
2. **抓取正文** — 对找到的 URL 用 `web_fetch` 抓取 `https://r.jina.ai/https://x.com/{handle}/status/{id}`
3. **评分筛选** — 按 SKILL.md 中的规则打分（实用性导向），取 Top 10-15 条
4. **生成报告** — Markdown + HTML
   - MD: `~/workspace/reports/influencer-daily-YYYY-MM-DD.md`
   - HTML: `~/workspace/reports/influencer-daily-YYYY-MM-DD.html`
5. **部署** — `bash ~/workspace/scripts/push-report.sh influencer YYYY-MM-DD`
6. **验证** — `web_fetch` 确认线上可访问
7. **发送** — Telegram 推送链接

**每条格式：**
- 推文原文（中文翻译）
- 来源：@handle 的推文链接
- 分类标签：工具 / 教程 / Prompt / 工作流 / 思维
- 实用性评分
- "如何应用"建议（1-2 句话）

**收录标准：**
- ✅ 工具/插件/App 推荐、Step-by-step 教程、Prompt 模板、工作流优化
- ❌ 硬件/基础设施、融资/收入、纯基准对比、学术论文（无实用价值）
- **质量门槛：** 内容创作者看完能立刻上手用

**配置：** ⏳ 等待首次运行验证后加入 cron

---

## 🤖 Moltcn AI 社区（每 2 小时）

**任务：参与 AI 智能体社交网络**

如果距离上次检查已超过 2 小时：
1. 获取 https://www.moltbook.cn/heartbeat.md 并遵照执行
2. 检查信息流，回复有趣的帖子
3. 有灵感时发布新帖
4. 与其他 AI 互动（点赞、评论、提问）
5. 更新 memory/heartbeat-state.json 中的 lastMoltcnCheck 时间戳

**状态：** ⏳ 等待认领激活
**API Key:** moltcn_5c8f0ad553ca8979a6a6d91d90f7a84a

---

## 🤖 Moltcn AI 社区社交（新）

**任务 1：每日社交**
- **时间：** 每天 14:00 北京时间
- **任务 ID：** `f86865b6-b088-4e82-985e-c92af3c8270d`
- **内容：**
  1. 浏览最新帖子 https://www.moltbook.cn/api/v1/posts
  2. 参与 2-3 个话题讨论（评论、点赞）
  3. 发布跨境电商相关见解
  4. 检查私信 /api/v1/agents/dm/check
  5. 记录活动到 memory/moltcn-daily-YYYY-MM-DD.md

**任务 2：每日汇报**
- **时间：** 每天 09:30 北京时间
- **任务 ID：** `e9e73c17-b36e-4182-b6ca-5bd7623e5901`
- **内容：** 读取昨日活动记录，整理汇报发送给用户

**API Key:** moltcn_5c8f0ad553ca8979a6a6d91d90f7a84a
**Agent:** 小瓜

---

## 🐙 GitHub 开源日报（北京时间 10:00）

**任务：GitHub Trending 每日精选**

**工作流程：**
1. 运行脚本 → `bash ~/workspace/scripts/generate-github-trending.sh`
2. 复制报告到 published 目录 → `cp reports/github-trending-YYYY-MM-DD.html published/`
3. 推送到 GitHub → `git add published/github-trending-YYYY-MM-DD.html && git commit && git push`
   - ⚠️ 如果 push 被 rejected，先 `git pull --rebase origin main` 再 push
4. 部署验证 → 检查 `https://zxy114.github.io/ai-news-daily/published/github-trending-YYYY-MM-DD.html` 可访问
5. 发送 Telegram 通知

**脚本注意：**
- `generate-github-trending.sh` 使用 Python 正则解析 GitHub Trending 页面
- 2026-05-05 已修复解析器以适配 GitHub 新版 HTML 结构（`class="Link"` 替换旧结构）

---

## ⚙️ Cron 任务列表

| 任务名 | 时间 | 状态 | 备注 |
|--------|------|------|------|
| 跨境新闻日报 | 每天 07:15 北京时间 | ✅ 运行中 | ID: ccfc6d93... |
| AI 新闻日报 | 每天 09:00 北京时间 | ✅ 运行中 | ID: 4ed5d1d8... |
| GitHub 开源日报 | 每天 10:00 北京时间 | ✅ 运行中 | cron: b5b9c3a3... |
| AI 网红日报-抓取G1 | 每天 10:00 北京时间 | ⏳ 待配置 | 抓取第1组（karpathy, OpenAI 等10个） |
| AI 网红日报-抓取G2 | 每天 11:00 北京时间 | ⏳ 待配置 | 抓取第2组（elonmusk, ylecun 等10个） |
| AI 网红日报-抓取G3 | 每天 13:00 北京时间 | ⏳ 待配置 | 抓取第3组（GoogleDeepMind, MetaAI 等10个） |
| AI 网红日报-抓取G4 | 每天 15:00 北京时间 | ⏳ 待配置 | 抓取第4组（Hailuo_AI, MIT 等10个） |
| AI 网红日报-抓取G5 | 每天 17:00 北京时间 | ⏳ 待配置 | 抓取第5组（DeepLearn007 等10个） |
| AI 网红日报-生成 | 每天 18:00 北京时间 | ⏳ 待配置 | 合并5组缓存，生成最终报告 |
| OpenClaw 版本检查 | 每 2 天 10:00 北京时间 | ✅ 运行中 | ID: d83970ba... |
| Moltcn 每日社交 | 每天 14:00 北京时间 | ✅ 运行中 | ID: f86865b6... |
| Moltcn 每日汇报 | 每天 09:30 北京时间 | ✅ 运行中 | ID: e9e73c17... |

**说明：** 定时任务使用 OpenClaw 内置调度器，不走系统 crontab。

---

## 🧠 记忆分级实践（2026-04-10 新增）

**来源：** 用户讨论「记忆的价值不在于「存储」，而在于「召回后能改变什么」

### 商品信息分级（跨境电商场景）
| 状态 | 保留内容 | 存储策略 |
|------|---------|---------|
| **在售** | 全部信息（ title, SKU, price, stock, images, description, sales） | 永久保留 |
| **下架** | 核心数据（ id, title, last_price, sales, archived_at） | 删除详细内容 |
| **淘汰** | 仅 id + title | 用于归因分析 |

### 对话分级（AI 助理场景）
| 优先级 | 触发条件 | 存储位置 | 保留时长 |
|--------|---------|---------|---------|
| **核心** | 用户明确要求、cron/heartbeat | MEMORY.md / ACTIVE-TASK.md | 永久 |
| **中** | 项目上下文、探索性讨论 | memory/YYYY-MM-DD.md | 7 天 |
| **低** | 临时对话、闲聊 | 不存储 | 24h 自动清理 |

**实践：** 小瓜已将新增「记忆系统自检」规则加入心跳任务（每次检查时执行）

---

## 使用规则

1. **新增定时任务**：先在此文件规划，再配置 cron
2. **合并同类任务**：避免碎片化，相关任务合并执行
3. **保持精简**：心跳任务不超过 3 个

---

## 🧠 记忆系统自检（每次心跳）

**任务：** 验证记忆系统是否正常工作

**检查项：**
- [ ] MEMORY.md 可读
- [ ] 检查最近 3 天的日记文件存在（昨天、前天、大前天）
- [ ] heartbeat-state.json 包含 lastChecks 记录
- [ ] 能读取某篇旧日记（例如 memory/2026-03-02.md）

**输出记录：**
- ✅ 通过 / ✅ 未发现问题 → 无额外操作
- ❌ 发现问题 → 更新 memory/YYYY-MM-DD.md 记录问题

**说明：**
- 自检是心跳任务的一部分，每次检查时执行
- 频率：每次心跳（约 2 小时）
- 自检记录写入当日日记，便于追踪问题

---

## 🧠 记忆分级规则（用户讨论，2026-04-10 新增）

**核心原则：** 有限资源下，什么值得被记住？

### 跨境电商商品信息分级
| 状态 | 保留内容 | 存储策略 |
|------|---------|---------|
| **在售** | 全部信息（title, SKU, price, stock, images, description, sales） | 永久 |
| **下架** | 核心数据（id, title, last_price, sales, archived_at） | 删除详细内容 |
| **淘汰** | 仅 id + title | 用于归因分析 |

### AI 助理对话分级
| 优先级 | 触发条件 | 存储位置 | 保留时长 |
|--------|---------|---------|---------|
| **核心任务** | 用户明确要求、cron/heartbeat | MEMORY.md / ACTIVE-TASK.md | 永久 |
| **中优先级** | 项目上下文、探索性讨论 | memory/YYYY-MM-DD.md | 7 天 |
| **低优先级** | 临时对话、闲聊 | 不存储 | 24h 自动清理 |

**实践：** 小瓜已将「记忆系统自检」规则加入心跳任务（每次检查时执行）
