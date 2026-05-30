# Drission-Page Skills — [EN](README.md)

DrissionPage 专项 skill 集合，为支持技能功能的 AI 编程助手（Claude Code、Codex、Cursor 等）提供 DrissionPage 库的精准知识支持。
当你在项目中使用 DrissionPage 时，AI 助手会自动调用相应 skill，给出基于正确 API 的回答，避免与 selenium 等其他库混淆。

## 目录结构

```
Drission-Page/
├── README.md                        本文件
├── SKILL.md                         总路由（入口）
├── quickstart/SKILL.md              快速入门
├── find-elements/SKILL.md           元素定位
├── browser-ops/SKILL.md             浏览器操作
├── element-ops/SKILL.md             元素交互
├── network/SKILL.md                 网络功能
├── advanced/SKILL.md                高级功能
├── update/SKILL.md                  版本检查与升级
└── evals/                           评估测试用例
```

## Skill 一览

| Skill 名称 | 行数 | 触发场景 |
|-----------|:----:|---------|
| `Drission-Page` | 148 | **总路由** — 复合任务规划、子 skill 调度 |
| `Drission-Page:quickstart` | 176 | 安装、三种页面对象选择（ChromiumPage / SessionPage / WebPage）、第一个示例 |
| `Drission-Page:find-elements` | 284 | 元素定位语法（`#id` / `.class` / `@attr=value` / `text:` / `css:` / `xpath:`）、NoneElement、父子兄弟元素遍历 |
| `Drission-Page:browser-ops` | 292 | 多标签页管理、页面导航、截图（含全页）、iframe 切换、弹窗处理、Cookie 管理、执行 JS、页面滚动 |
| `Drission-Page:element-ops` | 295 | 点击（含 JS 穿透）、输入（含模拟真人速度）、下拉框 select、文件上传、拖拽、键盘操作、获取属性/文本/HTML |
| `Drission-Page:network` | 323 | 网络数据包监听（`listen` API）、SessionPage HTTP 请求、文件下载管理、会话 Cookie 保持 |
| `Drission-Page:advanced` | 395 | 智能等待机制、反爬检测绕过、Shadow DOM 处理、ChromiumOptions 配置、无头模式、多实例并发、Cookie 持久化 |
| `Drission-Page:update` | 203 | 版本检查、PyPI 最新版对比、升级操作、破坏性变更说明、版本回退、环境冲突排查 |
| **合计** | **2116** | |

> 经 4.1.1.2 版本 API 验证，已确认所有代码示例使用正确的 API。

## 安装

### 方式一：直接克隆（推荐）

```bash
# 克隆到 AI 助手的 skills 目录（以 Claude Code 为例）
git clone https://github.com/bg1cnw/DrissionPage-Skill.git ~/.claude/skills/Drission-Page
```

安装后重启 AI 助手即可自动生效。当你在对话中提到 DrissionPage 相关需求时，AI 助手会自动调用对应 skill。

### 方式二：手动复制

下载本仓库所有文件，放入对应 AI 助手的 skills 目录（如 Claude Code 为 `~/.claude/skills/Drission-Page/`）。

### 验证安装

在 AI 助手中输入 `/Drission-Page`，如果看到路由提示则安装成功。

### 依赖

Skills 本身无需额外依赖，但你使用的 DrissionPage 脚本需要安装 DrissionPage 库：

```bash
pip install DrissionPage>=4.1.1.4
```

## 使用方式

### 自动触发（推荐）

直接用中文描述需求，AI 助手会自动识别并调用对应 skill：

```
"帮我写一个登录后抓取数据的爬虫"    → Drission-Page（总路由规划）
"ele() 找不到元素怎么办"           → Drission-Page:find-elements
"怎么同时操作多个标签页"            → Drission-Page:browser-ops
"DrissionPage 有新版本吗"          → Drission-Page:update
```

### 手动调用

```
/Drission-Page                      触发总路由，进行任务规划
/Drission-Page:quickstart           直接获取入门指导
/Drission-Page:find-elements        元素定位语法手册
/Drission-Page:browser-ops          浏览器操作手册
/Drission-Page:element-ops          元素交互手册
/Drission-Page:network              网络功能手册
/Drission-Page:advanced             高级功能手册
/Drission-Page:update               检查并升级库版本
```

## 总路由工作方式

**单一问题** → 识别类型，直接调用对应子 skill，给出答案。

**复合任务**（如"帮我写一个完整的爬虫"）→ 执行三步流程：

```
1. 理解目标    分析用户完整需求
      ↓
2. 拆解子任务  标注每步由哪个子 skill 负责
      ↓
3. 逐步执行    按顺序给出可运行的代码
```

示例拆解：
```
任务：登录 + 抓取列表 + 保存 CSV

① [quickstart]    → 选 ChromiumPage（需要浏览器登录）
② [browser-ops]   → 访问登录页
③ [element-ops]   → 填写表单、点击登录
④ [advanced]      → 等待登录完成（wait.ele_loaded）
⑤ [find-elements] → 批量定位列表元素（eles()）
⑥ [network]       → 若有 Ajax 接口则监听（listen）
⑦ [browser-ops]   → 翻页循环
⑧ Python 标准库   → 写入 CSV
```

## 知识覆盖的 DrissionPage 版本

- 当前已安装版本：**4.1.1.4**
- PyPI 最新版本：**4.1.1.4**（截至 2026-05）
- Skills 内容基于 **4.1.1.4 API**，已通过运行时验证（与 3.x 有破坏性变更，见 `Drission-Page:update` skill）
- 4.1.1.4 新增依赖 **DrissionGet**（多线程下载）和 **DrissionRecord**（数据记录）

## 评估结果（Iteration 1）

| 测试场景 | 有 Skill | 无 Skill | 提升 |
|---------|:-------:|:-------:|:----:|
| 快速入门 + API 正确性 | 100% | 40% | +60pp |
| 元素定位语法完整性 | 100% | 40% | +60pp |
| Ajax 监听（listen API）| 100% | 25% | +75pp |
| **平均** | **100%** | **35%** | **+65pp** |

无 Skill 时，AI 助手倾向于混用 selenium 风格 API（`find_element(By.ID, ...)`、`send_keys()` 等），这些方法在 DrissionPage 中不存在。

## 相关链接

- 官方文档：https://DrissionPage.cn
- GitHub：https://github.com/g1879/DrissionPage
- PyPI：https://pypi.org/project/DrissionPage/
- 更新日志：https://github.com/g1879/DrissionPage/releases
