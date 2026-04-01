一、项目概述

Zen Briefing 是一个 AI 信息聚合平台，整合官方发布与社区讨论，以报纸风格呈现每日/每周 AI 简报。

核心理念：你不需要更多信息，你需要更少的噪音。

产品定位是 AI 时代的信息编辑室——自动采集、AI 判断、报纸排版，帮用户 3 分钟看完本周 AI 领域发生了什么。核心价值不是聚合信息，而是注意力服务：帮用户确认"该知道的都知道了"。


二、核心功能

1. 信息采集

自动爬取 AI 领域的官方更新、科技新闻和社区讨论，覆盖 Anthropic、OpenAI、Cursor、Gemini、DeepMind、Hugging Face 等主要玩家。支持 RSS 解析、GitHub Releases 追踪、Playwright 动态页面爬取等多种采集方式。

2. AI 内容分析

DeepSeek API 驱动的内容富化：自动分析、摘要生成、文章撰写。每 30 分钟执行一次 AI Enrichment，每日 8:00 AM 生成 AI 日报（中英双语）。

3. 报纸风格 UI

Newspaper 页面以报纸化周视图呈现，将官方更新与社区反馈聚合展示。Live Feed（Admin）页面提供实时信息流，支持分类筛选、热度排序和趋势雷达。

4. 定时任务调度

|任务|频率|说明|
|---|---|---|
|Official Updates|每天 7:55 AM|爬取官方更新|
|Community Posts|每 2 小时|爬取 Reddit/HN|
|AI Enrichment|每 30 分钟|AI 内容分析 + 文章生成|
|Daily Digest|每天 8:00 AM|AI 日报生成|
|Data Cleanup|每天 3:00 AM|清理过期数据|


三、技术栈

|类别|技术栈详情|
|---|---|
|Monorepo|Turborepo + pnpm workspaces|
|前端|Next.js 16, React 19, Tailwind CSS 4, shadcn/ui (Radix)|
|后端|Express 5, TypeScript, Playwright (爬虫)|
|AI|DeepSeek API (OpenAI-compatible)|
|数据库|MongoDB (Atlas 云端)|
|部署|Vercel (web) + Railway (worker)|
|代码规范|ESLint + Prettier + Husky|


四、项目结构

```
zen-briefing/
├── apps/
│   ├── web/                # Next.js 前端 (port 7100)
│   │   └── src/
│   │       ├── app/        # App router pages & API routes
│   │       │   ├── (newspaper)/  # 报纸路由组
│   │       │   └── (admin)/      # Live Feed 路由组
│   │       ├── components/
│   │       │   ├── newspaper/    # Newspaper 页面组件
│   │       │   ├── live-feed/    # Live Feed 组件
│   │       │   └── ui/           # shadcn/ui 基础组件
│   │       └── lib/
│   │           ├── heat.ts       # Heat 计算 + 分类推断
│   │           ├── api.ts        # API 客户端
│   │           └── mongodb.ts    # 数据库连接
│   │
│   └── worker/             # Express 后端 + 爬虫 (port 7101)
│       └── src/
│           ├── scraper/    # 官方 + 社区爬虫
│           ├── services/   # MongoDB, DeepSeek AI
│           ├── api/        # Express HTTP routes
│           └── jobs/       # Cron 调度, enrichment, digest
│
├── packages/
│   └── shared/             # 共享类型定义
│
├── package.json            # Root workspace config
├── pnpm-workspace.yaml
└── turbo.json
```


五、数据源

官方更新 (Official Updates)

|源|采集方式|
|---|---|
|Anthropic Blog|博客 RSS + 全文爬取|
|Claude Code|GitHub Releases|
|OpenAI Blog|RSS + Playwright + Models API|
|Cursor|Changelog 页面爬取|
|Gemini|Changelog 页面 (Playwright)|
|DeepMind|博客 RSS + Playwright 全文爬取|
|Hugging Face|博客 RSS + 全文爬取|
|Qwen|HuggingFace API (新模型发布追踪)|
|Kimi|MoonshotAI GitHub Releases|
|GitHub Copilot|GitHub Blog Changelog RSS|

科技新闻 (Tech News)

|源|采集方式|
|---|---|
|TechCrunch|AI 分类 RSS|
|VentureBeat|全站 RSS + AI 关键词过滤|

社区内容 (Community)

|源|采集方式|
|---|---|
|Reddit|公开 JSON API|
|Hacker News|Algolia API|


六、页面与 API

页面路由

|路径|说明|
|---|---|
|`/`|Newspaper — 报纸化周视图|
|`/admin`|Live Feed — 实时信息流 + 管理|
|`/article/[id]`|文章详情页|

Web API (port 7100)

- `/api/updates` — 官方更新（支持 `?grouped=true` 按产品分组）
- `/api/community` — 社区帖子
- `/api/newspaper/[date]` — 报纸数据（周视图）
- `/api/digest/[date]` — AI 日报
- `/api/trends` — 热门话题和工具

Worker API (port 7101)

- `POST /api/official-updates/fetch` — 触发官方更新爬取
- `POST /api/community-posts/fetch` — 触发社区帖子爬取
- `POST /api/daily-digest/generate` — 触发日报生成
- `POST /api/content/:type/:id/enrich` — 触发内容 AI 分析


七、环境变量

|变量|说明|
|---|---|
|`MONGODB_URI`|MongoDB 连接串（Atlas）|
|`DATABASE_NAME`|数据库名，默认 `ai_pulse`|
|`DEEPSEEK_API_KEY`|AI 分析服务 API Key|
|`OPENAI_API_KEY`|可选，用于获取模型列表|


八、Roadmap

Phase 1 — 可分享（当前阶段）

目标：让 10-20 个朋友开始用，收集真实反馈。

- 鉴权系统：邀请码注册 + session 登录，admin/reader 两个角色
- 移动端阅读优化：响应式排版 + PWA 支持
- 数据源扩展：Meta/Llama、Mistral

Phase 2 — 可订阅

目标：验证留存和口碑。

- 推送订阅：邮件周报 + Webhook 通知
- 个性化：自定义关注源、关键词过滤
- 阅读体验：历史周刊归档、全文搜索

Phase 3 — 可收费

目标：验证付费意愿，建立可持续模式。

- 免费层 vs Pro 层分级
- 采集策略从"按源爬取"扩展到"按话题聚合"
- AI 自动识别值得追踪的新产品/新动态

决策记录

|日期|决策|理由|
|---|---|---|
|2026-03-10|移动端用响应式 PWA 而非小程序|核心价值在内容排版，不依赖平台能力|
|2026-03-10|收费逻辑是注意力服务而非内容付费墙|信息本身公开，价值在于编辑判断和时间节省|
|2026-03-10|鉴权优先于移动端优化|没有鉴权就没法分享，没有分享就没有反馈|
