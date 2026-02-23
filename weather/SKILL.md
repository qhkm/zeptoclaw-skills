---
name: weather
description: Get current weather and forecasts without API keys.
metadata: {"zeptoclaw":{"emoji":"🌤️","requires":{"anyBins":["curl"]}}}
---

# Weather Skill

No API keys needed.

## wttr.in

Quick status:
```bash
curl -s "wttr.in/Kuala+Lumpur?format=3"
```

Detailed fields:
```bash
curl -s "wttr.in/Kuala+Lumpur?format=%l:+%c+%t+%h"
```

Tips:
- Replace spaces with `+`
- Add `?m` for metric format
- Add `?1` for today-only view
