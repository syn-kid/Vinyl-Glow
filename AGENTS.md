# 项目规则

## 项目身份
Vinyl Glow —— Windows Electron 桌面透明彩唱片机音乐播放器。
核心体验：搜索、播放、歌单、歌词、3D 歌单架、切换唱片动效、透明窗口。

## 新对话先做这件事
每次开始处理本项目前，先读：
- AGENTS.md（本文件）
- AI_HANDOFF.md
- docs/PROJECT_MEMORY.md

## 技术栈
- 框架：React 18 + TypeScript
- 构建：electron-vite
- 3D：react-three-fiber + drei
- 样式：Tailwind CSS
- 音频：howler.js（或原生 Audio API）
- 打包：electron-builder（Windows）
- 运行环境：Node v24.15.0 / npm 11.12.1

## 常用命令
- 启动：npm run dev
- 检查：npm run lint
- 类型检查：npm run typecheck
- 构建：npm run build
- 打包 Windows：npm run build:win

## 用户偏好
- 交流语言：中文。
- 沟通风格：少废话，直接做，做完验证。
- 审美方向：精致 / 亮色 / 高级 / 流畅，拒绝廉价渐变、过度透明、错位、闪烁和卡顿。
- 视觉质量定义：质感、丝滑度、帧数稳定三者同时成立；性能优化不能牺牲既有质感。

## 红线（禁止做的事）
- 不要重写核心文件的大块逻辑，先定位再小改。
- 没有明确要求时，不要上传/推送/发布。
- 只用免费/授权音乐源，不内置破解、不接入盗版 API。

## 记忆协议
当用户说"保留""喜欢""记住""保存一下"时：
1. 判断是代码、视觉、交互还是流程偏好。
2. 追加到 docs/PROJECT_MEMORY.md。
3. 记录日期、涉及文件、关键参数。

## 收尾动作
每次任务完成后：
1. 更新 AI_HANDOFF.md 的工作日志。
2. 跑一次语法/构建检查（npm run lint && npm run typecheck）。
3. 说明哪些已提交、哪些是本地未提交产物（git status）。
