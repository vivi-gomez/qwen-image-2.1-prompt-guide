# 動作例プロンプト集

公式Qwenソースと情報量の多いコミュニティ投稿から厳選。出典を付けてあります。自由に改変して構いませんが、引用符と保持宣言のパターンは維持してください。

## 生成 — Qwen-Image-2.1公式（2026-09-20）

```text
A neon shop sign that reads "QWEN IMAGE 2.1", rainy night, reflections on wet pavement
A capybara reading a book by candlelight
A ceramic teapot on a wooden table
Clean flat vector infographic titled "FROM CHERRY TO CUP" showing five numbered steps left to right
```

## テキスト主体の生成 — 公式Qwen-Imageブログ（2025-08-04）

書店のウィンドウ展示（読めるテキスト4層）:

```text
Bookstore window display. A sign displays "New Arrivals This Week". Below, a shelf
tag with the text "Best-Selling Novels Here". To the side, a colorful poster
advertises "Author Meet And Greet on Saturday" with a central portrait of the
author. There are four books on the bookshelf, namely "The light between worlds"
"When stars are scattered" "The slient patient" "The night circus"
```

映画ポスター（行ごとのテキスト指定）:

```text
A movie poster. The first row is the movie title, which reads "Imagination
Unleashed". The second row is the movie subtitle, which reads "Enter a world
beyond your imagination". The third row reads "Cast: Qwen-Image". The fourth row
reads "Director: The Collective Imagination of Humanity". … At the bottom edge,
the text "Launching in the Cloud, August 2025" appears in bold, modern
sans-serif font …
```

店面のミックスメディア（チョーク＋ネオン＋ポスター＋数字）:

```text
A coffee shop entrance features a chalkboard sign reading "Qwen Coffee 😊 $2 per
cup," with a neon light beside it displaying "通义千问". Next to it hangs a poster
showing a beautiful Chinese woman, and beneath the poster is written
"π≈3.1415926-53589793-23846264-33832795-02384197".
```

## 編集 — 2.1公式

```text
Change the background to a sunset beach
Let this mascot dance under the moon
```

## 複数参照 — 2.1公式

```text
These three characters are sitting around a campfire in a forest
```

タグ参照を使う場合（PE-I2I規約、入力2枚以上）:

```text
Place <image1>'s character in the forest camp of <image2>. Keep hairstyle,
clothing, and facial features identical.
```

## 透明画像 — 2.1公式

```text
This is an RGBA image with transparency. A cute cartoon dragon sticker.
The image has alpha channel and the background is transparent.
```

## コミュニティの編集パターン（Reddit r/StableDiffusionプレイブック、2025-08-27）

```text
Replace the sign text with 'GRAND OPENING'. Keep original font, size, color, and
perspective. Do not alter background or signboard.

Re-render this scene in a Studio Ghibli art style. Preserve character identity,
clothing, and layout.

Within the red box, replace the lower component of the character '稽' with '旨'.
Match stroke thickness and calligraphy style. Leave everything else unchanged.

Relight the scene with a warm key light from the right and cool rim light from
the back. Keep pose and background unchanged.

Render with a 35 mm lens, shallow depth of field, focus on subject's face.
Preserve environment blur.

Place the same character in a desert environment. Keep hairstyle, clothing, and
facial features identical.
```

## カメラ・品質表現（apiyi、fal.ai — Qwen-Image-2512時代）

```text
shot on Canon EOS R5, 85mm f/1.4 lens, professional photography, RAW format
, Ultra HD, 4K, cinematic composition.        ← 初代公式のサフィックス。
                                               2.1では品質ブースター非推奨のため省略
```

公式2512ポートレート例に同梱されていたネガティブプロンプト:

```text
低分辨率，低画质，肢体畸形，手指畸形，画面过饱和，蜡像感，人脸无细节，过度光滑，
画面具有AI感。构图混乱。文字模糊，扭曲。
```

## 日本語テキストの回避策（Zenn、Edit-2509、2025-10-01）

描画済みテキストを筆文字に変換させる例（編集パスへの指示は日本語で可）:

```text
画像に書かれたテキストを習字風のフォントに変換してください。1画ごとの配置を忠実に
なぞり、抜け漏れがないようにしてください。左下に、赤い四角形の「通义千问」という
印をつけてください
```

テキスト画像を部屋の絵に合成する例（番号参照）:

```text
1枚目の部屋に飾られている絵について、額縁は残して、その内部を2枚目の画像で表す文字に
置き換えてください。フォントは入力されたゴシック体ではなく、習字のような行書体に変更
してください。最後に、赤い四角のハンコを絵の左下端に加えてください
```

## 出典一覧

| 出典 | URL |
|---|---|
| Qwen-Image-2.1 GitHub | https://github.com/QwenLM/Qwen-Image-2.1 |
| Qwen-Image-2.1 HFモデルカード | https://huggingface.co/Qwen/Qwen-Image-2.1 |
| PE-T2I / PE-I2I モデルカード | https://huggingface.co/Qwen/Qwen-Image-2.1-PE-T2I ・ …-PE-I2I |
| 公式ブログ（中国語） | https://qwen.ai/blog?id=qwen-image-2.1 |
| ComfyUI公式投稿 | https://blog.comfy.org/p/qwen-image-21-in-comfyui-open-weight |
| Qwen-Imageブログ（テキスト描画） | https://qwenlm.github.io/blog/qwen-image/ |
| QwenLM/Qwen-Image README＋公式拡張ツール | https://github.com/QwenLM/Qwen-Image |
| Reddit編集プレイブック | https://www.reddit.com/r/StableDiffusion/comments/1n1n81o/ |
| fal.ai 2512プロンプトガイド | https://fal.ai/learn/devs/qwen-image-2512-text-to-image-prompt-guide |
| apiyi 実測23ケース | https://help.apiyi.com/en/qwen-image-2512-prompt-guide-test-cases-en.html |
| Zenn: Editで日本語を描く | https://zenn.dev/kota_iizuka/articles/33219ebb8aff99 |
| Zenn: プロンプト攻略ガイド | https://zenn.dev/rick_lyric/articles/ffd10bbb59e8b6 |
