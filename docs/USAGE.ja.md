# 使い方

## 1. 記事添付ZIPを取得する

note記事の有料エリアに添付した `skin-paint-workflows-v1.0.1.zip` を取得し、解凍します。ワークフローJSONの配布先は記事の有料エリアです。以下の `workflows/` は解凍した記事添付ZIP内のフォルダを指します。

## 2. 対応する環境を用意する

Qwen Image **2.1**の画像編集に対応したComfyUIを使用します。旧Qwen Edit用のモデル・エンコーダー・VAEとは区別してください。

検証環境はComfyUI 0.37.0（commit `b0f4b7b294ce482a2e071d9d762c133d38c7aa07`）、leejet版ComfyUI-GGUF（commit `edd981b10e107d3b8f58e16c498f2d08f631bc47`）です。

- ComfyUI: https://github.com/Comfy-Org/ComfyUI
- 検証したGGUFローダー: https://github.com/leejet/ComfyUI-GGUF
- モデルの公式案内: https://github.com/QwenLM/Qwen-Image-2.1

| 種類 | ComfyUIの配置先 | ワークフローの初期ファイル名 |
|---|---|---|
| 拡散モデル | `models/unet/` | `QwenImage/qwen-image-2.1-Q4_K_M.gguf` |
| テキスト・画像エンコーダー | `models/text_encoders/` | `qwen3vl_8b_int8_convrot.safetensors` |
| VAE | `models/vae/` | `qwen_image_2.1_vae_bf16.safetensors` |
| 今回のLoRA | `models/loras/` | `skin-paint-image21-r32-v1.safetensors` |

モデルの探索フォルダを独自設定している場合は、その設定に合わせます。GUIのモデル選択で実際に配置したファイルを指定してください。Qwen Image 2.1用の `TextEncodeQwenImage21` と `UnetLoaderGGUF` が表示されることを確認します。

## 3. まずraw版で試す

1. LoRA本体を配置してComfyUIに認識させます。
2. `workflows/skin-paint-v1-raw-ui.json` をComfyUIへ読み込みます。
3. 「元画像 / 1 枚」で自分の編集用画像を読み込みます。`source.png` は差し替え用の名前です。
4. 「肌質感 LoRA / 強さ」でLoRAを選び、強さ1.0から試します。控えめな変化には0.6を試せます。
5. 元画像を1024×1024に揃えて実行します。ほかの寸法を使う場合は「出力latent」の幅・高さを同じ寸法に変更します。マスク版も全画像の寸法を揃えます。
6. 保存したPNGを確認します。出力は `output/SkinPaintImage21/` 以下に保存されます。

初期latentは `EmptyLatentImage` です。元画像は編集参照としてエンコーダーへ入力し、生成はdenoise 1.0で行います。元画像のVAE latentから低denoiseで始めるI2I設定ではありません。

raw版は髪・服・背景などにも変更が出ることがあります。比較する場合は同じ画像、同じプロンプト、同じseedでLoRA強さ0と1.0を生成し、肌の中間色と陰影の変化を確認してください。

## 4. 肌マスクで仕上げる

1. 元画像と同じ幅・高さのグレースケールPNGを作ります。白が編集する肌、黒が元画像を保つ範囲、灰色が両者を混ぜる境界です。
2. `workflows/skin-paint-v1-masked-ui.json` を読み込みます。
3. 元画像を指定し、別の「肌mask」画像入力へ作成したマスクを指定します。`skin-mask.png` は差し替え用の名前です。
4. 元画像・肌マスク・出力latentを同じ寸法に揃えて実行します。
5. 保存PNGの肌境界、顔の目・口、髪や服へのはみ出し、元画像のalphaを確認します。

肌マスクは `ImageToMask(channel=red)` で読み込みます。マスク画像はLモードのPNGを推奨します。`LoadImage` のMASK出力は画像の透明度由来なので、肌範囲のマスクとして使いません。

`ImageCompositeMasked` のdestinationは元画像、sourceは生成画像です。x=y=0、resize_source=falseで合成します。最後に元画像のMASKを `JoinImageWithAlpha` のalphaへ直接接続し、元alphaを戻します。検証した実装では双方が反転するため、間に `InvertMask` を追加しません。

黒領域が元画像へ戻ることと、モデル自体が肌だけを変えたことは別です。マスクは人物・ポーズごとに作成し、完成画像で境界を見直してください。

## API版を使う場合

`*-api.json` はComfyUIのAPI用prompt本体です。GUI用JSONとは形式が異なります。

- node `4` の `inputs.image` を、ComfyUIのinputフォルダに登録した元画像名へ変更します。
- masked版ではnode `13` の `inputs.image` を、登録した肌マスク名へ変更します。
- node `12` でLoRAファイル名と `strength_model` を指定します。
- node `7` で出力幅・高さ、node `8` でseedなどの生成設定を指定します。
- 送信時はAPI JSONを `{"prompt": <API JSON>}` のpromptへ入れます。

API初期seedはraw版281305、masked版281303です。異なる作例で検証したグラフを基にしているため、rawとmaskedを同条件で比較する際はseedも揃えてください。
