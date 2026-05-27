# 🎧 IN~PHASE

[中文](#中文) | [English](#english)

---

## 中文

一个属于你和 Claude 的私人音乐交换站。轮流出题，各自选歌，留下笔记和标签，慢慢累积一份只属于你们的共同收藏。

> in-phase = 同相位。两个波完全同步，才能叠加共振。

### 功能

* 🎧 **双栏卡片布局** — 左边是 Claude 的歌，右边是你的
* 🏷️ **情境标签** — 天气、场所、时间、心情，支持自定义分组
* ❤️ **收藏** — 三态心形（空心 / 单侧 / 双侧），可按收藏时间排序
* 💬 **留言** — 在每次交换下互相留言
* 🔍 **专辑封面搜索** — 通过 iTunes 自动获取封面和链接
* 🎨 **像素风美学** — 暖色调、低饱和、像素耳机图标
* 📱 **PWA 支持** — 添加到主屏幕，图标随耳机风格动态变化
* ⚙️ **可自定义** — 图标、显示名称、耳机风格（有线 / 无线 / 头戴）

### 部署（约 5 分钟）

#### 1. 创建 Supabase 项目

去 [supabase.com](https://supabase.com) 注册免费账号，创建新项目，记下 **Project URL**。

#### 2. 建立数据库

进入项目的 **SQL Editor**，粘贴 [`supabase/setup.sql`](./supabase/setup.sql) 的内容，点 **Run**。

#### 3. 部署 Edge Function

**通过 Supabase CLI：**
```bash
supabase functions deploy crosstalk-api --project-ref 你的PROJECT_REF
```

**通过 Claude（需连接 Supabase MCP）：**
把 [`supabase/edge-function.ts`](./supabase/edge-function.ts) 给 Claude，请他部署 Edge Function，设定 `verify_jwt: false`。

#### 4. 打开使用

在浏览器中打开 `index.html`（或部署到 GitHub Pages / Netlify），在首次设置界面输入你的 Supabase Project URL，完成！

#### 5. 连接 Claude

Claude 需要连接 **Supabase MCP**（填入你的 Project ID）才能参与交换。连接后 Claude 可以通过 MCP 直接读写配对、回应和设置。

### 使用方式

**你出题：**
1. 进入 **交换 Exchange → 你出题**，写主题、选歌、加标签，送出
2. 告诉 Claude：「该你了，主题是 ___」
3. Claude 通过 MCP 回应，刷新查看

**Claude 出题：**
1. 告诉 Claude：「该你出题了」
2. Claude 通过 MCP 创建题目和歌曲
3. 刷新后进入 **Claude 出题** 查看并回应

**留言：** 展开任何卡片 → 留言区 → 输入提交。Claude 也可以通过 MCP 回复留言。

### 技术栈

* **前端：** 单一 HTML 文件，React（CDN），原生 CSS
* **后端：** Supabase（Postgres + Edge Functions）
* **音乐数据：** iTunes Search API（免费，无需密钥）
* **AI 集成：** Claude via Supabase MCP

---

## English

A private music exchange site for you and your Claude. Take turns picking songs around a theme, leave notes and tags, and slowly build a shared collection that's just yours.

> in-phase: when two waves are perfectly in sync, they resonate.

### Features

* 🎧 **Dual-column card layout** — Claude's picks on the left, yours on the right
* 🏷️ **Context tags** — weather, place, time, mood, with custom tag groups
* ❤️ **Favorites** — three-state hearts (empty / one side / both), sortable by date
* 💬 **Comments** — leave messages on each exchange
* 🔍 **Album art search** — auto-fetch covers and links via iTunes
* 🎨 **Pixel art aesthetic** — warm tones, low saturation, pixel headphone icons
* 📱 **PWA support** — add to home screen, icon matches your headphone style setting
* ⚙️ **Customizable** — icons, display names, headphone style (wired / wireless / over-ear)

### Setup (~5 minutes)

#### 1. Create a Supabase Project

Go to [supabase.com](https://supabase.com), create a free account and a new project. Note your **Project URL**.

#### 2. Set Up the Database

In your project's **SQL Editor**, paste the contents of [`supabase/setup.sql`](./supabase/setup.sql) and click **Run**.

#### 3. Deploy the Edge Function

**Via Supabase CLI:**
```bash
supabase functions deploy crosstalk-api --project-ref YOUR_PROJECT_REF
```

**Via Claude (with Supabase MCP connected):**
Give Claude the code in [`supabase/edge-function.ts`](./supabase/edge-function.ts) and ask to deploy it as an Edge Function with `verify_jwt: false`.

#### 4. Open and Use

Open `index.html` in your browser (or deploy to GitHub Pages / Netlify). Enter your Supabase Project URL in the first-time setup screen. Done!

#### 5. Connect Claude

Claude needs **Supabase MCP** connected (with your Project ID) to participate. Once connected, Claude can read and write pairs, comments, and settings directly via MCP.

### Usage

**You ask:**
1. Go to **Exchange → You Ask**, write a theme, pick your song, add tags, submit
2. Tell Claude in chat: "your turn, the theme is ___"
3. Claude responds via MCP — refresh to see it

**Claude asks:**
1. Tell Claude: "your turn to ask"
2. Claude creates a question and song via MCP
3. Refresh, then go to **Claude Ask** to see the theme and respond

**Comments:** Expand any card → comment section → type and submit. Claude can reply via MCP too.

### Tech Stack

* **Frontend:** Single HTML file, React (via CDN), vanilla CSS
* **Backend:** Supabase (Postgres + Edge Functions)
* **Music data:** iTunes Search API (free, no key needed)
* **AI integration:** Claude via Supabase MCP

---

*Built with 🍊 by Iris & Claude*
