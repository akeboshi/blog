---
title: ActionText がどのようにHTMLを保存しているか
date: 2021-02-26T07:12:12.132Z
draft: false
---
ActionText の本文は、 action_text_rich_texts.body に保存される。どのように保存されるか見ていく。
active storage を使った画像データとのリンクは個々では触れない

## has_rich_text

任意のモデルで、 has_rich_text を定義すると、 `ActionText::RichText` への `has_one` が定義される。
これで、任意のモデルから `ActionText::RichText` モデルへのポリモーフィックな依存が作られる
```ruby
      def has_rich_text(name)
        class_eval <<-CODE, __FILE__, __LINE__ + 1
          def #{name}
            rich_text_#{name} || build_rich_text_#{name}
          end
          def #{name}?
            rich_text_#{name}.present?
          end
          def #{name}=(body)
            self.#{name}.body = body
          end
        CODE

        has_one :"rich_text_#{name}", -> { where(name: name) },
          class_name: "ActionText::RichText", as: :record, inverse_of: :record, autosave: true, dependent: :destroy

        scope :"with_rich_text_#{name}", -> { includes("rich_text_#{name}") }
        scope :"with_rich_text_#{name}_and_embeds", -> { includes("rich_text_#{name}": { embeds_attachments: :blob }) }
      end
```
https://github.com/rails/rails/blob/master/actiontext/lib/action_text/attribute.rb#L27-L47

## RichText

https://github.com/rails/rails/blob/master/actiontext/app/models/action_text/rich_text.rb

table 名が `action_text_rich_texts` になっている
```ruby
self.table_name = "action_text_rich_texts"
```

body に入れた文字列はそのままの文字列では保存されず serialize された形で保存される。

```ruby
serialize :body, ActionText::Content
```

serialize は引数に何も取らなければ、yaml 形式でシリアライズして、dbに何でも突っ込める (https://qiita.com/jnchito/items/68e91e9bf46f960a79e4) が基本的には使わないほうが良い。

ActionText では、 `ActionText::Content` 型に serialize したものを database に保存している。
シリアライズ、デシリアライズは `load`, `dump` を呼ぶので、それの定義を探してあげれば、シリアライズ、デシリアライズで何を読んでいるかわかる

Serializer をインクルードしていて、 https://github.com/rails/rails/blob/master/actiontext/lib/action_text/content.rb#L7

本体はここ https://github.com/rails/rails/blob/master/actiontext/lib/action_text/serialization.rb

```ruby
      def load(content)
        new(content) if content
      end

      def dump(content)
        case content
        when nil
          nil
        when self
          content.to_html
        else
          new(content).to_html
        end
      end
```

db に保存される際は、 `to_html` を呼んで、読み込む際は initializer を呼んでいるのがわかる