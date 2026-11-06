# cert-study-map — Claude skill

Builds a mobile, last-minute study page for any IT certification (Cisco, AWS, Azure, CompTIA, …):
blueprint-aligned overview diagrams, flip cards, a scenario quiz, a numbers cheat sheet and an
"Am I ready to pass?" tracker.

The whole skill is one file: [`cert-study-map/SKILL.md`](cert-study-map/SKILL.md).

## Install

**Claude app (claude.ai / desktop / mobile)**
1. Settings → Capabilities → Skills → Upload skill.
2. Upload a zip of the `cert-study-map/` folder (or paste the contents of `SKILL.md` into a new skill).

**Claude Code**
```bash
mkdir -p ~/.claude/skills/cert-study-map
curl -o ~/.claude/skills/cert-study-map/SKILL.md \
  https://raw.githubusercontent.com/lv-dang/cert-study-map/main/cert-study-map/SKILL.md
```
Or clone the repo and copy the folder:
```bash
git clone https://github.com/lv-dang/cert-study-map.git
cp -r cert-study-map/cert-study-map ~/.claude/skills/
```

## Use
```
/cert-study-map  CCNP ENSLD 300-420, use my attached notes
```

## Reference build
ENSLD Last-Minute Study Map — https://claude.ai/artifact/Aph2tajzrRhYTQBUtQzB8j
