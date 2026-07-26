---
title: "Bland–Altman analysisとは？一致性を評価するための基本と読み方"
date: 2026-07-26 00:00:00 +0900
categories: [Statistics, Research]
tags: [Bland-Altman, Agreement, Repeatability, ICC, R]
---

# Bland–Altman analysisとは？

医療研究では、

- 2つの測定方法は同じ結果を示すのか？
- 同じ測定を繰り返したとき、どの程度ばらつくのか？

を評価したい場面が数多くあります。

そのような場面で最も広く用いられている統計手法の一つが**Bland–Altman analysis（ブランド・アルトマン解析）**です。

Bland–Altman解析は1986年にMartin BlandとDouglas Altmanによって提唱され、現在でも医学論文における一致性（agreement）の評価の標準的な方法として広く利用されています。

---

# Bland–Altman解析は何を評価するのか？

Bland–Altman解析は、

> **2つの測定値がどの程度一致しているか（agreement）**

を評価する方法です。

例えば、

- 新しい検査機器と従来機器の比較
- 同じ検査を2回行った場合の再現性
- 2人の測定者間の一致性

などを評価する際によく用いられます。

---

# 相関係数では一致性は評価できない

「相関係数が高いから一致している」と考えてしまうのはよくある誤解です。

例えば次のようなデータを考えてみます。

| Method A | Method B |
|---------:|---------:|
|100|110|
|120|130|
|140|150|
|160|170|

この2つの測定値の相関係数は

**r = 1.00**

になります。

しかしMethod Bは常に10だけ高い値を示しています。

つまり、

- 相関は非常に高い
- しかし一致していない

という状態です。

Bland–Altman解析では「差」を評価することで、この問題を解決できます。

---

# Bland–Altman plotとは？

Bland–Altman plotでは、

横軸に

**2つの測定値の平均**

縦軸に

**2つの測定値の差**

をプロットします。

つまり、

$$
\text{Mean}=\frac{A+B}{2}
$$

$$
\text{Difference}=A-B
$$

となります。

このグラフから、

- 系統誤差（bias）
- 測定誤差
- 外れ値
- 比例誤差

を視覚的に評価できます。

---

# Bland–Altman解析で重要な3つの指標

## 1. Bias（平均差）

Biasは

$$
\text{Bias}=\overline{A-B}
$$

で計算されます。

Biasは平均的なずれを意味します。

例えば

Bias = +2 μm

なら、

Method Aは平均して2 μm厚く測定していることになります。

Biasが0に近いほど、系統誤差が少ないことを意味します。

---

## 2. Limits of Agreement（LoA）

最も重要な指標が

**95% Limits of Agreement**

です。

計算式は

$$
\text{LoA}=\text{Bias}\pm1.96\times SD_{\text{difference}}
$$

となります。

例えば

Bias = 1 μm

SD = 5 μm

なら

Lower LoA

$$
1-1.96\times5=-8.8
$$

Upper LoA

$$
1+1.96\times5=10.8
$$

となります。

つまり、

95%の測定差は

**−8.8〜10.8 μm**

の範囲に入ると考えられます。

LoAが狭いほど一致性が高いことを意味します。

---

## 3. 外れ値

95% LoAの外側にあるデータは、

通常よりも大きな測定誤差を示している可能性があります。

例えば、

- 固視ずれ
- 測定ミス
- 画像品質の低下

などが原因であることがあります。

---

# Bland–Altman plotの見方

理想的なBland–Altman plotでは、

- Biasが0付近
- 点がランダムに散らばる
- Limits of Agreementが狭い

という特徴があります。

逆に、

- Biasが大きい
- LoAが広い
- 測定値が大きくなるにつれて差も大きくなる

場合は、一致性に問題がある可能性があります。

---

# 比例誤差（Proportional bias）

測定値が大きくなるほど差も大きくなる場合があります。

例えば、

小さい値では差が2 μmしかないのに、

大きな値では20 μmも差が生じるようなケースです。

このような現象を

**比例誤差（proportional bias）**

と呼びます。

比例誤差が疑われる場合には、

- 回帰分析
- Deming regression
- Passing–Bablok regression

なども併せて検討されます。

---

# Repeatability研究との関係

Repeatability（再現性）の研究では、

同じ被験者を複数回測定します。

その測定差を評価するため、

Bland–Altman解析は非常によく利用されます。

例えば、

- OCT
- OCTA
- 眼軸長
- 眼圧
- 網膜厚

など、多くの眼科研究で採用されています。

---

# Repeatability Coefficientとの関係

Repeatability研究では、

Repeatability Coefficient（RC）

もよく報告されます。

RCは

$$
RC=1.96\times\sqrt2\times S_w
$$

で計算されます。

一方、

Bland–Altman解析では

$$
LoA=\pm1.96\times SD_{\text{difference}}
$$

を用います。

繰り返し測定では

$$
SD_{\text{difference}}=\sqrt2\times S_w
$$

となるため、

最終的には

$$
RC=LoA
$$

となります。

つまり、

**Repeatability Coefficientは、Bland–Altman解析における95% Limits of Agreementと数学的に同じ概念です。**

---

# ICCとの違い

ICCも再現性評価でよく用いられますが、

評価している内容は異なります。

| Bland–Altman | ICC |
|--------------|-----|
| 実際にどの程度ずれるか | 個体間の順位がどれだけ保たれるか |
| 単位あり（μmなど） | 単位なし（0〜1） |
| 一致性を評価 | 信頼性を評価 |

例えば、

ICC = 0.99

であっても、

Limits of Agreementが±30 μmであれば、

順位は保たれていても測定誤差は大きいことになります。

そのため、近年の論文では

- ICC
- Bland–Altman解析
- Sw
- Repeatability Coefficient

をセットで報告することが推奨されています。

---

# RでBland–Altman解析を行う

```r
library(ggplot2)

mean_value <- (data$Measurement1 + data$Measurement2) / 2
difference <- data$Measurement1 - data$Measurement2

bias <- mean(difference)
sd_diff <- sd(difference)

upper <- bias + 1.96 * sd_diff
lower <- bias - 1.96 * sd_diff

ggplot(data.frame(mean_value, difference),
       aes(mean_value, difference)) +
  geom_point() +
  geom_hline(yintercept = bias, color = "blue") +
  geom_hline(yintercept = upper, linetype = 2, color = "red") +
  geom_hline(yintercept = lower, linetype = 2, color = "red") +
  theme_classic()
```

---

# まとめ

Bland–Altman解析は、2つの測定値の一致性を評価するための最も基本的な統計手法です。

ポイントは以下の通りです。

- 相関係数では一致性は評価できない
- Biasは平均的なずれを表す
- Limits of Agreementは95%の測定差の範囲を示す
- Repeatability Coefficientと数学的に同じ概念である
- ICCと組み合わせることで、測定の信頼性をより包括的に評価できる

眼科領域では、OCT、眼軸長測定、眼圧測定など、多くの検査機器の再現性評価に利用されており、臨床研究を行う上で理解しておきたい重要な統計手法の一つです。

---

## 関連記事

- ICCとは？
- ICCの種類（ICC(1)、ICC(2)、ICC(3)）
- Sw（Within-subject Standard Deviation）とは？
- Repeatability Coefficientとは？
- CV（Coefficient of Variation）とは？
- RでICCを計算する方法（psychパッケージとirrパッケージの比較）
