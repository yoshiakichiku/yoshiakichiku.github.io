---
title: "Repeatability Coefficientはなぜ2.77×Swなのか？"
date: 2026-07-27 00:00:00 +0900
categories: [Statistics, Research]
tags: [Repeatability, Repeatability Coefficient, Sw, Bland-Altman, Reliability, Statistics]
math: true
---

# Repeatability Coefficientはなぜ2.77×Swなのか？

Repeatability（再現性）の論文では、次の式を目にすることがよくあります。

$$
95\% \text{ difference threshold}
=
1.96 \times \sqrt{2} \times Sw
$$

あるいは

$$
RC = 2.77 \times Sw
$$

という式です。

しかし、

- なぜ **1.96** を掛けるのでしょうか？
- なぜ **√2** が必要なのでしょうか？

この記事では、この式がどのように導かれるのかを順を追って解説します。

---

# Swとは何か？

Sw（Within-subject standard deviation）は、**同じ対象を繰り返し測定したときの測定誤差の標準偏差**です。

例えば、同じ患者の網膜厚を3回測定したとします。

| 測定回数 | 網膜厚 |
|----------|-------:|
| 1回目 | 281 μm |
| 2回目 | 279 μm |
| 3回目 | 282 μm |

測定値は少しずつ異なります。

このばらつきの大きさを表すのがSwです。

つまり、1回の測定誤差は

$$
e \sim N(0,Sw^2)
$$

という正規分布に従うと考えます。

---

# 比較したいのは2回の測定値の差

臨床では、

- 前回と今回
- 測定者Aと測定者B
- 1回目と2回目

のように、**2回の測定値の差**を評価したいことがほとんどです。

そこで、

$$
D=X_1-X_2
$$

という「差」を考えます。

---

# 真の値は打ち消し合う

測定値を

$$
X_1=T+e_1
$$

$$
X_2=T+e_2
$$

と表します。

ここで、

- \(T\)：真の値
- \(e\)：測定誤差

です。

すると、

$$
D=X_1-X_2
$$

は

$$
D=(T+e_1)-(T+e_2)
$$

となり、

$$
D=e_1-e_2
$$

になります。

つまり、**真の値は消え、測定誤差だけが残ります。**

---

# なぜ√2が現れるのか？

ここが最も重要なポイントです。

独立した2つの変数では、

$$
Var(a-b)=Var(a)+Var(b)
$$

という性質があります。

したがって、

$$
Var(D)
=
Var(e_1)+Var(e_2)
$$

となります。

両方ともSw²なので、

$$
Var(D)
=
Sw^2+Sw^2
=
2Sw^2
$$

となります。

標準偏差は分散の平方根なので、

$$
SD(D)
=
\sqrt{2Sw^2}
=
\sqrt2\times Sw
$$

となります。

つまり、

**2回測定値の差の標準偏差は、1回の測定誤差より√2倍大きくなる**のです。

これが式の中の**√2**の意味です。

---

# なぜ1.96を掛けるのか？

差Dは正規分布に従うと仮定しています。

正規分布では、

約95%のデータが

$$
\pm1.96SD
$$

の範囲に入ります。

したがって、

差の95%範囲は

$$
\pm1.96\times\sqrt2\times Sw
$$

となります。

これが

**95% Difference Threshold**

あるいは

**Repeatability Coefficient（RC）**

です。

---

# なぜ2.77になるのか？

論文では

$$
RC=2.77\times Sw
$$

と書かれることもあります。

これは、

$$
1.96\times\sqrt2
=
2.7718
$$

だからです。

つまり、

$$
RC
=
2.77\times Sw
$$

は、

$$
RC
=
1.96\times\sqrt2\times Sw
$$

を簡略化した表現にすぎません。

---

# Bland–Altman解析との関係

Bland–Altman解析では、

95% Limits of Agreementは

$$
\text{Mean difference}
\pm
1.96\times SD_{\text{difference}}
$$

と表されます。

ここで、

$$
SD_{\text{difference}}
=
\sqrt2\times Sw
$$

であれば、

Limits of Agreementの幅は

$$
1.96\times\sqrt2\times Sw
$$

となります。

つまり、

**Repeatability Coefficientは、Bland–Altman解析における「同一対象を繰り返し測定した場合の95%差」と同じ考え方から導かれています。**

---

# まとめ

Repeatability Coefficientの式

$$
RC
=
1.96\times\sqrt2\times Sw
$$

は、次の3つの考え方から導かれます。

1. Swは1回の測定誤差の標準偏差
2. 2回測定値の差の標準偏差は√2×Swになる
3. 正規分布の95%範囲は±1.96SDで表される

この3つを組み合わせることで、

$$
RC
=
1.96\times\sqrt2\times Sw
=
2.77\times Sw
$$

という式が得られます。

この式は経験的に決められたものではなく、**正規分布と分散の性質から数学的に導かれる**ことが分かります。

---

## 関連記事

- Sw（Within-subject Standard Deviation）とは？
- Repeatability Coefficientとは？
- Bland–Altman解析とは？
- ICC（Intraclass Correlation Coefficient）とは？
- CV（Coefficient of Variation）とは？

{% include post-footer.html %}
