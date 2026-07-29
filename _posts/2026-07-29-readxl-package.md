---
title: "readxlとは？ExcelファイルをRで読み込むための基本パッケージをわかりやすく解説"
date: 2026-07-29 00:00:00 +0900
categories: [R]
tags: [R, readxl, Excel, データ解析, tidyverse]
---

readxlは、Excelファイル（.xlsx、.xls）をRへ簡単に読み込めるパッケージです。

CSVへ変換することなく直接Excelを読み込めるため、研究やデータ解析で最もよく利用されるパッケージの一つです。

本記事では、readxlの基本的な使い方から実践的な活用方法まで、初心者向けにわかりやすく解説します。

---

# readxlとは？

**readxl**は、ExcelファイルをRに読み込むためのパッケージです。

Rでは多くの解析がデータフレームを対象として行われます。

そのため、最初にExcelデータをRへ読み込む作業が必要になります。

以前はCSVへ変換してから読み込むことが一般的でしたが、現在ではreadxlを使うことでExcelファイルをそのまま読み込めます。

---

# readxlの特徴

readxlには次のような特徴があります。

- Excelファイルを直接読み込める
- `.xlsx`と`.xls`の両方に対応
- インストールが簡単
- 余分な依存パッケージが少ない
- tidyverseとの相性が良い

特に医学研究では、電子カルテや研究データがExcel形式で管理されていることが多く、readxlは非常に便利です。

---

# インストール方法

初めて利用する場合は、

```r
install.packages("readxl")
```

を実行します。

その後、

```r
library(readxl)
```

で読み込みます。

---

# Excelファイルを読み込む

最も基本的な使い方は、

```r
library(readxl)

data <- read_excel("data.xlsx")
```

です。

これだけでExcelファイルがデータフレームとして読み込まれます。

---

# シートを指定する

Excelに複数のシートがある場合は、

```r
data <- read_excel("data.xlsx", sheet = "Sheet2")
```

または

```r
data <- read_excel("data.xlsx", sheet = 2)
```

のように指定できます。

シート名でも番号でも指定可能です。

---

# シート名を確認する

どのようなシートがあるか確認したい場合は、

```r
excel_sheets("data.xlsx")
```

を実行します。

例えば、

```text
[1] "Patient"
[2] "Control"
[3] "Summary"
```

のように表示されます。

---

# 読み込む範囲を指定する

一部だけ読み込みたい場合は、

```r
data <- read_excel(
  "data.xlsx",
  range = "A1:E100"
)
```

と指定できます。

必要な範囲だけ読み込めるため、大きなExcelファイルでは便利です。

---

# 列名がない場合

列名が存在しないデータでは、

```r
data <- read_excel(
  "data.xlsx",
  col_names = FALSE
)
```

を利用します。

すると、

```
...1
...2
...3
```

のような列名が自動で付けられます。

---

# 読み込む列の型を指定する

通常は自動判定されますが、

必要に応じて

```r
data <- read_excel(
  "data.xlsx",
  col_types = c(
    "text",
    "numeric",
    "date"
  )
)
```

のように指定できます。

これにより、文字列や日付を意図した型で読み込めます。

---

# 読み込んだデータを確認する

読み込み後は、

```r
head(data)
```

で先頭6行を確認します。

構造を確認するには、

```r
str(data)
```

が便利です。

さらに、

```r
summary(data)
```

を実行すると、各変数の概要を確認できます。

---

# dplyrと組み合わせる

readxlは、tidyverseのdplyrと組み合わせることでさらに便利になります。

例えば、

```r
library(readxl)
library(dplyr)

data <- read_excel("data.xlsx")

data %>%
  filter(Group == "Control")
```

のように、そのままデータ解析へ進めます。

---

# 医学研究での活用例

医学研究では、

- 患者背景
- OCT測定値
- 血液検査結果
- 視力データ
- アンケート結果

などをExcelで管理していることが多くあります。

readxlを利用すれば、

解析用にCSVへ変換する必要がなく、Excelファイルを直接Rへ読み込めます。

これにより、データ管理の手間を減らし、解析作業を効率化できます。

---

# よくあるエラー

## ファイルが見つからない

```
Error: path does not exist
```

ファイル名や保存場所を確認しましょう。

作業フォルダ（Working Directory）が正しいかも重要です。

---

## シート名が違う

```
Sheet 'Data' not found
```

シート名のスペルや大文字・小文字を確認してください。

まず

```r
excel_sheets("data.xlsx")
```

でシート名を確認すると安心です。

---

## 列の型がおかしい

数値が文字列として読み込まれることがあります。

この場合は、

- Excel側のセルの書式
- 欠損値
- col_types

を確認しましょう。

---

# readxlとopenxlsxの違い

Excelを扱うパッケージにはreadxl以外にもopenxlsxがあります。

| パッケージ | 主な用途 |
|------------|----------|
| readxl | Excelを読み込む |
| openxlsx | Excelを読み書き・編集する |

解析だけならreadxlで十分ですが、

解析結果をExcelへ出力したい場合はopenxlsxが便利です。

---

# まとめ

readxlは、ExcelファイルをRへ直接読み込めるシンプルで高速なパッケージです。

医学研究やデータ解析ではExcel形式のデータを扱う機会が多く、readxlは最初に覚えておきたいパッケージの一つです。

dplyrやggplot2などのtidyverseと組み合わせることで、データの読み込みから前処理、解析、可視化までを効率よく進められるようになります。
