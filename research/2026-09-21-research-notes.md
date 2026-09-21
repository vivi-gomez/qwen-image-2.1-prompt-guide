# Qwen-Image-2.1 プロンプトガイドスキル — 調査ノート（2026-09-21）

プロンプトガイドAgent Skill作成のための一次調査まとめ。調査日: 2026-09-21（モデル公開翌日）。

---

## 1. モデル概要

- **正式名称**: Qwen-Image-2.1（表記ゆれ: "Qwen Image 2.1" / 「千问图像2.1」）
- **開発元**: Alibaba Qwen チーム（杭州通义实验室）
- **発表日**: **2026-09-20**（GitHub News 表記、HFライセンス文書も同日）
- **位置づけ**: 「Qwen現最強のオープンソース画像生成モデル」。生成と編集を1モデルに統合（"小模强效，创改一体"）
- **系譜**: Qwen-Image (2025-08, 20B, Apache 2.0) → Edit-2509 (2025-09) → Edit-2511 / Qwen-Image-2512 / Layered (2025-12) → Qwen-Image-2.0 (2026-02, API専用) → Qwen-Image-3.0/3.0-Pro (2026-07, API専用・クローズド) → **Qwen-Image-2.1 (2026-09-20, 7B, オープンウェイト)**

### 重要な注意点
- ナンバリングが直感的でない: 3.0の方が新しいが非公開。オープンウェイト系列の最新が2.1
- **ライセンスがApache 2.0ではなく「Qwen Research License Agreement」**（研究・評価目的の非商用のみ無償。商用は model-business@notice.qwencloud.com へ個別交渉。再配布時はライセンス同梱・帰属表示必須）
- Alibaba Cloud 百炼APIには 2026-09-21時点で "qwen-image-2.1" は未掲載（リリース直後のため）

## 2. バリエーションと提供形態

| モデル | 内容 | パラメータ |
|---|---|---|
| Qwen/Qwen-Image-2.1 | 本体。生成＋編集の統一モデル | 7B |
| Qwen/Qwen-Image-2.1-PE-T2I | 公式プロンプトリライター（生成用）。短い指示→詳細英語プロンプト＋`wh_ratio` | 9.4B (Qwen3.5-VL 9Bベース) |
| Qwen/Qwen-Image-2.1-PE-I2I | 公式プロンプトリライター（編集用）。曖昧な指示→厳密な編集ディレクティブ | 同上 |
| Comfy-Org/Qwen-Image-2.1 | ComfyUI用重みパック（Day-0対応） | — |

- Day-0対応フレームワーク: **Diffusers（推奨）**、ComfyUI、vLLM-Omni、SGLang、LightX2V
- Turbo/蒸留版は現時点でなし
- デモ: HF Spaces（Qwen/Qwen-Image-2.1）

## 3. 仕様・能力

### アーキテクチャ
- 32層 Single-Stream DiT（最適化されたMMDiT）、7B
- テキストエンコーダ: Qwen3-VL 8B / VAE: 64ch RGBA・16×圧縮
- Flow Matching（Euler＋dynamic shifting）、Prefix KV Cache再利用、混合粒度注意

### 機能（前世代からの新機能）
1. **生成＋編集の統一**（Qwen-ImageとQwen-Image-Editが1つに）
2. **ネイティブRGBA透明画像**（透明生成・透明レイヤー編集・被写体抽出）
3. **最大10枚の参照画像入力**（複数人物合成、キャラ・商品・背景・スタイル参照。ComfyUIノードは16枚まで）
4. **局所編集**: 円を描く・ペイント注釈・マスクで編集範囲指定（旧Edit系のネイティブControlNet相当はこの形式に置き換え）
5. **人物・商品のアイデンティティ保持**
6. **ネイティブ2K・高精細テキストレンダリング**（ポスター・タイポグラフィ・インフォグラフィック・パノラマ・絵コンテ）

### アスペクト比（幅×高さ・2K）
| 比率 | 解像度 |
|---|---|
| 1:1 | 2048×2048 |
| 4:3 / 3:4 | 2400×1792 / 1792×2400 |
| 3:2 / 2:3 | 2528×1696 / 1696×2528 |
| 16:9 / 9:16 | 2752×1536 / 1536×2752 |

- デフォルト 2048×2048、40推論ステップ
- PEリライターは上記7種に加え 2:1, 21:9, 9:21, 4:5, 3:1, 5:4, 1:3, 18:39, 9:20, 7:3, 9:5, 5:7 等を意味論的に選択し、パイプライン側で最も近い解像度にマップ
- **diffusersは `negative_prompt` と `true_cfg_scale`（デフォルト4.0）をサポート**（true_cfg>1＋ネガ指定で有効化）。SGLang/vLLMの蒸留運用例は guidance 1 でネガなし

---

## 4. 公式プロンプトガイドの要点

単一の「プロンプトガイド」文書は存在しない。公式の指針は以下に分散:
1. GitHub READMEの「Prompt Rewriting」セクション
2. **PE-T2I / PE-I2I 各リポジトリ同梱の `system_prompt.txt`（事実上の公式プロンプトガイド本体）**
3. モデルカードの透明画像テンプレート

### (A) 生成（T2I）— PE-T2I システムプロンプトから

**基本方針**: 完成した画像を「観察者」の視点で描写する、**1つの長い英語段落（約20文・400〜500語）**。短い指示も長い指示も同じ分量に拡張される（短い指示=大部分を補完して発明）。

8ステップ構成:
1. **固定要素と自由要素を分離**: ユーザー指定のテキスト文字列・物体名・個数・色・位置・比率は**一字一句そのまま維持**。「使用上の注記」（"4Kでノイズなし"等）は描写に反映するが文言としてエコーしない
2. **フレーム決定**: 比率は `wh_ratio` フィールドのみに記載し、**説明文中に比率・解像度・ピクセル数を書かない**。デフォルトは横物3:2・縦物2:3。1:1（バッジ・アイコン）、16:9（シネマ・プレゼン）、9:16（スマホ・縦バナー）等は意味論で選択
3. **冒頭文（約20語）**: 「The image is a ⟨縦/横/正方形⟩ ⟨スタイル⟩ ⟨写真・ポスター・イラスト…⟩ of ⟨被写体⟩, ⟨背景とパレット⟩.」— メディア名詞は省略不可、スタイル語はここで一度だけ命名
4. **インベントリ**: 全要素にフレーム内位置（upper-left, across the top, lower-third, in the centre…）を割り当て。**位置フレーズ8〜14個（目安10個）**、四隅・端・中央まで満遍なく
5. **フレームウォーク**: レイアウト画像なら「背景→最上帯→本体（左→中央→右）→最下帯」。単一被写体なら「背景→配置→頭・顔→体・衣装→手持ち物→縁」。**文の約1/3は位置フレーズで開始**
6. **テキスト設定**: 読み取れる文字はすべて読み順に `a bold black headline across the top reads "…"` の形で**直引用符**内に元スクリプトのまま（中国語は中国語のまま）記述。ウェイト・色・大小も指定。読ませない文字は "blurred, indistinct, too small to read"。チャートの軸・目盛り・凡例・セル値も書き出す。テキストなし画像は約3割あり、看板をでっち上げない
7. **ライティング専用の一文**: "The lighting is …"（光源・方向・質、影とハイライト）
8. **全体構図で締め**: "The overall composition ⟨is/uses/feels⟩ …" **1文だけ**

**文体ルール**:
- 現在形・三人称・宣言文。「you」「create」「make sure」禁止
- **品質ブースター禁止**（"masterpiece", "8K", "highly detailed", "award-winning"）
- 不確実なものはヘッジ（"appears to be"）。ユーザー固定要素にのみ断定
- **色は修飾語付き**（deep navy, muted olive, pale cream）。Hexはユーザー指定時のみ
- **材質まで書く**（brushed metal, matte plastic, frosted glass, weathered wood）
- 「いくつかのアイテム」等の要約禁止。**列挙する**。小さい個数は単語で（three, five, twelve）
- 人物は観察可能な表面で記述。**年齢は数値でなく人生段階**（a young adult, in her thirties）
- ブランド名でなくクラスで（a silver laptop, a mirrorless camera）
- 写真・デザイン語彙歓迎（shallow depth of field, bokeh, backlit, negative space）
- 物理的整合性（影は光と逆方向、反射の一貫性）
- **描写言語は常に英語**（依頼言語に関わらず）。画像内テキストのみ元スクリプト維持
- 出力: `{"rewritten_prompt": "<描写>", "wh_ratio": "<例: 3:2>"}`

### (B) 編集（I2I）— PE-I2I システムプロンプトから

- **統轄原理「属性のもつれ解除（Attribute Disentanglement）」**: 指名した属性のみ強く明確に編集し、他は入力画像の忠実度で保持。失敗モードは「漏れ」と「過小編集」の対称的2つ。**保持は内容をロックするもので編集強度を抑えるものではない**
- **保持対象は型・位置・役割で命名し、外観を再描写しない**（再描写すると生成指示として読まれてドリフト）。包括的保持節1つを優先
- **アイデンティティは最難の不変量**: 顔・アクセサリ・製品意匠・レンダリング媒体は明示的に狙わない限り全編集で維持。参照画像由来のアイデンティティは**言葉で描述せず画像タグで指す**
- **画像内テキストはリテラル**: 読める文字は出力に現れるなら全要素を引用符付きで正確にコミット。省略・要約禁止。読めない文字は追加しない
- **複数画像参照の必須ルール**: N≥2では `<image1>`, `<image2>` …のタグ参照が**必須**（「图1」「the first image」等の自然言語参照は禁止）。単一画像ではタグを使わない。各画像の役割（キャンバス/素材提供元）を明示
- **言語決定の2層化**: (A) 説明文の言語（中国語指示→中国語、英語→英語、その他→英語）と (B) **画像内に描画されるテキストの言語**（①ユーザー明示指定＞②入力画像のテキストの支配言語＞③指示言語）を混同しない。描画テキストは**モノリンガル必須**（混在禁止）
- **出力サイズ**: `wh_ratio` と `ratio_follow`（"<image1>" 等の出力追従先）は排他。デフォルトは入力画像の比率に追従。「新規シーン生成」のみ意味論的に選択。比率キーワード対応表: 正方形/头像→1:1、横版/PPT→16:9、海报→2:3、证件照/小红书→3:4、全景→2:1、名片→9:5、A4→5:7/7:5、iPhone画面→18:39、Android→9:20、cinemascope→21:9。**"2K/4K/8K" は品質記述子であり比率判定に使わない**
- **書式**: 改行なし単一段落。画像に描画する文字のみ二重引用符。**比率・解像度情報をプロンプト文中に含めない**。肯定形で記述（「禁止改变背景」でなく「保持背景不变」）。決定的に（ヘッジ禁止）

### (C) 透明画像（RGBA）公式テンプレート

> `This is an RGBA image with transparency. <your description>. The image has alpha channel and the background is transparent.`

### (D) 公式推奨ワークフロー

短いプロンプトはそのまま書かず、**PEリライターを通して詳細化してから QwenImage21Pipeline に渡す**（README「Prompt Rewriting」推奨）。リライター出力の `wh_ratio` を解像度表にマップし40ステップで生成。

---

## 5. コミュニティ知見（旧世代含む・適用バージョン明記）

### 構造
- **自然文推奨、SD式タグ羅列・`(red hair:1.5)` 重み付け構文は不可**（全系）→ "with vibrant, striking red hair" で強調
- MMDiTは**トークン位置と具体性に重み**→被写体を文頭にFront-load。順序: Subject → Style → Details → Composition → Lighting（fal.ai, 2512）
- 実測では構造化（"Subject: ..."）より**物語調の自然文**が高得点。正順で95%被写体明確（apiyi, 2512）
- 長さ: **1〜3文が最適**の実測（31語>82語、生成も速い）。ポートレートは英語200語以内（公式ツール）

### テキスト描画
- **描きたい文字列は必ず二重引用符**（公式・コミュニティ全系一致）
- 引用符のみで正確率85%、+CFG引き上げ+ステップ増で96%（ベースライン65%）
- **大文字小写・句読点・改行・縦書き/横書きまで忠実に転写**される。プロンプト側の表記に従う
- 書体・色・サイズ・呈示方式（ネオン/LED/印刷/刺繍/グラフィティ）まで指定
- 複数テキストブロックは**各行・位置を個別に記述**
- 暗黙の文字（「リストを表示」）は失敗。**具体文字列を提示**。締めに "No other text appears in the image." で混入防止
- 中国語描画が最強（94.1）＞英語＞**日本語はT2I直接生成では崩れやすい**。回避策: 白地黒字テキスト画像を入力してEditで「忠実になぞる」指示（Zenn, 2509）

### 編集
- 基本は**短い命令文**: "Change the background to a sunset beach"
- **編集対象+保持対象をセットで**: "Replace X with Y. Keep original font, size, color, and perspective. Do not alter background."
- **1回に詰め込まず2〜3の小さい編集に分割**してチェーン
- 汎用キーワード: Replace X with Y / Add X / Remove X / Leave everything else unchanged / Rotate to show the back

### 複数参照
- 公式例は**空間的言語**で配置指定: "The magician bear is on the left, the alchemist bear is on the right, facing each other…"
- 実用上「1枚目の部屋に…2枚目の画像で…」の番号+場所併記も機能（Zenn）
- ※2.1のPE-I2Iは `<image1>` タグ参照を必須化（上記(B)）— 新公式規約

### カメラ・ライティング
- "shot on Canon EOS R5, 85mm f/1.4 lens" + "professional photography, RAW format"
- "Relight the scene with a warm key light from the right and cool rim light from the back. Keep pose and background unchanged."
- 初代公式サフィックス: `, Ultra HD, 4K, cinematic composition.`

### 失敗パターンと回避策
| 失敗パターン | 対象ver | 回避策 |
|---|---|---|
| タグ羅列・重み構文が効かない | 全系 | 自然文で強調 |
| 被写体が曖昧 | 全系 | 被写体を文頭に、具体性を上げる |
| 冗長で優先順位が崩れる | 2512系 | 1〜3文・31語級に圧縮 |
| 矛盾するスタイル指定 | 2512系 | 主スタイル1つに絞る |
| 曖昧語（"beautiful", "a list"） | 全系 | 具体値・具体文字列に置換 |
| 文字の誤字・欠け | 全系 | 引用符+CFG 6〜8+ステップ35〜50。数字・記号は簡素化。中国語＞英語＞日本語の順で安定 |
| 日本語テキスト破綻 | Edit-2509 | T2I直描せずテキスト画像をEditで「忠実になぞる」 |
| 手・指の破綻 | 全系 | ネガ "extra fingers, deformed hands"＋肯定 "natural hand posture, five fingers"（60%→85%） |
| 編集が全体を書き変える | Edit系 | "Keep everything else unchanged" 常時付与、1編集1指示 |
| 顔・アイデンティティ崩れ | Edit系 | "Preserve face/clothing features" 明記 |
| ネガティブが無視される | 蒸留運用 | guidance 1では非対応。true_cfg>1の実装で使う |
| 出力がノイジー・ポーズが硬い | 2.1 | 公開直後の報告、確立した回避策なし（経過観察） |

---

## 6. 動作例プロンプト（原文・出典付き）

### Qwen-Image-2.1 公式（2026-09-20）
- テキスト描画: `A neon shop sign that reads "QWEN IMAGE 2.1", rainy night, reflections on wet pavement`
- 編集: `Change the background to a sunset beach`
- 編集（動き）: `Let this mascot dance under the moon`
- 複数参照: `These three characters are sitting around a campfire in a forest`
- 透明: `This is an RGBA image with transparency. A cute cartoon dragon sticker. The image has alpha channel and the background is transparent.`
- T2I: `A capybara reading a book by candlelight` / `A ceramic teapot on a wooden table`
- Comfy公式: `Clean flat vector infographic titled "FROM CHERRY TO CUP" showing five numbered steps left to right`

### 初代Qwen-Image公式（テキスト描画の宝庫）
- 書店ウィンドウ: `Bookstore window display. A sign displays "New Arrivals This Week". Below, a shelf tag with the text "Best-Selling Novels Here". To the side, a colorful poster advertises "Author Meet And Greet on Saturday" with a central portrait of the author. There are four books on the bookshelf, namely "The light between worlds" "When stars are scattered" "The slient patient" "The night circus"`
- 映画ポスター: `A movie poster. The first row is the movie title, which reads "Imagination Unleashed". The second row is the movie subtitle, which reads "Enter a world beyond your imagination". The third row reads "Cast: Qwen-Image". … At the bottom edge, the text "Launching in the Cloud, August 2025" appears in bold, modern sans-serif font …`
- 看板＋ネオン＋π: `A coffee shop entrance features a chalkboard sign reading "Qwen Coffee 😊 $2 per cup," with a neon light beside it displaying "通义千问". Next to it hangs a poster showing a beautiful Chinese woman, and beneath the poster is written "π≈3.1415926-53589793-23846264-33832795-02384197".`

### 2512公式実装例（ネガティブ付き）
- prompt: `A 20-year-old East Asian girl with delicate, charming features and large, bright brown eyes—expressive and lively... She stands indoors at an anime convention, surrounded by banners, posters, or stalls. Lighting is typical indoor illumination—no staged lighting—and the image resembles a casual iPhone snapshot...`
- negative: `低分辨率，低画质，肢体畸形，手指畸形，画面过饱和，蜡像感，人脸无细节，过度光滑，画面具有AI感。构图混乱。文字模糊，扭曲。`

### コミュニティ（Reddit プレイブック, 2025-08-27）
- テキスト差し替え: `Replace the sign text with 'GRAND OPENING'. Keep original font, size, color, and perspective. Do not alter background or signboard.`
- スタイル転送: `Re-render this scene in a Studio Ghibli art style. Preserve character identity, clothing, and layout.`
- 赤枠部分編集: `Within the red box, replace the lower component of the character '稽' with '旨'. Match stroke thickness and calligraphy style. Leave everything else unchanged.`
- ライティング: `Relight the scene with a warm key light from the right and cool rim light from the back. Keep pose and background unchanged.`
- レンズ: `Render with a 35 mm lens, shallow depth of field, focus on subject's face. Preserve environment blur.`
- 身維持転置: `Place the same character in a desert environment. Keep hairstyle, clothing, and facial features identical.`

### 日本語テキスト（Zenn, 2025-10-01, Edit-2509）
- 習字変換: `画像に書かれたテキストを習字風のフォントに変換してください。1画ごとの配置を忠実になぞり、抜け漏れがないようにしてください。左下に、赤い四角形の「通义千问」という印をつけてください`
- 絵の置換（番号参照実例）: `1枚目の部屋に飾られている絵について、額縁は残して、その内部を2枚目の画像で表す文字に置き換えてください。フォントは入力されたゴシック体ではなく、習字のような行書体に変更してください。最後に、赤い四角のハンコを絵の左下端に加えてください`

---

## 7. 情報源一覧

### 公式
| 種別 | URL | 日付 |
|---|---|---|
| 公式ブログ（中） | https://qwen.ai/blog?id=qwen-image-2.1 | 2026-09-20 |
| GitHub | https://github.com/QwenLM/Qwen-Image-2.1 | 2026-09-20 |
| HF モデルカード | https://huggingface.co/Qwen/Qwen-Image-2.1 | 2026-09-20 |
| PE-T2I（system_prompt.txt同梱） | https://huggingface.co/Qwen/Qwen-Image-2.1-PE-T2I | 2026-09-20 |
| PE-I2I（system_prompt.txt同梱） | https://huggingface.co/Qwen/Qwen-Image-2.1-PE-I2I | 2026-09-20 |
| HF デモ | https://huggingface.co/spaces/Qwen/Qwen-Image-2.1 | — |
| ComfyUI 公式 | https://blog.comfy.org/p/qwen-image-21-in-comfyui-open-weight | 2026-09-20 |
| diffusers PR（true_cfg） | https://github.com/huggingface/diffusers/pull/14804 | 2026-09-20 |
| vLLM レシピ | https://recipes.vllm.ai/Qwen/Qwen-Image-2.1 | — |
| SGLang クックブック | https://docs.sglang.io/cookbook/diffusion/Qwen-Image/Qwen-Image-2.1 | — |
| 初代Qwen-Imageブログ | https://qwenlm.github.io/blog/qwen-image/ | 2025-08-04 |
| QwenLM/Qwen-Image README＋公式拡張ツール prompt_utils_2512.py | https://github.com/QwenLM/Qwen-Image | 2025-08〜2026-02 |
| Edit-2509 HF | https://huggingface.co/Qwen/Qwen-Image-Edit-2509 | 2025-09-22 |
| Edit-2511 HF | https://huggingface.co/Qwen/Qwen-Image-Edit-2511 | 2025-12-23 |

### コミュニティ
| 種別 | URL | 日付 |
|---|---|---|
| Reddit プレイブック | https://www.reddit.com/r/StableDiffusion/comments/1n1n81o/ | 2025-08-27 |
| Reddit 2.1初期テスト | https://www.reddit.com/r/StableDiffusion/comments/1wkvqlj/ | 2026-09-19 |
| fal.ai 2512ガイド | https://fal.ai/learn/devs/qwen-image-2512-text-to-image-prompt-guide | 2026-01-07 |
| apiyi 実測23ケース | https://help.apiyi.com/en/qwen-image-2512-prompt-guide-test-cases-en.html | 2026-01-18 |
| Zenn 日本語描画 | https://zenn.dev/kota_iizuka/articles/33219ebb8aff99 | 2025-10-01 |
| Zenn 攻略ガイド | https://zenn.dev/rick_lyric/articles/ffd10bbb59e8b6 | 2025-11-22 |

---

## 8. スキル構成案への示唆

1. **中核はPE-T2I/PE-I2Iのsystem_prompt.txtの翻案** — 公式の「理想的プロンプト」仕様が全て書かれている。ただし2モードで方針が対立する部分があります（T2I=400〜500語の長文観察描写 vs コミュニティ実測=1〜3文が最適、編集=短い命令文）。スキルでは「用途別に使い分ける」構成に
2. **SKILL.mdはルーティング＋共通原則に薄く**、詳細はreferences/に分割（progressive disclosure）
3. 画像内テキスト描画（引用符規約・日本語の注意）、編集の保持節、複数参照の `<image1>` タグ規約、RGBAテンプレート、アスペクト比表は必須コンテンツ
4. プロンプト出力は**英語で生成**（公式仕様）。スキルの説明文は日本語で良い
5. 免責: ライセンスが非商用（Qwen Research License）である点をREADMEに明記すべき
