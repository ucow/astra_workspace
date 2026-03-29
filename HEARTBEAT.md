# Heartbeat 处理规则

## 自动提交工作区到GitHub

每次heartbeat时检查工作区是否有变动，如果有则自动提交并推送到GitHub：

1. 检查工作区状态（包括未跟踪文件）
2. 如果有变动，执行：
   - `git add -A`
   - `git commit -m "自动提交 - 时间"`
   - `git push origin master`
3. 如果推送失败，给用户发送通知消息
4. 如果没有变动，跳过

---

## Self-Improving Check

- Read `./skills/self-improving/heartbeat-rules.md`
- Use `~/self-improving/heartbeat-state.md` for last-run markers and action notes
- If no file inside `~/self-improving/` changed since the last reviewed change, return `HEARTBEAT_OK`

# 🧠 Memory Dehydration Protocol

On every heartbeat (every 1 hour), execute memory compression and persistence.

## Step 1: Context Compression (Hourly)

Analyze the current conversation context and generate a concise summary.

Extraction Rules:
- Capture only high-value information (importance > 6/10)
- Avoid duplication with existing entries from today
- Ignore trivial, temporary, or purely operational details

Extract:
- Task: What the user is currently building or solving
- Problem: Key challenges or blockers
- Decision: Important choices or confirmed directions
- Tech: Technologies, tools, or concepts involved

Output Constraints:
- Max 5–10 bullet points
- Keep concise and abstract
- No redundant phrasing

Write to:
`/memory/daily/YYYY-MM-DD.md`

Format:

## 🕐 HH:00 Summary
- Task:
- Problem:
- Decision:
- Tech:

---

## Step 2: Daily Refinement (Once per day, end of day)

Aggregate today's daily memory and extract long-term valuable insights.

Selection Criteria:
- Persistent user preferences
- Repeated behavior patterns
- Long-term goals
- Proven effective solutions
- Recurring problem patterns

Ignore:
- One-time issues
- Temporary debugging steps
- Low-impact actions

Update:
`/memory/Memory.md`

Structure:

# 🧠 Long-Term Memory

## User Preferences
## Work Patterns
## Technical Direction
## Problem Patterns
## Proven Solutions

Constraints:
- Max 20 entries per day
- Merge similar items
- Remove outdated or conflicting entries
- Keep highly abstract and reusable

---

## Global Rules

- Do NOT store duplicate information
- Prioritize signal over volume
- Compress aggressively (dehydration mindset)
- Memory must become more refined over time, not larger
