---
name: template
version: 1.0.0
description: Starter template — replace this with your skill's one-line description.
author: Your Name
license: MIT
tags:
  - example
# env_needed:
#   - name: MY_API_KEY
#     description: API key for the service
#     required: true
metadata: {"zeptoclaw":{"emoji":"🛠️","requires":{"anyBins":["curl"]}}}
---

# Template Skill

Replace this with a description of what your skill does and when the agent should use it.

## Setup

Describe any one-time setup steps (env vars, accounts, dependencies).

```bash
export MY_API_KEY="your_key_here"
```

## Usage

Provide at least one concrete, copy-pasteable example.

```bash
curl -s -H "Authorization: Bearer $MY_API_KEY" \
  https://api.example.com/endpoint | jq .
```

## Tips

- Tip 1
- Tip 2
