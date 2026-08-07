---
title: "Segment Anything Model（SAM）とは？初心者向けに基本原理を解説"
date: 2026-08-07 09:35:00 +0900
description: "Segment Anything Model（SAM）の基本原理を初心者向けに解説します。"
toc: true
math: false
---

# Segment Anything Model（SAM）とは？ ― 画像セグメンテーションを初心者向けに理解する

## はじめに

**Segment Anything Model（SAM）**は、Meta AIが開発した画像セグメンテーションモデルです。

画像セグメンテーション（image segmentation）とは、画像の中から目的とする領域をピクセル単位で抽出する処理です。

例えば眼科OCT画像で、

* 網膜
* 網膜下液（subretinal fluid: SRF）
* 網膜内液（intraretinal fluid: IRF）

などの領域を塗り分ける処理がセグメンテーションに相当します。

SAMの大きな特徴は、

> **「何を抽出するか」をクリックや矩形などで人間が指示できる**

ことです。

---

# 1. そもそもセグメンテーションとは？

画像AIにはさまざまなタスクがあります。

### Classification

画像全体を分類します。

```text
OCT画像
   ↓
 AI
   ↓
SRFあり / SRFなし
```

どこにSRFが存在するかまでは分かりません。

### Object detection

対象物の位置を矩形（bounding box）として検出します。

```text
┌────────────────────┐
│                    │
│       ┌──────┐     │
│       │ SRF  │     │
│       └──────┘     │
│                    │
└────────────────────┘
```

### Segmentation

対象物そのものをピクセル単位で抽出します。

```text
OCT image
      ↓
Segmentation model
      ↓
████████
████████  ← SRF mask
 ██████
```

つまり、

**「このピクセルはSRF、このピクセルはSRFではない」**

という判定を画像全体について行います。

---

# 2. 従来のセグメンテーションAI

従来の医用画像セグメンテーションでは、U-Netなどのモデルが広く利用されています。

例えばSRFを抽出したい場合、

```text
大量のOCT画像
      +
SRFの正解マスク
      ↓
   training
      ↓
SRF segmentation model
```

という流れになります。

この方法では、

> **SRFを抽出するための専用AI**

を作ることになります。

別の対象を抽出したければ、基本的にはその対象について教師データを用意して学習する必要があります。

---

# 3. SAMでは何が違うのか？

SAMでは発想が異なります。

ユーザーが画像を見ながら、

> 「ここを抽出してほしい」

とAIに指示できます。

この指示を **prompt（プロンプト）** と呼びます。

代表的なpromptには、

* point
* bounding box
* mask

などがあります。

例えば、

```text
OCT image

────────────────────────
        ●
      ↑
  「ここがSRF」
────────────────────────

          ↓ SAM

────────────────────────
       ███████
     ███████████
       ██████
────────────────────────
```

のようになります。

つまりSAMは、

**人間の指示を利用して対象領域を推定するAI**

と考えると分かりやすいです。

---

# 4. Positive pointとNegative point

point promptには大きく2種類あります。

### Positive point

「ここは抽出したい対象である」

という指示です。

```text
      SRF

   █████████
 █████●██████
   ███████

       ↑
 positive point
```

### Negative point

「ここは対象ではない」

という指示です。

例えばSAMが網膜組織までSRFとして認識してしまった場合、

```text
████████████████  retina
        ×
    ███████
    ███●███       SRF
    ███████
```

`●` をpositive point、

`×` をnegative point

として指定できます。

SAMはこれらの情報から、

> 「●を含むが×を含まない領域はどこか？」

を推定します。

この**対話的な修正が可能**なのがSAMの重要な特徴です。

---

# 5. SAMの内部構造

SAMは大きく3つの部分から構成されます。

```text
                SAM

Image
  │
  ▼
┌───────────────────┐
│   Image Encoder   │
└───────────────────┘
          │
          │ image embedding
          ▼
┌───────────────────┐
│   Mask Decoder    │
└───────────────────┘
          ▲
          │
┌───────────────────┐
│  Prompt Encoder   │
└───────────────────┘
          ▲
          │
      point / box
```

重要なのは次の3つです。

1. **Image Encoder**
2. **Prompt Encoder**
3. **Mask Decoder**

---

# 6. Image Encoder

Image Encoderは、

> **画像の特徴をAIが理解できる情報に変換する部分**

です。

例えばOCT画像そのものは大量のピクセル値から構成されています。

```text
OCT image
   ↓
Image Encoder
   ↓
Image embedding
```

Image Encoderは画像から、

* 境界
* 形
* texture
* 構造

などの情報を抽出し、**image embedding**という特徴表現に変換します。

SAMではVision Transformer（ViT）がこの中心的な役割を担っています。

---

# 7. Prompt Encoder

Prompt Encoderは、人間から与えられた指示をAIが扱える形に変換します。

例えば、

```text
positive point
negative point
bounding box
```

などです。

```text
click
  ↓
Prompt Encoder
  ↓
Prompt embedding
```

つまり、

**Image Encoder = 画像を理解する**

**Prompt Encoder = 人間の指示を理解する**

と考えると分かりやすいです。

---

# 8. Mask Decoder

最後にMask Decoderが、

```text
Image embedding
       +
Prompt embedding
       ↓
 Mask Decoder
       ↓
segmentation mask
```

という処理を行います。

つまりSAM全体を非常に単純化すると、

```text
画像の情報
    +
人間からの指示
    ↓
「この人が指定している領域はどこか？」
    ↓
Mask
```

というAIです。

---

# 9. なぜクリックするたびに高速に結果が出るのか？

SAMを理解するうえで重要なポイントです。

画像そのものの解析はImage Encoderによって行われます。

```text
Image
  ↓
Image Encoder
  ↓
Image embedding
```

この処理は比較的重い処理です。

しかし一度image embeddingを計算してしまえば、その後はクリックするたびに画像全体を最初から解析する必要はありません。

```text
                 Image
                   ↓
             Image Encoder
                   ↓
           Image embedding
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
     click       click       click
       ↓           ↓           ↓
     mask        mask        mask
```

そのため、

**クリック → segmentation → 修正 → segmentation**

というインタラクティブな操作が可能になります。

これはnapariなどの画像viewerとSAMを組み合わせる際にも重要な性質です。

---

# 10. SAMは「SRFを知っている」のか？

ここは医用画像にSAMを使うときに特に重要です。

SAMにOCT画像を与えて、

```text
ここはSRF
   ↓
   ●
```

とクリックしたとしても、SAMが医学的に

> 「これはsubretinal fluidである」

と診断しているわけではありません。

SAMが行っているのは、

> **promptで指定された対象に対応すると考えられる画像領域を抽出すること**

です。

したがって、

**SAM = 疾患診断AI**

ではありません。

むしろ、

**SAM = 非常に高性能な領域抽出AI**

と理解した方が適切です。

---

# 11. SAMと完全自動AIの違い

例えばSRF segmentationを完全自動化すると、

```text
OCT
 ↓
AI
 ↓
SRF mask
```

となります。

一方、SAMでは、

```text
OCT
 ↓
Human prompt
 ↓
SAM
 ↓
SRF mask
```

となります。

つまり、

### 完全自動AI

```text
AI → segmentation
```

### SAM

```text
Human + AI → segmentation
```

です。

SAMは特に、

**human-in-the-loop segmentation**

と相性のよいモデルです。

---

# 12. 医用画像でSAMが面白い理由

医用画像のアノテーションは非常に大変です。

例えばOCT volumeが512 B-scansあり、それぞれにSRFを手作業で塗る場合、

```text
slice 1    手動
slice 2    手動
slice 3    手動
...
slice 512  手動
```

となります。

SAMを利用すると、

```text
slice
 ↓
数回クリック
 ↓
SAM segmentation
 ↓
人間が確認・修正
 ↓
mask
```

というワークフローにできる可能性があります。

つまりSAMは、

> **人間を完全に置き換えるというより、人間によるアノテーションを高速化する**

用途にも向いています。

---

# 13. ただし、通常のSAMは医用画像専用ではない

SAMは一般画像を中心として開発されたfoundation modelです。

したがって、

* OCT
* MRI
* CT
* 病理画像

などにそのまま使用した場合、必ずしも十分な性能が得られるとは限りません。

特にOCTでは、

* SRF
* IRF
* PED
* retinal layers

など、一般画像とは大きく異なる構造を扱います。

この問題を解決するため、SAMを医用画像に適応させたモデルも研究されています。

代表例の一つが **MedSAM** です。

---

# 14. SAMを一言で表すと

SAMを最も単純化すると、

> **画像と人間からのpromptを受け取り、指定された対象のmaskを生成するfoundation model**

です。

内部では、

```text
Image
  ↓
Image Encoder
  ↓
Image embedding
       │
       │
       ▼
   Mask Decoder ───→ Mask
       ▲
       │
Prompt embedding
       ▲
       │
Prompt Encoder
       ▲
       │
 point / box
```

という処理が行われています。

初心者の段階では、

**Image Encoder**

= 画像を理解する

**Prompt Encoder**

= 人間の指示を理解する

**Mask Decoder**

= 両方を組み合わせて領域を塗る

と理解しておけば十分です。

---

# まとめ

SAMを理解するうえで重要なのは次の4点です。

1. **SAMは画像セグメンテーションを行うAIである**
2. **pointやboxなどのpromptによって抽出対象を指示できる**
3. **Image Encoder・Prompt Encoder・Mask Decoderから構成される**
4. **医学的診断をしているのではなく、指定された対象に対応する画像領域を推定している**

特に医用画像研究では、

```text
完全手動annotation
        ↓
SAMによるinteractive annotation
        ↓
Human correction
        ↓
高品質な教師データ
        ↓
医用画像専用segmentation model
```

という使い方が非常に重要になります。

SAMを単なる「自動セグメンテーションAI」と考えるより、

> **人間とAIが対話しながら画像をセグメンテーションするための基盤モデル**

と理解すると、その役割が分かりやすくなります。
