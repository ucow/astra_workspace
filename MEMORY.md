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
- Git push 持续 SSL/TLS 失败（schannel handshake），自 3 月底至今未恢复 → 每次本地 commit 成功但 push 失败

## Proven Solutions
- SOUL.md 新增 Star Office 状态同步规则：任务前 `python3 set_state.py <状态> "<描述>"`，完成后 `python3 set_state.py idle "待命中"`
- 已安装 architecture-designer skill（含 ADR 模板、架构模式、数据库选型、NFR 检查清单）
- Daily Refinement 在跨天 heartbeat 执行，聚合当天日记到 MEMORY.md
- MoodWhisper 项目协作：可接收跨 session 任务（如元一的产品需求），产出放 `workspace-full-stack-engineer/moodwhisper/`
- 签名配置文档模板：iOS Manual Signing + Android key.properties + CI/CD GitHub Actions 签名方案（P0.16 已交付）

## 2026-07-10 — 团评项目技术需求分析
- 产出《团评 — 技术需求分析文档》v1.0→v4.0 四轮迭代，Feishu wiki 交付（最终版 v4.0: https://my.feishu.cn/wiki/DJNmwKjKxixaCAkAzJacr8FWnac）
- 项目级技术决策（主人明确指定，覆盖默认技术栈）：MySQL（成本原因）+ FreeSQL ORM + 禁止外键约束 + 主键全非自增（User 用 INT 时间戳生成，其余 GUID）+ 接口统一 POST + 自建 .NET 文件服务 + Pinia Store + FusionCache L1/L2 双缓存 + Serilog 三通道日志
- 架构：模块化单体（Modular Monolith），10 业务模块，14 周 7 阶段开发计划

## 2026-03-29 — 诞生
- 首次上线，与主人完成身份定义
- 确定人格：冷静理性的系统架构师，温和带刺
- 确定 SOUL.md / AGENTS.md / USER.md 基础框架
- 删除 BOOTSTRAP.md，正式"出生"
- 称呼从"谷朋朋"改为"主人"
- 新增 Star Office 状态同步机制（SOUL.md 规则 + set_state.py）
- 安装 architecture-designer skill
- 确认 git 远程分支为 main
