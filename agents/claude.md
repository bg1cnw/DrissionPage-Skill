# Drission-Page 技能集

DrissionPage 专项技能集合，由总路由器 + 7 个子技能组成，覆盖 DrissionPage 库的完整使用场景。

## 技能架构

本技能集采用**路由 + 子技能**架构：

- **`Drission-Page`**（总路由）：理解用户意图，将复合任务拆解为子步骤，按顺序调用对应子技能
- **7 个子技能**：各自覆盖一个领域，精准触发，上下文高效

## 子技能一览

| 技能 | 触发场景 |
|------|----------|
| `Drission-Page` | 复合任务规划、子技能调度 |
| `Drission-Page:quickstart` | 安装、三种页面对象选择、第一个示例 |
| `Drission-Page:find-elements` | 元素定位语法、NoneElement、遍历 |
| `Drission-Page:browser-ops` | 标签页、导航、截图、iframe、弹窗、Cookie、JS 执行 |
| `Drission-Page:element-ops` | 点击、输入、下拉框、文件上传、拖拽、键盘 |
| `Drission-Page:network` | 网络监听、HTTP 请求、文件下载、会话 Cookie |
| `Drission-Page:advanced` | 等待机制、反爬、Shadow DOM、Options 配置、并发 |
| `Drission-Page:update` | 版本检查、升级、破坏性变更 |

## 使用指引

处理 DrissionPage 相关任务时：

1. 单一领域问题 → 对应子技能直接处理
2. 复合任务 → `Drission-Page` 总路由先拆解，再调度子技能逐步执行
3. 需要查 API 细节 → 读 `references/docs-map.md` 定位文档，再读对应 `references/docs/` 中的详细文档
4. 文档优先于记忆：始终先查 `references/docs/`，不依赖模型训练数据中的 API 记忆

## 参考文档

所有子技能共享 `references/docs/` 中的详细中文文档。文档映射见 `references/docs-map.md`。

需要浏览器诊断时优先配合 Chrome DevTools MCP，协作规范见 `references/chrome-devtools-mcp.md`。
