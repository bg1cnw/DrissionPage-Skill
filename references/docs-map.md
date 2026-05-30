# 文档映射与任务手册

本文件只引用已经复制到当前 skill 内的资料，保证技能集可单独安装使用。

## 快速查找表

> 根据你要做的事情，直接找到对应文档和子技能。

| 我想要… | 子技能 | 首选文档 | 补充文档 |
|---------|--------|----------|----------|
| 安装、导入、第一个示例 | `:quickstart` | `references/docs/入门指南/🌏 安装.md` | `references/docs/入门指南/🌏 导入.md`、`references/docs/入门指南/☀️ 基本概念.md` |
| 了解对象关系 / 选型 | `:quickstart` | `references/docs/入门指南/☀️ 基本概念.md` | `references/docs/特性与示例/💖 特性.md` |
| 定位元素 / 查找失败 | `:find-elements` | `references/docs/控制浏览器/🔦 定位语法.md` | `references/docs/控制浏览器/🔦 语法速查表.md`、`references/docs/控制浏览器/🔦 页面或元素内查找.md`、`references/docs/控制浏览器/🔦 相对定位.md`、`references/docs/控制浏览器/🔦 简化写法.md` |
| 页面导航 / 访问网页 | `:browser-ops` | `references/docs/控制浏览器/🛰️ 访问网页.md` | `references/docs/控制浏览器/🛰️ Page 对象.md` |
| 标签页管理 | `:browser-ops` | `references/docs/控制浏览器/🛰️ 标签页管理.md` | `references/docs/控制浏览器/🛰️ 浏览器对象.md` |
| 截图 / 录像 | `:browser-ops` | `references/docs/控制浏览器/🛰️ 截图和录像.md` | — |
| iframe 操作 | `:browser-ops` | `references/docs/控制浏览器/🛰️ iframe 操作.md` | — |
| Cookie 管理 | `:browser-ops` | `references/docs/控制浏览器/🥦 设置 cookies.md` | — |
| 弹窗处理 | `:browser-ops` | `references/docs/控制浏览器/🛰️ 页面交互.md` | — |
| 配置浏览器启动 | `:advanced` | `references/docs/控制浏览器/🛰️ 浏览器启动设置.md` | `references/docs/控制浏览器/🛰️ 连接浏览器.md`、`references/docs/控制浏览器/🥦 创建全新的浏览器.md` |
| WebPage / 模式切换 | `:advanced` | `references/docs/入门指南/🗺️ 模式切换.md` | `references/docs/特性与示例/⭐ 模式切换.md` |
| 元素点击 / 输入 / 交互 | `:element-ops` | `references/docs/控制浏览器/🛰️ 元素交互.md` | `references/docs/控制浏览器/🛰️ 动作链.md` |
| 下拉框 / select | `:element-ops` | `references/docs/控制浏览器/🛰️ 元素交互.md` | — |
| 文件上传 | `:element-ops` | `references/docs/控制浏览器/🛰️ 上传文件.md` | — |
| 获取元素属性 / 文本 | `:element-ops` | `references/docs/控制浏览器/🛰️ 获取元素信息.md` | `references/docs/特性与示例/⭐ 获取元素属性.md` |
| 获取页面信息 | `:element-ops` | `references/docs/控制浏览器/🛰️ 获取网页信息.md` | — |
| 等待元素 / 页面加载 | `:advanced` | `references/docs/控制浏览器/🛰️ 等待.md` | — |
| 网络数据包监听 | `:network` | `references/docs/控制浏览器/🛰️ 监听网络数据.md` | — |
| SessionPage / HTTP 请求 | `:network` | `references/docs/SessionPage/🛩️ 概述.md` | `references/docs/SessionPage/🛩️ 访问网页.md`、`references/docs/入门指南/🗺️ 收发数据包.md` |
| 下载文件 | `:network` | `references/docs/下载文件/⤵️ 概述.md` | `references/docs/下载文件/⤵️ download方法.md`、`references/docs/下载文件/⤵️ 浏览器下载.md` |
| 无头模式 | `:advanced` | `references/docs/控制浏览器/🥦 无头模式.md` | — |
| 浏览器多开 / 并发 | `:advanced` | `references/docs/控制浏览器/🥦 浏览器多开.md` | `references/docs/实战示例/🌠 多线程多标签页采集.md` |
| 反爬 / 指纹浏览器 | `:advanced` | `references/docs/控制浏览器/🌐 连接 TgeBrowser 指纹浏览器.md` | `references/docs/控制浏览器/🛰️ 浏览器启动设置.md` |
| 全局设置 / 命令行 | `:advanced` | `references/docs/进阶使用/⚙️ 全局设置.md` | `references/docs/进阶使用/⚙️ 命令行的使用.md`、`references/docs/进阶使用/⚙️ 配置文件的使用.md` |
| 看代码示例写法 | `:quickstart` | `references/docs/实战示例/` | `references/docs/入门指南/🗺️ 自动登录.md` |
| DevTools MCP 协作 | `:browser-ops` | `references/chrome-devtools-mcp.md` | `references/docs/控制浏览器/🛰️ 页面交互.md`（`cdp()` 方法） |
| 检查版本 / 升级 | `:update` | 见该子技能内部说明 | — |

> 所有文档均为中文。

## 常见任务执行顺序

### 编写或修改脚本

1. 先确认属于 `ChromiumPage`、`SessionPage` 还是 `WebPage`。
2. 先看 `references/docs/实战示例/` 是否已有接近案例。
3. 再用 `references/docs/` 各栏目文档核对预期行为和参数说明。
4. 涉及真实网页时，优先用 `chrome-devtools-mcp` 或最小复现确认请求、等待点和失败阶段。
5. 只改当前任务所需的一层逻辑，不要无依据扩散到多个对象。
6. 做最小 smoke test。

### 排查 locator 或元素查找问题

1. 先看 `references/docs/控制浏览器/🔦 定位语法.md`。
2. 再看 `references/docs/控制浏览器/🔦 页面或元素内查找.md` 和 `references/docs/控制浏览器/🔦 语法速查表.md`。
3. 用最小复现确认到底是定位写法问题、页面结构变化，还是等待时机问题。

### 编写示例或补文档

1. 默认先从 `references/docs/实战示例/` 选最接近的案例改写。
2. 如果实战示例不够，再参考 `references/docs/入门指南/` 中的入门示例。
3. 示例仍不足时，从 `references/docs/特性与示例/` 获取写法参考。
4. 复用现有写法和对象命名。
5. 示例尽量最小化，聚焦一个能力点。

## 验证建议

| 改动类型 | 验证命令 |
|----------|----------|
| 普通项目脚本 | 最小脚本直接运行，或构造最小 `ChromiumPage()` / `WebPage()` 场景 |
| 纯导入 / 签名改动 | `python -c "from DrissionPage import ChromiumPage, SessionPage, WebPage"` |
| 浏览器相关改动 | 记录本地浏览器、端口、配置文件前提后做 smoke test |
