# プロンプトと設定

## LoRA適用時の固定プロンプト

```text
Apply skin_paint_v1 to the exposed skin of image1 while preserving the person and all non-skin content.
```

`skin_paint_v1` は今回の学習に使った固定した塗り表現の識別語です。入力は編集したい元画像1枚です。negative promptは空です。

| 項目 | 初期値 |
|---|---|
| 画像サイズ | 1024×1024 |
| バッチ | 1 |
| Steps | 25 |
| CFG | 1.0 |
| Sampler | euler |
| Scheduler | simple |
| denoise | 1.0 |
| 参照のresolution | 1024 |
| LoRA強さ | 1.0 |
| raw seed | 281305 |
| masked seed | 281303 |
| seed変更 | fixed |

LoRAは `LoraLoaderModelOnly` でモデルのみに適用し、CLIPへは適用しません。

## LoRAなしの自然言語編集を比べる場合

以下は初期の評価人物1系列で使った対照プロンプトです。LoRA強さ0またはLoRAを外したモデルで使います。

```text
Repaint only the exposed skin of the adult person in image1 with softly blended, dimensional anime skin painting. Use broad smooth shadows, gentle reflected light, subtle warm color variations on the cheeks and joints, and restrained highlights. Preserve the original person's overall skin complexion and light direction. Preserve the character identity, facial features, body shape, hands and pose. Keep hair, eyes, lips, clothing, objects, background, line art and composition unchanged.
```

自然言語編集でも柔らかい塗りを得られた例があります。LoRAが常に優れているとは評価していません。同じ元画像とseedで比べ、好みの塗りと肌以外への変化を見て選んでください。
