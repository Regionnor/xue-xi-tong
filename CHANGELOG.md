# 更新日志 (Changelog)

本文件记录「学习通AI答题小助手（Region改良款）」的所有重要改动。

> 本脚本基于 [ScriptCat #711《🥇超星学习通｜知到智慧树——网课小助手》](https://scriptcat.org/zh-CN/script-show-page/711)（原版 v0.2.7）改良而来，在此向原作者致敬。

格式参考 [Keep a Changelog](https://keepachangelog.com/)，版本号遵循语义化版本。

---

## [1.16] - 2026-10-28

### 🛠 修复 (Fixed) - 2026-10-28
- **彻底解决「启用脚本后所有超星页面卡死」**：根因有两层——
  1. v1.12 引入的 AI 预设下拉使用了 `el-select`，在 closed shadow DOM 内触发 ResizeObserver 死循环（已在 1.15 改为原生 `<select>` 修复）；
  2. 脚本浮动面板在**每个超星页面都会挂载**，而根组件的 `el-tabs` 会**急切渲染所有 `el-tab-pane`**（仅用 `v-show` 隐藏），导致「答题」标签页（含上述 `el-select`）在页面加载时就被挂载、即便未切换到该标签页也会转起来 —— 表现为「任意超星页面一启用脚本就卡死」。
- **本次加固**：把各 `el-tab-pane` 的内容改为**懒挂载**——仅当 `configStore.menuIndex === tab.name`（即该标签页被选中）时才渲染对应组件，未选中的标签页内容渲染为 `null` 不挂载。这样页面加载时只挂载默认「首页」标签页；即便停留在旧版本，非默认标签页也不会在加载期被挂起空转。
- 注意：若你仍停留在 1.14 或更早，请务必升级到 1.16（含 1.15 的原生 select 修复 + 本次懒挂载）。升级后建议硬刷新（Ctrl+F5）或重新拖入脚本以绕过 jsDelivr ~12h 缓存。

---

## [1.15] - 2026-10-28

### 🛠 修复 (Fixed) - 2026-10-28
- **修复安装 v1.12+ 后「答题」设置页卡死、CPU 飙高（风扇狂转）**：v1.12 新增的 AI 预设下拉使用了 Element Plus 的 `el-select`。该组件内部依赖 **teleport + popper 弹层 + ResizeObserver 尺寸测量**，而整个面板挂载在 **closed shadow DOM** 内、Element Plus 样式无法穿透，导致 `el-select` 的尺寸测量失稳、触发 **ResizeObserver 死循环**，表现为页面卡死、CPU 占满。
  - 改为使用 **原生 `<select>` + `<option>`**（内联样式，在 shadow DOM 内始终生效，无 teleport/popper/ResizeObserver 负担），与之前把 `el-scrollbar` 换成原生 `<div>` 是同一思路。
  - 「测试连接」按钮继续沿用安全的 `el-button`（自 v1.11 导出按钮起已在 shadow DOM 内稳定使用），未改动。
  - 若此前在 1.12~1.14 上遇到卡死，升级到 1.15 后应恢复正常。

---

## [1.14] - 2026-10-28

### 🔧 修正 (Fixed) - 2026-10-28
- **修正小米 MiMo 预设端点**：将预设「小米 MiMo」的接口地址由社区流传值 `https://api.miwifi.com/v1` 更正为官方 OpenAI 兼容地址 `https://api.xiaomimimo.com/v1`（官方同时提供 Anthropic 兼容路径 `https://api.xiaomimimo.com/anthropic`，本预设采用 OpenAI 兼容）。API Key 格式为 `sk-xxxxx`（于官方 API Keys 页面创建）。现已移除「端点请核实」提示。模型字段仍留空，由用户按账户实际可用模型填写。

---

## [1.13] - 2026-10-28

### 🔧 调整 (Changed) - 2026-10-28
- **预设接口源不再预填「模型」，仅填「地址」**：根据反馈，各厂商 / 账户可用的模型名差异较大，预设下拉（OpenAI / DeepSeek / 通义千问 / 豆包 / Kimi / 小米 MiMo / 空白）现只一键填好接口地址，模型字段留空由用户自行填写（占位提示：`AI 模型，如 gpt-4o-mini / deepseek-chat`）。选任一项会同步清空模型框，避免沿用旧模型导致不匹配。

---

## [1.12] - 2026-10-17

### ✨ 新增 (Added) - 2026-10-17
- **AI 接口「预设接口源」下拉框**：在「答题」设置 → AI 接口答题区，地址输入框上方新增一个 `el-select` 预设源，选一项即可一键填好「地址」与「模型」：
  - `OpenAI (GPT)` → `https://api.openai.com/v1` / `gpt-4o-mini`
  - `DeepSeek (深度求索)` → `https://api.deepseek.com/v1` / `deepseek-chat`
  - `通义千问 (Qwen)` → `https://dashscope.aliyuncs.com/compatible-mode/v1` / `qwen-plus`
  - `豆包 (火山方舟)` → `https://ark.cn-beijing.volces.com/api/v3` / `doubao-seed-1-6-250615`
  - `Kimi (月之暗面)` → `https://api.moonshot.cn/v1` / `moonshot-v1-8k`
  - `小米 MiMo（端点请核实）` → `https://api.miwifi.com/v1` / `mimo`（⚠️ 该端点为社区流传值，如不通请自行核对官方地址后修改）
  - `空白（自定义）` → 清空地址与模型，手动填写自有接口
- **AI 接口「测试连接」按钮**：在自定义要求输入框下方新增「测试连接」按钮。点击后用当前「地址 / 密钥 / 模型」向 `/chat/completions` 发送一条极简 `ping` 请求（`max_tokens:5`），并在按钮右侧实时显示结果：
  - 成功（HTTP 2xx 且返回 `choices`）→ 绿色「连接成功（HTTP xxx）」
  - 失败 → 红色提示，区分「HTTP 非 2xx + 响应片段」「返回异常（非标准 JSON）」「请求出错（地址/网络/跨域）」「请求超时（20s）」「请先填写 API Key / 接口地址」等情形。

---

## [1.11] - 2026-10-16

### 🛠 修复 (Fixed) - 2026-10-16
- **修复首页日志区「往下拉的条拉不动」（滚动失效）**：首页 `首页` 标签页原本用 Element Plus 的 `<el-scrollbar>` 包裹日志列表。由于整个面板挂载在 **closed shadow DOM**（`attachShadow({mode:"closed"})`）内，而 Element Plus 样式只通过 `GM_addStyle` 注入到页面 `document`，**不会穿透 shadow 边界**，导致 `el-scrollbar` 的滚动条/滑块样式缺失、无法拖拽。
  - 改为使用**原生可滚动 `<div class="log">`**（通过内联 `style` 设置 `max-height:230px; overflow-y:auto`，内联样式在 shadow DOM 内始终生效），彻底摆脱对 `el-scrollbar` 样式的依赖，日志现在可正常上下滚动。

### ✨ 新增 (Added) - 2026-10-16
- **首页日志「导出」功能**：在日志区顶部新增工具栏（左侧「运行日志」标题，右侧「导出」按钮）。点击「导出」将当前 `logList` 逐条格式化为 `时间  类型  内容`（自动剔除 HTML 标签），以 UTF-8 文本 Blob 触发浏览器下载为 `chaoxing-log-<时间戳>.txt`。导出逻辑在脚本页面上下文内执行，无需额外 `@grant` 权限。

---

## [1.10] - 2026-10-15

### 🛠 修复 (Fixed) - 2026-10-15
- **修正「最高倍速」被错误地设为 16x 导致播放失败**：超星播放器速度菜单默认上限为 **2x**（个别课程更高），原本按 `[16,8,4,3,2,…]` 由高到低试设会越界，超出播放器允许范围而播放异常。
  - 改为**优先读取播放器速度菜单里可选的最高项**（解析 `.vjs-playback-rate`/`.speedMenu` 等节点中 `Nx` 标签或 `data-rate` 属性，取最大值），默认即 2x，课程若开放更高则自动取更高。
  - 若菜单尚未加载（刚定位到视频时），**退回「不超过 2x」的安全范围** `[2,1.75,1.5,1.25]` 由高到低试设，绝不越界。
  - 定位到视频约 2.5s 后**再次探测菜单**（此时菜单通常已就绪），若发现更高可选项则上调倍速。
  - `ratechange` 重设逻辑改为「仅在播放器把倍速**调低**到低于目标值时，才重新设回目标值」，不再强行反复写 16x，避免与播放器互相打架。
  - 日志由「已设置最高倍速：Nx」改为「已设置倍速：Nx（取播放器可选上限，避免超出导致播放失败）」，语义更准确。

---

## [1.9] - 2026-10-13

### ✨ 新增 (Added) - 2026-10-13
- **视频加速与智能跳播（四项优化，均在 `processMedia` 中处理视频任务时生效）**：
  1. **「任务已完成」不再重复点击播放**：无论是轮询开始时、播放前还是播放后的检查，只要 `isFinishedTask(iframe)` 判定任务点已标记完成（`.ans-job-finished`），立即 `finish()` 跳过并放行下一节，避免无意义的反复点击、浪费时间。
  2. **自动最高倍速**：定位到 `<video>` 后，按 `[16,8,4,3,2,1.75,1.5,1.25]` 由高到低尝试设置 `playbackRate`，取浏览器接受的最高值（通常为 16x）并持续在 `ratechange` 事件上重新声明，对抗播放器强行复原，日志打印 `已设置最高倍速：Nx`。
  3. **支持拖拽时跳到结尾前几秒**：若视频时长可用，将 `currentTime` 设置为 `duration - 5s` 实现「拖到结尾前」以极速过完。约 1.5s 后校验实际跳转结果：成功则提示「已拖拽进度条至结束前约 5s」；若被复位则提示「进度条无法拖动，转为正常高倍速播放」。
  4. **「教师设置无法拖动进度条」提示识别**：拖拽后若页面/任务 iframe 内出现该文本（轮询检测），按用户设定**视为已接近结尾**，不再重试拖拽，转为正常高倍速播放直至 `ended` 自然完成，避免无谓报错与卡顿。
- ⚠️ 说明：第 3/4 项属于「能拖就拖、不能拖就倍速播完」的兼容策略；当教师禁止拖动时，脚本会自动回退到最高倍速正常播放，不会卡死也不会反复报错。

---

## [1.8] - 2026-10-13

### 🔧 优化 (Changed) - 2026-10-13
- **启动日志带上版本号**：`脚本加载成功，正在解析网页（当前版本 vX.X）`，便于一眼确认当前运行的是哪一版，避免「刷新页面以为更新了、其实还是旧版」的误判（问题排查时特别有用）。
- 版本号取自 `GM_info.script.version`，与脚本头 `@version` 保持一致。

---

## [1.7] - 2026-10-13

### 🛠 修复 (Fixed) - 2026-10-13
- **在 v1.6 基础上进一步强化视频定位**（应对「播放器不在任务 iframe 内」的情况）：
  - 新增**整页兜底查找**：若本 iframe（含其所有嵌套 iframe）仍找不到 `<video>/<audio>`，则**在本 iframe 之外、遍历整页所有 iframe** 定位真正带 `src` 的播放器（约 5s 后触发一次），覆盖播放器与任务 iframe 不同层/不同源的情况。
  - 找不到媒体元素时的「放弃等待」时间由约 30s 缩短到**约 12s**（避免长时间卡住），届时打印 `[诊断] 仍未找到 …` 并结束本任务点，保证后续章节仍能切换。
  - `[诊断]` 日志更明确：区分「本 iframe 内找到」与「本 iframe 之外找到」。
- ⚠️ 说明：v1.5 及更早版本在找不到 `<video>` 时会**永久挂起**，导致视频不播放且**不切下一节**；v1.6/v1.7 已修复。请务必**重新导入本脚本**（仅刷新页面不会更新已安装的旧版本）。

---

## [1.6] - 2026-10-13

### 🛠 修复 (Fixed) - 2026-10-13
- **修复「视频不再自动播放」且「看完不自动切下一节」两个问题（同一根因）**：
  - 根因：`processMedia` 只在 `iframe` 的 `documentElement` 直接层 `querySelector("video")` 找播放器。超星当前页面里 `<video>` 常嵌套在更深的 iframe（或播放器另起一层）中，导致 `mediaElement` 一直为 `null`——于是 **播放逻辑从未执行（日志停在「正在尝试播放」、无「播放成功」、无报错）**，并且 `processMedia` 因找不到元素而**永久挂起不 resolve**，导致 `watchIframe` 整批任务无法「完成」，下一节切换（仅在整批完成后触发）也一并失效。
  - 媒体元素查找改为 `findMediaDeep`：**先查当前 iframe 文档，查不到则递归进入嵌套 iframe（深度 ≤ 4）**，覆盖播放器被包一层 iframe 的情况（原有直接查找作为第一优先，无回归风险）。
  - **防挂起**：连续 12 次（约 30s）仍未找到媒体元素则主动结束本任务点，避免卡住后续章节切换。
  - `play()` 失败（如被浏览器自动播放策略 `NotAllowedError` 拒绝）时不再直接判失败，而是**再尝试点击 `.vjs-big-play-button` 等播放按钮**兜底；并新增诊断日志明确提示「若仍被拦截，请先在页面/视频上点一下授权后刷新」。
  - 找到媒体元素时打印其 `src` 便于确认是否定位到正确的播放器。
- **下一节切换更稳**：「自动下一章节」触发时不再只认单一 `.jb_btn…nextChapter` 选择器，改为**依次尝试多种选择器**（`#prevNextFocusNext` / `.jb_btn…nextChapter` / `.nextChapter` 等），并补上「任务未完成」确认弹窗（`.jobFinishTip .nextChapter`）的自动确认，降低因超星改版导致按钮点不到、切不了节的概率。
- 诊断日志前缀统一为 `[诊断]`。

---

## [1.5] - 2026-10-13

### ✨ 新增 (Added) - 2026-10-13
- **脚本自动检查更新能力**：在脚本头部新增 `@downloadURL` / `@updateURL`（均指向 jsDelivr 的 `@main` 分支稳定链接 `https://cdn.jsdelivr.net/gh/Regionnor/xue-xi-tong@main/chaoxing-ai-answer-assistant.user.js`）。
  - 启用后，Tampermonkey / ScriptCat 会在打开超星页面时自动比对版本号，有新版本则提示更新，无需手动重新安装。
  - ⚠️ 自动更新依赖本仓库 `main` 分支保持最新：每次改完脚本请 `git push origin main`，jsDelivr 才会同步到最新内容（jsDelivr 对 GitHub 文件默认有约 12 小时 CDN 缓存，紧急更新可手动在 jsDelivr 链接后加 `?_=时间戳` 或前往 jsDelivr 后台 Purge 刷新）。

---

## [1.4] - 2026-10-13

### 🛠 修复 (Fixed) - 2026-10-13
- **真正修复文档/PPT 自动下滑但页面不移动的问题**：此前 `processPpt` 只 `scrollTo(window)`，而超星 PDF 阅读器内容实际渲染在嵌套的 `mooc-ans/screen/file` iframe（或 #panView）中，window 无溢出故滚动无效。
  - `processPpt` 的滚动改为 `scrollDeep`：**滚动 window / scrollingElement / body + 文档内所有可滚动容器 + 递归进入全部嵌套 iframe**（深度上限 8），确保滑到真正的内容渲染层。
  - 文档识别新增 `screen/file` 来源，避免漏判该渲染层 iframe。
  - 新增诊断日志：`[诊断] 文档滚动：本次尝试滚动 N 个可滚动容器`，便于确认是否真的滚到了内容。

## [1.3] - 2026-10-13

### 🔧 优化 (Changed) - 2026-10-13
- 抑制对**明显非任务 iframe**（章节容器页 `knowledge/cards` / `mycourse/studentstudy`、空 src、`about:blank`、`javascript:`）的「未识别到可处理任务点」告警，避免误导用户以为功能异常；仅在真正无法判定、且非上述已知容器的 iframe 上才保留诊断日志。

### ✅ 验证 (Verified)
- 用户实测日志确认 v1.2 的识别升级已生效：章节内 `ananas/modules/pdf/index.html` 文档被识别为阅读器，触发 `processPpt`（「发现一个文档，正在解析」→「阅读完成」），视频任务也正常播放。

## [1.2] - 2026-10-13

### 🛠 修复 (Fixed) - 2026-10-13
- **继续修复 PPT / 文档任务点不自动下滑阅读的问题**：v1.1 仅按 iframe 来源类型（`modules/xxx`）判定仍不足以命中部分超星页面。
  - 任务点识别升级为**三重兜底**：①来源类型（放宽为 `modules/ppt|doc|pdf|...` 或 `/{type}/index.html`）；②**按 iframe 内部内容识别**（含 `#panView` 阅读器容器或 `embed/object[type*=pdf]` 即视为文档/PPT 阅读器）；③沿祖先链（最多 6 层）查找 `.ans-job-icon`。
  - 当三条都不命中时，日志会**打印该 iframe 的真实 `src` 与 `.ans-job-icon` 有无**，便于精确适配页面结构。
  - 新增顶层诊断日志：每次枚举到新的 iframe 组合时，打印本页实际枚举到的**全部 iframe 地址**（去重），用于判断目标 iframe 是否因 `contentDocument` 为空被丢弃。

## [1.1] - 2026-10-13

### 🛠 修复 (Fixed) - 2026-10-13
- **修复网页上的 PPT / 文档任务点不再自动下滑阅读、且提前报「任务完成」的问题**：
  - 根因：`processIframeBody` 的任务点路由**强依赖**在 `iframe.parentElement` 中查找 `.ans-job-icon` 元素，超星改版后该元素位置/类名变化即导致匹配不到，从而跳过 `processPpt`/`processBook` 的下滑逻辑，最终章节循环结束误报完成。
  - 改为**优先按 iframe 来源类型（`modules/ppt`、`modules/doc`、`modules/pptx`、`modules/docx`、`modules/pdf`、`modules/innerbook` 等）判定**任务点类型并触发对应处理，不再被 `.ans-job-icon` 的 DOM 位置卡住；`.ans-job-icon` 仅作为来源无法识别时的兜底诊断。
  - 补充更清晰的跳过原因日志：明确提示「只答题，不做其他」开关开启会跳过自动阅读，以及来源不匹配且找不到 `.ans-job-icon` 时的诊断信息，便于后续排查。

---

## [1.0] - 2026-10-13

首个改良版本，基于 ScriptCat #711（v0.2.7）重构答题逻辑。

### ✨ 新增 (Added)
- **AI 答题能力**：答题模块对接**任意 OpenAI 兼容 AI 接口**（OpenAI / DeepSeek / 通义千问 / 本地 Ollama 等）。在脚本面板「答题」设置中填写接口地址、API Key、模型即可自动作答。
- 新增 `aiConfig` 配置项（启用开关 / 接口地址 / API Key / 模型 / 温度 / 自定义提示词），随脚本配置自动持久化。
- 支持调用 AI 自动完成**视频弹题、章节测试、作业与考试**作答；解析逻辑兼容单选、多选、判断、填空、简答题型。

### 🗑 移除 (Removed)
- **第三方题库与题库密钥**：不再依赖任何第三方题库接口与 token，仅保留 AI 答题。
- **知到智慧树（zhihuishu）相关代码**：移除 `ZhsQuestionHandler`、`useZhsAnswerLogic`、`hookWebpack`、`hookError`、`XMLHttpRequestInterceptor` 及对应配置 / 分发 / 平台匹配，仅专注**超星学习通**。
- **远程公告 / 版本更新功能**及其相关拉取逻辑。
- 头部过期题库域名 `@connect`（tikuhai / icodef / 62.234.36.191），仅保留 `@connect *` 以支持任意 AI 域名。

### 🔧 变更 (Changed)
- 元信息：`@name` → 学习通AI答题小助手--Region改良款；`@author` → Region；`@namespace` → `https://github.com/Region`；`@version` → `1.0`。
- `@description` 重写，强调 AI 接口兼容性与改良来源。

### 🛠 修复 (Fixed) - 2026-10-13
- 修复章节任务点处理时读取 `contentWindow` 报 `TypeError: Cannot read properties of null` 导致整批任务「任务处理失败」的问题：
  - `IframeUtils.getIframes` / `getAllNestedIframes`：元素、iframe、`documentElement` 空值判空，`contentDocument` 读取加 try/catch。
  - `processIframe`：iframe 为空或已从文档分离时直接跳过，`src` 读取容错。
  - `processPpt` / `processBook`：`iframeWindow` 与页面函数（`onchangepage` / `getFrameAttr`）判空并 try/catch。
  - `processMedia`：入参判空，`documentElement` 查询容错。
- 单个 iframe 任务点处理异常不再中断整批任务：改为记录警告日志后继续处理下一个任务点。

### 🌐 兼容性 (Compatibility)
- `@match` 覆盖超星学习通相关域名：`*.chaoxing.com`、`*.edu.cn`、`*.nbdlib.cn`、`*.hnsyu.net`、`*.gdhkmooc.com`。
- 适用环境：Tampermonkey / ScriptCat 等支持 `@require`、`@connect` 的 userscript 管理器。

---

## 使用提示
1. 在 userscript 管理器（Tampermonkey / ScriptCat）中安装本脚本。
2. 进入超星学习通对应页面，打开脚本面板「答题」页。
3. **勾选启用 AI 接口答题**，并填写你的 AI 接口地址与 API Key。
4. 进入视频、章节测试、作业或考试页面即可自动调用 AI 作答。

> ⚠️ 本脚本仅用于学习辅助，请遵守所在平台的使用条款与学校/机构的考试纪律要求。
