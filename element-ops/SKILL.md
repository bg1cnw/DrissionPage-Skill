---
name: Drission-Page:element-ops
description: |
  DrissionPage 元素交互操作大全。当用户询问如何点击元素、如何在输入框输入文字、
  如何处理下拉选择框（select）、如何上传文件、如何拖拽元素、如何模拟键盘按键、
  如何获取元素属性/文本/尺寸/位置、如何操作 checkbox 和 radio 时触发。
  也适用于"点击没有反应怎么办"、"input 输入不进去"、"怎么清空输入框"、
  "如何模拟鼠标悬停/拖拽"、"怎么获取元素的 HTML 内容"等问题。
  使用此 skill 掌握 ChromiumElement 的完整交互能力。
author: claude
version: 4.1.1.4
---

# DrissionPage 元素操作大全

> **参考文档**：`references/docs/控制浏览器/🛰️ 元素交互.md` — 点击、输入、hover、check
> **补充文档**：`references/docs/控制浏览器/🛰️ 动作链.md`、`references/docs/控制浏览器/🛰️ 上传文件.md`、`references/docs/控制浏览器/🛰️ 获取元素信息.md`、`references/docs/控制浏览器/🛰️ 获取网页信息.md`、`references/docs/特性与示例/⭐ 获取元素属性.md`
> **文档映射**：`references/docs-map.md`

---

## 点击操作

```python
ele = page.ele('#submit-btn')

# 普通点击（自动等待可用、滚动到视口、判断遮挡）
ele.click()
# 等价于 click.left(by_js=False, timeout=1.5, wait_stop=True)

# 强制 JS 点击（穿透遮挡，只关心元素在 DOM 中）
ele.click(by_js=True)

# 带超时的模拟点击
ele.click.left(by_js=False, timeout=10)

# 右键点击
ele.click.right()

# 中键点击（常用于打开新标签）
ele.click.middle()

# 双击
ele.click.multi(times=2)

# 带偏移量的点击（相对元素左上角，正值向右/下）
ele.click.at(50, 10)      # 偏右50、偏下10像素处

# 点击触发下载（自动处理下载等待）
ele.click.to_download(save_path=r'C:\Downloads', rename='myfile.pdf')

# 点击触发上传
ele.click.to_upload(r'C:\file.jpg')

# 点击后等待某元素出现
ele.click()
result = page.wait.eles_loaded('#result', timeout=5)
```

---

## 文本输入

```python
inp = page.ele('#search-input')

# 清空并输入（默认行为）
inp.input('hello world')

# 追加输入（不清空原内容）
inp.input('more text', clear=False)

# 仅清空输入框
inp.clear()

# 模拟真人逐字符输入（触发完整 keydown/keypress/keyup 事件）
# 逐字符输入，配合 page.wait() 实现慢速效果
for char in '慢速输入':
    inp.input(char, clear=False)
    page.wait(0.05, 0.15)  # 随机间隔 50-150ms

# 输入后直接按 Enter
from DrissionPage.common import Keys
inp.input('search text' + Keys.ENTER)
```

---

## 键盘按键

```python
from DrissionPage.common import Keys

ele = page.ele('#textarea')

# 特殊按键
ele.input(Keys.ENTER)
ele.input(Keys.TAB)
ele.input(Keys.ESC)
ele.input(Keys.BACKSPACE)
ele.input(Keys.DELETE)
ele.input(Keys.SPACE)
ele.input(Keys.F5)

# 方向键
ele.input(Keys.UP)
ele.input(Keys.DOWN)
ele.input(Keys.LEFT)
ele.input(Keys.RIGHT)

# 组合键
ele.input(Keys.CTRL + 'a')          # Ctrl+A 全选
ele.input(Keys.CTRL + 'c')          # Ctrl+C 复制
ele.input(Keys.CTRL + 'v')          # Ctrl+V 粘贴
ele.input(Keys.CTRL + Keys.ENTER)   # Ctrl+Enter
ele.input(Keys.SHIFT + Keys.TAB)    # Shift+Tab
```

---

## 下拉选择框（Select）

```python
select = page.ele('tag:select#country')

# 按显示文本选择（直接调用 select 对象）
select('中国')
# 也可显式调用方法
select.by_text('中国')

# 按 value 属性选择
select.by_value('CN')

# 按选项索引（从 0 开始）
select.by_index(2)

# 多选框——同时选中多个
select(['选项1', '选项2', '选项3'])

# 取消选中
select.cancel_by_text('选项1')
select.clear()                          # 取消全部选中

# 获取当前选中项
print(select.selected_option.text)
print(select.selected_option.attr('value'))
print(select.selected_options)          # 多选时返回列表
```

---

## Checkbox 和 Radio

```python
# Checkbox — 点击切换状态
cb = page.ele('tag:input@type=checkbox@@name=agree')
cb.click()

# 确保 checkbox 处于选中状态
if not cb.states.is_checked:
    cb.click()

# 确保 checkbox 处于未选中状态
if cb.states.is_checked:
    cb.click()

# Radio — 选中某个选项
radio = page.ele('@type=radio@@value=male')
radio.click()
```

---

## 文件上传

```python
# 方法一：直接向 input[type=file] 输入文件路径
file_input = page.ele('tag:input@type=file')
file_input.input(r'C:\Users\user\Documents\file.jpg')

# 上传多个文件（需要 multiple 属性）
file_input.input([r'C:\file1.jpg', r'C:\file2.png', r'C:\file3.pdf'])

# 方法二：拦截文件对话框（适用于点击按钮触发上传的情况）
page.set.upload_files(r'C:\file.jpg')       # 预设要上传的文件路径
page.ele('#upload-btn').click()             # 点击触发文件选择框
page.wait.upload_paths_inputted()           # 等待文件路径注入完成
```

---

## 获取元素信息

```python
ele = page.ele('#product-card')

# 文本内容
print(ele.text)          # 纯文本（去除 HTML 标签、整理空白）
print(ele.raw_text)      # 保留格式的原始文本（含换行、空格）

# HTML 内容
print(ele.html)          # 元素的 outer HTML（含自身标签）
print(ele.inner_html)    # 元素的 inner HTML（不含自身标签）

# 属性值
print(ele.attr('class'))
print(ele.attr('href'))
print(ele.attr('data-id'))
print(ele.attrs)         # 所有属性的字典

# 标签名
print(ele.tag)           # 例如 'div'、'a'、'input'

# 元素尺寸与坐标
print(ele.rect.size)       # (宽度, 高度)，单位像素
print(ele.rect.location)   # (x, y)，相对视口左上角
print(ele.rect.midpoint)   # (x, y)，元素中心点
print(ele.rect.corners)    # 四个角的坐标
```

---

## 元素状态检测

```python
ele = page.ele('#submit-btn')

print(ele.states.is_displayed)      # 是否可见（未被 display:none 隐藏）
print(ele.states.is_enabled)        # 是否可用（非 disabled 状态）
print(ele.states.is_checked)        # 是否被选中（checkbox/radio）
print(ele.states.is_in_viewport)    # 是否在当前视口内
print(ele.states.is_covered)        # 是否被其他元素遮挡
```

---

## 滚动到元素

```python
ele = page.ele('#footer-section')

ele.scroll.to_see()          # 滚动使元素出现在视口中
ele.scroll.to_top()          # 滚动到元素内部的顶部
ele.scroll.to_bottom()       # 滚动到元素内部的底部
ele.scroll.down(200)         # 在元素内向下滚动 200px
```

---

## 鼠标动作链（连续动作）

```python
# 通过 page.actions 执行连续鼠标动作
ac = page.actions

# 移动到元素（触发 hover 效果）
ac.move_to('#menu-item')

# 悬停在元素上停留一段时间
ac.move_to('#hover-btn').wait(0.8)

# 简单拖拽：从源元素拖到目标元素
source = page.ele('#drag-handle')
target = page.ele('#drop-zone')
ac.drag(source, target)

# 精确拖拽：按坐标拖拽
ac.drag_to((100, 300), (500, 300))

# 滑块验证码处理（按住后缓慢移动）
slider = page.ele('#slider-btn')
ac.hold_on(slider)           # 按下鼠标不松开
ac.move(200, 0, duration=1)  # 缓慢向右移动200px，用时1秒
ac.release()                 # 松开鼠标

# 键盘+鼠标组合（Ctrl+点击多选）
ac.key_down('ctrl')
ac.click('#item1')
ac.click('#item2')
ac.key_up('ctrl')
```

---

## 等待元素交互就绪

```python
# 等待元素出现后操作
ele = page.wait.eles_loaded('#ajax-result', timeout=10)
if ele:
    print(ele.text)
    ele.click()

# 等待元素可见后操作
page.wait.ele_displayed('#confirm-dialog')
page.ele('#confirm-btn').click()

# 等待加载动画消失
page.wait.ele_hidden('#loading-mask', timeout=15)
page.ele('#real-content').text
```

---

## 相关 skills

- `Drission-Page:find-elements` — 如何定位/查找元素
- `Drission-Page:browser-ops` — 浏览器级别的操作（截图、标签页）
- `Drission-Page:advanced` — 等待机制、Shadow DOM、反爬技巧
