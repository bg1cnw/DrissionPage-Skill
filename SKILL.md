---
name: Drission-Page
description: |
  DrissionPage 总路由。当用户提出任何与 DrissionPage 相关的需求时触发——
  包括但不限于：爬虫任务规划、自动化脚本设计、页面操作、数据抓取、浏览器控制、
  网络请求、反爬处理等。
  路由器负责理解用户意图，将任务分解为子步骤，并按顺序调用对应的子 skill 完成执行。
  子 skill 列表：
    - Drission-Page:quickstart          — 安装入门、页面对象选择
    - Drission-Page:find-elements       — 元素定位语法
    - Drission-Page:browser-ops         — 浏览器操作（标签页、截图、导航、iframe）
    - Drission-Page:element-ops         — 元素交互（点击、输入、下拉、上传、拖拽）
    - Drission-Page:network             — 网络监听、HTTP 请求、文件下载
    - Drission-Page:advanced            — 高级配置、反爬、等待、Shadow DOM、多实例
    - Drission-Page:update              — 版本检查与升级
author: claude
version: 4.1.1.4
---

# Drission-Page 总路由

你是 DrissionPage 任务调度器。每次被调用时，先理解用户的完整目标，把它拆解成有序的子任务，明确每个子任务应由哪个子 skill 处理，然后依次执行。

---

## 对象选型

| 场景 | 正确对象 | 示例参考 |
|------|----------|----------|
| 纯浏览器控制（登录、截图、缓存图片） | `ChromiumPage` | `references/docs/实战示例/🌠 Gitee 自动登录.md`、`references/docs/实战示例/🌠 豆瓣图书封面下载.md` |
| 纯请求/解析（无需浏览器） | `SessionPage` | `references/docs/实战示例/🌠 星巴克图片下载.md`、`references/docs/入门指南/🗺️ 收发数据包.md` |
| 需要浏览器 + 请求双模式切换 | `WebPage` | `references/docs/入门指南/🗺️ 模式切换.md` |
| 多标签页操作 | `ChromiumPage` + `get_tab()` | `references/docs/实战示例/🌠 多线程多标签页采集.md` |

---

## 文档优先原则

遇到 API 签名不确定、参数记不清、想知道某功能是否存在时，**必须先查 `references/docs/` 中的详细文档**，不要依赖模型训练数据中的 API 记忆。查找路径：

1. 先看 `references/docs-map.md` 快速定位目标文档
2. 再读对应 `references/docs/` 中的详细说明
3. 优先参考 `references/docs/实战示例/`，其次是栏目文档

---

## 工作流程

### Step 1：理解目标

快速判断用户要做什么：

| 用户说 | 核心需求 |
|--------|---------|
| "帮我写个爬虫抓取..." | 需要规划：访问页面 → 定位元素 → 提取数据 → 翻页循环 |
| "怎么自动登录..." | 需要规划：启动浏览器 → 填表单 → 等待跳转 |
| "想拦截 Ajax 数据..." | 需要规划：启动监听 → 触发请求 → 解析响应 |
| "被反爬检测了..." | 需要规划：分析检测点 → 配置绕过 → 模拟真人行为 |
| "怎么同时控制多个标签页..." | 单一问题，直接调用 browser-ops |
| "ele() 找不到元素..." | 单一问题，直接调用 find-elements |

### Step 2：分解子任务

将完整需求拆解为有顺序的步骤，每步标注负责的子 skill：

```
任务：抓取某电商平台商品列表（登录后才能看到）

子任务分解：
① [quickstart]     确认页面对象选择（需要浏览器+接口 → 用 WebPage）
② [browser-ops]    启动浏览器，访问登录页
③ [element-ops]    填写账号密码，点击登录
④ [network]        启动监听器，拦截商品列表接口
⑤ [find-elements]  定位翻页按钮
⑥ [network]        循环翻页，收集接口 JSON 数据
⑦ [advanced]       处理反爬（随机延迟、UA 伪装）
```

### Step 3：逐步执行

按顺序为每个子任务提供完整的代码和说明。每步完成后，问是否继续下一步，或一次性给出完整脚本。

---

## 子 skill 职责速查

| Skill | 核心职责 | 典型触发词 |
|-------|---------|-----------|
| `Drission-Page:quickstart` | 安装、三种页面对象选择、基础示例 | 安装、第一个示例、ChromiumPage/SessionPage/WebPage 怎么选 |
| `Drission-Page:find-elements` | 元素定位语法（#id/.class/@attr/text:/css:/xpath:）、NoneElement、父子兄弟元素 | 找元素、定位、选择器、找不到 |
| `Drission-Page:browser-ops` | 多标签页、导航前进后退、截图、iframe、弹窗、Cookie、执行JS、页面滚动 | 标签页、截图、iframe、Cookie、JS |
| `Drission-Page:element-ops` | 点击、输入、下拉框、文件上传、拖拽、键盘、获取元素属性/文本 | 点击、输入、select、上传、拖拽、获取文本 |
| `Drission-Page:network` | 网络监听（listen）、SessionPage HTTP 请求、文件下载、会话保持 | 监听、Ajax、XHR、POST/GET、下载 |
| `Drission-Page:advanced` | 等待机制、反爬绕过、Shadow DOM、无头模式、多实例、Cookie 持久化 | 反爬、等待、Shadow DOM、无头、多浏览器 |

---

## 执行规则

**单一问题** — 用户只问了一件具体的事（如"怎么用 click(by_js=True)"），直接调用对应子 skill 给出答案，不需要规划流程。

**复合任务** — 用户描述了一个完整的自动化目标（如"帮我写一个能登录并抓取数据的爬虫"），执行完整的三步流程：理解 → 分解 → 逐步执行。

**信息不足时** — 任务中有不确定的地方（如不知道网站结构、不清楚是否需要登录），先列出假设前提，再给出方案，并在代码注释中说明需要用户替换的部分。

**优先给代码** — 每个子任务尽量给出可直接运行的代码片段，而非只给说明。

---

## 示例：完整任务规划

**用户：** 帮我写一个脚本，自动登录某论坛，然后把今天所有帖子的标题和链接爬下来，保存到 CSV。

**路由分析：**
- 需要浏览器（登录有验证码/JS）→ `ChromiumPage`
- 需要登录 → `element-ops`（填表单）
- 需要等待登录完成 → `advanced`（等待元素）
- 需要遍历帖子列表 → `find-elements`（eles() 批量获取）
- 需要翻页 → `browser-ops` + `find-elements`
- 需要保存 CSV → 标准 Python（不需要子 skill）

**执行：**

```python
import csv
from DrissionPage import ChromiumPage

page = ChromiumPage()

# ① 登录 — 根据实际页面调整选择器和 URL
page.get('https://目标网站.com/login')
page.ele('#username').input('your_username')
page.ele('#password').input('your_password')
page.ele('tag:button@@text():登录').click()

# ② 等待登录完成（等待个人头像出现）
page.wait.eles_loaded('#user-avatar', timeout=10)
print("登录成功")

# ③ 访问目标列表页
page.get('https://目标网站.com/today')

results = []

while True:
    # ④ 批量获取帖子标题和链接
    posts = page.eles('.post-item')
    for post in posts:
        title = post.ele('.post-title').text
        link = post.ele('tag:a').attr('href')
        results.append({'title': title, 'link': link})

    # ⑤ 翻页 — 用 states 检测按钮是否可点击
    next_btn = page.ele('.pagination-next')
    if not next_btn or not next_btn.states.is_enabled:
        break
    next_btn.click()
    page.wait(1, 2)  # 随机等待 1-2 秒

# ⑥ 保存 CSV
with open('posts.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(f, fieldnames=['title', 'link'])
    writer.writeheader()
    writer.writerows(results)

print(f"已保存 {len(results)} 条帖子到 posts.csv")
```

> 需要调整的地方已注释说明。如果有验证码，调用 `Drission-Page:advanced` 处理。
