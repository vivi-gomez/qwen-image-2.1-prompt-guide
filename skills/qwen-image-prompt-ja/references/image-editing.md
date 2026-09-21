# 画像の編集・スタイル変更・合成

Qwen-Image-2.1は生成と編集が統一されています。入力画像1枚（編集）、参照画像複数枚（合成）、なし（生成）を同じモデルで扱います。**編集プロンプトは短い命令文**です — 生成のロングフォームとは正反対です。

## 統轄原理: 属性のもつれ解除（attribute disentanglement）

変えたい属性だけを名指しし、残りすべてを明示的な保持宣言で固定します。失敗モードは2つで対称です:

- **漏れ（leakage）** — 指定していないものに編集が波及する（「夜にして」でコートのデザインまで変わる等）。
- **過小編集** — 出力が入力と見分けがつかない。

保持宣言は*内容*をロックするもので、編集の強さを弱めません。**必ず1つ入れます。**

```text
Replace the sign text with 'GRAND OPENING'. Keep the original font, size,
color, and perspective. Leave the background and signboard unchanged.
```

## ルール

- **短い命令文で書く。** "Change the background to a sunset beach." / "Remove the parked car." / "Add a wool scarf around her neck." / "Rotate the shoe to show the sole."
- **保持対象は役割・位置で名指しし、外観を再描写しない。** 保持対象を具体的に描写すると生成指示と読まれてドリフトします。個別の保持節を並べるより、包括的な1節（"Keep everything else unchanged"）を優先します。
- **アイデンティティは最も難しい不変量。** 顔、特徴的なアクセサリ、製品の意匠、レンダリング媒体（写真/アニメ/イラスト/3D）は、明示的に狙わない限りどの編集でも維持されます。参照画像に由来するアイデンティティは画像タグで指します — 言葉で描写すると類似性が劣化します。
- **画像内テキストはリテラルに。** 出力に現れる読める文字はすべて引用符付きで正確にコミットします。言い換え・要約は不可。入力のタイポグラフィと言語を維持。ソースで読めない文字は追加しません。
- **肯定形で書く。** 「背景を変えないで」でなく "Keep the background unchanged"。
- **決定的に書く。** ヘッジ（maybe, or）と未解決の選択肢は不可 — 曖昧さは編集を割らせます。
- **改行なしの1段落。** *描く*文字だけ二重引用符で囲みます。比率・解像度の語は文中に入れません（後述）。
- **小さい編集をつなぐ。** 大掛かりな変更は1つの巨大な指示ではなく、2〜3回の逐次パス（編集→確認→次の編集）に分割します。定型の合成レシピ（例: 商品を新しい台面へ配置）は複数箇所に触れても1つの論理的変更と数えます。連鎖は短く — 毎パスで画像全体が再エンコードされるため、パスが重なるほどアイデンティティやラベルの忠実度は削れます。毎パス後に再確認してください。

## 複数参照画像の合成（2〜10枚）

入力画像が2枚以上のとき、公式2.1リライターは**タグ参照を必須**にしています:

- 入力は `<image1>`, `<image2>` … で参照する — "the first image", "image A", "1枚目の画像" などの自然言語参照は禁止（※公式リライター経由時。後述のZenn実例のように自然言語の番号参照が機能した報告も旧世代にはありますが、2.1ではタグが正式規約）。
- 入力1枚の編集ではタグを*使わない*。
- 各画像に役割を割り当てる — どちらが**キャンバス**（編集の土台）で、どちらが素材（人物・商品・背景・スタイルの提供元）か。
- キャンバス内のレイアウトは空間の言葉で決める: "facing each other", "on the left / on the right"。

定型レシピ:

| タスク | プロンプトの形 |
|---|---|
| 人物をシーンへ | `Place <image1>'s character in the forest camp of <image2>. Keep hairstyle, clothing, and facial features identical.` |
| 商品を設定へ | `Put the product from <image2> onto the marble counter in <image1>. Preserve the product's shape, materials, and label text exactly.` |
| 背景差し替え | `Replace the background of <image1> with the scenery from <image2>. Keep the subject, pose, and lighting direction unchanged.` |
| スタイル転送 | `Re-render <image1> in the art style of <image2>. Preserve subject identity, clothing, and layout.` |
| グループ合成 | `The characters from <image1>, <image2>, and <image3> are sitting around a campfire in a forest.`（2.1公式例） |

## 言語の2層化

独立した2つの言語決定を混同しないこと:

1. **説明文の言語**: 編集指示は英語が安全なデフォルト（公式リライターは中国語依頼には中国語、それ以外は英語を出力）。唯一の例外は日本語テキストのトレース編集で、日本語指示の実績あるテンプレートが存在します — `text-rendering.md` を参照。
2. **画像内テキストの言語**: (a) ユーザーの明示指定が最優先。なければ (b) 入力画像のテキストの支配言語。さらになければ (c) 指示の言語。描画テキストは**モノリンガル**（1つの文字列に複数スクリプトを混ぜない）。

## 出力サイズ

- デフォルトは**入力画像の比率に追従**（リライターのスキーマでは `ratio_follow: "<image1>"`）。`wh_ratio` と `ratio_follow` の同時設定は無効 — 排他です。
- 例外は「新規シーン生成」（撮影シーン、コスプレ、新設定）: 生成と同じく意味から比率を選びます。
- 公式リライターのキーワード→比率対応: 正方形/アバター→1:1、横長/PPT→16:9、ポスター→2:3、証明写真/小紅書→3:4、パノラマ→2:1、名刺→9:5、A4→5:7/7:5、iPhone画面→18:39、Android→9:20、シネマスコープ→21:9。
- 「2K/4K/8K」は品質記述子であり比率のヒントではない — 出力は常に約2K。比率を推測する材料にしない。
- アウトペインティング（外拡張）: 拡張方向から新しい比率を推論します（追加面積は30〜50%を見込む）。

## 局所編集

2.1では旧世代のControlNet式の条件付けを、キャンバス上への注釈に置き換わっています。編集したい範囲を円で囲む・塗りつぶす・マスクで指定した上で、その範囲にスコープした指示を書きます — "Within the red box, replace … Leave everything else unchanged."
