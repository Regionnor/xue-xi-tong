# 更新日志 (Changelog)

本文件记录「学习通AI答题小助手（Region改良款）」的所有重要改动。

> 本脚本基于 [ScriptCat #711《🥇超星学习通｜知到智慧树——网课小助手》](https://scriptcat.org/zh-CN/script-show-page/711)（原版 v0.2.7）改良而来，在此向原作者致敬。

格式参考 [Keep a Changelog](https://keepachangelog.com/)，版本号遵循语义化版本。

---

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
