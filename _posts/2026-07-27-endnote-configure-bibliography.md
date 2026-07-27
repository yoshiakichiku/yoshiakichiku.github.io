---
title: "EndNoteのConfigure Bibliographyの使い方｜参考文献の書式が勝手に変わるときの対処法"
date: 2026-07-27 00:00:00 +0900
categories: [Research, EndNote]
tags: [EndNote, Configure Bibliography, Reference, Word, Citation]
---

EndNoteを使用していると、

> 「Update Citations and Bibliographyを押したら参考文献の書式が変わってしまった」

という経験はないでしょうか。

例えば、

- フォントが変わる
- 文字サイズが変わる
- 行間が変わる
- 参考文献タイトルが変わる

などです。

私自身も論文投稿前にこの現象で困りました。

しかし、原因はWordではなく**EndNoteの「Configure Bibliography」**にありました。

この記事では、Configure Bibliographyの使い方と、参考文献の書式を修正する方法を解説します。

---

# なぜ書式が変わるのか？

EndNoteでは、

**Update Citations and Bibliography**

を実行すると、引用だけでなく**参考文献リスト全体を再生成**します。

その際、EndNoteに保存されている設定が適用されるため、Wordで手動変更した書式は上書きされることがあります。

そのため、

Word側で何度修正しても、更新のたびに元へ戻ってしまいます。

---

# Configure Bibliographyを開く

Wordを開き、

**EndNoteタブ**

↓

**Configure Bibliography**

をクリックします。

ここで参考文献リストの表示方法を変更できます。

---

# Configure Bibliographyで変更できる項目

主な設定項目は次のとおりです。

- Output Style
- Bibliography Title
- フォント
- 文字サイズ
- 行間
- レイアウト

Word上で直接修正するのではなく、こちらで設定することで更新後も設定が維持されます。

---

# フォントを変更する

例えば投稿規定で

- Times New Roman
- 12 pt

が指定されている場合は、

Configure Bibliographyで

**Font**

を

```
Times New Roman
```

に設定します。

次に

**Size**

を

```
12 pt
```

に設定します。

これでUpdate Citations and Bibliographyを実行しても、設定したフォントが維持されます。

---

# タイトルも変更できる

参考文献タイトルも変更できます。

例えば

```
References
```

や

```
Bibliography
```

など、投稿規定に合わせて設定できます。

タイトルが不要な場合は空欄にできる場合もあります。

---

# Wordで直接修正しない方がよい理由

Word上で参考文献リストを直接編集しても、

Update Citations and Bibliography

を実行すると、EndNoteが再び参考文献リストを作成します。

そのため、手動で修正した内容は失われる可能性があります。

まずはConfigure Bibliographyを確認しましょう。

---

# Configure Bibliographyで直らない場合

Configure Bibliographyで変更できるのは、主に参考文献リストの表示です。

それでも思いどおりにならない場合は、

- Output Style（.ensファイル）
- Wordのスタイル（Bibliographyスタイル）

が影響している可能性があります。

この場合はOutput Styleを編集することで解決できることがあります。

---

# 投稿前のおすすめ

投稿前には、

1. Output Styleを投稿先のジャーナルに変更する
2. Configure Bibliographyで参考文献の見た目を確認する
3. Update Citations and Bibliographyを実行する

という流れがおすすめです。

これにより、投稿規定に沿った参考文献リストを効率よく作成できます。

---

# まとめ

EndNoteで参考文献の書式が勝手に変わる場合は、Wordではなく**Configure Bibliography**を確認しましょう。

特に、

- フォント
- 文字サイズ
- タイトル
- レイアウト

などはConfigure Bibliographyから変更できます。

Word上で直接修正するよりも、EndNote側で設定した方が更新後も設定が維持されるためおすすめです。

---


{% include post-footer.html %}
