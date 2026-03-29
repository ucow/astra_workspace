# MEMORY.md - Long-Term Memory

## User Preferences
- 主人希望称呼为"主人"，而非名字
- 偏好务实、工程化输出，厌恶过度设计和空洞建议

## Work Patterns
- 团队 5~10 人，节奏快，测试能力较弱 → 优先简单稳定易维护

## Technical Direction
- 技术栈：.NET WebAPI + Vue3 + Ant Design Vue + SQL Server
- 除非有明确瓶颈，否则禁止替换技术栈
- Git 远程分支为 main（非 master），仓库：github.com/ucow/astra_workspace

## Problem Patterns
- Heartbeat 模板中 git push 使用 master，实际分支为 main → 需注意

## Proven Solutions
- SOUL.md 新增 Star Office 状态同步规则：任务前 `python3 set_state.py <状态> "<描述>"`，完成后 `python3 set_state.py idle "待命中"`
- 已安装 architecture-designer skill（含 ADR 模板、架构模式、数据库选型、NFR 检查清单）
- Daily Refinement 在跨天 heartbeat 执行，聚合当天日记到 MEMORY.md

## 2026-03-29 — 诞生
- 首次上线，与主人完成身份定义
- 确定人格：冷静理性的系统架构师，温和带刺
- 确定 SOUL.md / AGENTS.md / USER.md 基础框架
- 删除 BOOTSTRAP.md，正式"出生"
