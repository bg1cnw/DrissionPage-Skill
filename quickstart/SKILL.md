---
name: Drission-Page:quickstart
description: |
  DrissionPage 快速入门指导。当用户询问 DrissionPage 怎么安装、如何开始使用、
  三种页面对象如何选择（ChromiumPage / SessionPage / WebPage）、第一个示例怎么写时触发。
  也适用于"DrissionPage 和 selenium/requests 有什么区别"、"我想用 Python 控制浏览器"、
  "怎么用 Python 爬网页"等场景。如果用户刚接触 DrissionPage 或需要基础示例，务必使用此 skill。
author: claude
version: 4.1.1.4
---

# DrissionPage 快速入门

DrissionPage 是一个 Python 网页自动化工具，既能控制浏览器，也能收发数据包，还能把两者合一。
语法简洁优雅，无需 webdriver，无需为浏览器版本匹配驱动。

## 安装

```bash
pip install DrissionPage
```

**系统要求：**
- Python 3.6+
- Windows / Linux / macOS
- Chromium 内核浏览器（Chrome、Edge 等）

---

## 三种页面对象，按需选择

| 对象 | 用途 | 何时用 |
|------|------|--------|
| `ChromiumPage` | 纯浏览器控制 | 需要 JS 渲染、交互操作、登录等 |
| `SessionPage` | 纯 HTTP 请求 | 静态页面、API 接口、高性能爬取 |
| `WebPage` | 两者合一 | 需要随时在浏览器和请求间切换 |

---

## ChromiumPage 示例（浏览器控制）

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()

# 访问网页
page.get('https://www.baidu.com')

# 查找输入框并输入（#kw 是 id 选择器）
page.ele('#kw').input('DrissionPage')

# 点击搜索按钮（@value 是属性选择器）
page.ele('@value=百度一下').click()

# 获取所有结果标题
for item in page.eles('.result h3'):
    print(item.text)
```

---

## SessionPage 示例（HTTP 请求，无浏览器）

```python
from DrissionPage import SessionPage

page = SessionPage()

# 发送 GET 请求
page.get('https://httpbin.org/get')
print(page.json)            # JSON 格式响应
print(page.raw_data)        # 原始响应内容

# 发送 POST 请求
page.post('https://httpbin.org/post', data={'key': 'value'})
page.post('https://api.example.com/login', json={'user': 'admin'})

# 像浏览器模式一样查找元素
ele = page.ele('tag:h1')
print(ele.text)
```

---

## WebPage 示例（浏览器 + 请求合一）

```python
from DrissionPage import WebPage

page = WebPage()  # 默认 'd' 模式（浏览器驱动）

# 用浏览器处理登录
page.get('https://example.com/login')
page.ele('#username').input('admin')
page.ele('#password').input('123456')
page.ele('button[type=submit]').click()

# 切换到 's' 模式（请求模式），自动继承登录 Cookie
page.change_mode('s')
page.get('https://example.com/api/data')
data = page.json
```

---

## 简洁调用语法

DrissionPage 支持把页面对象当函数调用来查找元素：

```python
page = ChromiumPage()
page.get('https://www.baidu.com')

# 以下三种写法等价
ele = page.ele('#kw')
ele = page('#kw')           # 简写
ele = page('#kw', index=1)  # 明确指定第1个匹配
```

---

## 配置浏览器启动选项

```python
from DrissionPage import ChromiumPage, ChromiumOptions

co = ChromiumOptions()
co.headless()    # 无头模式（不显示窗口）
co.no_imgs()     # 不加载图片（节省流量、加速）
co.mute()        # 静音
co.incognito()   # 隐身模式

page = ChromiumPage(co)
page.get('https://example.com')
```

---

## 常见问题

**Q: 启动失败，找不到浏览器？**
```python
co = ChromiumOptions()
# 手动指定 Chrome 路径
co.set_browser_path(r'C:\Program Files\Google\Chrome\Application\chrome.exe')
page = ChromiumPage(co)
```

**Q: 如何复用已打开的浏览器（调试用）？**
```python
# 先用命令行启动 Chrome 并开启调试端口：
# chrome.exe --remote-debugging-port=9222

co = ChromiumOptions().set_local_port(9222)
page = ChromiumPage(co)
# 直接接管已有浏览器，保留登录状态
```

**Q: 提示需要关闭已有浏览器？**

DrissionPage 默认接管同端口的 Chrome。如果冲突，换一个端口：
```python
co = ChromiumOptions().set_local_port(9333)
page = ChromiumPage(co)
```

---

## 相关 skills

- `Drission-Page:find-elements` — 元素定位语法大全
- `Drission-Page:browser-ops` — 浏览器操作（标签页、截图、导航）
- `Drission-Page:element-ops` — 元素交互（点击、输入、拖拽）
- `Drission-Page:network` — 网络监听与 HTTP 请求
- `Drission-Page:advanced` — 高级功能（等待、iframe、Shadow DOM）
