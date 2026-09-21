# モデルの能力・パラメータ・実行

## モデル概要（2026-09-21時点）

- **Qwen-Image-2.1** — 2026-09-20公開。7B Single-Stream DiT（MMDiT系）、Qwen3-VL 8Bテキストエンコーダ、64チャンネルRGBA VAE、Flow Matching。生成＋編集の統一モデル。
- 公式プロンプトリライター: **PE-T2I** と **PE-I2I**（各9.4B）。短い依頼を本スキルが文書化しているスタイルに拡張します。
- 提供形態: Hugging Face / ModelScopeでオープンウェイト配布。Day-0対応はDiffusers（推奨）、ComfyUI、vLLM-Omni、SGLang、LightX2V。HF Spaceにデモあり。**2026-09-21時点でAlibaba Cloud Model Studio（百炼）APIには未掲載。**
- 系譜の注意: Qwen-Image-3.0 / 3.0-Pro（2026-07）は番号が新しいがクローズドAPI専用。2.1がオープンウェイト系列の現行フラッグシップです。

## アスペクト比と解像度（ネイティブ2K）

| 比率 | ピクセル | | 比率 | ピクセル |
|---|---|---|---|---|
| 1:1 | 2048×2048 | | 3:2 | 2528×1696 |
| 4:3 | 2400×1792 | | 16:9 | 2752×1536 |
| 3:4 | 1792×2400 | | 2:3 | 1696×2528 |
| — | — | | 9:16 | 1536×2752 |

- 公式リライターは 2:1, 21:9, 9:21, 4:5, 3:1, 5:4, 1:3, 18:39, 9:20, 7:3, 9:5, 5:7 も出力することがあります — 利用パイプラインで最も近い対応解像度にマップします。
- vLLM系APIは `"size": "1024x1024"` 形式の文字列を直接受け付けます。

## 生成パラメータ

- **ステップ数**: 40（公式デフォルト）。テキストを含む生成・編集は50 — 文字化け時に最初に上げる値（`text-rendering.md` 参照）。
- **CFG / ネガティブプロンプト**: Diffusersパイプラインは `negative_prompt` ＋ `true_cfg_scale`（デフォルト4.0。>1 かつネガ指定でtrue CFGが有効）に対応。ガイダンス1の蒸留運用ではネガティブは完全に無視されます — 実行環境を確認してからネガティブの効果を約束すること。文字化け対策には true_cfg を6〜8に引き上げてテキスト修復ネガティブを併用します（この値が4.0のデフォルトより優先）。
- **シード**: 完全制御可（`torch.Generator(...).manual_seed(n)`、vLLMの `--seed`）。プロンプトを変えずにシリーズ（商品の正面/側面/上面など）を揃えたいときはシード固定が有効。
- 実用的なネガティブ: 手の修復 `extra fingers, deformed hands`。テキスト修復 `misspelled text, garbled letters, unreadable font`。

## 透明画像（RGBA）

公式テンプレート — 説明部分だけ差し替えてそのまま使います:

```text
This is an RGBA image with transparency. <your description>.
The image has alpha channel and the background is transparent.
```

透明レイヤー編集・被写体切り抜きにも同様の形式で扱えます。

## 複数参照画像入力

最大**10枚**（ComfyUIノードは16枚まで）。タグ参照の規約（`<image1>`, …）は `references/image-editing.md` を参照。

## 最小実行スニペット

```python
# Diffusers（推奨）
from diffusers import QwenImage21Pipeline
pipe = QwenImage21Pipeline.from_pretrained("Qwen/Qwen-Image-2.1", torch_dtype=torch.bfloat16)
image = pipe(prompt, negative_prompt=" ", true_cfg_scale=4.0,
             height=1536, width=2752, num_inference_steps=40).images[0]
```

```text
vLLM: qwen-image-2.1 エンドポイント — {"prompt": "...", "size": "1024x1024", "seed": 42}
```

ネガティブを使わない場合は空文字でなくスペース1つ（`" "`）を渡します。

## 既知の失敗パターン → 修正

| 症状 | 修正 |
|---|---|
| タグ式・重み付け構文が無視される | 流暢な自然文に書き直す |
| 被写体が違う・曖昧 | 被写体を文頭に移動し、具体性を上げる |
| 長いプロンプトで構図が崩れる | 1〜3文に圧縮し、前寄せに再構成 — ただし**レイアウト/ポスター/テキスト主体は例外**（ロングフォームが正解。崩れの真因は位置未指定・配置されない要素の溢れであることが多い） |
| 画像内テキストの文字化け | 引用符で正確にコミット。true_cfg 6〜8、ステップ35〜50。数値表記の簡素化。日本語は編集トレース方式へ |
| 編集が未指定部分に波及 | 保持宣言を追加・拡充。より小さい編集に分割して連鎖 |
| 顔・アイデンティティのドリフト | "Preserve face/clothing features" を明記。参照は言葉でなくタグで指す |
| 指・手の破綻 | ネガティブ＋肯定形の姿勢表現 |
| 出力がノイジー、ポーズが硬い | 2.1公開初日の報告。確立した修正なし — シードを変えて再試行、プロンプト長を削減 |

## ライセンス — 商用利用の前に必読

Qwen-Image-2.1は初代と異なりApache 2.0ではなく、**Qwen Research License Agreement**の下で配布されています:

- 無償利用は**研究・評価目的（非商用）のみ**。商用利用は個別ライセンスが必要（model-business@notice.qwencloud.com）。
- 再配布にはライセンス文の同梱、改変ファイルの明示、帰属表示が必須。
- 出力で他のAIモデルを訓練した場合は「Built with Qwen」表示が必要。派生物の名称に "Qwen" を先頭に使うことは禁止。
