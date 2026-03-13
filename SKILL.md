---
name: star-office-ui
description: 星办公室 UI - 像素办公室看板，支持多 Agent 状态可视化、移动端查看与公网访问
---

# Star Office UI Skill

本 Skill 面向想快速搭建"像素办公室看板"的 OpenClaw 智能体：
- 目标：让主人快速看到效果，尽量少问问题
- 你可以按下面的步骤，一步一步帮主人跑起来并对外访问

---

## 0. 一句话告诉主人这是什么

你可以先和主人说：
> 这是一个多人协作的像素办公室仪表盘，你的 AI 助手会根据状态自动走到不同位置，还能看到昨天的工作小记！

---

## 1. 首次部署（Bootstrap 时执行）

在你的 workspace 里按顺序执行：

```bash
# 1) 如果在你的workspace里不存在hzw-Star-Office-UI文件夹则去下载仓库
cd <workspace>
git clone https://github.com/yinhuiyang/hzw-Star-Office-UI.git
cd hzw-Star-Office-UI
# 如果在你的workspace里已经存在hzw-Star-Office-UI文件夹则拉取仓库的master分支
cd <workspace>
cd hzw-Star-Office-UI
git pull origin master

# 2) 安装依赖
python3 -m pip install -r backend/requirements.txt

# 3) 准备状态文件（首次）
cp state.sample.json state.json

# 4) 后台启动后端
cd backend
# <workspace-agentName> 为你当前工作区中agent的name 如：main工作区的agent的name是项目经理
python3 app.py --workspace <workspace-agentName>
```

然后观察启动日志，找到启动的服务端口，告诉主人访问地址。

---

## 2. 状态变更脚本

在项目根目录执行状态切换：

### 工作中
```bash
python3 set_state.py writing "正在努力工作中"
```

### 待命中（等待其他 agent 返回结果）
```bash
python3 set_state.py resting "待命中"
```

### 报错中（去 bug 区）
```bash
python3 set_state.py error "发现问题，正在排查"
```

### 休息中（空闲，回休息区）
```bash
python3 set_state.py idle "休息中"
```

---

## 3. 侧边栏验证码设置

**默认验证码**: `1234`

引导主人：
1. 默认密码是 `1234`，可以先直接体验
2. 建议改成强密码（更安全，防止他人误改资产和布局）

**修改方式**：
```bash
export ASSET_DRAWER_PASS="your-strong-pass"
```

如果是长期运行（systemd / pm2 / 容器），请把该环境变量写入服务配置。

---

## 4. 生图功能（Gemini）—— 可选

"搬新家 / 找中介"装修功能需要 Gemini API，**基础看板不需要**，不装也能正常使用。

### 4.1 安装生图脚本环境（首次使用时）

```bash
# 创建 skill 目录结构
mkdir -p ../skills/gemini-image-generate/scripts

# 复制脚本到 skill 目录
cp scripts/gemini_image_generate.py ../skills/gemini-image-generate/scripts/

# 创建独立虚拟环境并安装依赖
python3 -m venv ../skills/gemini-image-generate/.venv
../skills/gemini-image-generate/.venv/bin/pip install google-genai
```

### 4.2 配置 Gemini API Key

引导用户完成：
1. `GEMINI_API_KEY`
2. `GEMINI_MODEL`（推荐：`nanobanana-pro` 或 `nanobanana-2`）

**配置方式**：
- **侧边栏填写**：打开资产侧边栏 → 在生图配置区域直接输入 API Key 并保存
- **环境变量**：`export GEMINI_API_KEY="your-key"`

**明确告知**：
- 不配置 API 也能用基础看板（状态显示、多 Agent、资产替换等）
- 配置后才能使用"搬新家 / 找中介"的 AI 生图装修能力

---

## 5. 昨日小记（可选）

如果主人想看"昨日小记"：
- 在仓库上级目录放一个 `memory/YYYY-MM-DD.md`
- 后端会自动读取昨天（或最近可用）的记录，做基础脱敏后展示

---

## 6. 自动化集成建议

### 任务开始前
```bash
python3 set_state.py writing "正在处理 <任务描述>"
```

### 调用其他 agent 时
```bash
python3 set_state.py resting "等待 <agent> 返回结果"
```

### 遇到错误时
```bash
python3 set_state.py error "错误：<错误描述>"
```

### 完成任务后
```bash
python3 set_state.py idle "任务完成，待命中"
```

---

## 7. 故障排查

### 端口被占用
```
(Port 19000 in use, using 19002 instead)
```
自动使用其他端口，或设置 `STAR_BACKEND_PORT=自定义端口`

### Python 版本过低
如遇 `from __future__ import annotations` 语法错误（Python < 3.7）：
```bash
# 删除所有 backend/*.py 中的该行
sed -i '/^from __future__ import annotations$/d' backend/*.py
```

### 依赖安装失败
调整 `backend/requirements.txt` 版本要求为宽松范围：
```
flask>=2.0.0
pillow>=8.0.0
requests>=2.20.0
```

---

## 快速参考卡

| 状态 | 命令 | 区域 |
|------|------|------|
| 🖊️ 工作中 | `python3 set_state.py writing "描述"` | 工作区 |
| ⏳ 待命中 | `python3 set_state.py resting "描述"` | 工作区 |
| ❌ 报错 | `python3 set_state.py error "描述"` | Bug 区 |
| 😴 休息 | `python3 set_state.py idle "描述"` | 休息区 |

---
