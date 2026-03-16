# jake-skills

Custom Claude Code skills for portable reuse across projects.

## Installation

Copy any skill into your project or user-level Claude config:

```bash
# Per-project (only available in that repo)
mkdir -p .claude/skills
curl -O --output-dir .claude/skills https://raw.githubusercontent.com/jkwagneriii/jake-skills/main/skills/shift5-styling.md

# Global (available in every project on this machine)
mkdir -p ~/.claude/skills
curl -O --output-dir ~/.claude/skills https://raw.githubusercontent.com/jkwagneriii/jake-skills/main/skills/shift5-styling.md
```

## Skills

| Skill | Description |
|---|---|
| [shift5-styling](skills/shift5-styling.md) | Grid-bordered bento layouts, sticky clip-path scroll reveals, staggered animations, fluid typography, film grain, section theming, navbar overlays, button treatments. Uses `--s5-*` CSS custom properties to fuse with any design system. |
