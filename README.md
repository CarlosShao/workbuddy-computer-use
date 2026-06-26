# workbuddy-computer-use

Windows 桌面自动化 Skill for WorkBuddy — 让 AI Agent 像人一样操控 Windows 桌面应用。

[English](#english)

---

## 这是什么？

对标 OpenAI Codex / Anthropic Claude 的 **Computer Use** 能力，但是：

- ✅ **跑在你自己的 Windows 机器上**，不需要远程 VM
- ✅ **零按 token 计费**，截图/操作不限次数
- ✅ **支持中文输入**（通过剪贴板）
- ✅ **支持任意 Windows 应用**：Win32 / WinForms / WPF / Qt / Electron / UWP

当 WorkBuddy 遇到浏览器覆盖不到的场景（原生桌面应用），自动加载本 Skill。

---

## 能做什么？

| 能力 | 典型命令 |
|------|---------|
| 📸 **截图** | 全屏截图、区域截图、返回 base64 |
| 🖱️ **鼠标** | 移动、单击、双击、右键、拖拽、滚轮 |
| ⌨️ **键盘** | 输入文本（含中文）、组合快捷键、按键按下/抬起 |
| 🪟 **窗口管理** | 列出所有窗口、激活、最小化、最大化、关闭 |
| 🔍 **UI 自动化** | 按控件名/类型查找元素、点击、填文本、等待元素出现 |
| 🖼️ **图像模板匹配** | OpenCV 找图、点击图标、等待图像出现（NMS 去重） |
| 🔤 **OCR（可选）** | 屏幕文字识别（需安装 Tesseract） |
| 🛡️ **安全机制** | Emergency Stop、Failsafe（鼠标甩到角落急停） |

---

## 安装

### 1. 安装 Skill（WorkBuddy 内）

```
/install-skill workbuddy-computer-use
```

或从 GitHub 手动安装：

```bash
git clone https://github.com/CarlosShao/workbuddy-computer-use.git \
  ~/.workbuddy/skills/workbuddy-computer-use/
```

### 2. 安装 Python 依赖

本 Skill 使用**隔离 venv**，不影响系统 Python：

```bash
# 进入 skill 目录
cd ~/.workbuddy/skills/workbuddy-computer-use/

# 创建隔离虚拟环境
python -m venv .venv

# 安装依赖（Windows）
.venv/Scripts/pip.exe install pyautogui pywinauto opencv-python numpy mss pillow pytesseract

# 安装依赖（macOS / Linux）
.venv/bin/pip install pyautogui pywinauto opencv-python numpy mss pillow pytesseract
```

> **注意**：Python 版本要求 ≥ 3.10。如果系统没有 `python` 命令，请用 `python3` 替代。

### 3.（可选）安装 Tesseract OCR

如果需要 OCR 功能：

- 下载：https://github.com/UB-Mannheim/tesseract/wiki
- 安装后确保 `tesseract` 命令在 PATH 中
- 中文语言包确认：`tesseract --list-langs` 含 `chi_sim`

---

## 使用示例

### 示例 1：自动填表（记事本）

```
用户：帮我在记事本里输入"你好 WorkBuddy"，然后保存为 test.txt
```

WorkBuddy 加载本 Skill 后自动执行：
1. `list-windows` 找到记事本
2. `activate-window` 激活窗口
3. `type "你好 WorkBuddy"` 输入中文
4. `hotkey ctrl s` 触发保存
5. `set-text` 填文件名
6. `click-element` 点保存按钮

### 示例 2：找图点击

```
用户：屏幕上有个"确定"按钮的图标，帮我找到并点击它
```

### 示例 3：UI Automation（计算器）

```
用户：打开计算器，算一下 123 * 456
```

---

## CLI 全命令参考

<details>
<summary>点击展开全部命令</summary>

```bash
python cli.py --help

# 截图
screenshot --output <path> [--base64]
screen-size
pixel --x <n> --y <n>

# 鼠标
mouse-position
move --x <n> --y <n> [--duration <s>]
click [--x <n> --y <n>] [--button left|middle|right]
double-click [--x <n> --y <n>]
right-click [--x <n> --y <n>]
drag --x1 <n> --y1 <n> --x2 <n> --y2 <n>
scroll [--clicks <n>] [--x <n> --y <n>]

# 键盘
type --text <str>
hotkey <key1> [key2 ...]
key-press --key <str>
key-down --key <str>
key-up --key <str>
wait [--seconds <n>]

# 窗口
list-windows [--filter <str>]
find-window --title <str>
activate-window --title <str>
minimize --title <str>
maximize --title <str>
restore --title <str>
close-window --title <str>
window-rect --title <str>

# UI Automation (pywinauto)
find-element --title <str> --control_type <str> [--name <str>] [--auto_id <str>]
click-element ...
set-text --title <str> --control_type <str> --auto_id <str> --value <str>
element-text ...
wait-element ...

# 图像匹配 (OpenCV)
find-image <template_path> [--threshold <0-1>] [--region x,y,w,h]
click-image <template_path> [--threshold <0-1>]
wait-image <template_path> [--timeout <s>]
count-image <template_path> [--threshold <0-1>]

# OCR (需 Tesseract)
ocr [--region x,y,w,h]
ocr-words [--region x,y,w,h]

# 安全
emergency-stop
clear-stop
failsafe [on|off]
stop-status
```

</details>

---

## 安全机制

| 机制 | 说明 |
|------|------|
| **Emergency Stop** | 调用后所有操作命令被拒绝，直到 `clear-stop` |
| **Failsafe** | 鼠标快速甩到屏幕四角任一角，立即终止所有进行中的操作 |
| **坐标越界保护** | 所有鼠标移动前校验坐标在屏幕范围内 |

---

## 测试

本仓库包含 `TEST_PLAN.md`，覆盖 Lv1（冒烟）→ Lv5（安全+E2E）共 55+ 测试用例。

建议在新 WorkBuddy 会话中逐级执行，确认每级通过后再推送到 SkillHub。

---

## 技术栈

- **PyAutoGUI** — 鼠标/键盘底层控制
- **OpenCV (cv2)** — 图像模板匹配 + NMS 去重
- **pywinauto** — Windows UI Automation（控件树遍历）
- **mss** — 高性能截图（比 PyAutoGUI 快 3-5x）
- **pytesseract** — OCR（可选，需 Tesseract 二进制）
- **Pillow** — 图像处理

---

## License

MIT

---

## English

## What is this?

A Windows desktop automation skill for WorkBuddy that mirrors the "Computer Use" capability of OpenAI Codex / Anthropic Claude — but runs entirely on **your own Windows machine**, no remote VM, no per-token cost.

**Supports any Windows app**: Win32, WinForms, WPF, Qt, Electron, UWP.

### Features

- 📸 Screenshot (mss, 3-5x faster than PyAutoGUI)
- 🖱️ Mouse control (move, click, drag, scroll)
- ⌨️ Keyboard input (supports **Chinese** via clipboard)
- 🪟 Window management (list, activate, minimize, close)
- 🔍 UI Automation via pywinauto (find elements by name/automation id)
- 🖼️ OpenCV template matching with NMS (Non-Maximum Suppression)
- 🔤 OCR via Tesseract (optional)
- 🛡️ Safety: emergency stop + failsafe (mouse corner kill switch)

### Install

```bash
# Clone to WorkBuddy skills dir
git clone https://github.com/CarlosShao/workbuddy-computer-use.git \
  ~/.workbuddy/skills/workbuddy-computer-use/

# Enter skill directory and create isolated venv
cd ~/.workbuddy/skills/workbuddy-computer-use/
python -m venv .venv

# Install Python deps (Windows)
.venv/Scripts/pip.exe install pyautogui pywinauto opencv-python numpy mss pillow pytesseract

# Install Python deps (macOS / Linux)
.venv/bin/pip install pyautogui pywinauto opencv-python numpy mss pillow pytesseract
```

> **Note**: Requires Python >= 3.10. Use `python3` if `python` is not available.

### Quick Start

```
User: Open Notepad, type "hello WorkBuddy", and save as test.txt
```

WorkBuddy will auto-load this skill and execute the full desktop automation flow.

### Links

- GitHub: https://github.com/CarlosShao/workbuddy-computer-use
- Issues: https://github.com/CarlosShao/workbuddy-computer-use/issues
