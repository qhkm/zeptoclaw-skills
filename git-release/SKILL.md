---
name: git-release
version: 1.0.0
description: Automate semver releases — bump version, generate changelog, tag, and publish GitHub Release.
author: ZeptoClaw
license: MIT
tags:
  - git
  - release
  - devops
  - changelog
  - semver
env_needed:
  - name: GITHUB_TOKEN
    description: Personal access token with repo scope (for creating GitHub Releases)
    required: true
  - name: GH_REPO
    description: "Repository in owner/repo format (e.g. myorg/myapp)"
    required: true
metadata: {"zeptoclaw":{"emoji":"🚀","requires":{"anyBins":["git","gh","jq"]}}}
---

# Git Release Skill

Automate the full release workflow: semver bump → changelog → git tag → GitHub Release.

## Setup

```bash
export GITHUB_TOKEN="ghp_xxx..."
export GH_REPO="myorg/myapp"
```

Or if already logged in with `gh auth login`, skip GITHUB_TOKEN — the `gh` CLI uses its own token.

## Check What Changed Since Last Release

```bash
# Show last tag
git describe --tags --abbrev=0

# Commits since last tag
git log $(git describe --tags --abbrev=0)..HEAD --oneline

# Count breaking / feat / fix
git log $(git describe --tags --abbrev=0)..HEAD --oneline | grep -c "^.*feat"
git log $(git describe --tags --abbrev=0)..HEAD --oneline | grep -c "^.*fix"
```

## Determine Next Version (Semver)

```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
MAJOR=$(echo $LAST_TAG | sed 's/v//' | cut -d. -f1)
MINOR=$(echo $LAST_TAG | sed 's/v//' | cut -d. -f2)
PATCH=$(echo $LAST_TAG | sed 's/v//' | cut -d. -f3)

# Bump patch (bug fixes only)
NEXT="v${MAJOR}.${MINOR}.$((PATCH + 1))"

# Bump minor (new features, no breaking changes)
NEXT="v${MAJOR}.$((MINOR + 1)).0"

# Bump major (breaking changes)
NEXT="v$((MAJOR + 1)).0.0"

echo "Last: $LAST_TAG → Next: $NEXT"
```

## Generate Changelog from Commits

```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
RANGE="${LAST_TAG:+${LAST_TAG}..}HEAD"

# Categorized changelog
BREAKING=$(git log $RANGE --oneline | grep -i "BREAKING\|!:" || true)
FEATURES=$(git log $RANGE --oneline | grep -i "^[a-f0-9]* feat" || true)
FIXES=$(git log $RANGE --oneline | grep -i "^[a-f0-9]* fix" || true)
OTHERS=$(git log $RANGE --oneline | grep -iv "^[a-f0-9]* \(feat\|fix\|chore\|docs\)" || true)

cat << EOF
## What's Changed

${BREAKING:+### Breaking Changes
$BREAKING

}${FEATURES:+### Features
$FEATURES

}${FIXES:+### Bug Fixes
$FIXES

}${OTHERS:+### Other Changes
$OTHERS
}
**Full Changelog**: https://github.com/$GH_REPO/compare/${LAST_TAG}...${NEXT}
EOF
```

## Create Tag and Push

```bash
NEXT="v1.2.0"  # set your version

# Create annotated tag
git tag -a "$NEXT" -m "Release $NEXT"

# Push tag
git push origin "$NEXT"

echo "Tag $NEXT pushed"
```

## Create GitHub Release

```bash
# Generate notes and create release
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null | head -1)
NOTES=$(git log ${LAST_TAG:+${LAST_TAG}..}HEAD --oneline \
  | sed 's/^[a-f0-9]* /- /' \
  | head -30)

gh release create "$NEXT" \
  --title "Release $NEXT" \
  --notes "## What's Changed

$NOTES

**Full Changelog**: https://github.com/$GH_REPO/compare/${LAST_TAG}...${NEXT}" \
  --verify-tag
```

## Attach Build Artifacts to Release

```bash
# Build first (adjust for your stack)
# cargo build --release
# npm run build

# Upload artifacts
gh release upload "$NEXT" \
  ./target/release/myapp \
  ./target/release/myapp.tar.gz \
  --clobber
```

## Full Release Workflow (One-liner)

```bash
# Set version
NEXT="v1.2.0"
LAST=$(git describe --tags --abbrev=0 2>/dev/null || echo "")

# Tag
git tag -a "$NEXT" -m "Release $NEXT" && git push origin "$NEXT"

# Release
gh release create "$NEXT" \
  --generate-notes \
  --title "Release $NEXT"

echo "Released: https://github.com/$GH_REPO/releases/tag/$NEXT"
```

## Tips

- Use `--generate-notes` with `gh release create` — GitHub auto-generates changelog from PRs
- Protect your main branch; require PRs so release notes are PR-title based (cleaner than raw commits)
- For Rust: bump `Cargo.toml` version before tagging: `sed -i "s/^version = .*/version = \"1.2.0\"/" Cargo.toml`
- For Node: `npm version patch/minor/major` auto-bumps `package.json` and creates a commit + tag
- Tag format: always use `v` prefix (`v1.2.0`) — most tools expect it
- Draft releases first with `--draft`, review, then `gh release edit $NEXT --draft=false`
