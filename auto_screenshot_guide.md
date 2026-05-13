# auto_screenshot_guide.md

# 自动截图与截图整理指南

本文用于配合 \`Claude_Codex_Deployment_Guide.md\` 的插图占位符完成截图采集、命名、整理和后续插入。

> 安全原则：任何截图都不得包含真实 API Key、访问令牌、授权码、完整邮箱、私有仓库地址、账单信息、设备序列号或组织内部信息。涉及账号页面时，先打码再入文档。

---

# 1. 如何手动截图

## 1.1 Windows 截图快捷键

| 快捷键 | 作用 | 适合场景 |
|---|---|---|
| \`Win + Shift + S\` | 打开系统截图工具，手动框选区域 | 安装器、设置窗口、终端局部截图 |
| \`PrtSc\` | 截取整个屏幕到剪贴板 | 快速记录全屏状态 |
| \`Alt + PrtSc\` | 截取当前活动窗口 | 安装器窗口、VS Code 窗口 |
| \`Win + PrtSc\` | 截取全屏并保存到图片文件夹 | 快速保留现场 |

推荐优先使用 \`Win + Shift + S\`，因为它可以只截取需要的区域，避免暴露隐私信息。

## 1.2 Snipping Tool 使用方法

1. 按 \`Win + S\` 搜索 \`Snipping Tool\` 或“截图工具”。
2. 点击“新建”，框选需要的区域。
3. 使用标注工具圈出重点区域。
4. 保存为 PNG，并按截图命名规范放入 \`screenshots/\` 文件夹。

## 1.3 浏览器截图方法

1. 打开官方网页。
2. 按 \`F11\` 进入全屏，减少无关区域。
3. 使用 \`Win + Shift + S\` 截取下载按钮、版本号或关键设置区域。
4. 保存为 PNG，例如 \`02-01_vscode_homepage.png\`。

浏览器开发者工具全页截图：按 \`F12\`，再按 \`Ctrl + Shift + P\`，输入 \`screenshot\`，选择 \`Capture full size screenshot\`。

## 1.4 VS Code 截图方法

1. 打开对应项目或设置页面。
2. 隐藏无关侧栏、账号头像和通知。
3. 放大到适合阅读的字号。
4. 使用 \`Win + Shift + S\` 截取核心区域。
5. 对关键按钮或选项进行红框标注。

---

# 2. 自动化截图方案

## 2.1 Playwright 自动截图

Playwright 适合自动打开网页并截取页面。优点是稳定、支持全页截图；缺点是首次需要安装浏览器依赖。

## 2.2 Selenium 自动截图

Selenium 适合使用本机 Edge 或 Chrome 自动截图。优点是生态成熟；缺点是浏览器驱动版本需要匹配。

## 2.3 浏览器全页截图

长网页可使用 Chrome/Edge 开发者工具的 \`Capture full size screenshot\`，或 Playwright 的 \`page.screenshot(full_page=True)\`。

---

# 3. 自动截图脚本

## 3.1 Python + Playwright 版本

安装依赖：

\`\`\`powershell
python -m pip install playwright
python -m playwright install chromium
\`\`\`

脚本文件：\`scripts/screenshot_with_playwright.py\`

\`\`\`python
from pathlib import Path
from playwright.sync_api import sync_playwright

SCREENSHOTS = Path("screenshots")
SCREENSHOTS.mkdir(exist_ok=True)

PAGES = [
    ("02-01_vscode_homepage.png", "https://code.visualstudio.com/download"),
    ("03-01_git_download.png", "https://git-scm.com/downloads/win"),
    ("04-01_nodejs_download.png", "https://nodejs.org/en/download/"),
    ("05-01_python_download.png", "https://www.python.org/downloads/windows/"),
]

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1440, "height": 1000})
    for filename, url in PAGES:
        print(f"Capturing {url} -> {filename}")
        page.goto(url, wait_until="networkidle", timeout=60000)
        page.screenshot(path=str(SCREENSHOTS / filename), full_page=True)
    browser.close()
\`\`\`

运行：

\`\`\`powershell
python scripts\\screenshot_with_playwright.py
\`\`\`

## 3.2 Python + Selenium 版本

安装依赖：

\`\`\`powershell
python -m pip install selenium
\`\`\`

脚本文件：\`scripts/screenshot_with_selenium.py\`

\`\`\`python
from pathlib import Path
from selenium import webdriver
from selenium.webdriver.edge.options import Options
from selenium.webdriver.support.ui import WebDriverWait

SCREENSHOTS = Path("screenshots")
SCREENSHOTS.mkdir(exist_ok=True)

PAGES = [
    ("02-01_vscode_homepage.png", "https://code.visualstudio.com/download"),
    ("03-01_git_download.png", "https://git-scm.com/downloads/win"),
    ("04-01_nodejs_download.png", "https://nodejs.org/en/download/"),
    ("05-01_python_download.png", "https://www.python.org/downloads/windows/"),
]

options = Options()
options.add_argument("--headless=new")
options.add_argument("--window-size=1440,1000")

driver = webdriver.Edge(options=options)
try:
    for filename, url in PAGES:
        print(f"Capturing {url} -> {filename}")
        driver.get(url)
        WebDriverWait(driver, 30).until(lambda d: d.execute_script("return document.readyState") == "complete")
        driver.save_screenshot(str(SCREENSHOTS / filename))
finally:
    driver.quit()
\`\`\`

运行：

\`\`\`powershell
python scripts\\screenshot_with_selenium.py
\`\`\`

---

# 4. PowerShell 批处理方案

## 4.1 自动创建 screenshots 文件夹

\`\`\`powershell
New-Item -ItemType Directory -Force -Path .\\screenshots
New-Item -ItemType Directory -Force -Path .\\scripts
\`\`\`

## 4.2 自动整理截图命名

\`\`\`powershell
Get-ChildItem "$env:USERPROFILE\\Downloads" -Filter *.png |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 20 Name, LastWriteTime, Length
\`\`\`

## 4.3 使用 Edge Headless 截图

\`\`\`powershell
$edge = "C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\msedge.exe"
New-Item -ItemType Directory -Force -Path .\\screenshots
& $edge --headless --disable-gpu --window-size=1440,1000 --screenshot="$(Resolve-Path .\\screenshots\\02-01_vscode_homepage.png)" "https://code.visualstudio.com/download"
\`\`\`

如果出现权限、沙箱、公司策略限制，请改用手动截图或 Playwright。

---

# 5. 截图命名规范

## 5.1 基本格式

\`\`\`text
章节号-图序号_英文描述.png
\`\`\`

示例：

\`\`\`text
02-01_vscode_homepage.png
02-02_vscode_install.png
03-01_git_download.png
04-01_nodejs_download.png
05-02_python_path_checkbox.png
10-01_openai_api_keys.png
11-05_ai_todo_running.png
\`\`\`

## 5.2 命名规则

| 规则 | 示例 | 说明 |
|---|---|---|
| 两位章节号 | \`02\` | 第 2 章 |
| 两位图序号 | \`01\` | 本章第 1 张图 |
| 英文小写描述 | \`vscode_homepage\` | 避免中文路径兼容问题 |
| PNG 格式 | \`.png\` | 技术文档截图推荐格式 |

## 5.3 Markdown 插入格式

\`\`\`markdown
![图2-1 VS Code 官方下载页面](screenshots/02-01_vscode_homepage.png)
\`\`\`

插入真实图片后，建议保留原来的插图说明，便于后续排版成 Word 或技术博客。
