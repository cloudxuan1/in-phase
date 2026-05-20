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

**Step 4：Filter（已完成，merged to main）**
- 统一为单一搜索框（移除 moreQ），输入时在搜索框下方浮出匹配标签建议 panel
- 建议 panel 点击外部自动收起，搜索词保留
- 「標籤：」行 + 「常用：」行，带行前缀标签
- 更多面板：「預設：」行标签 + 分类 chip 折叠展开；标签卡片对齐点击的 chip，右侧溢出自动右对齐；点击外部自动收起
- 自定义分类超 15 个折叠，可展开全部
- 选中状态：25% 分类色淡底 + 彩色边框 + 彩色文字（替代实心填充）
- 搜索范围：歌名、歌手、主题、note、标签、留言
- 筛选逻辑：searchMatch && tagMatchAny（OR）

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

## 已知问题（待处理）

### iPhone Safari 展开卡片触发整页缩放（优先级高，未解决）

**症状**：手机上点开卡片展开详情，顶部 header/nav 会缩小变形，整页被 Safari 自动缩放。

**根因（已确认）**：`.columns` 是 `display:flex`，`.col` 有 `flex:1` 但没有 `min-width:0`。展开卡片时某个子元素的 min-content-width 超出列宽，撑宽 `.col`，两列合计 > 视口，Safari 缩页。

**已试过且失败的方案（不要再试）**：
- PR#5：全局 `.col { min-width:0 }` → 两列失衡，展开交互坏掉（被 revert）
- PR#6：sticky header / layout shell → 只遮症状，不解根因（被 revert）
- PR#7 (V4)：expanded detail 改 `position:absolute; top:100%` → detail 覆盖下方卡片，原地展开感消失（被 revert）
- PR#10 (V5)：`.col { min-width:0 }` + `.theme-title` flex 规则 → 列失衡 + 留言 input 坏掉（被 revert）
- PR#12 (V6)：只加 `.theme-title { min-width:0 }`、`.comment-input input { min-width:0 }`、`.card.expanded .input { box-sizing:border-box }` → Safari 仍缩页
- V6b：在 V6 基础上加 `.col { overflow-x:hidden }` → 卡片展开后视觉上不再变大（overflow 把内容裁掉了）

**当前 main 状态**：PR#4 干净基线（Filter UI 完成），以上所有改动已全部 revert。

**下一步必须先做诊断，再改代码**：
1. 用 Mac 连 iPhone，Safari 开发者工具 → Elements 面板，展开卡片时找到实际比视口宽的元素
2. 或者加一段临时调试 JS，展开时把所有 `offsetWidth > window.innerWidth` 的元素打印到页面上
3. 确认溢出元素后，只针对那个元素加约束，不要碰 `.col` 本身

**已确认不能动的方向**：
- 不加 `position:sticky` 多层吸顶
- 不加全局 `overflow:hidden` / `overflow:clip`
- 不让 expanded card 脱离 normal flow（破坏原地展开感）
- 不加 `.col { min-width:0 }` 或 `.col { overflow-x:hidden }`（反复验证会破坏列宽平衡）

### 旧标签数据迁移问题
旧数据格式：`{ "weather": "晴 Sunny", "mood": "懷念 Nostalgic" }`（带英文）
新标签格式：`"晴"`, `"懷念"`（纯中文）

现状：`getSongTags()` 能读出旧数据的字符串值（如 `"晴 Sunny"`），但和新标签 `"晴"` 是两个不同字符串，筛选时无法合并命中。

处理方案（v5 或单独 PR）：
- 方案 A：SQL 批量 UPDATE，把旧标签字符串替换成新格式（需要手写映射表）
- 方案 B：`getSongTags()` 里做 normalize，自动去掉末尾英文（trim + 正则），让旧数据展示和筛选都和新标签一致
- 推荐方案 B，不改数据库，风险更低

## 改动规范

- 标签重构严格按 Step 1→2→3→4 顺序执行，每步独立 commit
- 小改动（CSS 参数调整）：直接改，写清楚 commit message
- 大改动（跨步骤的逻辑变更）：先出方案让我确认，再动代码
- 每次改动尽量原子化，一个 commit 做一件事，方便回退
