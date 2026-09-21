# 画像内に読める文字を描く

画像内テキストの忠実な描画はQwen-Image最大の強みです（中国語が最強、英語は堅実、**日本語は壊れやすい** — 後述）。コツのほとんどは引用の規律です。

## 最も重要なルール

**画像内に描く文字列は、描かれるべき形のまま二重引用符に入れる。**

```text
A neon shop sign that reads "QWEN IMAGE 2.1", rainy night, reflections on wet pavement
```

同系列での実測: 引用なし約65% → 引用あり約85% → 引用＋CFG引き上げ＋ステップ増で約96%。ユーザーの文字列を言い換え・翻訳・「きれいにする」ことは絶対にしません。大文字小文字・句読点・改行は書いた通りに再現されます。

## 提示方法を指定する

書体クラス、ウェイト、色、サイズ、呈示媒体すべてが効きます:

- `a bold black headline across the top reads "SPRING SALE"`
- `a chalkboard sign reading "Qwen Coffee $2 per cup"`
- `a neon light displaying "OPEN 24 HOURS"`
- 刺繍、彫刻、グラフィティ、筆文字 — 描画媒体もプロンプトの一部です。

## 複数のテキストブロック

各ブロックを読み順に、それぞれ位置を付けて個別に描写します:

```text
A movie poster. The first row is the movie title, which reads "Imagination Unleashed".
The second row is the movie subtitle, which reads "Enter a world beyond your
imagination". The third row reads "Cast: Qwen-Image". … At the bottom edge, the text
"Launching in the Cloud, August 2025" appears in bold, modern sans-serif font.
```

## レイアウト・ポスター・インフォグラフィック

`text-to-image.md` のロング観察描写を使い、読めるものは**すべて**書き出します: チャートの軸、目盛り値、凡例、表のセル値、ステップのラベル。公式ComfyUI投稿の番号付きインフォグラフィック例:

```text
Clean flat vector infographic titled "FROM CHERRY TO CUP" showing five numbered
steps left to right
```

（完全なプロンプトでは各ステップのラベルとキャプションを引用符付きで展開します。）

## アンチパターン

- **暗黙のテキスト**（「リストを表示して」「メニューを出して」）— 失敗します。文字列そのものを与えること。
- **入り組んだ数値表記** — `"#25 Jan 2026"` は崩れます。`"25 Jan 2026"` に簡素化。スラッシュ付き日付（`"9/28"`）や数字・記号の混ざりも同様に崩れやすい — ユーザーの文字列はそのままコミットしますが、崩れた場合は画像全体を作り直すのではなく、後述の編集トレース方式に切り替えます。
- **読める看板の捏造** — 遠景の文字を読ませたくないなら "blurred, indistinct, too small to read" と書きます。良い画像の約3割はテキストなし。看板をでっち上げない。
- **1つの文字列への複数スクリプト混在** — 描画テキストはモノリンガルで。

## 締めの一文

テキストを含むプロンプトは次で締めます:

```text
No other text appears in the image.
```

フレームの他の部分に紛れ込むゴミテキストを抑止する効果が実測されています。

## 日本語テキスト: 重要な注意

**テキストからの直接生成（T2I）で日本語は最も弱いスクリプトです**（中国語 ≫ 英語 > 日本語）— 漢字は画数が欠け、かなは歪みます。Qwen-Image-Edit-2509で検証済みの、今も推奨される回避策:

1. 描きたい日本語テキストをそのまま、白地に黒字のシンプルな画像として用意する（画像編集ツールやテキストマスク生成ノードで）。
2. それを編集モードに渡し、次のような指示を出す:
   `画像に書かれたテキストを忠実に再現してください。1画ごとの配置を崩さず、抜け漏れがないようにしてください` — つまり「入力画像のテキストを1画ごと忠実になぞって再現し、抜け漏れをなくす」指示。そのうちで配置を指定する（例: "place it inside the picture frame in <image1>, in a calligraphy style"）。

編集パスは入力テキストを「なぞる」方が、生成パスが「発明」するよりはるかに忠実です。中国語テキストは逆に、直接指定しても高い精度で描けます。

英語と日本語が混在するポスターでは: 日本語文字列を引用符に入れた直接生成を1回試します（短い文字列なら生き残ることがあります）。崩れたら、キャプション領域を「見出し直下の空き帯」（"a clean empty band beneath the headline, reserved for a caption"）として描写したベースを作り直し、上記の編集トレース方式で日本語を挿入します。

## 文字が手に関わる・結果が化けるとき

- CFG（true_cfg 6〜8）とステップ数（35〜50）を引き上げる。
- 手の破綻はネガティブ（`extra fingers, deformed hands`）＋肯定文（"natural hand posture, five fingers"）で修復 — 手の正常率が約60%→85%に上がった報告あり。
- 句読点を簡素化して再試行。それでも化ける場合は上記の編集トレース方式に切り替える。
