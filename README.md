# AI 作品展示平台

## 当前状态

静态 HTML 前端 + 可选 Supabase 云同步。产品方向：只展示实际有 AI 参与的作品。

- **未配置 Supabase 时**：纯静态 + localStorage 本地持久化（上传作品、待审核、点赞、收藏，刷新不丢失，仅本机可见）。
- **配置 Supabase 后**：新增登录 / 注册，作品、待审核、点赞、收藏云端同步（跨设备），上传文件走 Supabase Storage 公开 URL。

配置步骤见 **[SETUP.md](SETUP.md)**（Supabase 免费档：Postgres 500MB + 存储 1GB + Auth 5 万 MAU，¥0）。

## 页面

- `index.html`：PixCollect 主界面，包含信息流、分类、搜索、详情弹窗、发布入口、登录弹窗和云端同步。

## 本地运行

```bash
python3 -m http.server 8000
```

打开 `http://localhost:8000/index.html`。页面依赖外部 CDN（lucide 图标、supabase-js）和示例媒体。

## 上线流程（重要）

本地目录没有 `.git`，**不能直接 git push**；线上更新通过 MCP GitHub 工具（`create_or_update_file` / `push_files`）提交到远程 `main` 分支，GitHub Actions 自动重新部署 Pages。

线上地址：<https://hiderguardian-design.github.io/ai-collection-platform/>

## 下一阶段边界

1. 让所有访客看到已审核作品（需要公开作品表 + 公开读策略，当前审核后作品仅本人可见）。
2. 作品元数据、媒体存储和审核模型的正式设计。
3. AI 参与证明（证据字段 + 人工审核 + 申诉流程）。
4. 上传媒体的大小、格式、版权和安全策略（Supabase Storage 默认单文件 50MB）。
5. 收藏、点赞、评论、搜索索引和管理后台范围。
6. 流量超过免费档后再考虑 Cloudflare R2（流量永免费）。

## 当前限制

- 示例作品和点赞数据为硬编码演示；真实数据需走登录 + 上传 + 审核。
- 云同步以云端为最终版，多端同时编辑可能互相覆盖（demo 阶段可接受）。
- 外部图片、视频、音频、字体来自第三方地址。