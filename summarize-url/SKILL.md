---
name: summarize-url
version: 1.0.0
description: Fetch a URL and extract clean text content for summarization or analysis.
author: ZeptoClaw
license: MIT
tags:
  - web
  - scraping
  - summarization
  - research
env_needed:
  - name: BRAVE_API_KEY
    description: Brave Search API key for web search (optional, only needed for search-then-summarize)
    required: false
metadata: {"zeptoclaw":{"emoji":"🔗","requires":{"anyBins":["curl","python3"]}}}
---

# Summarize URL Skill

Fetch web pages and extract clean readable text, stripping ads, nav, and boilerplate.

## Setup

No required env vars. Optionally set Brave API key for the search-then-summarize workflow.

```bash
export BRAVE_API_KEY="BSAxxxx..."
```

## Fetch and Extract Text

```bash
# Fetch URL and strip HTML tags
URL="https://example.com/article"
curl -sL "$URL" \
  -H "User-Agent: Mozilla/5.0 (compatible; ZeptoClaw/1.0)" \
  | python3 -c "
import sys, re
html = sys.stdin.read()
# Remove scripts, styles, nav, footer
html = re.sub(r'<(script|style|nav|footer|header)[^>]*>.*?</\1>', '', html, flags=re.S|re.I)
# Remove all other tags
text = re.sub(r'<[^>]+>', ' ', html)
# Clean whitespace
text = re.sub(r'[ \t]+', ' ', text)
text = re.sub(r'\n{3,}', '\n\n', text)
# Decode HTML entities
import html as htmlmod
text = htmlmod.unescape(text)
print(text.strip()[:8000])
"
```

## Extract Article Content (Readability-style)

```bash
pip install readability-lxml 2>/dev/null

URL="https://example.com/blog/post"
curl -sL "$URL" \
  -H "User-Agent: Mozilla/5.0 (compatible; ZeptoClaw/1.0)" \
  | python3 -c "
import sys
from readability import Document
html = sys.stdin.read()
doc = Document(html)
print('TITLE:', doc.title())
print()
# Extract content text
content = doc.summary()
import re, html as h
text = re.sub(r'<[^>]+>', ' ', content)
text = h.unescape(re.sub(r'\s+', ' ', text))
print(text[:6000].strip())
"
```

## Summarize Multiple URLs

```bash
# urls.txt — one URL per line
while IFS= read -r url; do
  echo "=== $url ==="
  curl -sL "$url" \
    -H "User-Agent: Mozilla/5.0 (compatible; ZeptoClaw/1.0)" \
    --max-time 10 \
    | python3 -c "
import sys, re, html as h
txt = re.sub(r'<(script|style)[^>]*>.*?</\1>', '', sys.stdin.read(), flags=re.S|re.I)
txt = re.sub(r'<[^>]+>', ' ', txt)
txt = h.unescape(re.sub(r'\s+', ' ', txt)).strip()
print(txt[:2000])
" 2>/dev/null || echo "(failed to fetch)"
  echo
done < urls.txt
```

## Search Then Summarize (Brave)

```bash
QUERY="Lazada Malaysia seller fees 2026"
RESULTS=$(curl -s "https://api.search.brave.com/res/v1/web/search?q=${QUERY// /+}&count=3" \
  -H "Accept: application/json" \
  -H "X-Subscription-Token: $BRAVE_API_KEY" \
  | jq -r '.web.results[] | "URL: \(.url)\nTitle: \(.title)\nSnippet: \(.description)\n"')
echo "$RESULTS"

# Then fetch the top result for full content
TOP_URL=$(curl -s "https://api.search.brave.com/res/v1/web/search?q=${QUERY// /+}&count=1" \
  -H "Accept: application/json" \
  -H "X-Subscription-Token: $BRAVE_API_KEY" \
  | jq -r '.web.results[0].url')
curl -sL "$TOP_URL" -H "User-Agent: Mozilla/5.0" \
  | python3 -c "
import sys, re, html as h
txt = re.sub(r'<(script|style|nav|footer)[^>]*>.*?</\1>', '', sys.stdin.read(), flags=re.S|re.I)
txt = re.sub(r'<[^>]+>', ' ', txt)
print(h.unescape(re.sub(r'\s+', ' ', txt)).strip()[:4000])
"
```

## Extract JSON-LD Structured Data

```bash
URL="https://example.com/product/widget"
curl -sL "$URL" | python3 -c "
import sys, re, json
html = sys.stdin.read()
matches = re.findall(r'<script type=[\"\'']application/ld\+json[\"\''][^>]*>(.*?)</script>', html, re.S)
for m in matches:
    try:
        data = json.loads(m.strip())
        print(json.dumps(data, indent=2))
    except:
        pass
"
```

## Tips

- Add `--max-time 10` to curl to avoid hanging on slow sites
- Paywalled sites will return only the teaser paragraph — that's expected
- For JavaScript-rendered pages (SPAs), plain curl won't work — use ZeptoClaw's `web_fetch` tool instead which handles rendering
- Extract only the first 4000–8000 chars — most article content is in the first portion; LLMs don't need the full page
- Store extracted text to a file before passing to LLM: `... > /tmp/page.txt && cat /tmp/page.txt | zeptoclaw agent -m "Summarize this"`
- `readability-lxml` gives much cleaner extracts for news/blog sites than raw regex stripping
