t-guide# Qwen-Image-2.1 Prompt Guide — Agent Skills

[English](#english) 

Agent skills that teach coding assistants how to write effective prompts for **Qwen-Image-2.1**, Alibaba's open-weight image generation & editing model (released 2026-09-20). Available in English and Japanese — install the one matching your working language (or both).

---

## English

### What this is

Two self-contained agent skills, distilled from official Qwen sources (model cards, the official prompt rewriters PE-T2I/PE-I2I, GitHub READMEs) plus validated community findings:

- **`qwen-image-prompt-en/`** — English skill
- **`qwen-image-prompt-ja/`** — Japanese skill (identical structure)

Each skill covers:

- **Text-to-image**: the official two styles — compact 1–3-sentence prompts for simple subjects, and the long observational paragraph (~300–500 words) for posters, layouts, and infographics
- **Image editing**: attribute disentanglement, preserve clauses, chained small edits
- **In-image text rendering**: verbatim quoting rules, presentation control, the Japanese-text workaround (edit-tracing), anti-garbling settings
- **Multi-reference compositing**: `<image1>` tag conventions, canvas/donor roles, recipes for face swap, background replace, style transfer
- **Transparent (RGBA) output**, aspect ratio table (native 2K), negative prompts / true CFG / seeds / steps, minimal Diffusers & vLLM snippets
- **Failure patterns → fixes** and a curated, sourced example-prompt library

### Install

Copy the skill directory for your language into your agent's skills folder:

```bash
# ZCode / Claude-Code-compatible (~/.agents/skills is the cross-tool standard)
git clone https://github.com/kjranyone/qwen-image-2.1-prompt-guide
cp -r qwen-image-2.1-prompt-guide/skills/qwen-image-prompt-en ~/.agents/skills/
# Japanese version:
cp -r qwen-image-2.1-prompt-guide/skills/qwen-image-prompt-ja ~/.agents/skills/
```

Project-scoped installation works too — drop the directory into `<project>/.agents/skills/`. The skill triggers automatically when you ask for image generation/editing prompts involving Qwen image models; you can also force-load it with `/skill qwen-image-prompt-en <request>`. Compatible with ZCode, Claude Code, and any agent that discovers skills in `.agents/skills/`.

### Repository layout

```text
skills/
├── qwen-image-prompt-en/
│   ├── SKILL.md                  # core rules + routing (always loaded on trigger)
│   └── references/
│       ├── text-to-image.md      # official PE-T2I long-form style + compact style
│       ├── image-editing.md      # disentanglement, preserve clauses, <imageN> tags
│       ├── text-rendering.md     # quoting discipline, Japanese text workaround
│       ├── capabilities.md       # specs, ratios, CFG/negative/seed, snippets, license
│       └── examples.md           # sourced example-prompt library
research/
└── 2026-09-21-research-notes.md  # the underlying research (Japanese)
```

### License

- **This repository** — the skill files, documentation, and research notes — is released under the [MIT License](LICENSE).
- **The Qwen-Image-2.1 model** is a separate work under the **Qwen Research License Agreement**: free for research and evaluation (non-commercial) use only; commercial use requires a separate license from the Qwen team. This repository contains prompt-engineering guidance only and ships no model weights. See `references/capabilities.md` in either skill for details.

### Sources

Skill content is based on the official Qwen-Image-2.1 model card & README, the PE-T2I/PE-I2I prompt rewriter system prompts, the official Qwen-Image blog, and community evaluations (fal.ai, apiyi, Reddit, Zenn). Full source list with dates: [`skills/qwen-image-prompt-en/references/examples.md`](skills/qwen-image-prompt-en/references/examples.md) and [`research/2026-09-21-research-notes.md`](research/2026-09-21-research-notes.md).

---
