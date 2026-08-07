---
title: "Segment Anything Model（SAM）の仕組みを一歩深く理解する"
date: 2026-08-07 09:35:00 +0900
description: "Segment Anything Model（SAM）のアーキテクチャ、prompt encoding、mask decoder、曖昧性への対応、学習データ、zero-shot segmentationについて中級者向けに解説します。"
toc: true
math: true
---

## はじめに

前回の記事では、Segment Anything Model（SAM）を、

> 画像と人間からのpromptを受け取り、指定された対象のmaskを生成するモデル

として説明しました。

SAMの基本構造は次の3要素です。

1. Image Encoder
2. Prompt Encoder
3. Mask Decoder

初心者向けには、

* Image Encoder：画像を理解する
* Prompt Encoder：人間の指示を理解する
* Mask Decoder：両者を組み合わせてmaskを作る

という理解で十分です。

しかし、SAMを研究やソフトウェア開発に利用するには、さらに次の点を理解する必要があります。

* SAMは何を学習しているのか
* pointやboxはどのように数値化されるのか
* なぜ複数のmaskを出力するのか
* predicted IoUとは何か
* Automatic Mask Generatorは何をしているのか
* zero-shot segmentationとは何を意味するのか
* 医用画像で失敗するのはなぜか
* fine-tuningではどこを更新するのか

この記事では、SAMの原著論文と公式実装を基に、これらを順番に整理します。

SAMは単なるモデル名ではなく、Metaが提案した新しいsegmentation task、モデル、データセットを含むプロジェクトです。原著では、promptable segmentation modelと、11 million images・10億を超えるmaskからなるSA-1B datasetが提示されました。

---

## 1. SAMが解こうとしている問題

従来のsegmentation modelは、多くの場合、対象とするtaskごとに学習されます。

例えば、

```text
OCT image
    ↓
SRF専用モデル
    ↓
SRF mask
```

というモデルは、SRFのsegmentationには利用できます。

一方、そのままでは、

* IRF
* PED
* 網膜
* 脈絡膜
* 網膜色素上皮

などを自由に抽出できるわけではありません。

SAMが目指したのは、taskごとの専用モデルではなく、

> promptによって、抽出したい対象を後から指定できる汎用segmentation model

です。

この問題設定を、論文では **promptable segmentation task** と呼んでいます。

SAMへの入力は、概念的には次のように表せます。

[
M = f(I, P)
]

ここで、

* (I)：入力画像
* (P)：prompt
* (M)：出力mask
* (f)：SAM

です。

従来のモデルでは、何を抽出するかがモデルの学習段階で固定されることが多いのに対し、SAMではpromptによって推論時に対象を指定します。

---

## 2. SAMは対象物の名前を理解しているわけではない

SAMにOCT画像を入力し、SRFの内部をクリックしたとします。

```text
OCT image
     ↓
SRF内部をクリック
     ↓
SAM
     ↓
SRFに見える領域のmask
```

しかし、original SAMでは通常、

```text
"subretinal fluid"
```

という医学用語を入力しているわけではありません。

SAMが直接受け取る代表的なpromptは、

* foreground point
* background point
* bounding box
* mask

です。

したがってSAMが解いている問題は、

> これは何という疾患・組織か

ではなく、

> このpromptが指している画像領域はどこか

です。

この違いは重要です。

SAMはsemantic diagnosis modelではなく、基本的には**prompt-conditioned mask generation model**です。

なお、2026年時点では後継モデルとしてSAM 2やSAM 3も発表されています。SAM 2は画像と動画を統一的に扱い、SAM 3では短い名詞句や画像例によるconcept promptが導入されています。この記事では、構造を理解しやすいoriginal SAMを中心に扱います。

---

## 3. SAMの全体構造

SAMの推論処理は、次のように整理できます。

```text
Input image
    │
    ▼
Image Encoder
    │
    ▼
Image Embedding
    │
    ├──────────────────┐
    │                  │
    │             Prompt
    │                  │
    │                  ▼
    │           Prompt Encoder
    │                  │
    │                  ▼
    │           Prompt Embedding
    │                  │
    └─────────┬────────┘
              ▼
         Mask Decoder
              │
       ┌──────┴──────┐
       ▼             ▼
   Mask logits   Predicted IoU
```

この構造の重要な点は、**画像の処理とpromptの処理が分離されていること**です。

画像はImage Encoderによって一度embeddingに変換されます。

その後、ユーザーがpointを追加したり削除したりしても、同じimage embeddingを再利用できます。

これにより、

```text
click
  ↓
mask
  ↓
clickを追加
  ↓
maskを修正
```

というinteractive segmentationを高速に実行できます。

---

## 4. Image Encoderは何を出力しているのか

Image Encoderは、入力画像をそのまま「理解」しているというより、画像を低解像度の特徴マップへ変換しています。

[
E_I = f_{\mathrm{image}}(I)
]

ここで、

* (I)：入力画像
* (f_{\mathrm{image}})：Image Encoder
* (E_I)：image embedding

です。

original SAMでは、Image EncoderとしてVision Transformer（ViT）を基盤としたmasked autoencoder事前学習済みモデルが使用されています。

### 4.1 ピクセル値から特徴量への変換

例えば元画像が、

```text
1024 × 1024 × 3
```

のRGB画像だったとしても、Mask Decoderがすべてのピクセルを直接扱うわけではありません。

画像はpatchに分割され、各patchがtokenとして表現されます。

概念的には、

```text
Image
  ↓
patch分割
  ↓
patch embedding
  ↓
Transformer
  ↓
image embedding
```

という流れです。

Image Encoderは、

* 輪郭
* texture
* 色
* 局所構造
* 物体のまとまり
* 広域的な位置関係

など、segmentationに利用できる特徴をembeddingに保持します。

---

## 5. なぜImage Encoderが重いのか

TransformerベースのImage Encoderは、SAMの中で計算量の大きい部分です。

一方、Prompt EncoderとMask Decoderは比較的軽量です。

そのため、実際のinteractive segmentationでは、

```text
最初の1回
Image → Image Encoder → Image embedding
```

を計算し、その後は、

```text
新しいpoint
    +
保存済みimage embedding
    ↓
Mask Decoder
    ↓
新しいmask
```

と処理します。

つまり、クリックするたびに画像全体をImage Encoderへ通しているわけではありません。

この設計によって、SAMはブラウザやviewer上での対話的操作に適したモデルになっています。

---

## 6. Prompt Encoderの役割

Prompt Encoderは、人間が入力したpromptをembeddingに変換します。

[
E_P = f_{\mathrm{prompt}}(P)
]

ここで、

* (P)：point、box、maskなどのprompt
* (E_P)：prompt embedding

です。

SAMではpromptを、大きく次の2種類に分けています。

* sparse prompt
* dense prompt

---

## 7. Sparse prompt

Sparse promptは、少数の座標や領域指定で表現できるpromptです。

代表例は、

* positive point
* negative point
* bounding box

です。

### 7.1 Point prompt

point promptには、少なくとも次の情報があります。

* x座標
* y座標
* foregroundかbackgroundか

例えば、

```text
(x, y, label)
```

として、

```text
(420, 310, foreground)
```

のように表現できます。

ただし、座標値をそのままTransformerへ入力するわけではありません。

SAMでは座標位置に対してpositional encodingを行い、さらにpositive pointかnegative pointかを表すlearned embeddingを加えます。

概念的には、

[
E_{\mathrm{point}}
==================

E_{\mathrm{position}}(x,y)
+
E_{\mathrm{label}}
]

となります。

ここで、

* (E_{\mathrm{position}})：点の位置
* (E_{\mathrm{label}})：foregroundまたはbackground

を表します。

つまりSAMは、

```text
どこをクリックしたか
```

だけでなく、

```text
その点を含めたいのか、除外したいのか
```

も区別しています。

---

## 8. Bounding box prompt

bounding boxは、一般に2つの角で表現できます。

[
B = (x_1,y_1,x_2,y_2)
]

SAMでは、矩形の左上と右下に対応する座標をpositional encodingし、それぞれに異なるlearned embeddingを加えます。

概念的には、

```text
top-left corner
        +
bottom-right corner
        ↓
box embedding
```

となります。

box promptは、対象の大まかな位置と範囲を同時に与えられるため、point promptより安定することがあります。

特に、

* 対象が小さい
* 周囲に似た構造が多い
* 画像内に複数の候補がある

場合に有効です。

医用画像でも、単一pointよりbounding boxの方が良好な結果を示す場合があります。ただし、その性能は対象の大きさ、境界の明瞭さ、画像domainによって大きく変わります。

---

## 9. Dense prompt

dense promptの代表例はmask promptです。

mask promptでは、前回得られたmaskや粗いmaskを、新しい推論の入力として利用します。

```text
Previous mask
      +
new point
      ↓
SAM
      ↓
refined mask
```

これはinteractive segmentationで重要です。

例えば最初のpointで得られたmaskが一部不足していた場合、

1. 現在のmaskをSAMへ再入力する
2. positive pointを追加する
3. 新しいmaskを生成する

という反復ができます。

mask promptは画像と同様に空間的な広がりを持つため、sparse tokenではなく、dense embeddingとしてImage Encoderの特徴マップへ加えられます。

---

## 10. Mask Decoderは何をしているのか

Mask Decoderは、image embeddingとprompt embeddingを組み合わせてmaskを生成します。

[
(\hat{M},\hat{q})
=================

f_{\mathrm{decoder}}(E_I,E_P)
]

ここで、

* (\hat{M})：予測mask
* (\hat{q})：mask qualityの予測値
* (E_I)：image embedding
* (E_P)：prompt embedding

です。

SAMのMask Decoderは、Transformer decoderを基盤とした軽量な構造です。

概念的には、

```text
Image tokens
     ↕
Prompt tokens
     ↓
Two-way interaction
     ↓
Mask tokens
     ↓
Predicted masks
```

という処理を行います。

---

## 11. Two-Way Transformer

SAMのMask Decoderでは、prompt側から画像を見るだけでなく、画像側もprompt情報によって更新されます。

これを論文ではtwo-way Transformer decoderとして説明しています。

一般的なcross-attentionを単純化すると、

```text
prompt query
    ↓
image featuresを検索
```

となります。

SAMではさらに、

```text
prompt → image
image → prompt
```

という双方向の情報交換を行います。

これによって、

* クリックされた位置の周囲に何があるか
* box内部のどの境界が対象らしいか
* negative pointによってどの領域を除外するか

を統合します。

---

## 12. SAMが複数のmaskを出す理由

1つのpointだけを与えた場合、ユーザーが何を指しているかは必ずしも一意ではありません。

例えば人物の服をクリックした場合、そのpointは、

* 服の一部
* 服全体
* 人物全体
* 人物を含む前景全体

を意味している可能性があります。

OCTでも同様に、液体領域の内部を1点クリックした場合、

* クリック周囲の小さな低反射領域
* 連続したSRF全体
* 網膜剥離領域全体

など、異なる粒度の解釈が可能です。

この問題を **prompt ambiguity** と呼びます。

SAMは曖昧なpromptに対して、複数の妥当なmaskを出力するよう設計されています。

```text
1 point
   ↓
SAM
   ├─ Mask 1：部分
   ├─ Mask 2：物体全体
   └─ Mask 3：より広い領域
```

したがって、複数maskの出力は失敗ではなく、曖昧性への対応です。

---

## 13. Multimask output

公式実装の`SamPredictor.predict()`には、`multimask_output`という設定があります。

```python
masks, scores, logits = predictor.predict(
    point_coords=point_coords,
    point_labels=point_labels,
    multimask_output=True,
)
```

`multimask_output=True`では、曖昧なpromptに対して複数のmask候補が返されます。

一方、

```python
multimask_output=False
```

では、基本的に単一のmaskが返されます。

### 使い分け

最初のクリックだけの場合は、対象が曖昧になりやすいため、

```python
multimask_output=True
```

が有用です。

複数のpositive pointやnegative pointを追加して対象が明確になった後は、

```python
multimask_output=False
```

でも安定しやすくなります。

---

## 14. Predicted IoUとは何か

SAMはmaskだけでなく、そのmaskの品質を予測するscoreも出力します。

公式実装では、一般に次のような値を受け取ります。

```python
masks, scores, logits
```

ここで、

* `masks`：二値化されたmask
* `scores`：predicted IoU
* `logits`：二値化前のmask logits

です。

IoUは、予測maskと正解maskの重なりを表す指標です。

[
\mathrm{IoU}
============

\frac{|M_{\mathrm{pred}}\cap M_{\mathrm{true}}|}
{|M_{\mathrm{pred}}\cup M_{\mathrm{true}}|}
]

しかし、推論時には通常、正解maskは存在しません。

そこでSAMは、

> この予測maskが正解maskとどの程度重なると考えられるか

を自分で予測します。

これがpredicted IoUです。

---

## 15. Predicted IoUは真の精度ではない

predicted IoUは、実際にground truthと比較して計算されたIoUではありません。

あくまでモデルが予測したmask qualityです。

したがって、

```text
predicted IoU = 0.95
```

であっても、

```text
実際のDiceやIoUが高い
```

とは限りません。

特に、SAMの学習domainと大きく異なる画像では注意が必要です。

例えば、

* OCT
* MRI
* CT
* 超音波
* 病理画像

では、predicted IoUのcalibrationが自然画像と異なる可能性があります。

研究でSAMを評価するときには、predicted IoUだけでなく、

* Dice coefficient
* IoU
* sensitivity
* precision
* Hausdorff distance
* surface distance

などをground truthと比較して評価する必要があります。

---

## 16. Mask logitsとは何か

SAMが内部で最初から0と1の二値maskを生成しているわけではありません。

各位置について、

```text
対象である可能性が高い
対象である可能性が低い
```

という連続値を出力します。

これがmask logitsです。

概念的には、

[
L(x,y)\in\mathbb{R}
]

として、thresholdを適用して二値maskへ変換します。

[
M(x,y)
======

\begin{cases}
1 & L(x,y) > t\
0 & L(x,y) \le t
\end{cases}
]

ここで、

* (L(x,y))：mask logit
* (t)：threshold
* (M(x,y))：二値mask

です。

公式実装では、通常はSAM内部で設定されたthresholdによってmaskが二値化されます。

---

## 17. なぜlogitsを再利用するのか

interactive segmentationでは、前回のlow-resolution mask logitsを次のmask promptとして再利用できます。

```python
masks, scores, logits = predictor.predict(
    point_coords=point_coords,
    point_labels=point_labels,
    multimask_output=True,
)

best_index = scores.argmax()
previous_logits = logits[best_index]
```

次にpointを追加して、

```python
masks, scores, logits = predictor.predict(
    point_coords=new_point_coords,
    point_labels=new_point_labels,
    mask_input=previous_logits[None, :, :],
    multimask_output=False,
)
```

とします。

ここで二値maskではなくlogitsを再入力するのは、境界付近の不確実性を保持できるためです。

二値maskでは、

```text
0 または 1
```

しかありません。

一方、logitsには、

```text
明らかに背景
背景寄り
境界付近
前景寄り
明らかに前景
```

という連続的な情報が残っています。

---

## 18. SamPredictorの基本的な処理

公式repositoryでは、interactive prediction用に`SamPredictor`が用意されています。公式repositoryには推論コード、学習済みcheckpoint、example notebookが含まれています。

概念的な使用方法は次のとおりです。

```python
import numpy as np
import torch
from segment_anything import sam_model_registry, SamPredictor

device = "cuda" if torch.cuda.is_available() else "cpu"

sam = sam_model_registry["vit_b"](
    checkpoint="sam_vit_b_01ec64.pth"
)
sam.to(device=device)

predictor = SamPredictor(sam)
```

画像を設定します。

```python
predictor.set_image(image)
```

この時点でImage Encoderが実行され、image embeddingが保存されます。

次にpromptを指定します。

```python
point_coords = np.array([
    [500, 300],
])

point_labels = np.array([
    1,
])
```

`1`はforeground point、`0`はbackground pointを表します。

```python
masks, scores, logits = predictor.predict(
    point_coords=point_coords,
    point_labels=point_labels,
    multimask_output=True,
)
```

---

## 19. 座標系に注意する

SAMへpointを渡すときには、座標順序に注意が必要です。

多くのNumPy画像では、配列のindexは、

```python
image[y, x]
```

です。

しかし、SAMへ入力するpoint coordinateは通常、

```python
[x, y]
```

です。

つまり、

```python
point_coords = np.array([
    [x, y],
])
```

とします。

napariではlayerによって座標順序が、

```text
z, y, x
```

または、

```text
y, x
```

で扱われます。

したがって、napariからSAMへ座標を渡す際には、

```text
napari座標
    ↓
軸の順序を変換
    ↓
SAM座標
```

という処理が必要です。

この座標変換を誤ると、クリック位置とは異なる場所がpromptとして入力されます。

---

## 20. 画像の前処理

SAMへ入力する画像は、通常RGB画像として扱われます。

しかしOCT B-scanは多くの場合、grayscaleです。

例えば画像が、

```python
gray.shape == (height, width)
```

の場合、そのままでは期待される入力形式と異なることがあります。

簡単な変換方法は、同じgrayscale画像を3 channelへ複製することです。

```python
rgb = np.stack(
    [gray, gray, gray],
    axis=-1,
)
```

結果は、

```python
rgb.shape == (height, width, 3)
```

となります。

ただし、単純な3 channel化によって自然画像とのdomain gapが解消されるわけではありません。

---

## 21. SAM内部での画像resize

SAMでは、入力画像の長辺が所定の長さになるようresizeされます。

その後、必要に応じてpaddingされ、Image Encoderへ入力されます。

概念的には、

```text
Original image
      ↓
Resize longest side
      ↓
Padding
      ↓
Image Encoder
```

となります。

pointやbounding boxの座標も、画像のresizeに合わせて変換する必要があります。

`SamPredictor`を使用する場合、この座標変換は内部で処理されます。

一方、SAMの内部APIを直接扱う場合には、画像とpromptの座標変換を自分で管理する必要があります。

---

## 22. ViT-B、ViT-L、ViT-Hの違い

original SAMには、主に次のImage Encoder variantがあります。

* ViT-B
* ViT-L
* ViT-H

一般的には、

```text
ViT-B < ViT-L < ViT-H
```

の順にモデルが大きくなります。

大きなモデルほど、通常は、

* パラメータ数が多い
* VRAM使用量が多い
* image embeddingの計算が重い
* 推論に時間がかかる

という傾向があります。

### 実用的な選択

ローカルPCでnapariと組み合わせる場合は、まずViT-Bで動作確認するのが合理的です。

```text
ViT-B
  ↓
座標入力、mask表示、保存を実装
  ↓
必要に応じてViT-LまたはViT-H
```

モデルサイズを大きくしても、domain gapによる誤りが自動的に解決するとは限りません。

医用画像では、モデルサイズよりも、

* 適切なprompt
* 入力画像の前処理
* fine-tuning
* domain-specific training
* 後処理

の方が重要になる場合があります。

---

## 23. Automatic Mask Generatorとは何か

SAMはpointやboxを手動入力するだけでなく、画像全体から多数のmask候補を自動生成できます。

公式実装では`SamAutomaticMaskGenerator`が用意されています。

```python
from segment_anything import SamAutomaticMaskGenerator

mask_generator = SamAutomaticMaskGenerator(sam)
masks = mask_generator.generate(image)
```

これは、

> SAMが画像全体を一度に意味的に理解して、すべての物体名を認識している

という単純な処理ではありません。

実際には、画像上に多数のpoint promptを配置し、それぞれに対してmask候補を生成します。

```text
Image
  ↓
格子状にpointを配置
  ↓
各pointから複数maskを生成
  ↓
品質の低いmaskを除外
  ↓
重複maskを除外
  ↓
最終mask集合
```

---

## 24. Automatic Mask Generatorの主要パラメータ

Automatic Mask Generatorでは、複数のパラメータが結果に影響します。

### `points_per_side`

画像の各辺に何個のpointを配置するかを決めます。

```text
points_per_side = 4

●   ●   ●   ●
●   ●   ●   ●
●   ●   ●   ●
●   ●   ●   ●
```

値を大きくすると、小さな対象も検出しやすくなる一方、

* 計算時間
* GPU使用量
* mask候補数

が増加します。

### `pred_iou_thresh`

predicted IoUが一定未満のmaskを除外します。

```text
predicted IoU < threshold
           ↓
        discard
```

### `stability_score_thresh`

thresholdを少し変化させてもmask形状が安定しているかを評価します。

境界が不安定なmaskは除外されやすくなります。

### `min_mask_region_area`

指定面積より小さい領域を除去するために利用されます。

小さなnoiseを除外するのに有効ですが、小さな病変自体を消してしまう可能性があります。

---

## 25. Stability scoreとは何か

mask logitsのthresholdを少し変えたときに、maskが大きく変化する場合、そのmaskは不安定と考えられます。

例えば、

```text
threshold A → 大きなmask
threshold B → ほとんど消失
```

となる領域は、モデルの確信度が低い可能性があります。

一方、

```text
threshold A → ほぼ同じmask
threshold B → ほぼ同じmask
```

なら、安定したmaskと考えられます。

SAMのAutomatic Mask Generatorでは、この性質を利用して不安定なmaskを除外します。

ただし、医用画像の淡い境界や低contrast病変では、臨床的に正しい領域でもstability scoreが低くなる可能性があります。

---

## 26. Non-Maximum Suppression

多数のpointからmaskを生成すると、同じ対象に対して重複したmaskが多数作られます。

```text
Mask A：SRF全体
Mask B：ほぼ同じSRF
Mask C：少し広いSRF
```

これらをすべて残すと冗長です。

そこで、重複度の高い候補から代表的なmaskを残す処理が行われます。

このような重複除去にはNon-Maximum Suppression（NMS）の考え方が使われます。

```text
重複したmask候補
       ↓
scoreが高いmaskを保持
       ↓
強く重複するmaskを除去
```

---

## 27. Crop layer

Automatic Mask Generatorでは、画像全体だけでなく、画像をcropして再度mask生成することもできます。

```text
Whole image
    +
local crops
    ↓
small objectsを検出
```

元画像全体を縮小すると、小さな対象の情報が失われる可能性があります。

そこで局所領域を拡大して処理することで、小さな対象を検出しやすくします。

ただしcrop数を増やすと、計算時間も大きく増加します。

OCTの小さなcystや薄いSRFを扱う場合、この発想は有用ですが、重複maskや誤検出の管理が必要になります。

---

## 28. SAMの学習目標

SAMは、入力されたpromptに対応するvalid maskを出すよう学習されています。

学習時には、

* point
* box
* mask

などを模擬的に生成し、それらに対する正解maskを学習します。

重要なのは、1つの曖昧なpromptに対して、必ずしも唯一の正解が存在しないことです。

例えば物体内部の1点だけでは、

```text
part
object
group
```

のいずれを意味するか決められません。

SAMでは複数maskを予測し、そのうち少なくとも1つが妥当なmaskになるように学習します。

---

## 29. Loss functionの考え方

SAMのmask学習では、focal lossとDice lossが組み合わされています。

概念的には、

[
\mathcal{L}_{\mathrm{mask}}
===========================

\lambda_1\mathcal{L}*{\mathrm{focal}}
+
\lambda_2\mathcal{L}*{\mathrm{dice}}
]

です。

### Focal loss

foregroundとbackgroundの不均衡に対応し、難しいピクセルを重視します。

病変領域が画像全体に対して小さい場合、単純なpixel accuracyでは、

```text
すべて背景
```

と予測しても高いaccuracyになる可能性があります。

focal lossは、このようなclass imbalanceへの対応に利用されます。

### Dice loss

予測maskと正解maskの重なりを直接評価します。

Dice coefficientは、

[
\mathrm{Dice}
=============

\frac{2|M_{\mathrm{pred}}\cap M_{\mathrm{true}}|}
{|M_{\mathrm{pred}}|+|M_{\mathrm{true}}|}
]

です。

Dice lossは一般に、

[
\mathcal{L}_{\mathrm{dice}}
===========================

1-\mathrm{Dice}
]

という形で表現できます。

---

## 30. SA-1B dataset

SAMの汎用性を支えている重要な要素がSA-1B datasetです。

SA-1Bは、約1,100万枚の画像と10億を超えるmaskを含む大規模segmentation datasetとして構築されました。

従来のsegmentation datasetと比較して、mask数が極めて多いことが特徴です。

ただし、

> 大量のmaskがあるから、医学的構造も完全に理解している

という意味ではありません。

SA-1Bの中心は一般画像であり、OCTに固有の、

* 網膜層構造
* speckle noise
* 液体貯留
* shadow artifact
* motion artifact

を専門的に学習したdatasetではありません。

---

## 31. Data engine

10億を超えるmaskをすべて人間が最初から手作業で描くことは現実的ではありません。

そこでSAMでは、model-assisted annotationを段階的に利用するdata engineが構築されました。

概念的には、

```text
人間がmaskを作る
      ↓
初期モデルを学習
      ↓
モデルがannotationを補助
      ↓
より多くのmaskを収集
      ↓
モデルを改善
      ↓
さらにannotationを効率化
```

という循環です。

これは医用画像研究にも応用できる考え方です。

```text
少数の手動annotation
       ↓
初期segmentation model
       ↓
model-assisted annotation
       ↓
人間が修正
       ↓
教師データを拡大
       ↓
モデルを再学習
```

つまりSAMの価値は、最終的なsegmentation modelとしてだけでなく、**教師データ作成を加速するannotation tool**としても大きいと考えられます。

---

## 32. Zero-shot segmentationとは何か

zero-shotとは、対象taskの専用教師データで追加学習せずに、そのtaskへ適用することです。

SAMの場合、

```text
一般画像による大規模学習
        ↓
追加学習なし
        ↓
新しい画像domainへ適用
```

という状況をzero-shot transferと呼びます。

SAMの原著では、さまざまなsegmentation taskに対するzero-shot transferが評価され、専用学習モデルと競合する性能を示すtaskも報告されました。

ただしzero-shotは、

```text
未知domainでも必ず高精度
```

という意味ではありません。

正確には、

```text
対象domainの教師データで追加学習していない
```

という学習条件を表します。

---

## 33. Domain gap

SAMを医用画像へ適用するときの最大の問題の一つがdomain gapです。

### 自然画像

* RGB
* 物体境界が比較的明瞭
* textureや色が豊富
* 日常的な物体が中心

### OCT画像

* grayscale
* speckle noise
* 層状構造
* 境界が淡い
* 病変が小さい
* artifactが多い

この分布の違いにより、自然画像で獲得した特徴表現がOCTにそのまま最適とは限りません。

[
P_{\mathrm{train}}(X)
\neq
P_{\mathrm{OCT}}(X)
]

ここで、

* (P_{\mathrm{train}}(X))：SAMの学習画像分布
* (P_{\mathrm{OCT}}(X))：OCT画像分布

です。

---

## 34. 医用画像でSAMが失敗しやすい状況

SAMは一般に、視覚的境界が明確な大きな構造を比較的抽出しやすい一方、

* 小さな対象
* contrastが低い対象
* 境界が不明瞭な対象
* 複数の類似領域
* 薄く細長い構造

では不安定になりやすい傾向があります。

医用画像でSAMを評価した研究でも、対象の大きさや境界の明瞭さが性能に影響し、box promptがpoint promptより有効な場合が報告されています。

OCTでは、特に次のような誤りが想定されます。

### Under-segmentation

```text
実際のSRF全体
██████████████

SAMのmask
    █████
```

### Over-segmentation

```text
実際のSRF
    █████

SAMのmask
██████████████
```

### Leakage

境界を越えて隣接組織へmaskが漏れます。

### Fragmentation

連続する病変が複数の小領域へ分割されます。

### Missing small lesions

小さなIRF cystなどが検出されません。

---

## 35. SAMは2D modelである

original SAMは基本的に2D画像を入力します。

OCT volumeが、

[
V\in\mathbb{R}^{Z\times H\times W}
]

であっても、original SAMへの入力は通常、

[
I_z\in\mathbb{R}^{H\times W}
]

という1枚のB-scanです。

したがって、

```text
slice 100 → mask
slice 101 → mask
slice 102 → mask
```

は、それぞれ独立に処理されます。

SAM自身は、slice 100とslice 101が隣接していることを知りません。

---

## 36. 2D SAMを3D volumeへ使う方法

OCT volumeにSAMを使う場合、いくつかの方法が考えられます。

### 方法1：全sliceを独立に処理

```text
各B-scan
   ↓
SAM
   ↓
maskをstack
```

実装は単純ですが、slice間の連続性が保証されません。

### 方法2：key sliceのみSAMで処理

```text
代表slice
   ↓
SAM annotation
   ↓
隣接sliceへ伝播
```

隣接sliceへの伝播には、

* optical flow
* registration
* interpolation
* tracking
* 前sliceのmaskを次sliceのpromptに利用

などが考えられます。

### 方法3：SAMで教師データを作り、3D modelを学習

```text
SAM-assisted annotation
       ↓
3D training dataset
       ↓
3D U-Netなどを学習
```

最終的に3D整合性が必要なら、この方法は有力です。

---

## 37. Fine-tuningの選択肢

医用画像へSAMを適応させる方法は、すべてのparameterを更新するfull fine-tuningだけではありません。

主な選択肢は、

1. Mask Decoderのみ更新
2. Prompt EncoderとMask Decoderを更新
3. adapterを追加
4. LoRAを利用
5. Image Encoderを含めてfull fine-tuning

です。

---

## 38. Mask Decoderのみを更新する

Image Encoderを固定し、Mask Decoderだけを学習します。

```text
Image Encoder    frozen
Prompt Encoder   frozen
Mask Decoder     trainable
```

### 長所

* 学習parameterが少ない
* GPU memoryを抑えやすい
* 小規模datasetでも試しやすい
* original SAMの特徴を維持しやすい

### 短所

OCT画像に対するImage Encoderの特徴表現そのものが不十分な場合、Mask Decoderだけでは限界があります。

---

## 39. Adapterを使う

既存のImage Encoderのparameterを固定し、小さなtrainable moduleを追加します。

```text
Transformer block
      +
small adapter
```

### 長所

* full fine-tuningより軽量
* 学習parameterを削減できる
* domain adaptationに利用しやすい

### 短所

* adapterの挿入位置や構造を設計する必要がある
* 実装がMask Decoderのみの学習より複雑になる

---

## 40. LoRAを使う

LoRAは、大きな重み行列全体を更新せず、低rankの行列を追加して学習する方法です。

元の重みを、

[
W' = W + \Delta W
]

として、

[
\Delta W = BA
]

と低rank分解します。

ここで、

* (W)：固定された元の重み
* (A,B)：学習する小さな行列

です。

LoRAを使うことで、Image Encoderのattention層などを比較的少ないparameterで適応できます。

---

## 41. Full fine-tuning

Image Encoderを含む全体を更新します。

```text
Image Encoder    trainable
Prompt Encoder   trainable
Mask Decoder     trainable
```

### 長所

domainに最も強く適応できる可能性があります。

### 短所

* 大量のGPU memoryが必要
* 学習時間が長い
* 多くの教師データが必要
* 過学習の可能性
* foundation modelの汎用性を損なう可能性

小規模な単施設OCT datasetでは、最初からfull fine-tuningを選ぶより、

```text
zero-shot evaluation
        ↓
Mask Decoder fine-tuning
        ↓
adapter / LoRA
        ↓
必要ならfull fine-tuning
```

と段階的に進める方が現実的です。

---

## 42. SAMを教師データ作成に使う

SAMの最も実用的な利用法の一つは、annotation支援です。

```text
OCT B-scan
     ↓
人間がpointまたはboxを入力
     ↓
SAMがmask候補を生成
     ↓
人間が修正
     ↓
確定mask
```

このとき、SAMの出力をそのまま正解maskとして保存するのではなく、

> 人間が確認・修正したmaskをground truthとする

ことが重要です。

SAMの出力を無修正で教師データにすると、SAMのsystematic errorが教師データに混入します。

---

## 43. Annotation timeだけで評価しない

SAM-assisted annotationを評価する場合、

```text
手作業より速かった
```

だけでは不十分です。

少なくとも、

* annotation time
* point数
* 修正回数
* final Dice
* intergrader agreement
* intragrader agreement
* failure rate
* manual correction area

などを評価する必要があります。

例えば、

```text
SAM使用時は速いが境界精度が下がった
```

可能性があります。

逆に、

```text
時間差は小さいがgrader間のばらつきが減った
```

可能性もあります。

---

## 44. Prompt strategyを標準化する

SAMの性能は、promptの与え方によって変わります。

研究で比較する場合、prompt strategyを事前に決める必要があります。

例えば、

### Point prompt

```text
病変中心にpositive pointを1個
```

### Multi-point prompt

```text
病変内部にpositive pointを3個
病変外部にnegative pointを2個
```

### Box prompt

```text
病変全体を含む最小bounding box
```

### Iterative prompt

```text
最大誤差領域へ順次pointを追加
```

prompt数を自由にすると、症例ごとの人的判断が大きくなり、再現性が低下します。

---

## 45. 実験単位を明確にする

OCT研究では、評価単位として、

* pixel
* B-scan
* volume
* eye
* patient

が混在します。

例えば、同一眼の512 B-scansを512個の独立sampleとして扱うと、独立性を過大評価する可能性があります。

データ分割は、原則としてpatient levelまたはeye levelで行います。

```text
同一患者のslice
      ↓
trainとtestに分けない
```

同一volumeの隣接sliceは非常によく似ているため、slice level random splitではdata leakageが起きやすくなります。

---

## 46. SAM評価で使用する指標

### Dice coefficient

[
\mathrm{Dice}
=============

\frac{2TP}{2TP+FP+FN}
]

領域の重なりを評価します。

### Intersection over Union

[
\mathrm{IoU}
============

\frac{TP}{TP+FP+FN}
]

### Precision

[
\mathrm{Precision}
==================

\frac{TP}{TP+FP}
]

過剰segmentationが少ないかを評価します。

### Recall

[
\mathrm{Recall}
===============

\frac{TP}{TP+FN}
]

病変を取り逃していないかを評価します。

### Hausdorff distance

境界間の最大距離を評価します。

### Average surface distance

境界間の平均的な距離を評価します。

---

## 47. Diceだけでは不十分な場合

大きな病変では、境界が多少ずれてもDiceが高くなることがあります。

逆に小さな病変では、数pixelのずれでDiceが大きく低下します。

そのため、

```text
Dice
  +
surface distance
  +
volume error
```

のように複数指標を組み合わせる方が適切です。

SRF volumeを研究目的とするなら、

[
\mathrm{Volume\ Error}
======================

V_{\mathrm{pred}}-V_{\mathrm{true}}
]

や、

[
\mathrm{Relative\ Volume\ Error}
================================

\frac{V_{\mathrm{pred}}-V_{\mathrm{true}}}
{V_{\mathrm{true}}}
]

も重要です。

---

## 48. SAMをnapariに統合する構成

napariとSAMを組み合わせる場合、次の構成が考えられます。

```text
Image layer
    │
    ├── OCT volume
    │
Points layer
    │
    ├── positive points
    └── negative points
    │
    ▼
SAM inference
    │
    ▼
Labels layer
    │
    └── segmentation mask
```

処理の流れは、

```text
1. OCT volumeを読み込む
2. 現在のsliceを取得する
3. grayscaleをRGBへ変換する
4. predictor.set_image()を実行する
5. point座標を取得する
6. predictor.predict()を実行する
7. maskをLabels layerへ表示する
8. 人間が確認・修正する
9. maskを保存する
```

です。

---

## 49. Sliceごとにset_imageを繰り返す問題

OCT volumeでは、sliceを変更するたびに異なる画像になります。

そのため、基本的にはsliceごとに、

```python
predictor.set_image(current_slice)
```

を実行する必要があります。

Image Encoderは重いため、頻繁にsliceを移動すると遅延が生じます。

対策として、

* 現在sliceのみembeddingを計算
* 前後sliceを先読み
* 最近使用したembeddingをcache
* すべてのslice embeddingを事前計算

などが考えられます。

ただし全sliceのembeddingを保持すると、RAMやVRAM使用量が増加します。

---

## 50. Embedding cache

簡単なcacheは、辞書として実装できます。

```python
embedding_cache = {}
```

slice indexごとにembeddingを保存します。

```python
if slice_index not in embedding_cache:
    predictor.set_image(image)
    embedding_cache[slice_index] = {
        "features": predictor.features.clone(),
        "input_size": predictor.input_size,
        "original_size": predictor.original_size,
    }
```

ただし、`SamPredictor`の内部状態を直接保存・復元する場合、使用するSAM実装のversionやtensor deviceに注意が必要です。

初期実装では無理に全volumeをcacheせず、

```text
現在slice
前slice
次slice
```

程度のlimited cacheから始める方が安全です。

---

## 51. SAMの出力を3Dで表示する

各sliceのmaskを、

[
M_z(y,x)
]

として保存し、stackすると、

[
M(z,y,x)
]

という3D label volumeになります。

```python
mask_volume[slice_index] = predicted_mask
```

napariのLabels layerへ、

```python
viewer.add_labels(
    mask_volume,
    name="SRF segmentation",
)
```

として表示できます。

ただし3D表示が滑らかでも、それだけで3D segmentationが正確とは限りません。

slice間のmaskの飛びや不連続を確認する必要があります。

---

## 52. 研究用途で保存すべき情報

最終maskだけでなく、少なくとも次の情報を保存すると再現性が高まります。

* original image identifier
* slice index
* positive point coordinates
* negative point coordinates
* bounding box
* SAM model variant
* checkpoint
* software version
* predicted IoU
* selected mask index
* human correctionの有無
* annotation time
* annotator identifier

例えばJSONとして、

```json
{
  "image_id": "case001",
  "slice_index": 125,
  "model": "sam_vit_b",
  "positive_points": [[510, 320]],
  "negative_points": [[480, 270]],
  "predicted_iou": 0.91,
  "human_corrected": true
}
```

のように保存できます。

---

## 53. SAM利用時の再現性

SAMは同じ画像、同じcheckpoint、同じprompt、同じ前処理を使用すれば、基本的には同じ出力を再現できます。

しかし研究全体では、

* 画像resize
* intensity normalization
* grayscaleからRGBへの変換
* prompt位置
* mask選択方法
* threshold
* post-processing

によって結果が変わります。

したがってmethodには、単に、

```text
SAMを使用した
```

と書くだけでは不十分です。

---

## 54. 論文に記載すべき項目

SAMを用いた研究では、次の内容を明記する必要があります。

### Model

* SAMのversion
* model variant
* checkpoint
* repository version

### Input

* image size
* channel数
* normalization
* resize方法

### Prompt

* point、box、maskの種類
* point数
* pointの選択方法
* negative pointの有無

### Output

* multimask outputの有無
* mask選択方法
* threshold
* post-processing

### Human interaction

* annotator数
* 修正方法
* annotation time
* disagreementの解決方法

### Evaluation

* dataset split
* ground truth作成方法
* Dice、IoUなどの指標
* patient levelの統計処理

---

## 55. SAMの限界

SAMには大きな可能性がありますが、次の限界があります。

### 医学的意味を保証しない

maskが視覚的に自然でも、医学的に正しいとは限りません。

### Prompt依存性

異なるannotatorが異なるpointを置くと、異なるmaskが得られます。

### Domain gap

自然画像での性能をそのまま医用画像へ一般化できません。

### 2D処理

original SAMはvolumeの3D連続性を直接利用しません。

### 小病変

小さくcontrastの低い病変では失敗しやすい可能性があります。

### Quality score

predicted IoUは、実際のground truthとのIoUではありません。

### Computational cost

Image Encoderは比較的重く、多数sliceの処理には計算資源が必要です。

---

## 56. SAMをどう位置づけるべきか

SAMを、

```text
何でも自動で正確にsegmentationするAI
```

と捉えると、期待と実際の性能が乖離します。

むしろ、

> 人間のpromptを利用してmask候補を高速に生成する、汎用的なsegmentation基盤

と考える方が適切です。

医用画像では、さらに限定して、

> annotationを支援し、教師データ作成を高速化するhuman-in-the-loop tool

としての価値が大きいと考えられます。

---

## 57. OCT研究における現実的な開発順序

OCTへSAMを導入する場合、次の順序が現実的です。

### Step 1：Zero-shot評価

```text
SAMをそのまま使用
      ↓
代表症例でmaskを確認
```

### Step 2：Interactive annotation

```text
positive / negative point
      ↓
mask修正
      ↓
教師データ作成
```

### Step 3：定量評価

```text
manual annotation
      vs
SAM-assisted annotation
```

評価項目は、

* Dice
* annotation time
* 修正回数
* failure rate

などです。

### Step 4：Domain adaptation

```text
Mask Decoder fine-tuning
adapter
LoRA
```

を比較します。

### Step 5：半自動化

```text
現在sliceのmask
      ↓
隣接sliceへprompt伝播
      ↓
人間が確認
```

### Step 6：専用モデル

蓄積した教師データを用いて、

```text
2D U-Net
3D U-Net
nnU-Net
Transformer-based segmentation model
```

などを学習します。

---

## 58. Original SAM、SAM 2、SAM 3の位置づけ

2026年時点ではSegment Anythingシリーズはoriginal SAMだけではありません。

### Original SAM

* image segmentation
* point、box、mask prompt
* 画像ごとに独立処理
* 基本構造の理解に適している

### SAM 2

* imageとvideoを統一的に扱う
* streaming memoryを導入
* 動画内で対象を追跡
* 連続sliceへの応用を考えやすい構造

SAM 2は、画像を1 frameのvideoとして扱う統一的なpromptable visual segmentation modelとして設計されています。

### SAM 3

* concept prompt
* 短い名詞句
* image exemplar
* matching objectの検出・segmentation・tracking

SAM 3では、単なる視覚的point promptだけでなく、概念を用いたsegmentationへ拡張されています。

ただし、OCTのSRFなど専門的な医用概念が、text promptだけで安定して抽出できることを意味するわけではありません。

---

## まとめ

SAMを中級レベルで理解するうえで重要なのは、次の点です。

1. SAMはpromptable segmentation taskを解くモデルである
2. Image EncoderとMask Decoderを分離することでinteractive inferenceを高速化している
3. pointとboxはpositional encodingを含むsparse promptとして処理される
4. mask promptはdense promptとして処理される
5. Mask Decoderではimageとpromptの情報が双方向に交換される
6. 曖昧なpromptに対応するため複数maskを出力する
7. predicted IoUはモデルが予測した品質であり、実測精度ではない
8. Automatic Mask Generatorは多数のpoint promptと重複除去を組み合わせている
9. original SAMは2D modelであり、3D OCTのslice間連続性を直接利用しない
10. 医用画像ではdomain gapを考慮し、人間による確認が必要である

SAMは、単純な自動segmentation modelではありません。

その本質は、

[
\text{Image features}
+
\text{Human prompt}
\longrightarrow
\text{Candidate masks}
]

というhuman-in-the-loopの設計にあります。

医用画像研究においては、SAM単体にすべてを任せるよりも、

```text
SAM
  +
human correction
  +
domain-specific dataset
  +
dedicated segmentation model
```

という開発サイクルの出発点として利用するのが現実的です。

---

## 参考文献

1. Kirillov A, Mintun E, Ravi N, et al. Segment Anything. Proceedings of the IEEE/CVF International Conference on Computer Vision. 2023.
2. Meta AI Research. Segment Anything.
3. Meta AI Research. Segment Anything Model 2.
4. Ravi N, Gabeur V, Hu YT, et al. SAM 2: Segment Anything in Images and Videos. 2024.
5. Meta AI Research. SAM 3: Segment Anything with Concepts.
