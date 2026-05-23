# Claude.md — in-phase 项目说明

## 项目概述

in-phase 是一个双人音乐交换网站。两个人轮流出题（theme），各自选一首歌回应，附上 note 和标签。
部署在 GitHub Pages，数据存在 Supabase。
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
- 纯文档改动（Claude.md 等）和简单小修可以直接推 main；大改动必须走 PR 流程。
- 每次改动原子化，一个 commit 一件事，方便回退。

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
- `crosstalk_settings` — 设置（icon, custom_tags, fav_tags 等）

Song JSON 格式：
```json
{
  "title": "", "artist": "", "album": "", "cover": "", "link": "", "note": "",
  "tags": { "list": ["标签1", "标签2"] }
}
```

旧格式（兼容）：`{ "tags": { "weather": "晴 Sunny", "mood": "懷念 Nostalgic" } }`

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

**❌ 弯路**：逐个修 input / note 的 min-width、`contain: inline-size`（展开卡和折叠卡一样宽）、`position: absolute/sticky`、全局 `overflow: hidden`。

**教训**：用户说"顶部没有同步放大"就是答案——顶部列名也要跟着列宽走，不要只修卡片内部。

---

### ✅ iOS 输入框点击自动放大（PR#17）

**正确方案**：viewport meta 加 `user-scalable=no`，一次性禁止点击放大和捏合缩放。

**❌ 弯路**：给 input 加 `font-size: 16px`；`@media` 媒体查询写在普通规则前面会被后面的规则覆盖失效（CSS 层叠按文件顺序，后写优先，`@media` 不是"更强的规则"）。

---

### ✅ favTags 刷新后丢失（PR#15）

**原因**：`saveSettings` 只在设置弹窗点"保存"时触发。

**正确方案**：用 `useEffect` 监听 `favTags` 变化自动 PATCH `fav_tags`；用 `useRef(false)` 跳过首次渲染，避免挂载时触发空写入。

---

### ✅ GitHub Pages Jekyll 构建失败

**方案**：根目录加空文件 `.nojekyll`，Pages 直接伺服静态文件，不走 Jekyll。

---

### ✅ 按钮/标签高度不一致（PR#17）

**原因**：DotGothic16 字体在 12px 下不同汉字字形高度略有差异。

**方案**：所有按钮类加 `line-height: 1.4`（统一行高）+ `white-space: nowrap`（防换行）。

---

## 已完成功能

### ✅ 收藏标签（喜欢）系统（PR#15）
- Filter 新增"喜欢"行，显示已收藏标签，点击直接筛选
- ＋按钮打开大弹窗，按分类展示所有标签，支持搜索，点击收藏/取消
- `fav_tags JSONB` 列加到 `crosstalk_settings`，自动持久化

### ✅ 预设分类按钮化（PR#15）
- Filter 预设行从平铺标签改为分类按钮（天气/场所/时间/心情/自定义/常用）
- 点击任意分类按钮弹出小浮窗显示该类标签

### ✅ Exchange 导航重设计（main）
- 切换按钮改为内联格式：`‹ #39 '白月光' ›`，主题文字用 accent 颜色，整体更紧凑

### ✅ 搜索框 + 按钮统一（PR#17）
- 搜索框添加 SVG 图标，还原 placeholder，删除 16px 媒体查询 hack
- 所有按钮加 `line-height: 1.4` + `white-space: nowrap`

---

## 经验教训（调试 / 协作）

- **序号标记法**：在元素文字后加 ①②③ 调试序号，能快速识别哪些元素共用同一个 class，沟通字号调整效率大幅提升。
- **Python 脚本处理 unicode escape**：index.html 里的汉字以 `常` 形式存储时，Edit 工具字符串匹配会失败，用 Python 按字节操作可以绕过。
- **git rebase + force-with-lease**：PR 分支落后 main 时，`git rebase origin/main` 让提交重新接在最新 main 上，再 `git push --force-with-lease` 推送（比 `--force` 安全，会检查远端是否被改过）。
- **cherry-pick**：把另一个分支上的单个 commit 搬到当前分支，不用整个分支 merge。
- **color-mix()**：`color-mix(in srgb, var(--accent) 25%, transparent)` 直接生成半透明色，比手写 rgba 更灵活。

---

## 下一步

### 继续完善 Settings 里的标签管理（第三页）
目前设置 → 管理标签组 → 展开某组的交互和 UI 仍需改进：排版字号不协调、输入框偏小、整体体验待打磨。

### 喜欢的卡片收藏功能 ⏳
允许用户收藏喜欢的音乐交换卡片。具体交互和实现方式待定，需要和用户进一步讨论。

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
