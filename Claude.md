# Claude.md — in-phase 项目说明

## 项目概述

in-phase 是一个双人音乐交换网站。两个人轮流出题（theme），各自选一首歌回应，附上 note 和标签。
部署在 GitHub Pages，数据存在 Supabase。
名字含义：in-phase = 同相位，两个波完全同步才能叠加共振。

---

## 跟我协作的方式（每次新会话必读）

**我用产品视角描述问题。** 我说的是"看起来是什么样"，不是"代码怎么写"。

1. **我说的大白话，往往就是答案。** 不要包装成多选题，不要拆成"你是想 X 还是 Y"。如果听不懂，用大白话复述给我确认（"你是想 XX 对吗？"），不要用技术术语反问。

2. **当我说"就这么简单"，相信我。** 我可能已经观察了几天才说这句话。不要假设底层很复杂。

3. **不要逐个修元素来追症状。** 我描述的是"现象"，你要找的是"为什么不符合我的意图"。如果修一个问题需要 R1、R2、R3……，多半方向错了，先停下来问我意图。

4. **解释问题时，先说视觉效果，再说技术原因。** 不要上来就堆术语。比如先说"展开的卡会变得和折叠的一样宽，放大效果消失"，再说"因为 contain: inline-size"——这样我能先理解在说什么，再决定要不要深入。

---

## 技术栈

- 纯前端单文件：index.html（HTML + CSS + JS，React CDN）
- 数据库：Supabase（project ref: kfgtrcdjusfljnvmwpsu）
- 后端：Supabase Edge Function（/pairs /comments /settings /search）
- 部署：GitHub Pages，push 到 main 自动部署

---

## 设计系统

所有颜色用 CSS 变量，不要写死色值：

```css
--bg: #faf5ef;     --card: #fefbf6;
--border: #8a8078; --accent: #f4845f;
--cream: #faf0e6;  --text: #5a534c;
--muted: #a0907e;  --light: #e8ddd0;
--tan: #d4a574;    --shadow: #d4c4b0;
```

整体风格：暖色调、像素风细节、低饱和。不要引入高饱和或冷色调元素。

---

## 数据库（结构不能改）

三张表：
- `crosstalk_pairs` — 歌曲配对（theme, asked_by, claude_song, user_song）
- `crosstalk_comments` — 留言（pair_id, side, sender, text）
- `crosstalk_settings` — 设置（icon, custom_tags 等）

Song JSON 格式：
```json
{
  "title": "", "artist": "", "album": "", "cover": "", "link": "", "note": "",
  "tags": { "list": ["标签1", "标签2"] }
}
```

旧格式（兼容）：`{ "tags": { "weather": "晴 Sunny", "mood": "懷念 Nostalgic" } }`

---

## 绝对不能动

- 数据库表结构 / Supabase Edge Function / Supabase API 路径
- Reply / Ask 核心提交逻辑
- ⇄ 配对跳转逻辑
- 卡片双栏瀑布流总览布局
- 删除功能 / 留言系统 / 交换记录点击逻辑

## 不要做的事

- 不要加 position: sticky 多层吸顶（iPhone Safari 会错位）
- 不要改 Settings emoji 网格（5×4，前 19 固定 + 第 20 格是 +）
- 不要修改 icon-preview 的 65px 宽度

---

## 已解决问题（防止绕弯路）

### ✅ iPhone Safari 展开卡片触发整页缩放（V4 / PR#13 已上线）

**产品意图（最重要）**：
- 点开一张卡片，那一侧的列变大，另一侧缩小，**总宽不变**
- **顶部列名（"Claude" / "xuan"）要跟着同步缩放对齐** ← 这是关键，漏掉这里等于没修好
- 列宽由状态显式控制，不让内容自动撑宽

**正确方案**：
- 默认 50/50；左展开 65/35；右展开 35/65
- React 派生 className（`left-expanded` / `right-expanded`）加在 `.columns` 上
- `.col { min-width: 0 }` 阻止内容撑宽已定好的列
- `.song-title` / `.note` / `.comment-input input` 加 overflow-wrap / min-width 防御

**❌ 不要再走的弯路**：
- 逐个修 input / note / theme-title 的 min-width / word-break（治不完，根因不在这）
- `contain: inline-size`（展开卡和折叠卡会一样宽，放大效果消失）
- `position: absolute / sticky`（破坏原地展开感）
- 全局 `overflow: hidden`（裁掉内容）

**教训**：用户说"顶部没有同步放大"已经是完整的答案——顶部列名也要跟着列宽比例走。这句话被忽略了多次导致反复返工。听到类似描述，直接找顶部对应元素，不要只修卡片内部。

---

## 下一步：V5

### 1. 旧标签数据兼容 ⏳

旧数据 `"晴 Sunny"` 和新标签 `"晴"` 是两个不同字符串，筛选时无法合并命中。

方案（不改数据库）：在 `getSongTags()` 里自动去掉末尾英文，让旧数据展示和筛选都和新标签一致。

### 2. Settings 里可以编辑预设标签组 ⏳

现在 preset groups（weather / place / time / mood）是写死在代码里的。
V5 要在设置页面里让用户可以自己增删改预设标签组的内容。

---

## 标签系统（v3.3，已完成）

**已上线状态**：
- 存储：`tags.list` 数组，旧格式自动兼容
- 展示：统一胶囊样式（--muted 淡底 + --text 文字），不按分类染色，总览最多 4 个 + N
- 提交：自由输入框（回车/逗号添加），quick tags 按历史频率前 8，每首歌最多 8 个标签
- 搜索：单一搜索框，范围覆盖歌名、歌手、主题、note、标签、留言，筛选逻辑 OR
- `getSongTags(song)` 统一转换新旧格式，所有渲染/筛选/统计都用这个函数

**分组定义**：
- preset：weather / place / time / mood
- custom（展示分类，不写进数据）：scene / texture / memory / other

---

## 改动规范

- 小改动（CSS 参数）：直接改，commit message 写清楚
- 大改动（跨模块逻辑）：先描述方案让我确认，再动代码
- 每次改动原子化，一个 commit 一件事，方便回退
