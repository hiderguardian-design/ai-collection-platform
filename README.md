# AI 作品展示平台

> 只收录真实有 AI 参与的作品。

## 当前状态

这是一个纯静态 HTML 界面原型，用于验证「只展示实际有 AI 参与的项目」这一产品方向。当前没有后端、数据库、真实上传、用户系统或自动审核流程。

## 页面

- `index.html`：PixCollect 主界面，包含信息流、分类、搜索、详情弹窗和发布入口演示。

备用方案 `simple-minimal.html`（小集 AI）已于 2026-08-16 删除，当前只保留 PixCollect 主方案。

## 在线预览（GitHub Pages）

部署通过 GitHub Actions 自动完成（见 `.github/workflows/pages.yml`）。

启用步骤（首次只需一次）：

1. 打开仓库 **Settings → Pages**
2. 在 **Build and deployment → Source** 选择 **GitHub Actions**
3. 回到 Actions 页签，等待 `Deploy static content to Pages` 工作流运行完成

预览地址：`https://hiderguardian-design.github.io/ai-collection-platform/`

- 主界面：`/index.html`

## 本地运行

在项目目录执行：

```bash
python3 -m http.server 8000
```

然后打开 `http://localhost:8000/index.html`。页面依赖外部 CDN 和示例媒体，离线打开时部分资源可能无法加载。

## 下一阶段边界

如果继续开发，应先明确以下产品和技术决定：

1. ✅ 已确定保留 PixCollect 主方案（备用方案已删除）。
2. 作品元数据、媒体存储和用户身份的后端方案。
3. 如何证明作品确实有 AI 参与，以及人工审核和申诉流程。
4. 上传媒体的大小、格式、版权和安全策略。
5. 是否需要收藏、点赞、评论、搜索索引和管理后台。

在这些决定完成前，不把静态演示直接扩展成缺少数据模型的伪后端。

## 当前限制

- 页面中的作品和点赞数据是硬编码示例。
- 上传、搜索和标签操作主要是交互演示。
- 外部图片、视频、音频和字体来自第三方地址。
- 项目未初始化本地 Git 工作区，也没有自动化测试。
