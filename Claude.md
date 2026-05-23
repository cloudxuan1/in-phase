# Claude.md — in-phase 项目说明

## 项目概述

in-phase 是一个双人音乐交换网站。两个人轮流出题（theme），各自选一首歌回应，附上 note 和标签。
部署在 GitHub Pages，数据存在 Supabase。
名字含义：in-phase = 同相位，两个波完全同步才能叠加共振。

---

## 跟我协作的方式（每次新会话必读）

1. **我用产品视角描述问题。** 我说的是"看起来是什么样"，不是"代码怎么写"。

2. **我说的大白话，往往就是答案。** 优先按字面意思理解和执行。如果听不懂，用大白话复述给我确认（"你是想 XX 对吗？"），稍加技术术语配合确认。

3. **当我说"就这么简单"，先相信我的判断去试。** 我可能已经观察了几天才说这句话。如果你发现实际情况确实更复杂，再告诉我哪里不同。

4. **我描述的是现象，不是具体修哪行代码。** 先理解我想要的效果，再决定怎么改。如果一个问题要连续改好几处才能修好，先停下来跟我确认意图，可能思路需要调整。

5. **解释问题时，先用视觉效果描述**（比如"展开的卡会变得和折叠的一样宽，放大效果消失"），**再补技术原因**（比如"因为 contain: inline-size"）。这样我能先理解在说什么，再决定要不要深入。

6. **用术语解释，不要怕我听不懂。** 遇到需要说明技术原因或改动方案的地方，直接用正确的术语讲（class 名、CSS 属性、函数名等）。我看不懂会发给 GPT 帮我理解，不用刻意回避术语——回避之后我反而完全不知道你在改哪里。

---

## 分支与 main 规则（重要）

- **未经允许不得直接推 main。** 每次改动要在分支开发，等用户说"在 main 直接改"才能直接推。
- 用户说 **"在 main 直接改"** = 当前任务可以直接推 main，授权仅限该次，不延续到下一个任务。
- 纯文档改动（Claude.md 等）可直接推 main，无需走 PR 流程。

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

### ✅ iOS 输入框点击自动放大（PR#17 已上线）

**产品意图**：点搜索框弹出键盘，但页面不要缩放。

**正确方案**：viewport meta 加 `user-scalable=no`，一次性禁止所有缩放（点击放大 + 捏合）。

**❌ 不要再走的弯路**：
- 给 input 加 `font-size: 16px`（iOS 低于 16px 才触发放大，但改字号会影响布局，且 CSS 层叠顺序容易出错）
- `@media (max-width: 768px) { font-size: 16px }` 媒体查询放在普通规则前面时会被后面的规则覆盖，失效

**教训**：CSS 层叠是按文件中出现的顺序，后写的优先级更高。媒体查询不是"更强的规则"，它和普通规则同等竞争，放在前面就会被后面的普通规则盖掉。

---

### ✅ 按钮/标签高度不一致（PR#17 已上线）

**现象**：DotGothic16 字体在 12px 下，不同汉字字形高度略有差异，导致同行按钮高矮不齐。

**正确方案**：所有按钮类加 `line-height: 1.4`（统一行高）+ `white-space: nowrap`（防换行导致高度变化）。

---

### ✅ GitHub Pages Jekyll 构建失败

**原因**：Pages 默认会用 Jekyll 处理文件，遇到 `_` 开头路径会出错。

**方案**：在仓库根目录加一个空文件 `.nojekyll`，Pages 就直接伺服静态文件，不走 Jekyll。

---

### ✅ favTags 刷新后丢失（PR#15 已上线）

**原因**：`saveSettings` 只在设置弹窗点"保存"时触发，收藏标签的改动没有被保存。

**正确方案**：用 `useEffect` 监听 `favTags` 变化，每次变更自动 PATCH `fav_tags` 到 Supabase。用 `useRef(false)` 跳过首次渲染，避免组件挂载时立刻触发空写入。

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
- 切换按钮从独立 btn-small 改为内联格式：`‹ #39 '白月光' ›`
- 主题文字用 accent 颜色，‹ › 无边框，整体更紧凑

### ✅ 搜索框优化（PR#17）
- 添加 SVG 搜索图标，删掉过长 placeholder
- 字号统一到 12px，删除 16px 媒体查询 hack

### ✅ 全局按钮字号与高度统一（PR#17）
- 所有按钮类加 `line-height: 1.4` + `white-space: nowrap`

---

## 经验教训（调试 / 协作）

- **序号标记法**：在元素文字后加 ①②③ 调试序号，能快速识别哪些元素共用同一个 class，沟通字号调整时效率大幅提升。
- **Python 脚本处理 unicode escape**：index.html 里的汉字以 `常` 形式存储时，Edit 工具的字符串匹配会失败。用 Python 按字节操作可以绕过这个问题。
- **git rebase 而不是 merge**：PR 分支落后 main 时，用 `git rebase origin/main` 让 PR 分支的提交重新接在最新 main 上，再 `git push --force-with-lease` 推送，保持历史线性干净。`--force-with-lease` 比 `--force` 安全，会检查远端有没有被别人改过。
- **cherry-pick**：可以把另一个分支上的单个 commit 搬到当前分支，不用整个分支 merge。
- **CSS 层叠顺序**：媒体查询 `@media` 不是"更强的规则"，它和普通规则平等竞争，写在文件前面就会被后面的普通规则覆盖。
- **color-mix()**：`color-mix(in srgb, #f4845f 25%, transparent)` 直接生成半透明色，比手写 rgba 更灵活，可以引用 CSS 变量。
- **.nojekyll 文件**：GitHub Pages 部署静态 HTML 时必须在根目录放这个空文件，否则 Jekyll 构建会报错（38 次才解决）。

---

## 下一步

### 继续完善 Settings 里的标签管理（第三页）

目前标签管理页面（设置 → 管理标签组 → 展开某组）交互和 UI 仍需改进：
- 排版和字号不协调，输入框偏小
- Done 按钮行为：应该保存并返回上一级（当前分支 PR#16 已有改动，待审阅）
- 整体交互体验待打磨

### 旧标签数据兼容 ⏳

旧数据 `"晴 Sunny"` 和新标签 `"晴"` 是两个不同字符串，筛选时无法合并命中。

方案（不改数据库）：在 `getSongTags()` 里自动去掉末尾英文，让旧数据展示和筛选都和新标签一致。

---

## 改动规范

- 大改动（跨模块逻辑）：先描述方案让我确认，再动代码
- 每次改动原子化，一个 commit 一件事，方便回退

---

## 部署与发布流程

**站点**：https://calm-buttercream-7225c0.netlify.app/
**类型**：静态单文件，Build command 留空，Publish directory 填 `.`

### 代码改动流程（必须走 PR）

1. 在独立分支开发
2. 推送并开 PR（draft）
3. Netlify 自动生成 Deploy Preview —— **在回复结尾附上 PR 链接**
4. 等用户测试确认
5. 用户确认后再 merge 到 main

- 没有 preview 不要 merge，先说明原因和解决办法
- 简单改动可以问用户"这个直接推 main 可以吗"，用户说可以就直接推
- 不要顺手重构、改 UI、改数据库、改环境变量或改部署配置

### 纯文档改动（Claude.md 等）

可以直接推 main，不需要走 PR 流程。

### 新仓库接入 Netlify 步骤

1. Netlify → Add new project → Import an existing project → 选 GitHub → 选仓库
2. 部署设置：
   - 静态项目：Build command 留空，Publish directory 填 `.`
   - 其他项目：先确认正确的 build / publish 再填
3. Deploy → 等第一次部署成功
4. 开启 Deploy Previews：
   Project configuration → Build & deploy → Continuous deployment → Branches and deploy contexts → Configure → 打开 **Any pull request against your production branch / branch deploy branches**
