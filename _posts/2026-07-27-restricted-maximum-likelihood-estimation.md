---
title: "Restricted Maximum Likelihood Estimation（REML）とは？線形混合効果モデルで使われる理由を解説"
date: 2026-07-27 00:00:00 +0900
categories: [Statistics, Research]
tags: [REML, Linear Mixed Model, Mixed Effects Model, Variance Components, Statistics, R]
---

Rで線形混合効果モデル（Linear Mixed Model）を実行すると、

```
Linear mixed model fit by REML
```

という表示を目にすることがあります。

初めて見ると、

> REMLとは何だろう？

と思う方も多いのではないでしょうか。

この記事では、Restricted Maximum Likelihood Estimation（REML）の考え方と、なぜ線形混合効果モデルで広く使われているのかを解説します。

---

# REMLとは？

REML（Restricted Maximum Likelihood Estimation）は、

**分散成分（variance components）をより正確に推定するための最尤推定法**

です。

線形混合効果モデルでは、

- 固定効果（Fixed effects）
- ランダム効果（Random effects）

の両方を同時に推定します。

REMLは特に、

**ランダム効果や残差分散を推定する方法**

として広く用いられています。

---

# 最尤法（Maximum Likelihood）との違い

通常の最尤法（Maximum Likelihood; ML）は、

固定効果と分散成分を同時に推定します。

しかし、この方法では、

**固定効果を推定したことによる自由度の減少**

が考慮されません。

その結果、

分散成分がやや小さく見積もられることがあります。

特に症例数が少ない研究では、この影響が無視できません。

---

# REMLの考え方

REMLでは、

まず固定効果の影響を取り除きます。

その後、

残った変動だけを使って分散成分を推定します。

そのため、

自由度を適切に考慮した推定が可能になります。

言い換えると、

MLは

> 「データ全体から分散を推定する」

方法ですが、

REMLは

> 「固定効果を説明した後に残ったばらつきから分散を推定する」

方法です。

---

# なぜREMLの方がよいのか？

サンプルサイズが十分大きい場合、

MLとREMLの違いはほとんどありません。

しかし、

症例数が少ない場合には、

MLでは分散が過小評価される傾向があります。

REMLではこの偏りが補正されるため、

より信頼性の高い分散推定が得られます。

そのため、

医学研究ではREMLが標準的に使用されることが多くなっています。

---

# 線形混合効果モデルとの関係

例えば、

```R
library(lme4)

model <- lmer(
  measurement ~ 1 +
  (1 | subject) +
  (1 | subject:eye),
  data = dat
)
```

を実行すると、

```
Linear mixed model fit by REML
```

と表示されます。

これは、

分散成分がREMLによって推定されたことを意味します。

---

# いつMLを使うのか？

固定効果を比較したい場合には、

MLが推奨されます。

例えば、

モデルA

```
measurement ~ age
```

と

モデルB

```
measurement ~ age + sex
```

を比較したい場合、

AICや尤度比検定では

**ML**

を使用します。

Rでは

```R
lmer(
  formula,
  REML = FALSE
)
```

と指定します。

---

# いつREMLを使うのか？

一方、

分散成分を推定したい場合は、

REMLを使用します。

例えば、

- ICC
- Repeatability
- Sw
- Variance components

などを求める研究では、

REMLが一般的です。

---

# 私の研究での使用例

私たちのRepeatability解析では、

次の線形混合効果モデルを使用しました。

```R
measurement ~ 1 +
(1 | subject) +
(1 | subject:eye)
```

このモデルでは、

REMLにより

- Subject間分散
- Eye間分散
- Residual variance（Sw²）

を推定しました。

推定されたResidual varianceから、

Sw、

Repeatability Coefficient、

95% Difference Threshold

を計算しています。

---

# MLとREMLの使い分け

| 目的 | 推奨される方法 |
|------|----------------|
| 固定効果の比較 | ML |
| AICの比較 | ML |
| 尤度比検定 | ML |
| 分散成分の推定 | REML |
| ICCの計算 | REML |
| Swの計算 | REML |
| Repeatability解析 | REML |

---

# まとめ

REML（Restricted Maximum Likelihood Estimation）は、

固定効果による自由度の減少を考慮しながら分散成分を推定する方法です。

そのため、

ランダム効果や残差分散を評価する研究では、

通常のMaximum Likelihoodよりも信頼性の高い推定が可能になります。

一方で、

固定効果同士を比較する場合には、

MLを使用するのが一般的です。

線形混合効果モデルでは、

**「分散を知りたいならREML、モデルを比較したいならML」**

と覚えておくとよいでしょう。

---

{% include post-footer.html %}
