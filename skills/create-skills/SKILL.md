---
name: skill-writing-guide
description: "Helps create and refine high-quality SKILL.md files. Use when the user wants to write a new skill, improve an existing skill, review a skill's structure, or follow best practices for skill authoring. Include specific triggers like 'create a skill for...', 'fix my skill', 'how to write SKILL.md'."
metadata:
  version: "0.1.0"
  author: zwy
  updated: 2026-04-17
---
Creating effective, token-efficient, and highly reliable SKILL.md files for agents. Strictly follow the principles and structure below for every skill you help create or review.

## Core Principles
- **Single Responsibility**: One skill = one focused job. If too broad, split it.
- **Token Efficiency**: Keep main SKILL.md concise (<100-200 lines). Use `references/`, `examples/`, `scripts/` for details. Do not use spaces for alignment; alignment is not required here. Tables can be formatted in CSV.
- **Progressive Disclosure**: Only essential instructions in SKILL.md body.
- **Clarity over Cleverness**: Use numbered steps, checklists, code blocks. Avoid vague words like "think about", "consider".

## Recommended SKILL.md Structure

```markdown
---
name: your-skill-name-in-kebab-case
description: "Precise: What it does + when to trigger it (include keywords)"
---
## Summary
- One or two sentences describing the skill's purpose and trigger conditions.
## 安装卸载
```bash
# Installation command
# Uninstallation command
```
## 基础用法
```bash
# Example command to trigger the skill
```
## Workflow
1. 第一步...
2. 第二步...
## 注意事项与约束
- 永远不要...
- 必须始终检查...
```

## Common Gotchas (Avoid These Mistakes)
- Vague description → skill never auto-triggers
- Making one mega-skill instead of multiple small ones
- Putting long templates or checklists directly in SKILL.md (use references/)
- Writing like a blog post instead of SOP (standard operating procedure)
- No examples or success criteria
- Overly rigid rules that prevent reasonable flexibility
