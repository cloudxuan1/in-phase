# CLAUDE.md — in-phase 项目说明

## 项目概述

in-phase 是一个双人音乐交换网站。两个人轮流出题（theme），各自选一首歌回应，附上 note 和标签。
部署在 GitHub Pages / Netlify，数据存在 Supabase。
名字含义：in-phase = 同相位，两个波完全同步才能叠加共振。

---

## 跟我协作的方式（每次新会话必读）

1. **我描述现象和效果，不是代码位置。** 先理解我想要的效果再决定怎么改；如果一个问题要改好几处，先停下来跟我确认意图。

2. **我说的大白话往往就是答案，先按字面意思试。** 如果听不懂，用大白话复述确认（"你是想 XX 对吗？"）。我说"就这么简单"时先相信我，确实更复杂再告诉我哪里不同。

3. **解释时先说视觉效果，再补技术原因。** 比如"展开的卡和折叠的一样宽（因为 contain: inline-size）"，让我先理解在说什么，再决定要不要深入。

4. **直接用术语解释，不要怕我听不懂。** 遇到技术原因或改动方案，直接说 class 名、CSS 属性、函数名。我看不懂会发给 GPT，回避术语反而让我完全不知道你在改哪里。

---

## 分支与 main 规则（重要）

- **未经允许不得直接推 main。** 每次改动在分支开发，等用户说"在 main 直接改"才能直接推。
- **"在 main 直接改"** 授权仅限该次任务，不延续到下一个任务。
- 纯文档改动（CLAUDE.md 等）和简单小修可以直接推 main；大改动必须走 PR 流程。
- 每次改动原子化，一个 commit 一件事，方便回退。

---

## 技术栈

- 纯前端单文件：index.html（HTML + CSS + JS，React CDN，无 JSX，全用 React.createElement）
- 数据库：Supabase（project ref: kfgtrcdjusfljnvmwpsu）
- 后端：Supabase Edge Function（crosstalk-api）
- 部署：GitHub Pages + Netlify，push 到 main 自动部署

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

四张表：
- `crosstalk_pairs` — 歌曲配对（theme, asked_by, claude_song, user_song）
- `crosstalk_comments` — 留言（pair_id, side, sender, text）
- `crosstalk_settings` — 设置（claude_icon, user_icon, headphone_type, custom_tags, fav_tags, preset_groups 等）
- `crosstalk_favorites` — 收藏（pair_id, side ['claude'|'user'], created_at，UNIQUE pair_id+side）

Song JSON 格式：
```json
{
  "title": "", "artist": "", "album": "", "cover": "", "link": "", "note": "",
  "tags": { "list": ["标签1", "标签2"] }
}
```

旧格式（兼容）：`{ "tags": { "weather": "晴 Sunny", "mood": "懷念 Nostalgic" } }`

Edge Function 路由（`/crosstalk-api`）：
- GET/POST/PATCH `/pairs` `/pairs/:id`
- GET/POST `/comments`
- GET/PATCH `/settings`
- GET `/search?q=`
- GET/POST/DELETE `/favorites`

---

## 绝对不能动 / 不要做的事

- 数据库表结构 / Supabase Edge Function / Supabase API 路径
- Reply / Ask 核心提交逻辑 / ⇄ 配对跳转逻辑
- 卡片双栏瀑布流总览布局 / 删除功能 / 留言系统 / 交换记录点击逻辑
- 不要加 position: sticky 多层吸顶（iPhone Safari 会错位）
- 不要改 Settings emoji 网格（5×4，前 19 固定 + 第 20 格是 +）
- 不要修改 icon-preview 的 65px 宽度

---

## 已解决问题（防止绕弯路）

### ✅ iPhone Safari 展开卡片触发整页缩放（PR#13）

**产品意图**：点开一张卡片，那一侧的列变大，另一侧缩小，总宽不变；**顶部列名也要跟着同步缩放**。

**正确方案**：
- 默认 50/50；左展开 65/35；右展开 35/65
- React 派生 className（`left-expanded` / `right-expanded`）加在 `.columns` 上
- `.col { min-width: 0 }` 阻止内容撑宽已定好的列

**❌ 弯路**：逐个修 input / note 的 min-width、`contain: inline-size`、`position: absolute/sticky`、全局 `overflow: hidden`。

---

### ✅ iOS 输入框点击自动放大（PR#17）

**正确方案**：viewport meta 加 `user-scalable=no`，一次性禁止点击放大和捏合缩放。

**❌ 弯路**：给 input 加 `font-size: 16px`；`@media` 媒体查询写在普通规则前面会被后面的规则覆盖失效。

---

### ✅ favTags 刷新后丢失（PR#15）

**原因**：`saveSettings` 只在设置弹窗点"保存"时触发。

**正确方案**：`useEffect` 监听 `favTags` 变化自动 PATCH；用 `useRef(false)` 跳过首次渲染。

---

### ✅ GitHub Pages Jekyll 构建失败

**方案**：根目录加空文件 `.nojekyll`。

---

### ✅ 按钮/标签高度不一致（PR#17）

**原因**：DotGothic16 字体在 12px 下不同汉字字形高度略有差异。

**方案**：所有按钮类加 `line-height: 1.4` + `white-space: nowrap`。

---

### ✅ 更換 Supabase URL 按钮无效（PR#19）

**原因**：`getApiUrl()` 优先读 `window.location.hash`；`localStorage.removeItem` 后 reload，hash 仍在，URL 被重新写回。

**正确方案**：reload 前先 `history.replaceState(null, "", pathname + search)` 清除 hash。

---

### ✅ 删除确认 modal 点垃圾桶不出现（PR#19）

**原因**：`deleteConfirmKey && React.createElement(...)` 作为第三个参数传给了 `showResetConfirm` backdrop 的 `React.createElement`，导致它嵌套在 showResetConfirm 里——只有 showResetConfirm 为 true 时才渲染。

**正确方案**：两个 modal 在 `.overlay` 下并列，不嵌套。数括号时要逐层追踪。

---

## 已完成功能

### ✅ 收藏标签（喜欢）系统（PR#15）
- Filter 新增"喜欢"行，显示已收藏标签，点击直接筛选
- ＋按钮打开大弹窗，按分类展示所有标签，支持搜索
- `fav_tags JSONB` 列加到 `crosstalk_settings`，自动持久化

### ✅ 预设分类按钮化（PR#15）
- Filter 预设行从平铺标签改为分类按钮（天气/场所/时间/心情/自定义/常用）

### ✅ Exchange 导航重设计（main）
- 切换按钮改为内联格式：`‹ #39 '白月光' ›`，主题文字用 accent 颜色

### ✅ 搜索框 + 按钮统一（PR#17）
- 搜索框添加 SVG 图标，所有按钮加 `line-height: 1.4`

### ✅ 卡片收藏功能（PR#18）
- 新表 `crosstalk_favorites`，Edge Function 加 GET/POST/DELETE `/favorites`
- 三态心形：♡（未收藏）→ 淡♥（单侧收藏）→ 实♥（双侧都收藏）
- 心形在歌名行，点击乐观更新 + API 持久化
- `favSort` 状态：默认排序 / 最新收藏在前（按 created_at 降序）

### ✅ 标签管理 Accordion 重设计（PR#19）
- 一次只能展开一个标签组，点开另一个自动关闭
- 展开面板：组名内联编辑、原生色盘改色、标签 chip + 新增输入框（回车/逗号/+）
- 删除两条路径：垃圾桶 → modal 确认；左滑 → 两步确认
- 移除旧 editingKey / hexInputs / editingGroup 全套状态

### ✅ IN~PHASE 品牌化（PR#19）
- SetupScreen：CROSSTALK → IN~PHASE，🎵 → 像素耳机图（PH 组件）
- Loading 界面：Loading Crosstalk → Loading IN~PHASE
- PWA apple-touch-icon：Canvas 绘制像素耳机，随 `headphoneType` 动态更新

---

## 经验教训（调试 / 协作）

- **序号标记法**：在元素文字后加 ①②③ 调试序号，能快速识别哪些元素共用同一个 class。
- **Python 脚本处理 unicode escape**：index.html 里的汉字以 `常` 形式存储时，Edit 工具字符串匹配会失败，用 Python 按行操作可以绕过。
- **React.createElement 括号计数**：多层嵌套时极易把一个 modal 当成另一个 modal 的子元素传进去，导致条件渲染互相依赖。改之前先数清楚哪个 `)` 关哪个 `createElement`。
- **git rebase + force-with-lease**：PR 分支落后 main 时，`git rebase origin/main` 让提交重新接在最新 main 上，再 `git push --force-with-lease` 推送。
- **cherry-pick**：把另一个分支上的单个 commit 搬到当前分支。
- **color-mix()**：`color-mix(in srgb, var(--accent) 25%, transparent)` 直接生成半透明色，比手写 rgba 更灵活。
- **hash URL 持久化陷阱**：`localStorage.removeItem` + `reload` 不等于"清除配置"——`getApiUrl()` 先读 hash，hash 不清就白清。

---

## 下一步

### V7：盲选 Blind Exchange

同一主题下，双方各自选歌，提交前互相看不到对方选了什么，两边交齐后同时揭晓。

**流程：**
1. 一方创建主题（blind pair）
2. Claude / User 各自提交一首歌
3. 未揭晓前只显示"已提交 / 未提交"状态，不显示对方歌名、封面、note、标签
4. 双方都提交后自动揭晓
5. 揭晓后变回普通 pair 展示，双列卡片 / 标签 / note / 留言 / 收藏照常使用

**第一版范围：**
- 创建 blind pair
- 未揭晓状态 UI
- 提交状态
- 双方交齐后揭晓
- 揭晓后复用现有卡片展示

**暂不做：**
- 双人标签 / 年度总结 / 盲选统计 / 揭晓动画 / 复杂权限

---

## 部署与发布流程

**站点**：https://calm-buttercream-7225c0.netlify.app/
**类型**：静态单文件，Build command 留空，Publish directory 填 `.`

### 代码改动流程

1. 在独立分支开发，推送并开 PR（draft）
2. 等用户测试确认后再 merge 到 main
3. 不要顺手重构、改 UI、改数据库、改环境变量或改部署配置

### 新仓库接入 Netlify

1. Netlify → Add new project → Import an existing project → 选 GitHub → 选仓库
2. 静态项目：Build command 留空，Publish directory 填 `.`
3. Deploy → 等第一次部署成功
4. 开启 Deploy Previews：Project configuration → Build & deploy → Continuous deployment → Branches and deploy contexts → Configure → 打开 **Any pull request against your production branch**
