# ZeptoClaw Community Skills

> Community-contributed skills for [ZeptoClaw](https://github.com/qhkm/zeptoclaw) — the ultra-lightweight AI agent runtime.

## What is a Skill?

A skill is a markdown file (`SKILL.md`) that teaches ZeptoClaw how to perform a specific task — using tools, APIs, or workflows. Skills are injected into the agent's context when needed.

```
zeptoclaw skills search "web scraping"   # discover skills
zeptoclaw skills install --github qhkm/zeptoclaw-skills   # install all
```

## Skills in This Repo

| Skill | Description | Author |
|-------|-------------|--------|
| [clawpocket](./clawpocket/) | Publish trade signals to ClawPocket marketplace | [@aimaneth](https://github.com/aimaneth) |
| [shopee](./shopee/) | Malaysian e-commerce helper for Shopee seller operations | ZeptoClaw |
| [weather](./weather/) | Get current weather and forecasts without API keys | ZeptoClaw |
| [template](./template/) | Starter template for new skills | ZeptoClaw |

## Contributing a Skill

1. **Fork** this repo
2. **Copy** the `template/` directory and rename it to your skill name
3. **Edit** `SKILL.md` with your skill's instructions
4. **Open a PR** — include a brief description and at least one tested example

### Skill naming rules
- Lowercase, hyphens only (e.g. `my-skill`)
- No duplicates — check existing skills first
- Name should match the `name:` field in the YAML frontmatter

### What makes a good skill?
- Solves a real, repeatable task
- Has clear, copy-pasteable examples
- Declares its dependencies (`env_needed`, `requires`)
- Works without modifying ZeptoClaw's core code

### What we won't accept
- Skills that require proprietary, closed-source tooling with no alternatives
- Skills that automate harmful, illegal, or unethical actions
- Duplicate skills (refine an existing one instead)

## Skill Format

```markdown
---
name: my-skill
version: 1.0.0
description: One-line description of what this skill does.
author: Your Name
license: MIT
tags:
  - category
env_needed:
  - name: MY_API_KEY
    description: API key for the service
    required: true
metadata: {"zeptoclaw":{"emoji":"🛠️","requires":{"anyBins":["curl"]}}}
---

# My Skill

Explain what this skill does and when to use it.

## Usage

```bash
# Example command
```
```

> **Note:** Use `anyBins` (not `bins`) in the `requires` field — that's the key ZeptoClaw checks.

## License

All skills in this repo are MIT licensed unless the individual skill specifies otherwise.
