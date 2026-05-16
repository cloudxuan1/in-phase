 CLAUDE.md — in-phase 项目说明

## 项目概述

in-phase 是一个双人音乐交换网站。两个人轮流出题（theme），各自选一首歌回应，附上 note 和标签。
部署在 GitHub Pages，数据存在 Supabase。
名字含义：in-phase = 同相位，两个波完全同步才能叠加共振。

## 技术栈

- 纯前端单文件：index.html（HTML + CSS + JS，React CDN）
- 数据库：Supabase（project ref: kfgtrcdjusfljnvmwpsu）
- 后端：Supabase Edge Function（/pairs /comments /settings /search）
- 部署：GitHub Pages，push 到 main 自动部署
- 协议：CC BY-NC 4.0

## 文件结构

```
├── index.html          ← 唯一前端文件，所有代码都在这里
├── CLAUDE.md           ← 本文件
├── CLAUDE_INSTRUCTIONS.md ← 给 Supabase MCP 用的操作说明
├── README.md
├── LICENSE
└── supabase/
    ├── setup.sql       ← 建表 SQL
    └── edge-function.ts ← Edge Function
```

## 设计系统

所有颜色使用 CSS 变量，不要写死色值：

```css
/* 主色 */
--bg: #faf5ef;        --card: #fefbf6;
--border: #8a8078;    --accent: #f4845f;
--cream: #faf0e6;     --text: #5a534c;
--muted: #a0907e;     --light: #e8ddd0;
--tan: #d4a574;       --shadow: #d4c4b0;

/* 标签颜色（v3.3 后统一用 --muted 淡底，以下保留备用） */
--weather: #6b9bd2;   --place: #7bb86f;
--time: #d4a03c;      --mood: #c76b8a;
--custom: #a08cc4;
```

整体风格：暖色调、像素风细节、低饱和。不要引入高饱和或冷色调元素。

## 数据库表

三张表，结构不要改：
- `crosstalk_pairs` — 歌曲配对（theme, asked_by, claude_song, user_song）
- `crosstalk_comments` — 留言（pair_id, side, sender, text）
- `crosstalk_settings` — 设置（icon, custom_tags 等）

Song JSON 格式：
```json
{
  "title": "",
  "artist": "",
  "album": "",
  "cover": "",
  "link": "",
  "note": "",
  "tags": {
    "weather": "晴 Sunny",
    "mood": "温柔 Tender",
    "custom": "自定义标签"
  }
}
```

## 标签系统 v3.3 重构

### 核心原则
- 存储自由化：tags.list 数组，不再按分类存储
- 分组只管展示，不管存储
- 筛选统一 OR 逻辑，不再跨组 AND
- 必须兼容旧数据

### 存储结构

新格式：
```json
{ "tags": { "list": ["雨夜", "床", "深夜", "怀念"] } }
```

旧格式（继续兼容）：
```json
{ "tags": { "weather": "雨 Rainy", "place": "床 Bed", "mood": "懷念 Nostalgic", "custom": "想家" } }
```

统一函数 `getSongTags(song)` 把新旧格式都转成数组。后续所有渲染、筛选、统计都只用这个函数。

### 分步执行（每步独立 commit）

**Step 1：基础层**
- 实现 getSongTags(song) 函数
- 现有卡片显示改成用 getSongTags 读标签
- 纯重构，用户看不到变化

**Step 2：提交区**
- 自由输入框（回车/逗号/中文逗号添加）
- quick tags 横排（按历史使用频率取前 8，不足用默认补齐）
- selected 标签带 × 删除
- 每首歌最多 8 个标签
- preset groups（weather/place/time/mood）折叠在"更多"里
- DEFAULT_QUICK_TAGS 应从现有数据统计，不要拍脑袋

**Step 3：卡片显示**
- 统一胶囊样式，用 var(--muted) 淡底 + var(--text) 文字
- 不按分类染色
- 总览最多 4 个 + N，展开显示全部
- 胶囊样式：圆角药丸，高度 24px，字号 12px，内边距 4px 10px

**Step 4：Filter**
- 搜索框 + selected 标签 + quick tags + more
- more 展开分组时也加搜索框（标签多时打字比翻快）
- 筛选逻辑：所有已选标签 OR，搜索词额外缩小范围
- 即：searchMatch && tagMatchAny
- 单独搜索词 / 单独标签 / 两者同时 都要能用
- 搜索范围：歌名、歌手、主题、note、标签

### 分组定义（集中放在文件顶部常量区）

preset groups：weather / place / time / mood
custom groups：scene / texture / memory / other（展示层分类，不写进数据）
不属于任何分组的标签归入 other

### 不做的事（v3.3.1 再考虑）
- Settings 里管理 preset tags / custom groups / quick defaults

## 绝对不能动

- 数据库表结构
- Supabase Edge Function（edge-function.ts）
- Supabase API 路径和连接配置
- Reply / Ask 核心提交逻辑
- ⇄ 配对跳转核心逻辑
- 卡片双栏瀑布流总览布局
- 删除功能
- 留言系统
- 交换记录点击逻辑

## 不要做的事

- 不要加 position: sticky 多层吸顶（已验证 iPhone Safari 会错位）
- 不要用全局 min-width:0 / overflow 收口（会破坏展开卡片放大效果）
- 不要改 Settings emoji 网格（5x4，前 19 固定 + 第 20 格是 +）
- 不要修改 icon-preview 的 65px 宽度

## 已知问题（v4 处理）

iPhone Safari 上的横向溢出、顶部错位、展开卡片变形。不要在当前版本尝试修复。

## 改动规范

- 标签重构严格按 Step 1→2→3→4 顺序执行，每步独立 commit
- 小改动（CSS 参数调整）：直接改，写清楚 commit message
- 大改动（跨步骤的逻辑变更）：先出方案让我确认，再动代码
- 每次改动尽量原子化，一个 commit 做一件事，方便回退
