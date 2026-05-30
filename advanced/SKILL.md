---
name: Drission-Page:advanced
description: |
  DrissionPage 高级功能与进阶技巧。当用户询问智能等待机制详解、Shadow DOM 处理、
  ChromiumOptions 全面配置（无头/禁图/代理/扩展/证书错误/User-Agent）、
  如何进行性能优化、如何绕过反爬检测（webdriver 检测）、
  如何多线程/多实例同时运行、如何处理动态加载/懒加载内容、
  如何保存/复用 Cookie、ini 配置文件管理时触发。
  也适用于"程序运行一会就卡死"、"被网站识别为机器人怎么办"、
  "怎么让浏览器跑得更快"、"怎么保持登录状态跨会话"等问题。
author: claude
version: 4.1.1.4
---

# DrissionPage 高级功能与进阶技巧

> **参考文档**：`references/docs/控制浏览器/🛰️ 等待.md` — 智能等待机制
> **补充文档**：`references/docs/控制浏览器/🛰️ 浏览器启动设置.md`、`references/docs/控制浏览器/🛰️ 连接浏览器.md`、`references/docs/控制浏览器/🥦 创建全新的浏览器.md`、`references/docs/控制浏览器/🥦 无头模式.md`、`references/docs/控制浏览器/🥦 浏览器多开.md`、`references/docs/控制浏览器/🌐 连接 TgeBrowser 指纹浏览器.md`、`references/docs/入门指南/🗺️ 模式切换.md`、`references/docs/进阶使用/⚙️ 全局设置.md`、`references/docs/进阶使用/⚙️ 配置文件的使用.md`、`references/docs/进阶使用/⚙️ 命令行的使用.md`、`references/docs/进阶使用/⚙️ 实用工具.md`
> **文档映射**：`references/docs-map.md`

---

## 智能等待机制（告别 sleep）

DrissionPage 内置完整的等待体系，精确等待而非盲目 sleep：

### 等待元素状态变化

```python
page = ChromiumPage()

# 等待元素出现在 DOM 中（返回元素对象，失败返回 False）
ele = page.wait.eles_loaded('#result', timeout=10)
if ele:
    print(ele.text)

# 等待元素变为可见（从隐藏到显示）
page.wait.ele_displayed('#confirm-dialog', timeout=5)

# 等待元素消失（如加载动画遮罩）
page.wait.ele_hidden('#loading-overlay', timeout=15)

# 等待元素从 DOM 中彻底删除
page.wait.ele_deleted('#temp-toast', timeout=5)
```

### 等待页面/标签加载

```python
# 等待页面开始加载
ele.click()
page.wait.load_start(timeout=5)

# 等待文档完成加载
page.wait.doc_loaded(timeout=30)

# 等待新标签页出现
page.ele('a[target=_blank]').click()
new_id = page.wait.new_tab(timeout=5)
tab = page.get_tab(new_id)
```

### 随机延迟（模拟真人节奏）

```python
import random

# 随机等待 1~3 秒（第二个参数是上限）
page.wait(1, 3)

# 固定等待
page.wait(2)

# 操作间插入随机延迟
page.ele('#btn1').click()
page.wait(0.5, 1.2)   # 操作之间随机等待
page.ele('#btn2').click()
page.wait(1, 2)
```

---

## ChromiumOptions 全面配置

```python
from DrissionPage import ChromiumOptions, ChromiumPage

co = ChromiumOptions()

# ---- 常用模式 ----
co.headless()                    # 无头模式（后台运行，不显示窗口）
co.no_imgs()                     # 不加载图片（节省流量，加速）
co.mute()                        # 静音
co.incognito()                   # 隐身模式

# ---- 安全/证书 ----
co.ignore_certificate_errors()   # 忽略 SSL/HTTPS 证书错误

# ---- 网络 ----
co.set_proxy('http://127.0.0.1:7890')    # HTTP 代理
co.set_proxy('socks5://127.0.0.1:1080')  # SOCKS5 代理

# ---- 身份特征 ----
co.set_user_agent(
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
    'AppleWebKit/537.36 (KHTML, like Gecko) '
    'Chrome/120.0.0.0 Safari/537.36'
)

# ---- 反检测 ----
co.set_argument('--disable-blink-features=AutomationControlled')

# ---- 性能优化 ----
co.set_argument('--disable-extensions')
co.set_argument('--disable-gpu')           # 无头模式推荐加
co.set_argument('--no-sandbox')            # Docker / Linux root 用户必加
co.set_argument('--disable-dev-shm-usage') # Docker 环境防止内存不足

# ---- 窗口 ----
co.set_argument('--window-size=1920,1080')
co.set_argument('--start-maximized')

# ---- 路径 ----
co.set_paths(download_path=r'C:\Downloads')
co.set_browser_path(r'C:\Program Files\Google\Chrome\Application\chrome.exe')

# ---- 扩展程序 ----
co.add_extension(r'C:\path\to\my\extension')

page = ChromiumPage(co)
```

---

## 复用已有浏览器（调试神器）

无需每次重新启动浏览器，直接接管已有实例，保留登录状态：

```bash
# 第一步：命令行启动 Chrome 并开启远程调试
# Windows:
chrome.exe --remote-debugging-port=9222 --user-data-dir=C:\chrome_debug

# macOS:
# /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

```python
# 第二步：Python 连接已有浏览器
from DrissionPage import ChromiumOptions, ChromiumPage

co = ChromiumOptions().set_local_port(9222)
page = ChromiumPage(co)

# 直接操作，保留所有登录状态、Cookie 等
print(page.url)
page.ele('#some-btn').click()
```

---

## Shadow DOM 处理

```python
# 方式一：直接查找（DrissionPage 自动穿透 Shadow DOM）
ele = page.ele('#shadow-input')     # 在 shadow DOM 内也能找到

# 方式二：显式获取 shadow root
host = page.ele('#shadow-host')
shadow_root = host.shadow_root

inner = shadow_root.ele('.shadow-item')
inner.click()

# 多层嵌套 shadow DOM（链式穿透）
deep_ele = (
    page.ele('#outer-host')
        .shadow_root
        .ele('#inner-host')
        .shadow_root
        .ele('#target-element')
)
```

---

## 多标签页并发（多线程）

```python
from DrissionPage import ChromiumPage
import threading

page = ChromiumPage()

def fetch_page(url, results, index):
    tab = page.new_tab(url)
    tab.wait.doc_loaded()
    results[index] = {
        'url': tab.url,
        'title': tab.title,
        'content': tab.ele('tag:body').text[:100]
    }
    tab.close()

urls = [
    'https://www.baidu.com',
    'https://www.bing.com',
    'https://www.sogou.com',
]

results = [None] * len(urls)
threads = [threading.Thread(target=fetch_page, args=(url, results, i))
           for i, url in enumerate(urls)]

for t in threads:
    t.start()
for t in threads:
    t.join()

for r in results:
    print(r['title'])
```

---

## 多浏览器实例

```python
from DrissionPage import ChromiumOptions, Chromium

# 启动两个独立浏览器实例
co1 = ChromiumOptions().set_local_port(9222)
co2 = ChromiumOptions().set_local_port(9333)

browser1 = Chromium(co1)
browser2 = Chromium(co2)

tab1 = browser1.latest_tab
tab2 = browser2.latest_tab

# 两个浏览器同时操作（可用于多账号并行）
tab1.get('https://account1.example.com')
tab2.get('https://account2.example.com')
```

---

## 反爬对抗

```python
from DrissionPage import ChromiumOptions, ChromiumPage

co = ChromiumOptions()
# 移除 webdriver 标识
co.set_argument('--disable-blink-features=AutomationControlled')
# 设置真实 UA
co.set_user_agent('Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
                  'AppleWebKit/537.36 ...')

page = ChromiumPage(co)

# 注入 JS 消除 navigator.webdriver 特征
page.run_js('Object.defineProperty(navigator, "webdriver", {get: () => undefined})')

# 模拟真人操作节奏
import random, time

def human_click(ele):
    """带随机延迟的真人风格点击"""
    ele.scroll.to_see()
    time.sleep(random.uniform(0.3, 0.8))
    # 偏移中心点随机几像素
    w, h = ele.rect.size
    ele.click.at(random.randint(-w//4, w//4),
                 random.randint(-h//4, h//4))
    time.sleep(random.uniform(0.5, 1.5))

def human_type(ele, text):
    """逐字符输入，带随机间隔"""
    ele.click()
    for char in text:
        ele.input(char, clear=False)
        time.sleep(random.uniform(0.05, 0.18))
```

---

## 保存与复用 Cookie

```python
import json

page = ChromiumPage()

# ——— 首次运行：登录并保存 Cookie ———
page.get('https://example.com/login')
page.ele('#user').input('admin')
page.ele('#pass').input('123456')
page.ele('#login-btn').click()
page.wait.eles_loaded('#dashboard')

# 保存 Cookie 到文件
cookies = page.cookies().as_dict()
with open('cookies.json', 'w', encoding='utf-8') as f:
    json.dump(cookies, f, ensure_ascii=False)

# ——— 下次运行：加载 Cookie 跳过登录 ———
with open('cookies.json', encoding='utf-8') as f:
    cookies = json.load(f)

# 先访问域名（需要同域才能设置 Cookie）
page.get('https://example.com')
page.set.cookies(cookies)
# 刷新即可进入登录状态
page.get('https://example.com/dashboard')
```

---

## ini 配置文件管理

```python
from DrissionPage import ChromiumOptions

co = ChromiumOptions()
co.headless()
co.no_imgs()
co.set_proxy('http://127.0.0.1:7890')

# 保存到默认 ini 文件（下次程序启动自动读取）
co.save()

# 保存到指定路径
co.save(path='my_project.ini')

# 读取指定路径的配置
co2 = ChromiumOptions(ini_path='my_project.ini')
```

---

## 处理动态/懒加载内容

```python
# 方案一：滚动触发懒加载，再等待元素
page.scroll.to_bottom()
page.wait(1, 2)   # 随机等待加载
items = page.eles('.lazy-item')

# 方案二：结合网络监听，等待 Ajax 完成
page.listen.start('api/items')
page.scroll.to_bottom()
packet = page.listen.wait(timeout=8)
if packet:
    data = packet.response.body

# 方案三：以某个元素出现作为"加载完成"的信号
page.scroll.to_bottom()
page.wait.eles_loaded('.load-more-btn', timeout=10)
```

---

## 超时与异常配置

```python
from DrissionPage.common import Settings

# 等待失败时自动抛出异常（默认静默返回 False）
Settings.raise_when_wait_failed = True

# 点击失败时抛出异常
Settings.raise_click_failed = True

# 配置全局超时
co = ChromiumOptions()
co.set_timeouts(base=15, page_load=30, script=10)
page = ChromiumPage(co)

# 手动重试封装
def retry_get(page, url, max_retries=3):
    for i in range(max_retries):
        try:
            if page.get(url):
                return True
        except Exception as e:
            print(f"第{i+1}次失败: {e}")
            page.wait(2, 3)
    return False
```

---

## 相关 skills

- `Drission-Page:quickstart` — 快速入门与基础概念
- `Drission-Page:browser-ops` — 浏览器控制（截图、标签页等）
- `Drission-Page:network` — 网络监听与数据包拦截
- `Drission-Page:find-elements` — 元素定位（Shadow DOM 查找）
