# PixCollect 云同步接入指南（Supabase 免费档，¥0）

本文档说明如何把当前纯前端 + localStorage 版本升级为「GitHub Pages + Supabase + Cloudflare R2」推荐组合。

> 原则：**不配置也能用**。`index.html` 内置了双模式——没填 key 时完全按原来的 localStorage 行为运行；填了 key 并登录后，作品 / 待审核 / 点赞 / 收藏自动云端同步，跨设备可见。

## 总成本（免费额度，2026-08 确认）

| 服务 | 免费档额度 | 用途 |
| --- | --- | --- |
| GitHub Pages | 无限静态托管 + 流量 | 网站托管（已用） |
| Supabase | Postgres 500MB + 文件存储 1GB + 流量 5GB/月 + Auth 5 万 MAU + Edge 函数 50 万次/月 | 数据库 / 登录 / 上传存储 |
| Cloudflare R2（可选进阶） | 10GB + 100 万次写 / 1000 万次读 + 流量永免费 | 流量超过 Supabase 5GB/月后再上 |

⚠️ Supabase 免费项目 7 天无访问会自动休眠，需要手动恢复（控制台点一下即可）。Vercel/Render 等后端托管本期不需要。

## 一、注册并创建项目（约 5 分钟）

1. 打开 <https://supabase.com> → 用 GitHub 或邮箱注册（免费，无需绑卡）。
2. Dashboard → **New project**：
   - 名称随意（如 `pixcollect`）
   - **Database password** 记下来（后续 SQL 编辑器用，其实不用密码也能建表，但建议保存）
   - Region 选 `Southeast Asia (Singapore)`（离国内最近）
3. 等待 1-2 分钟项目创建完成。

## 二、建表 + 建存储桶（一次性）

进入项目 → 左侧 **SQL Editor** → New query，粘贴下面全部 SQL，Run：

```sql
-- 1) 用户数据表：每个用户一行，存作品 / 待审核 / 点赞 / 收藏
create table if not exists public.pix_state (
  id uuid primary key default gen_random_uuid(),
  user_id uuid unique references auth.users(id) on delete cascade,
  user_works jsonb not null default '[]'::jsonb,
  pending_works jsonb not null default '[]'::jsonb,
  liked jsonb not null default '{}'::jsonb,
  bookmarked jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

-- 行级安全：只能读写自己的数据
alter table public.pix_state enable row level security;
drop policy if exists pix_state_own on public.pix_state;
create policy pix_state_own on public.pix_state
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);

-- 2) 上传存储桶（图片 / 视频，公开可读）
insert into storage.buckets (id, name, public)
values ('pix-media', 'pix-media', true)
on conflict (id) do nothing;

-- 登录用户可上传到 pix-media
drop policy if exists pix_media_upload on storage.objects;
create policy pix_media_upload on storage.objects
  for insert to authenticated with check (bucket_id = 'pix-media');

-- 所有人可读取（公开作品预览）
drop policy if exists pix_media_public_read on storage.objects;
create policy pix_media_public_read on storage.objects
  for select to public using (bucket_id = 'pix-media');
```

## 三、拿 key 填进 index.html（2 分钟）

1. 项目左侧 **Settings → API**（或项目首页底部）。
2. 复制两个值：
   - **Project URL**（形如 `https://xxxxxxxx.supabase.co`）
   - **anon public key**（一长串 `eyJ...`）
3. 打开 `index.html`，找到文件顶部配置块（搜索 `SUPABASE_URL`）：

```js
const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';  // ← 替换成你的 Project URL
const SUPABASE_ANON_KEY = 'YOUR-ANON-PUBLIC-KEY';          // ← 替换成你的 anon key
```

4. 保存。

## 四、推送上线（本地已无 .git，必须走 GitHub 更新）

改完 index.html 后用 MCP GitHub 工具（`create_or_update_file`，需要当前 SHA）或网页端直接编辑提交到 `main` 分支，GitHub Actions 会自动重新部署到 Pages，1-2 分钟生效。

## 五、验证

1. 打开线上地址，右上角出现「登录」按钮。
2. 注册一个账号（邮箱 + 密码），会收到确认邮件（可选确认，不确认也能用）。
3. 登录后发布一个作品 → 换一台设备 / 无痕窗口登录同一账号 → 待审核里应出现同一作品。
4. 手机上强制刷新（无痕模式）确认看到「登录」按钮。

## 常见问题

- **没填 key / key 填错**：页面完全按本地模式运行，不报错，只是没有云同步；控制台（F12）会提示 `Supabase 初始化失败`。
- **登录提示未配置**：检查 URL 是否以 `https://` 开头、anon key 是否完整复制（超过 30 字符）。
- **上传失败**：确认第二步的 SQL 已执行（bucket `pix-media` 存在）；文件过大会超过免费档限制，先试小图。
- **想清空云端数据**：Supabase Dashboard → Table Editor → `pix_state` → 删掉该行；Storage → `pix-media` → 删文件。
- **R2 何时上**：当 Supabase 流量（5GB/月）不够时，把图片迁移到 Cloudflare R2（流量永免费），只需把作品 `src` 换成 R2 公开 URL，代码无需其他改动。

## 当前边界（诚实说明）

- 云同步以「云端为最终版」：登录时云端数据覆盖本机，多设备同时改可能互相覆盖（demo 阶段够用）。
- 审核后的作品是「本机可见」；要让所有访客看到已审核作品，需要加一张公开作品表 + 所有人可读策略（下一步再做）。
- 登录用的是 Supabase 邮箱密码；Google 登录需在 Authentication → Providers 开启并配 OAuth。
- 视频上传免费档有 50MB 单文件限制（Supabase Storage 默认限制），图片无碍。