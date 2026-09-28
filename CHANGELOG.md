# 更新日志 (Changelog)

本文件记录「学习通AI答题小助手（Region改良款）」的所有重要改动。

> 本脚本基于 [ScriptCat #711《🥇超星学习通｜知到智慧树——网课小助手》](https://scriptcat.org/zh-CN/script-show-page/711)（原版 v0.2.7）改良而来，在此向原作者致敬。

格式参考 [Keep a Changelog](https://keepachangelog.com/)，版本号遵循语义化版本。

---

## [0.1] - 2026-09-28

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
- 元信息：`@name` → 学习通AI答题小助手--Region改良款；`@author` → Region；`@namespace` → `https://github.com/Region`；`@version` → `0.1`。
- `@description` 重写，强调 AI 接口兼容性与改良来源。

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
