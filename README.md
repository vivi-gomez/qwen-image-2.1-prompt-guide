# Qwen-Image-2.1 Prompt Guide — Agent Skills

[English](#english) | [日本語](#日本語)

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
└── qwen-image-prompt-ja/         # same structure, Japanese
research/
└── 2026-09-21-research-notes.md  # the underlying research (Japanese)
```

### License

- **This repository** — the skill files, documentation, and research notes — is released under the [MIT License](LICENSE).
- **The Qwen-Image-2.1 model** is a separate work under the **Qwen Research License Agreement**: free for research and evaluation (non-commercial) use only; commercial use requires a separate license from the Qwen team. This repository contains prompt-engineering guidance only and ships no model weights. See `references/capabilities.md` in either skill for details.

### Sources

Skill content is based on the official Qwen-Image-2.1 model card & README, the PE-T2I/PE-I2I prompt rewriter system prompts, the official Qwen-Image blog, and community evaluations (fal.ai, apiyi, Reddit, Zenn). Full source list with dates: [`skills/qwen-image-prompt-en/references/examples.md`](skills/qwen-image-prompt-en/references/examples.md) and [`research/2026-09-21-research-notes.md`](research/2026-09-21-research-notes.md).

---

## 日本語

### これは何

**Qwen-Image-2.1**（アルババのオープンウェイト画像生成・編集モデル、2026-09-20公開）向けの効果的なプロンプトの書き方をコーディングエージェントに教えるAgent Skillsです。公式ソース（モデルカード、公式プロンプトリライターPE-T2I/PE-I2I、GitHub README）と検証済みのコミュニティ知見をもとに作成しています。

- **`qwen-image-prompt-en/`** — 英語版スキル
- **`qwen-image-prompt-ja/`** — 日本語版スキル（同一構成）

各スキルがカバーする内容:

- **テキストから画像生成**: 公式の2スタイル — 単純な被写体向けのコンパクトな1〜3文プロンプトと、ポスター・レイアウト・インフォグラフィック向けのロング観察描写（約300〜500語）
- **画像編集**: 属性のもつれ解除、保持宣言、小さい編集の連鎖
- **画像内テキスト描画**: 引用の規律、提示方法の制御、日本語テキストの回避策（編集トレース方式）、文字化け対策設定
- **複数参照画像の合成**: `<image1>` タグ規約、キャンバス/素材の役割、顔合成・背景差し替え・スタイル転送のレシピ
- **透明（RGBA）出力**、アスペクト比表（ネイティブ2K）、ネガティブプロンプト / true CFG / シード / ステップ数、Diffusers・vLLMの最小スニペット
- **失敗パターン→修正**と出典付きの動作例プロンプト集

### インストール

使いたい言語のスキルディレクトリを、エージェントのスキルフォルダにコピーします:

```bash
# ZCode / Claude Code 互換（~/.agents/skills がクロスツール標準）
git clone https://github.com/kjranyone/qwen-image-2.1-prompt-guide
cp -r qwen-image-2.1-prompt-guide/skills/qwen-image-prompt-ja ~/.agents/skills/
# 英語版:
cp -r qwen-image-2.1-prompt-guide/skills/qwen-image-prompt-en ~/.agents/skills/
```

プロジェクトスコープでのインストールも可能です — `<project>/.agents/skills/` にディレクトリを置いてください。Qwen画像モデルのプロンプトを頼むと自動でトリガーされます。`/skill qwen-image-prompt-ja <依頼>` で強制ロードもできます。ZCode、Claude Code、`.agents/skills/` を走査するエージェント全般で動作します。

### ライセンス

- **本リポジトリ**（スキルファイル・ドキュメント・調査ノート）は [MIT License](LICENSE) の下で公開します。
- **Qwen-Image-2.1モデル本体**は別個の作品で、**Qwen Research License Agreement**（無償利用は研究・評価目的の非商用のみ、商用はQwenチームとの個別ライセンスが必要）の下で配布されています。本リポジトリにはプロンプトエンジニアリングのガイドのみが含まれ、モデルの重みは同梱していません。詳細は各スキルの `references/capabilities.md` を参照してください。

### 情報源

スキルの内容は、Qwen-Image-2.1公式モデルカード・README、PE-T2I/PE-I2Iプロンプトリライターのシステムプロンプト、公式Qwen-Imageブログ、およびコミュニティの検証（fal.ai、apiyi、Reddit、Zenn）に基づきます。日付付きの完全な出典一覧は [`skills/qwen-image-prompt-ja/references/examples.md`](skills/qwen-image-prompt-ja/references/examples.md) と [`research/2026-09-21-research-notes.md`](research/2026-09-21-research-notes.md) にあります。
