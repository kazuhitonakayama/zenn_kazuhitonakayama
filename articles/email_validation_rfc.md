---
title: "メールアドレスのバリデーション、254文字制限の理由を探る 〜RFC Errataから学ぶ〜"
emoji: "📧"
type: "tech"
topics: ["email", "validation", "rfc"]
published: false
---

# はじめに

システム開発において、メールアドレスのバリデーションは基本的な要件の一つです。しかし、「メールアドレスの最大長は何文字にすべきか？」という質問に対して、明確な答えを持っている開発者は少ないのではないでしょうか？

今回は、RFC（Request for Comments）のErrata（正誤表）から、メールアドレスの制限に関する重要な知見を紹介します。

# Errataとは？

Errataとは、RFCなどの技術文書における誤りや不正確な記述を修正するための正誤表です。RFCは一度公開されると内容を変更できないため、発見された誤りや追加の説明が必要な部分はErrataとして管理されます。

今回取り上げるのは、[RFC 3696のErrata ID 1690](https://www.rfc-editor.org/errata/eid1690)です。このErrataは、メールアドレスの長さ制限に関する重要な修正を含んでいます。

# メールアドレスの構成要素

メールアドレスは以下の2つの部分から構成されています：

1. ローカルパート（@の前の部分）
   - 例：`example`@gmail.com
   - 最大64文字まで

2. ドメインパート（@の後の部分）
   - 例：example@`gmail.com`
   - ドメイン全体では最大255文字まで

これらの制限は、メールプロトコルの技術的な制約に基づいています。

# なぜメールアドレスの制限は254文字なのか？

RFC 3696のErrata ID 1690によると、メールアドレス全体の制限は**254文字**が適切とされています。これには以下のような理由があります：

1. **SMTPプロトコルの制限**
   - SMTPコマンドラインの最大長は512文字
   - MAIL FROM: や RCPT TO: などのコマンド文字列も考慮する必要がある

2. **実際の構成要素の制限**
   - ローカルパート：最大64文字
   - @記号：1文字
   - ドメインパート：最大255文字
   - 合計：320文字（理論上の最大）

3. **実用的な制限**
   - RFC 2821では、forward-pathに256文字の制限を設けています
   - forward-pathは `<` [ A-d-l ":" ] Mailbox `>` の形式で定義されています
   - つまり、メールアドレス（Mailbox）の前後に少なくとも山括弧（`<` と `>`）が必要です
   - この2文字分を考慮すると、メールアドレス自体は254文字が上限となります
# コードで表現すると

メールアドレスのバリデーションを実装する際は、以下のポイントを考慮することをお勧めします：

```ruby
def validate_email(email)
  return false if email.length > 254  # 全体の長さチェック
  
  local, domain = email.split('@')
  return false if local.nil? || domain.nil?  # @が含まれているかチェック
  return false if local.length > 64   # ローカルパートの長さチェック
  return false if domain.length > 255 # ドメインパートの長さチェック
  
  # その他の文字種チェックなど
  true
end
```

# まとめ

- RFCのErrataは、技術文書の正確性を保つための重要な補足資料です
- メールアドレスは、ローカルパート（最大64文字）とドメインパート（最大255文字）で構成されます
- 実用的なメールアドレスの最大長は254文字が適切です
  - これは、SMTPプロトコルの制限と実際の使用パターンを考慮した結果です

この制限を理解することで、より堅牢なメールアドレスバリデーションを実装することができます。また、このような技術的な制約の背景を知ることは、システム設計の際の重要な知見となります。

# 参考資料

- [RFC 3696 Errata ID 1690](https://www.rfc-editor.org/errata/eid1690)
- [RFC 5321 - Simple Mail Transfer Protocol](https://tools.ietf.org/html/rfc5321)
- [What Is the Maximum Length of an Email Address?](https://www.directedignorance.com/blog/maximum-length-of-email-address)
