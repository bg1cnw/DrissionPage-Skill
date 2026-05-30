---
name: Drission-Page:network
description: |
  DrissionPage 网络功能大全。当用户询问如何监听网络请求、拦截 Ajax/XHR/Fetch 数据包、
  直接获取 API 接口 JSON 数据、使用 SessionPage 发送 HTTP 请求（GET/POST）、
  管理文件下载任务、处理 Cookie/Session/请求头、设置代理时触发。
  也适用于"怎么抓取 XHR 数据"、"Ajax 接口返回的 JSON 怎么获取"、
  "如何模拟登录后保持会话"、"下载文件进度怎么监控"、"怎么批量下载"等问题。
  使用此 skill 掌握 DrissionPage 的全部网络层能力。
author: claude
version: 4.1.1.4
---

# DrissionPage 网络功能大全

> **参考文档**：`references/docs/控制浏览器/🛰️ 监听网络数据.md` — 网络数据包监听
> **补充文档**：`references/docs/SessionPage/🛩️ 概述.md`、`references/docs/SessionPage/🛩️ 访问网页.md`、`references/docs/入门指南/🗺️ 收发数据包.md`、`references/docs/下载文件/⤵️ 概述.md`、`references/docs/下载文件/⤵️ download方法.md`、`references/docs/下载文件/⤵️ 浏览器下载.md`
> **文档映射**：`references/docs-map.md`

---

## SessionPage — 纯 HTTP 请求（无浏览器）

SessionPage 适合不需要 JS 渲染的场景，速度快、内存占用低。

### 基础 GET / POST

```python
from DrissionPage import SessionPage

page = SessionPage()

# GET 请求
page.get('https://httpbin.org/get')
print(page.json)           # 自动解析 JSON 响应
print(page.raw_data)       # 原始响应体（bytes 或 str）
print(page.html)           # HTML 内容（可用 ele() 解析）
print(page.response.status_code)   # HTTP 状态码

# 带查询参数的 GET
page.get('https://api.example.com/users', params={'page': 1, 'size': 20})

# POST 表单数据
page.post('https://httpbin.org/post', data={'username': 'admin', 'password': '123'})

# POST JSON 数据
page.post('https://api.example.com/login',
          json={'username': 'admin', 'password': '123'})

# 自定义请求头（本次有效）
page.get('https://api.example.com/data',
         headers={'Authorization': 'Bearer token_here'})
```

### 会话与 Cookie（模拟登录）

```python
page = SessionPage()

# 登录，SessionPage 自动维持 Cookie
page.post('https://example.com/login',
          data={'username': 'admin', 'password': '123456'})

# 后续请求自动携带 Cookie
page.get('https://example.com/user/profile')
print(page.ele('#username').text)

# 手动设置 Cookie
page.set.cookies({'name': 'token', 'value': 'my_token_value'})
page.set.cookies([
    {'name': 'a', 'value': '1'},
    {'name': 'b', 'value': '2'},
])

# 全局设置请求头（每次请求都带上）
page.set.headers({
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',
    'Referer': 'https://example.com',
})
```

### SessionOptions 详细配置

```python
from DrissionPage import SessionPage, SessionOptions

so = SessionOptions()
so.set_timeout(30)                             # 请求超时（秒）
so.set_proxies('http://127.0.0.1:7890')        # HTTP 代理
so.set_proxies('socks5://127.0.0.1:1080')      # SOCKS5 代理
so.set_headers({'User-Agent': 'MyBot/1.0'})
so.set_retry(3, interval=2)                    # 重试次数和间隔

page = SessionPage(session_or_options=so)
```

---

## 网络监听（拦截浏览器数据包）

直接获取浏览器收发的 XHR/Fetch 接口数据，比解析 DOM 更快更准。

### 基础示例：等待并捕获

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()

# 启动监听（指定 URL 关键词）
page.listen.start('api/v1/list')

# 触发网络请求（访问页面、点击按钮等）
page.get('https://example.com/products')

# 等待捕获第一个匹配的数据包
packet = page.listen.wait()

print(packet.url)
print(packet.method)                  # 'GET' 或 'POST'
print(packet.response.status)         # HTTP 状态码
print(packet.response.body)           # 响应体（自动尝试解析 JSON）
print(packet.response.headers)        # 响应头字典
print(packet.request.headers)         # 请求头
print(packet.request.body)            # 请求体（POST 数据）
```

### 监听分页 Ajax 数据

```python
page = ChromiumPage()
page.get('https://gitee.com/explore/all')

page.listen.start('gitee.com/explore')

for page_num in range(1, 6):
    page('@rel=next').click()          # 点击下一页
    packet = page.listen.wait()        # 等待接口响应
    data = packet.response.body        # 直接拿到 JSON 数据
    print(f"第{page_num}页:", packet.url)
```

### 实时流式监听

```python
page.listen.start('api/data')
page.get('https://example.com')

# 用 steps() 迭代每个捕获到的数据包
for packet in page.listen.steps(count=10, timeout=5):
    print(packet.url)
    process(packet.response.body)
```

### 精细化监听控制

```python
# 监听多个 URL 关键词
page.listen.start(['api/users', 'api/orders'])

# 正则表达式匹配
page.listen.start(r'api/v\d+/list', is_regex=True)

# 只监听 POST 请求
page.listen.start('api/', method='POST')

# 监听所有请求（True 表示全捕获）
page.listen.start(True)

# 等待多个数据包
packets = page.listen.wait(count=3, timeout=10)
for p in packets:
    print(p.url, p.response.status)

# 带超时等待（超时返回 False）
packet = page.listen.wait(timeout=8)
if packet:
    print(packet.response.body)
else:
    print("超时，未捕获到数据包")

# 停止监听并清理
page.listen.stop()
```

---

## 文件下载管理

DrissionPage 4.1.1.4 内置 **DrissionGet** 多线程下载引擎，同时提供 `click.to_download()` 一键下载。

### 一键下载（推荐）

```python
page = ChromiumPage()
page.get('https://example.com/files')

# 点击即下载，自动处理等待和保存
mission = page.ele('#download-btn').click.to_download(
    save_path=r'C:\Downloads',
    rename='myfile.pdf'
)
print(f"下载完成：{mission.path}")
```

### 传统方式

```python
page = ChromiumPage()

# 设置下载目录
page.set.download_path(r'C:\Downloads')

# 点击下载链接
page.ele('#download-btn').click()

# 等待所有下载完成
page.wait.downloads_done(timeout=120)
print("全部下载完成")
```

### 监控单个下载任务

```python
page.set.download_path(r'C:\Downloads')

page.ele('#download-btn').click()

# 等待下载任务开始，获取任务对象
mission = page.wait.download_begin(timeout=10)
if mission:
    print(f"下载 URL: {mission.url}")
    print(f"保存路径: {mission.path}")
    print(f"文件名: {mission.name}")

    # 等待该任务完成
    mission.wait()
    print(f"下载完成！文件保存在: {mission.path}")
```

### 批量下载

```python
page.set.download_path(r'C:\Downloads')

# 点击多个下载链接
download_links = page.eles('css:.download-link')
for link in download_links:
    link.click()
    page.wait(0.5)  # 稍等，避免触发太快

# 等待全部完成
page.wait.downloads_done(timeout=300)
print("批量下载完成")
```

---

## WebPage 模式切换（浏览器 + 请求合一）

```python
from DrissionPage import WebPage

page = WebPage()

# 浏览器模式处理登录
page.change_mode('d')
page.get('https://example.com/login')
page.ele('#user').input('admin')
page.ele('#pass').input('123456')
page.ele('#login-btn').click()
page.wait.eles_loaded('#dashboard')  # 等待登录成功

# 切换到请求模式（自动继承浏览器的 Cookie）
page.change_mode('s')

# 直接请求 API，无需重新登录
page.get('https://example.com/api/user/list', params={'page': 1})
users = page.json

page.post('https://example.com/api/data/export',
          json={'format': 'json', 'fields': ['id', 'name']})
data = page.json
```

---

## 设置代理

```python
from DrissionPage import ChromiumOptions, ChromiumPage

co = ChromiumOptions()
co.set_proxy('http://127.0.0.1:7890')      # HTTP 代理
co.set_proxy('socks5://127.0.0.1:1080')    # SOCKS5 代理

page = ChromiumPage(co)
page.get('https://api64.ipify.org?format=json')
print(page.json)  # 显示当前出口 IP
```

---

## 实战：抓取 Ajax 分页数据

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()
page.listen.start('api/products', method='GET')
page.get('https://shop.example.com/list')

all_items = []
page_num = 1

while True:
    packet = page.listen.wait(timeout=8)
    if not packet:
        print("等待超时，停止")
        break

    resp = packet.response.body
    items = resp.get('data', {}).get('list', [])
    all_items.extend(items)
    print(f"第{page_num}页：{len(items)} 条")

    # 判断是否还有下一页
    if not resp.get('data', {}).get('hasMore'):
        break

    page.ele('.page-next').click()
    page_num += 1

print(f"共抓取 {len(all_items)} 条数据")
```

---

## 相关 skills

- `Drission-Page:quickstart` — 快速入门（SessionPage vs ChromiumPage 选择）
- `Drission-Page:browser-ops` — 浏览器 Cookie 管理、页面操作
- `Drission-Page:advanced` — 反爬处理、超时重试、多实例
