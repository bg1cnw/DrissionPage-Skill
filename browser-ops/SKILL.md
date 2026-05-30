---
name: Drission-Page:browser-ops
description: |
  DrissionPage 浏览器操作大全。当用户询问如何管理标签页、页面导航（前进/后退/刷新）、
  截图、录屏、窗口大小、同时控制多个标签页、等待新标签出现、
  操作 iframe、处理弹窗/对话框、设置代理/Cookie、滚动页面、执行 JS 时触发。
  也适用于"怎么同时控制多个标签页"、"打开新窗口后怎么操作"、"如何截取整页截图"、
  "怎么阻止加载图片加速爬取"等问题。使用此 skill 掌握 ChromiumPage 完整的浏览器控制能力。
author: claude
version: 4.1.1.4
---

# DrissionPage 浏览器操作大全

> **参考文档**：`references/docs/控制浏览器/🛰️ 访问网页.md`、`references/docs/控制浏览器/🛰️ 标签页管理.md`、`references/docs/控制浏览器/🛰️ 截图和录像.md`、`references/docs/控制浏览器/🛰️ iframe 操作.md`、`references/docs/控制浏览器/🛰️ 页面交互.md`、`references/docs/控制浏览器/🥦 设置 cookies.md`、`references/docs/控制浏览器/🛰️ 浏览器对象.md`、`references/docs/控制浏览器/🛰️ Page 对象.md`
> **文档映射**：`references/docs-map.md`

---

## 页面导航

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()

# 访问网址（连接失败自动重试）
page.get('https://www.example.com')

# 带超时控制
page.get('https://example.com', timeout=15)

# 前进 / 后退
page.back()         # 后退1步
page.back(2)        # 后退2步
page.forward()      # 前进1步
page.forward(2)

# 刷新页面
page.refresh()
page.refresh(ignore_cache=True)   # 强制刷新（忽略缓存）

# 停止加载
page.stop_loading()
```

---

## 获取页面基本信息

```python
print(page.url)              # 当前 URL
print(page.title)            # 页面标题
print(page.html)             # 完整 HTML 源码
print(page.cookies())        # Cookie 列表
print(page.cookies().as_dict())  # Cookie 字典格式
```

---

## 多标签页管理

DrissionPage 的核心优势：**多个标签页可同时操作，无需来回切换**。

```python
# 新建标签页并直接获取对象
tab = page.new_tab('https://www.google.com')

# 两个标签页同时操作，互不干扰
page.ele('#search').input('DrissionPage')
tab.ele('#q').input('Python')

# 新建空白标签页
tab2 = page.new_tab()
tab2.get('https://bing.com')

# 查看所有标签页
print(page.tabs_count)   # 标签数量
print(page.tabs)         # 所有标签页 ID 列表

# 通过 ID 获取指定标签页对象
specific_tab = page.get_tab(page.tabs[1])

# 获取最新打开的标签页
latest = page.get_tab(page.latest_tab)

# 激活（切换焦点到）某标签页
tab.set.activate()

# 关闭标签页
tab.close()
```

---

## 等待新标签页出现

```python
# 点击会打开新标签的链接后，等待新标签
page.ele('a[target=_blank]').click()
new_tab_id = page.wait.new_tab(timeout=5)   # 返回新标签 ID
tab = page.get_tab(new_tab_id)
print(tab.url)
tab.wait.doc_loaded()  # 等待新标签加载完毕
```

---

## iframe 操作

无需切入切出，像操作普通元素一样使用：

```python
# 获取 iframe 对象（推荐用 get_frame，IDE 有更好的类型提示）
iframe = page.get_frame('#myFrame')           # 按 id
iframe = page.get_frame('myFrameName')        # 按 name
iframe = page.get_frame(1)                    # 按索引（从1开始）

# 在 iframe 内部查找元素，无需切换上下文
ele = iframe.ele('#inner-btn')
ele.click()

# iframe 也支持独立导航
iframe.get('https://example.com')

# 在 page 上直接查找 iframe 内的元素（自动穿透）
ele = page.ele('#element-inside-iframe')

# 获取页面所有 iframe 对象
all_frames = page.get_frames()
print(len(all_frames))
```

---

## 截图

```python
# 截取当前视口，保存到文件
page.get_screenshot(path='screenshot.png')

# 截取完整页面（包含视口外的内容）
page.get_screenshot(path='full.png', full_page=True)

# 截图不保存文件，返回 bytes
img_bytes = page.get_screenshot(as_bytes='png')
img_bytes_jpg = page.get_screenshot(as_bytes='jpeg')

# 截取单个元素区域
ele = page.ele('#chart-container')
ele.get_screenshot(path='chart.png')

# 先滚动到元素再截图
ele.scroll.to_see()
ele.get_screenshot(path='element.png')
```

---

## 录屏

```python
# 开始录制浏览器操作
page.screencast.start(path='recording.mp4')

# 执行操作...
page.get('https://example.com')
page.ele('#btn').click()

# 停止录制
page.screencast.stop()
```

---

## 窗口控制

```python
page.set.window.max()                   # 最大化
page.set.window.min()                   # 最小化
page.set.window.full()                  # 全屏
page.set.window.normal()                # 还原普通状态
page.set.window.size(1280, 800)         # 设置尺寸
page.set.window.location(100, 100)      # 设置位置
```

---

## Cookie 操作

```python
# 设置单个 Cookie
page.set.cookies({'name': 'token', 'value': 'abc123', 'domain': 'example.com'})

# 批量设置
page.set.cookies([
    {'name': 'token', 'value': 'abc'},
    {'name': 'user',  'value': 'admin'},
])

# 读取所有 Cookie
for c in page.cookies():
    print(c.name, c.value, c.domain)

# 转为字典
cookie_dict = page.cookies().as_dict()

# 清除所有 Cookie
page.set.cookies.clear()
```

---

## 执行 JavaScript

```python
# 运行 JS 并获取返回值
title = page.run_js('return document.title')
print(title)

# 传递元素给 JS
ele = page.ele('#myDiv')
page.run_js('arguments[0].style.border = "2px solid red"', ele)

# 在元素上直接运行 JS
ele.run_js('this.scrollIntoView()')
ele.run_js('this.style.display = "none"')
```

---

## 页面滚动

```python
# 页面级滚动
page.scroll.to_bottom()       # 滚动到最底部
page.scroll.to_top()          # 回到顶部
page.scroll.down(500)         # 向下滚动 500px
page.scroll.up(300)
page.scroll.right(200)
page.scroll.left(200)
page.scroll.to_see(ele)       # 滚动使某元素进入视口

# 元素内部滚动（如可滚动的 div）
container = page.ele('#scroll-container')
container.scroll.to_bottom()
container.scroll.down(200)
```

---

## 处理 Alert / Confirm / Prompt 弹窗

```python
# 触发弹窗的操作
page.ele('#alert-btn').click()

# 接受（点确定）
page.handle_alert(accept=True)

# 取消（点取消）
page.handle_alert(accept=False)

# 在 prompt 中输入文字后确认
page.handle_alert(accept=True, send='输入的内容')

# 等待弹窗关闭
page.wait.alert_closed()
```

---

## 屏蔽资源加载（提速）

```python
from DrissionPage import ChromiumOptions, ChromiumPage

co = ChromiumOptions()
co.no_imgs()    # 不加载图片

page = ChromiumPage(co)

# 运行时动态屏蔽指定 URL（支持通配符）
page.set.blocked_urls(['*.png', '*.jpg', '*.gif', 'ads.example.com/*'])
```

---

## 相关 skills

- `Drission-Page:quickstart` — 快速入门与页面对象创建
- `Drission-Page:find-elements` — 元素定位语法
- `Drission-Page:element-ops` — 元素交互操作
- `Drission-Page:network` — 网络监听与下载管理
- `Drission-Page:advanced` — 高级配置（无头模式、反爬、多实例）
