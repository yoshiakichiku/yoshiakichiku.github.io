---
title: "線形混合効果モデル（LMM）から分散成分を推定してICCを求める方法"
date: 2026-07-28 00:00:00 +0900
categories: [Statistics, Research]
tags: [LMM, Linear Mixed Model, ICC, Variance Components, Repeatability, R]
math: true
---

線形混合効果モデル（LMM）を用いると、データに含まれるばらつきを複数の要因に分解し、それぞれの分散成分を推定できます。
ICC（Intraclass Correlation Coefficient）は、この分散成分を利用して算出できるため、再現性や信頼性の評価によく用いられます。
本記事では、LMMから分散成分を推定し、ICCを求める考え方を数式とともに解説します。

# なぜ分散成分を推定するのか？

繰り返し測定データでは、観測値のばらつきは1種類ではありません。

例えばOCTで同じ眼を3回測定した場合、観測値の違いには次のような要因があります。

- 被験者ごとの違い
- 左右眼ごとの違い
- 同じ眼を繰り返し測定したときの測定誤差

これらを1つの分散として扱ってしまうと、どこにばらつきの原因があるのか分かりません。

そこでLMMを用いて、それぞれの分散成分を推定します。

---

# 線形混合効果モデル

例えば次のモデルを考えます。

$$
\mathrm{measurement}
\sim
1
+
(1|\mathrm{subject})
+
(1|\mathrm{subject:eye})
$$

このモデルでは、

- **subject**：被験者
- **subject:eye**：被験者内の左右眼

をランダム効果として扱います。

このモデルから推定される分散は、

- 被験者間分散
- 眼間分散
- 残差分散

の3つです。

---

# 分散成分とは？

推定される分散成分は

$$
\sigma^2_{subject}
$$

$$
\sigma^2_{eye}
$$

$$
\sigma^2_{error}
$$

です。

それぞれ

- 被験者によるばらつき
- 左右眼によるばらつき
- 同一眼の繰り返し測定による誤差

を表しています。

---

# 総分散

観測値全体の分散は

$$
\sigma^2_{total}
=
\sigma^2_{subject}
+
\sigma^2_{eye}
+
\sigma^2_{error}
$$

と考えられます。

LMMでは、この総分散を各要因へ分解して推定します。

---

# ICCとは？

ICC（Intraclass Correlation Coefficient）は、

「観測値全体のばらつきのうち、どの程度が真の個体差によるものか」

を示す指標です。

一般には

$$
ICC
=
\frac{\sigma^2_{between}}
{\sigma^2_{between}
+
\sigma^2_{within}}
$$

と表されます。

値は0〜1になります。

---

# Eye ICCの計算

左右眼を含むモデルでは、

眼単位で評価するICCは

$$
ICC_{eye}
=
\frac{
\sigma^2_{subject}
+
\sigma^2_{eye}
}{
\sigma^2_{subject}
+
\sigma^2_{eye}
+
\sigma^2_{error}
}
$$

となります。

つまり、

**眼ごとの差による分散が、全体の分散に占める割合**

を意味します。

---

# なぜ残差分散だけを誤差と考えるのか？

LMMでは、

残差分散

$$
\sigma^2_{error}
$$

は

同じ眼を何度測定しても生じる測定誤差

を表します。

したがって、

再現性を評価するときには

この残差分散が最も重要になります。

例えば

- Sw（Within-subject SD）
- Repeatability Coefficient
- 測定誤差

はいずれも残差分散から計算されます。

---

# Rでの実装例

lme4パッケージでは

```r
library(lme4)

fit <- lmer(
  measurement ~ 1 +
    (1 | subject) +
    (1 | subject:eye),
  data = data
)
```

でモデルを構築できます。

分散成分は

```r
VarCorr(fit)
```

で取得できます。

例えば

```text
subject      14.2
subject:eye   7.5
Residual      2.3
```

という結果が得られた場合、

Eye ICCは

```r
icc_eye <-
(14.2^2 + 7.5^2) /
(14.2^2 + 7.5^2 + 2.3^2)
```

で計算できます。

---

# 従来のANOVAとの違い

ICCは古くからANOVAでも計算されてきました。

しかしLMMには次のような利点があります。

- 不均衡データに対応できる
- 欠測値を扱える
- 多段階のランダム効果を扱える
- REMLによる安定した分散推定が可能

現在では、繰り返し測定データや階層構造をもつデータではLMMを用いてICCを算出する方法が広く利用されています。

---

# 臨床研究での応用

眼科研究では、

- OCT網膜厚
- 脈絡膜厚
- 血流指標
- 視野検査
- ERG

などの再現性評価によく利用されます。

特に

- 被験者
- 左右眼
- 繰り返し測定

という階層構造をもつデータでは、LMMによる分散成分の推定が非常に有用です。

---

# まとめ

LMMでは観測値のばらつきを被験者間分散、眼間分散、残差分散に分解できます。

ICCはこれらの分散成分から計算され、測定値の信頼性や再現性を評価する代表的な指標です。

近年では、不均衡データや階層構造を扱える利点から、ANOVAよりもLMMを用いてICCを推定する方法が広く採用されています。

---

# 参考文献

1. Hedges LV, Hedberg EC. Intraclass correlation values for planning group-randomized trials in education. *Educational Evaluation and Policy Analysis*. 2007;29:60–87.
