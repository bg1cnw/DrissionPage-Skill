---
name: Drission-Page:find-elements
description: |
  DrissionPage 元素定位与查找语法大全。当用户询问如何查找/定位元素、
  CSS 选择器怎么写、XPath 怎么用、怎么按文本/属性/标签查找、
  find 方法返回 NoneElement 怎么处理、如何遍历子元素、如何获取兄弟/父/相邻元素时触发。
  也适用于"怎么定位动态加载的元素"、"元素找不到怎么办"、"怎么同时查找多个元素"等问题。
  DrissionPage 有独特的简洁定位语法，不同于 selenium，使用此 skill 可避免走弯路。
author: claude
version: 4.1.1.4
---

# DrissionPage 元素定位语法大全

> **参考文档**：`references/docs/控制浏览器/🔦 定位语法.md` — 定位语法详解
> **补充文档**：`references/docs/控制浏览器/🔦 语法速查表.md`、`references/docs/控制浏览器/🔦 页面或元素内查找.md`、`references/docs/控制浏览器/🔦 相对定位.md`、`references/docs/控制浏览器/🔦 简化写法.md`、`references/docs/控制浏览器/🔦 行为模式.md`、`references/docs/控制浏览器/🔦 在结果列表中筛选.md`
> **文档映射**：`references/docs-map.md`

DrissionPage 提供了一套独特的简洁定位语法，比 CSS/XPath 更易写，同时也完全兼容标准选择器。

---

## 基础查找方法

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `ele(loc, index=1)` | 查找单个元素 | `ChromiumElement` / `NoneElement` |
| `eles(loc)` | 查找所有匹配元素 | `List[ChromiumElement]` |
| `page(loc)` | `ele()` 的简写调用 | 同上 |

```python
# 查找单个元素
ele = page.ele('#login-btn')
ele = page('#login-btn')   # 简写，等价

# 查找所有匹配
items = page.eles('.item')

# 获取第2个匹配（index 从1开始，负数从末尾计）
ele = page.ele('.item', index=2)
ele = page.ele('.item', index=-1)  # 最后一个
```

---

## DrissionPage 原生定位语法

### 按 ID 查找

```python
ele = page.ele('#username')         # id="username"
```

### 按 Class 查找

```python
ele = page.ele('.nav-item')         # class 含 "nav-item"
ele = page.ele('.btn.primary')      # 同时含两个 class
```

### 按标签名查找

```python
ele = page.ele('tag:div')
ele = page.ele('tag:input')
ele = page.ele('tag:a')
```

### 按文本内容查找

```python
# 模糊匹配（包含该文本的元素）
ele = page.ele('确认提交')

# 精确匹配（文本完全等于）
ele = page.ele('text=确认提交')

# 包含匹配（同默认模糊匹配）
ele = page.ele('text:确认')
```

### 按属性查找

```python
# 属性精确匹配（=）
ele = page.ele('@name=username')
ele = page.ele('@type=submit')
ele = page.ele('@placeholder=请输入用户名')

# 属性包含匹配（:）
ele = page.ele('@class:nav')

# 多属性 AND 组合（@@ 分隔）
ele = page.ele('@type=text@@name=email')

# 属性以某值开头（^）
ele = page.ele('@href^=https://')

# 属性以某值结尾（$）
ele = page.ele('@src$=.png')
```

---

## 四种匹配模式

| 符号 | 含义 | 示例 |
|------|------|------|
| `=` | 精确匹配 | `@name=user` |
| `:` | 包含匹配（模糊） | `@class:nav` |
| `^` | 开头匹配 | `@href^=https` |
| `$` | 结尾匹配 | `@src$=.jpg` |

文本查找同样支持这四种模式：

```python
page.ele('text=完全匹配的文字')
page.ele('text:包含这段')
page.ele('text^=以此开头')
page.ele('text$=以此结尾')
```

---

## CSS 选择器

```python
ele = page.ele('css:#main .list > li')
ele = page.ele('css:input[type="text"]')
ele = page.ele('css:.container > div:first-child')
ele = page.ele('css:a[href*="example"]')
```

---

## XPath

```python
ele = page.ele('xpath://div[@id="content"]/p')
ele = page.ele('xpath://button[text()="提交"]')
ele = page.ele('xpath://input[@type="text" and @name="user"]')
ele = page.ele('xpath://li[contains(@class,"item")][2]')  # 第2个 li
```

---

## 从元素出发查找子/后代元素

```python
container = page.ele('#container')

# 查找后代元素（自动处理层级）
item = container.ele('.item')
items = container.eles('.item')

# XPath 查找直接子元素
first_child = container.ele('xpath:child::*[1]')
all_children = container.eles('xpath:child::*')
```

---

## 相对定位：父/兄弟/前后元素

```python
ele = page.ele('#target')

# 父元素
parent = ele.parent()               # 直接父元素
parent2 = ele.parent(2)             # 向上2级
parent_div = ele.parent('tag:div')  # 向上找第一个 div

# 下/上一个兄弟元素
next_ele = ele.next()               # 下一个兄弟
prev_ele = ele.prev()               # 上一个兄弟
next_div = ele.next('tag:div')      # 下一个 div 兄弟

# 所有兄弟元素列表
all_next = ele.nexts()
all_prev = ele.prevs()

# DOM 中任意位置的前/后元素（不限兄弟）
after_table = ele.after('tag:table')
before_header = ele.before('.header')

# 所有满足条件的前/后元素
all_after = ele.afters('tag:div')
all_before = ele.befores('tag:p')
```

---

## 处理找不到元素（NoneElement）

DrissionPage 找不到元素时返回 `NoneElement`，**不会抛出异常**：

```python
ele = page.ele('#not-exist')

# 方式一：用 if 判断（推荐）
if ele:
    ele.click()
else:
    print("未找到元素")

# 方式二：判断类型
from DrissionPage import NoneElement
if not isinstance(ele, NoneElement):
    ele.click()

# NoneElement 的文本/属性访问不报错，返回空值
print(ele.text)       # 输出 ''
print(ele.attr('id')) # 输出 None
```

---

## 等待元素出现（动态页面必备）

```python
# 等待元素出现在 DOM 中，返回元素对象
ele = page.wait.eles_loaded('#dynamic-result', timeout=10)
if ele:
    print(ele.text)

# 等待元素变为可见
page.wait.ele_displayed('#loading-result')

# 等待元素消失（如加载动画）
page.wait.ele_hidden('#spinner')

# 在 ele() 中直接设置等待超时
ele = page.ele('#slow-element', timeout=10)
```

---

## Shadow DOM 元素定位

DrissionPage 支持自动穿透 Shadow DOM，多数情况下可直接查找 shadow 内的元素。复杂场景（多层嵌套、显式获取 shadow root）参考 `Drission-Page:advanced`。

```python
# 自动穿透 — 直接查 shadow DOM 内元素
ele = page.ele('#shadow-input')

# 更多高级用法见 Drission-Page:advanced
```
---

## 组合定位技巧

```python
# tag + class 组合
ele = page.ele('tag:a@class:active')

# tag + 文本内容组合
ele = page.ele('tag:button@@text():登录')

# 批量获取并过滤
links = page.eles('tag:a')
valid_links = [a for a in links if a.attr('href') and 'example' in a.attr('href')]

# 获取元素文本
print(ele.text)       # 纯文本（去标签）
print(ele.raw_text)   # 保留格式的原始文本

# 获取元素属性
print(ele.attr('href'))
print(ele.attr('data-id'))
print(ele.attrs)      # 所有属性的字典
```

---

## 相关 skills

- `Drission-Page:quickstart` — 快速入门
- `Drission-Page:element-ops` — 对找到的元素进行操作（点击、输入等）
- `Drission-Page:advanced` — iframe/Shadow DOM 高级定位技巧与等待机制
