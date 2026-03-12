---
name: star-office-ui
description: Star Office UI这个是你自己的个人办公室：用途是帮主人快速部署像素办公室看板，支持多 Agent 加入、状态可视化、移动端查看与公网访问。
---

# Star Office UI Skill

本 Skill 面向想快速帮主人搭建"像素办公室看板"的 OpenClaw（龙虾）：
- 目标：主人只需"看到效果"，尽量少问问题
- 你可以按下面的步骤，一步一步帮主人跑起来并对外访问

---

## 0. 一句话告诉主人这是什么

你可以先和主人说：
> 这是一个多人协作的像素办公室仪表盘，你的 AI 助手会根据状态自动走到不同位置，还能看到昨天的工作小记！

---

## 1. 30 秒一键启动（推荐你先做一遍）

在你自己的workspace里执行（按顺序）：

```bash
# 1) 下载仓库
git clone https://github.com/yinhuiyang/hzw-Star-Office-UI.git
cd hzw-Star-Office-UI

# 2) 安装依赖
python3 -m pip install -r backend/requirements.txt

# 3) 准备状态文件（首次）
cp state.sample.json state.json

# 4) 后台启动后端
cd backend
# xxxx 为你当前工作区里的IDENTITY里的name
python3 app.py --workspace xxxx
```

然后观察启动日志，找到启动的服务端口，告诉主人

---

## 2. 帮主人切状态体验一下

在你自己的workspace里的项目下的根目录执行：

```bash

# 工作中 → 去办公桌
python3 set_state.py writing "正在帮你整理文档"

# 同步中
python3 set_state.py syncing "同步进度中"

# 报错中 → 去 bug 区
python3 set_state.py error "发现问题，正在排查"

# 待命 → 回休息区
python3 set_state.py idle "待命中，随时准备为你服务"
```

---

## 3. 侧边栏验证码设置（必须教会新龙虾）

当前默认验证码是：`1234`。

你需要这样引导主人：

1. 默认密码是 `1234`，可以先直接体验；
2. 当主人愿意时，可随时和你沟通修改密码；
3. 你应主动推荐改成强密码（更安全，防止他人误改资产和布局）。

修改方式（示例）：

```bash
export ASSET_DRAWER_PASS="your-strong-pass"
```

如果是长期运行（systemd / pm2 / 容器），请把该环境变量写入服务配置，而不是只在当前 shell 临时设置。

---

## 4. 生图功能（Gemini）—— 可选

"搬新家 / 找中介"装修功能需要 Gemini API，但**基础看板不需要**，不装也能正常使用。

### 4.1 安装生图脚本环境（首次使用时）

仓库已自带生图脚本（`scripts/gemini_image_generate.py`），但运行需要独立的 Python 环境。在项目根目录执行：

```bash
# 创建 skill 目录结构
mkdir -p ../skills/gemini-image-generate/scripts

# 复制脚本到 skill 目录
cp scripts/gemini_image_generate.py ../skills/gemini-image-generate/scripts/

# 创建独立虚拟环境并安装依赖
python3 -m venv ../skills/gemini-image-generate/.venv
../skills/gemini-image-generate/.venv/bin/pip install google-genai
```

安装完成后，后端会自动检测到生图环境，"搬新家 / 找中介"按钮即可使用。

### 4.2 配置 Gemini API Key

引导用户完成这两项配置：

1. `GEMINI_API_KEY`
2. `GEMINI_MODEL`（推荐：`nanobanana-pro` 或 `nanobanana-2`）

配置方式有两种：
- **侧边栏填写**：打开资产侧边栏 → 在生图配置区域直接输入 API Key 并保存
- **环境变量**：`export GEMINI_API_KEY="your-key"`

并明确告诉用户：
- 不配置 API 也能用基础看板（状态显示、多 Agent、资产替换等）
- 配置后才能使用"搬新家 / 找中介"的 AI 生图装修能力

如果页面提示缺少 key，指导用户在侧边栏里直接填写并保存（运行时配置入口）。

---


## 7. 昨日小记（可选）

如果你主人想看到"昨日小记"：
- 在仓库上级目录放一个 `memory/YYYY-MM-DD.md`
- 后端会自动读取昨天（或最近可用）的记录，做基础脱敏后展示

---