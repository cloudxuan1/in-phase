# 🎧 IN~PHASE

一个属于你和 Claude 的私人音乐交换站。

围绕一个主题，你和 Claude 各自选一首歌，留下 note、标签、留言和收藏。时间久了，它会慢慢变成一份只属于你们的共同音乐记忆。

> in-phase = 同相位。两个波完全同步，才能叠加共振。

---

## 这是什么？

IN~PHASE 不是普通歌单，也不是音乐推荐工具。

它更像一个“双人音乐日记”：

- 一个人出一个主题；
- 两个人分别用一首歌回应；
- 每首歌可以写 note、贴标签、留言；
- 喜欢的卡片可以收藏；
- 很久以后，你可以用标签、收藏、交换记录，把某一段情绪重新找回来。

比如：

```text
主题：下雨天的心情

Claude 选了一首歌
你也选了一首歌

然后这一组交换就变成了一张 pair：
Claude 的回应 | 你的回应
```

---

## 它是怎么运作的？

IN~PHASE 的核心是一个共享数据库。

你在网页里提交主题、歌曲、标签和留言。  
Claude 通过 Supabase MCP 直接读写同一个数据库。

所以它不是“你把内容发给 Claude，Claude 再发回来”这种临时聊天记录，而是：

```text
网页前端
  ↓
Supabase 数据库
  ↑
Claude MCP
```

你和 Claude 都在同一个资料库里留下东西。

网页负责展示和交互。  
Claude 负责参与交换、补歌、留言、整理内容。  
Supabase 负责保存所有 pair、comments、settings 和 favorites。

---

## 页面功能介绍

### 1. Home：首页总览

首页是整个音乐交换站的主视图。

这里会以双栏卡片显示所有交换：

```text
Claude 的歌 | 你的歌
```

每一组交换都是一个 pair。  
左边是 Claude 的回应，右边是你的回应。

你可以在首页做这些事：

- 看所有交换记录；
- 展开某张卡片；
- 查看歌曲 note、标签、留言；
- 点心形收藏卡片；
- 用筛选器按标签找歌；
- 在 Claude / User / Both 视角之间切换。

---

### 2. 双栏卡片

每张卡片代表一首歌。

卡片里通常包含：歌名、歌手、专辑封面、note、标签、收藏心形、留言入口。

点击卡片后会展开，显示更多信息。

移动端展开时，当前这一侧会变宽，另一侧缩小，但整体仍保持在屏幕内。

---

### 3. 交换 pair

一组 pair 是一次完整的音乐交换。

它包含：

```text
主题 theme
Claude song
User song
留言 comments
收藏 favorites
```

pair 可以理解成“这一次我们围绕同一个主题发生了什么”。

例如：

```text
#39 白月光
Claude 选的歌
你选的歌
你们的留言
你们各自的收藏状态
```

---

### 4. Exchange：发起和回应

Exchange 是提交歌曲的地方。

目前有两种常规流程：

#### 你出题

你进入 Exchange，选择“你出题”，填写：

- 主题、歌名、歌手、专辑；
- 封面、链接；
- note、标签。

提交后，这个 pair 会先出现你的歌。  
然后你可以告诉 Claude：

```text
该你了，主题是：____
```

Claude 通过 Supabase MCP 写入它的回应。

#### Claude 出题

你也可以让 Claude 先出题。

Claude 会通过 MCP 创建主题和它的歌。  
你刷新后，在 Exchange 里看到 Claude 的题目，再提交你的回应。

---

### 5. 标签系统

标签是 IN~PHASE 里很重要的一层记忆索引。

它不是传统音乐分类器，不是为了精确区分 genre。  
它更像情绪、场景和记忆锚点。

默认标签包括：天气、场所、时间、心情。

你也可以在 Settings 里管理标签组，添加自己的标签。

比如：

```text
深夜 Late Night
雨 Rainy
床 Bed
怀念 Nostalgic
给冒牌克宝的回信
```

标签可以帮助你以后找回某种感觉，而不是只按歌名搜索。

---

### 6. Filter：筛选

Filter 用来按标签筛选卡片。

你可以点一个标签，查看所有包含这个标签的歌。

筛选逻辑是 OR：

```text
选了 A 和 B
= 显示包含 A 或 B 的卡片
```

这适合用来找“某一类感觉”的歌，比如：

```text
深夜
雨天
怀念
```

---

### 7. 喜欢的标签

Filter 里有“喜欢”标签区。

你可以把常用标签收藏起来，让它们一直显示在前面。  
这样不用每次都去完整标签列表里找。

适合收藏这些高频标签：

```text
深夜 Late Night
晴 Sunny
怀念 Nostalgic
平静 Calm
```

---

### 8. 收藏卡片

每张歌卡都有心形收藏。

收藏是分 Claude / User 两侧的：

```text
Claude 卡有 Claude 的收藏状态
User 卡有你的收藏状态
```

心形有三种状态：

```text
♡  未收藏
淡 ♥  单侧收藏
实 ♥  双侧都收藏
```

在 Both 总览里，你可以同时看到两边的收藏状态。  
切到 Claude 或 User 视角时，可以查看各自的收藏库。

收藏库支持两种排序：

- 默认排序；
- 最新收藏在前。

---

### 9. 留言

每组交换下面都有留言区。

你可以在展开卡片后留言。  
Claude 也可以通过 Supabase MCP 回复留言。

留言适合记录：

- 为什么选这首歌；
- 听完对方歌的反应；
- 对某个主题的补充；
- 后来回看时的新感受。

---

### 10. Settings：设置

Settings 用来管理这个站的个性化内容。

目前可以设置：

- Claude 的名字、你的名字；
- Claude / User 图标、耳机样式；
- 标签组、自定义标签、喜欢的标签。

标签组管理现在是 accordion 展开式：

```text
点标签组 → 原地展开
改组名 / 改颜色 / 加标签 / 删标签
Done 保存
```

删除标签组有确认流程，避免误删。

---

### 11. 封面搜索

提交歌曲时，可以通过 iTunes Search API 搜索歌曲信息。

它可以自动带出：歌名、歌手、专辑、封面、Apple Music / iTunes 链接。

这个功能不需要额外 API key。

---

### 12. PWA / 手机使用

IN~PHASE 支持添加到手机主屏幕。

图标是像素耳机风格，并且会根据 Settings 里的耳机类型变化：

- 有线；
- 无线；
- 头戴。

整体视觉风格是：

```text
暖色调
低饱和
像素风
私人音乐盒
```

---

## 当前功能

- 🎧 双栏音乐交换
- 🏷️ 情境标签系统
- ⭐ 喜欢的标签
- ❤️ 三态卡片收藏
- 💬 留言
- 🔍 iTunes 封面搜索
- ⚙️ Settings 个性化
- 🎨 像素耳机视觉
- 📱 PWA 主屏幕图标
- 🤖 Claude via Supabase MCP

---

## 下一步计划

### V7：盲选 Blind Exchange

盲选是下一阶段计划中的功能。

同一个主题下，两个人各自选歌，但提交前互相看不到对方选了什么。

流程大概是：

```text
创建主题
↓
两边各自提交歌曲
↓
未交齐前只显示“已提交 / 未提交”
↓
双方都提交后同时揭晓
```

这个功能适合看：

```text
同一个情绪下，我们选的歌到底有多像，或者差多远？
```

---

## 部署方式

### 1. 创建 Supabase 项目

去 Supabase 创建一个新项目，记下 Project URL。

### 2. 建立数据库

进入 Supabase SQL Editor，运行：

```text
supabase/setup.sql
```

### 3. 部署 Edge Function

部署：

```text
supabase/edge-function.ts
```

函数名：

```text
crosstalk-api
```

并设置：

```text
verify_jwt: false
```

### 4. 打开网页

打开 `index.html`，或者部署到 GitHub Pages / Netlify。

首次进入时，输入你的 Supabase Project URL。

### 5. 连接 Claude

Claude 需要连接 Supabase MCP。  
连接后，Claude 就可以通过 MCP 参与交换、回应歌曲、写留言和管理数据。

---

## 技术栈

- 前端：单文件 `index.html`
- UI：React CDN + 原生 CSS
- 数据库：Supabase Postgres
- 后端：Supabase Edge Function
- 音乐搜索：iTunes Search API
- AI 协作：Claude via Supabase MCP

---

Originally forked from [onlonlonl/crosstalk](https://github.com/onlonlonl/crosstalk). Reimagined and extended as IN~PHASE by 轩, with GPT-5.5 Thinking & Claude.
