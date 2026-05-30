---
name: Drission-Page:update
description: |
  DrissionPage 库版本检查与升级。当用户询问 DrissionPage 当前版本、
  是否有新版本、如何升级、升级后是否有破坏性变更、如何回退版本、
  或报告导入报错/API 不存在等可能由版本不匹配引起的问题时触发。
  也适用于"我的 DrissionPage 是最新的吗"、"升级会不会影响现有代码"、
  "怎么查看更新日志"、"pip install 之后还是旧版本怎么办"等场景。
author: claude
version: 4.1.1.4
---

# DrissionPage 库版本检查与升级

---

## 快速检查当前状态

```bash
# 查看已安装版本
pip show DrissionPage

# 查看 PyPI 最新版本（同时对比已安装版本）
pip index versions DrissionPage

# 一行命令对比
python -c "import DrissionPage; print('installed:', DrissionPage.__version__)"
```

---

## 检查是否有新版本

```bash
# 方法一：pip list 过滤（需要 pip >= 22.0）
pip list --outdated | grep DrissionPage

# 方法二：直接查询 PyPI JSON API
python -c "
import urllib.request, json
url = 'https://pypi.org/pypi/DrissionPage/json'
with urllib.request.urlopen(url) as r:
    latest = json.loads(r.read())['info']['version']
import DrissionPage
print(f'已安装: {DrissionPage.__version__}')
print(f'最新版: {latest}')
print('已是最新' if DrissionPage.__version__ == latest else '有新版本可升级')
"
```

---

## 升级操作

```bash
# 升级到最新版（推荐）
pip install --upgrade DrissionPage

# 升级到指定版本
pip install DrissionPage==4.1.1.4

# 国内镜像加速（升级慢时使用）
pip install --upgrade DrissionPage -i https://pypi.tuna.tsinghua.edu.cn/simple

# 验证升级成功
python -c "import DrissionPage; print(DrissionPage.__version__)"
```

---

## 跨大版本升级注意事项

DrissionPage 的版本号格式为 `大版本.次版本.补丁.修订`，大版本升级（如 3.x → 4.x）可能有**破坏性变更**：

### 3.x → 4.x 主要变更

| 3.x 写法 | 4.x 新写法 | 说明 |
|---------|-----------|------|
| `MixPage` | `WebPage` | 主类重命名 |
| `DriverPage` | `ChromiumPage` | 浏览器模式独立类 |
| `SessionPage` | `SessionPage`（保留） | 无变化 |
| `.find()` | `.ele()` | 查找方法重命名 |
| `.finds()` | `.eles()` | 批量查找重命名 |
| `page.change_mode()` | `page.change_mode()` | 保留，用法一致 |
| `DrissionPage.easy_set` | `DrissionPage.easy_set`（部分变化） | 配置模块调整 |

> **升级前建议：** 先在测试环境验证，跑一遍现有脚本，确认无报错再升级生产环境。

---

## 版本回退

```bash
# 查看所有历史版本
pip index versions DrissionPage

# 安装指定旧版本（会自动覆盖当前版本）
pip install DrissionPage==4.1.0.18

# 锁定版本（防止 pip upgrade all 时被自动升级）
pip install DrissionPage==4.1.1.4
# 在 requirements.txt 中写：DrissionPage==4.1.1.4
```

---

## 常见升级问题排查

### 升级后 ImportError / AttributeError

```python
# 检查实际加载的模块路径（排除多 Python 环境冲突）
import DrissionPage, os
print(os.path.dirname(DrissionPage.__file__))
print(DrissionPage.__version__)
```

```bash
# 如果路径不对，说明 pip 安装到了错误的 Python 环境
# 用 python -m pip 确保安装到当前使用的 Python
python -m pip install --upgrade DrissionPage
```

### 虚拟环境中升级

```bash
# 确认当前激活的虚拟环境
which python   # Linux/Mac
where python   # Windows

# 在 venv 中升级
.\venv\Scripts\pip install --upgrade DrissionPage   # Windows
source venv/bin/activate && pip install --upgrade DrissionPage  # Linux/Mac
```

### pip 本身过旧导致安装失败

```bash
python -m pip install --upgrade pip
pip install --upgrade DrissionPage
```

---

## 查看更新日志

- **GitHub Releases：** https://github.com/g1879/DrissionPage/releases
- **官方文档：** https://DrissionPage.cn
- **PyPI 页面：** https://pypi.org/project/DrissionPage/#history

---

## 一键检查 + 升级脚本

```python
"""DrissionPage 版本检查与自动升级工具"""
import subprocess, sys, json
from urllib.request import urlopen

def check_and_upgrade(auto_upgrade=False):
    # 获取已安装版本
    result = subprocess.run(
        [sys.executable, '-m', 'pip', 'show', 'DrissionPage'],
        capture_output=True, text=True
    )
    installed = None
    for line in result.stdout.splitlines():
        if line.startswith('Version:'):
            installed = line.split(':', 1)[1].strip()
            break

    if not installed:
        print("DrissionPage 未安装。运行：pip install DrissionPage")
        return

    # 获取 PyPI 最新版本
    try:
        with urlopen('https://pypi.org/pypi/DrissionPage/json', timeout=5) as r:
            latest = json.loads(r.read())['info']['version']
    except Exception as e:
        print(f"无法连接 PyPI：{e}")
        print(f"已安装版本：{installed}")
        return

    print(f"已安装版本：{installed}")
    print(f"PyPI 最新版：{latest}")

    if installed == latest:
        print("已是最新版本，无需升级。")
    else:
        print(f"有新版本可用：{installed} → {latest}")
        if auto_upgrade:
            print("正在升级...")
            subprocess.run([sys.executable, '-m', 'pip', 'install',
                           '--upgrade', 'DrissionPage'])
        else:
            print("运行以下命令升级：")
            print("  pip install --upgrade DrissionPage")

if __name__ == '__main__':
    # 改为 True 可自动升级
    check_and_upgrade(auto_upgrade=False)
```
