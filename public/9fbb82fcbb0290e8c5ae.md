---
title: iso-2022-jpの文字列をutf-8に変換する
tags:
  - Ruby
  - Rails
private: false
updated_at: '2021-01-13T12:45:27+09:00'
id: 9fbb82fcbb0290e8c5ae
organization_url_name: ateam-life-design
slide: false
ignorePublish: false
---
メールを受け取った際にiso-2022-jpで送られたために文字化けしてしまっていました。

rubyのencodeを使ってutf-8に変換します

検証するためにiso-2022-jpでエンコードされた文字列を生成します

```ruby
puts "ほげ".encode('iso-2022-jp')
# => $B$[$2(B
```

encodeを以下のように記述します

```ruby
str.encode(変換先, 変換元, 変換オプション)
```

今回はiso-2022-jpの文字列をutf-8に変換したいので次のようになります

```ruby
str = "ほげ".encode('iso-2022-jp')
puts str
# => $B$[$2(B
puts str.encode('utf-8', 'iso-2022-jp', invalid: :replace, undef: :replace, replace: '')
```

今回は変換できない文字を空文字に置き換えるオプションを指定しています

変換元がわかっている場合でないとこの方法で変換することはできませんが、
今回のようにメールが文字化けしている場合などでは有効に使えそうです

## 参考
https://docs.ruby-lang.org/ja/latest/method/String/i/encode.html
