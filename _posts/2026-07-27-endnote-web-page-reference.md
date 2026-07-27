---
title: "EndNoteでホームページやGitHubを参考文献として登録する方法"
date: 2026-07-27 00:00:00 +0900
categories: [Research, EndNote]
tags: [EndNote, GitHub, Web Page, Reference, Citation]
---

# EndNoteでホームページやGitHubを参考文献として登録する方法

近年では、論文だけでなくWebサイトやGitHub Repositoryを参考文献として引用する機会が増えています。

例えば、

- GitHub Repository
- 研究者のホームページ
- ソフトウェアの配布ページ
- 学会や行政機関のWebサイト
- オンラインマニュアル

などを引用することがあります。

しかし、EndNoteを使い始めたばかりの頃は、

> 「Webサイトはどう登録すればいいの？」

と迷う方も多いのではないでしょうか。

この記事では、EndNoteでホームページやGitHubを参考文献として登録する方法を解説します。

---

# Step 1. 新しいReferenceを作成する

EndNoteを開き、

**References → New Reference**

を選択します。

デフォルトでは **Journal Article** になっています。

---

# Step 2. Reference Typeを「Web Page」に変更する

画面上部の **Reference Type** を

```
Journal Article
```

から

```
Web Page
```

へ変更します。

これが最も重要なポイントです。

---

# Step 3. 必要な情報を入力する

代表的な入力項目は以下のとおりです。

| 項目 | 内容 |
|------|------|
| Author | 著者または組織名 |
| Year | 公開年または更新年 |
| Title | ページタイトル |
| Website Title | Webサイト名 |
| URL | ページのURL |
| Access Date | 閲覧日 |

すべて入力できなくても問題ありません。

分かる範囲で入力しておけば、多くの引用スタイルで適切に表示されます。

---

# GitHub Repositoryを登録する例

例えばGitHub Repositoryを登録する場合は、次のようになります。

| 項目 | 入力例 |
|------|---------|
| Author | Chiku Y |
| Year | 2026 |
| Title | OCT-repeatability-analysis |
| Website Title | GitHub |
| URL | https://github.com/username/OCT-repeatability-analysis |
| Access Date | July 27, 2026 |

GitHubで公開している解析コードやソフトウェアも、この方法で管理できます。

---

# 研究者ホームページを登録する例

研究者のホームページであれば、例えば次のようになります。

| 項目 | 入力例 |
|------|---------|
| Author | Yoshiaki Chiku |
| Year | 2026 |
| Title | Yoshiaki Chiku Research Blog |
| Website Title | GitHub Pages |
| URL | https://yoshiakichiku.github.io/ |
| Access Date | July 27, 2026 |

---

# Wordへ引用する方法

登録後は、通常の論文と同じようにWordから引用できます。

WordのEndNoteタブで

**Insert Citation**

をクリックし、登録したWebページを選択するだけです。

論文と同じように参考文献リストへ追加されます。

---

# 表示形式は雑誌ごとに異なる

EndNoteでは、投稿先雑誌の引用スタイル（Output Style）に応じて、自動的に参考文献の形式が変更されます。

例えば、

- JJO
- Scientific Reports
- IOVS
- Nature
- Vancouver

などでは表示方法が少しずつ異なります。

しかし、EndNoteに入力する情報は基本的に共通です。

そのため、一度登録してしまえば、投稿先に合わせて自動で参考文献の形式を変更できます。

---

# DOIがある場合は？

WebページであってもDOIが付与されている場合は、**Web Page**ではなく**Journal Article**として登録する方が適切です。

一方、GitHub Repositoryや研究者ホームページなどDOIがない場合は、**Web Page**として登録します。

---

# よくある質問

## GitHub Repositoryは論文で引用できますか？

はい。

近年では解析コードやソフトウェアをGitHubで公開し、それを参考文献として引用する論文が増えています。

再現可能な研究（Reproducible Research）の観点からも、GitHub Repositoryを引用することは一般的になりつつあります。

---

## Access Dateは必要ですか？

多くの引用スタイルでは、Webページは内容が変更される可能性があるため、閲覧日（Access Date）の記載が推奨されています。

---

# まとめ

EndNoteでホームページを参考文献として登録する手順は非常に簡単です。

1. **Reference Typeを「Web Page」に変更する**
2. **Author・Title・URL・Access Dateを入力する**
3. **通常の論文と同じようにWordへ引用する**

GitHub Repositoryや研究者ホームページを公開する機会が増えている現在、この登録方法を覚えておくと論文執筆がよりスムーズになります。

---

## 関連記事

- GitHub Repositoryを論文で引用する方法
- GitHubで解析コードを公開するメリット
- EndNoteでDOIから文献を自動登録する方法
- EndNoteの便利な使い方まとめ

{% include post-footer.html %}
