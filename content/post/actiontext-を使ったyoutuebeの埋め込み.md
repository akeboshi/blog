---
title: ActionText を使ったYoutuebeの埋め込み
date: 2021-02-26T07:13:38.603Z
draft: true
---
## Youtube を埋め込む

ボタンを増築して、youtubeの動画URLを貼るだけで、youtubeの埋め込みを作ろうぜという話。
やっているのは、画像を埋め込む時の処理の応用っぽい。
動画だと簡単にやってるけど、ちゃんと理解しようとすると大変だと思う

参考: https://railsconf.com/2020/video/chris-oliver-advanced-actiontext-attaching-any-model-in-rich-text
github: https://github.com/excid3/railsconf-2020-actiontext

### JS で、ボタンを作る

ボタンを作る部分は、 trix の toolbar の表示の部分が丸々 config になっているので、それをコピペして付け足すだけ。

https://github.com/excid3/railsconf-2020-actiontext/blob/master/app/javascript/youtube.js#L5-L52

https://github.com/excid3/railsconf-2020-actiontext/blob/master/app/javascript/packs/application.js#L19-L23

### JSコントローラ部

バックエンドに、表示用のサムネイルと、リソース特定用のIDをjson形式でもらって、書き出す

https://github.com/excid3/railsconf-2020-actiontext/blob/master/app/javascript/youtube.js#L54-L120

※ https://github.com/basecamp/bc3-api/blob/master/sections/rich_text.md#inserting-an-image-or-file-attachment にかいてあるようにsgidで、どのidのものかを識別するようにしている。
json の中のcontentの中身をaction-textの入力する部分に表示させている (https://github.com/basecamp/trix#inserting-a-content-attachment)

### 表示や埋め込みのhtmlを作るController, Modelを作る

ボタンを押したときに `/youtube/${id}` にリクエストが送られるので、sgid, と表示用のhtml (今回はサムネイル)を返すように controller を作る

https://github.com/excid3/railsconf-2020-actiontext/blob/master/app/controllers/youtube_controller.rb
https://github.com/excid3/railsconf-2020-actiontext/blob/master/app/models/youtube.rb

```
  include GlobalID::Identification # sgid 作るため
  include ActionText::Attachable # これをつけると、show画面で、表示するときに埋め込みyoutubeに置き換えて埋め込んでくれる。また、trixの形式で送られてきたrequestをrailsで解釈できるようなタグに変換できるようにしてる
```

database に登録されるのは 下記のようになる

```
<action-text-attachment sgid="BAh7CEkiCGdpZAY6BkVUSSItZ2lkOi8vYXBwL1lvdXR1YmUvaTRWM0tkb05IeE0_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--fdb0cca585ade8269ef92d193548405b311e183e"></action-text-attachment>
```

こんな形なのを、action text のオブジェクトを表示する際 (ActionText::Content#to_s) に、helperが呼ばれ、そこで Attachable のobject も 指定のtemplateで render される (template: https://github.com/excid3/railsconf-2020-actiontext/blob/master/app/views/youtubes/_youtube.html.erb)

下記のようなHTMLが吐かれる

```
<iframe id="ytplayer" type="text/html" width="640" height="360" src="https://www.youtube.com/embed/i4V3KdoNHxM" frameborder="0"></iframe>
```

### 特定のhtmlタグを許可する

デフォルトだと iframe がsanitizeで消されてしまう

https://github.com/rails/rails/blob/7118c435996a78effd7989e5c61536e817e6e716/actiontext/app/helpers/action_text/content_helper.rb#L17

allow_tags で許可するタグを決めている

https://github.com/rails/rails/blob/7118c435996a78effd7989e5c61536e817e6e716/actiontext/app/helpers/action_text/content_helper.rb#L8

ここに、iframeを追加する

https://github.com/excid3/railsconf-2020-actiontext/blob/master/config/application.rb#L20

これで、iframeが削除されること無く表示される

## Trix

Action Text で用いられているWYSIWYG Editor
入力された内容は Trix が持っているModelに変換され、render される。
hidden で持っている input と、text area を別で持っていて、見えている内容と全く同じに保存されているわけではない

### 機能の追加や設定

* ボタンを押したときにどのタグ名を当てるか

  * https://github.com/basecamp/trix/blob/master/src/trix/config/block_attributes.coffee
  * https://github.com/basecamp/trix/blob/master/src/trix/config/text_attributes.coffee
* ツールバーのカスタマイズ

  * https://github.com/basecamp/trix/blob/master/src/trix/config/toolbar.coffee
* style を当てる簡単な機能を追加する

  * https://jsfiddle.net/javan/egg7fgvv/
* 色を当てれそう？ (未検証)

  * https://gist.github.com/javan/a8a237f0db7648ba88d66cf9a50fa1f5

Youtube の埋め込みは、Controller から作っているので、Trix内部をちゃんと知らないと作るのは難しい… (これを理解して同等のもの作るとなるともう一週間くらい欲しい。)

class名の追加は今の所サポートされてないので、内部まで結構いじらないと難しそう [:github: PR](https://github.com/basecamp/trix/pull/791/files) があるので、これがマージされると、いい感じにできそう

### 参考 (表示されるものとか、リクエストの内容とか)

![](img/image.png "image")

これのリクエストとしては、 以下の input が送られる。

```
<input type="hidden" name="article[body]" id="article_body_trix_input_article" value="<div><figure data-trix-attachment=&quot;{&amp;quot;contentType&amp;quot;:&amp;quot;image/png&amp;quot;,&amp;quot;filename&amp;quot;:&amp;quot;スクリーンショット 2020-06-16 11.25.02.png&amp;quot;,&amp;quot;filesize&amp;quot;:162206,&amp;quot;height&amp;quot;:276,&amp;quot;sgid&amp;quot;:&amp;quot;BAh7CEkiCGdpZAY6BkVUSSIwZ2lkOi8vYXBwL0FjdGl2ZVN0b3JhZ2U6OkJsb2IvMzE_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--d3494cdca81ece9d8360b982dcef17a417fd095e&amp;quot;,&amp;quot;url&amp;quot;:&amp;quot;http://localhost:3000/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBKQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--c29f83c871e80f8bf195754fb8d5ef31dca4284f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202020-06-16%2011.25.02.png&amp;quot;,&amp;quot;width&amp;quot;:310}&quot; data-trix-content-type=&quot;image/png&quot; data-trix-attributes=&quot;{&amp;quot;presentation&amp;quot;:&amp;quot;gallery&amp;quot;}&quot; class=&quot;attachment attachment--preview attachment--png&quot;><img src=&quot;http://localhost:3000/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBKQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--c29f83c871e80f8bf195754fb8d5ef31dca4284f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202020-06-16%2011.25.02.png&quot; width=&quot;310&quot; height=&quot;276&quot;><figcaption class=&quot;attachment__caption&quot;><span class=&quot;attachment__name&quot;>スクリーンショット 2020-06-16 11.25.02.png</span> <span class=&quot;attachment__size&quot;>158.4 KB</span></figcaption></figure><br><br></div><h2>Header 1</h2><div><strong>bold</strong></div>">
```

表示されるのは↓

```
<trix-editor id="article_body" input="article_body_trix_input_article" class="trix-content" data-direct-upload-url="http://localhost:3000/rails/active_storage/direct_uploads" data-blob-url-template="http://localhost:3000/rails/active_storage/blobs/:signed_id/:filename" contenteditable="" role="textbox" trix-id="1" toolbar="trix-toolbar-1"><div><!--block--><span data-trix-cursor-target="left" data-trix-serialize="false">&#65279;</span><figure contenteditable="false" data-trix-attachment="{&quot;contentType&quot;:&quot;image/png&quot;,&quot;filename&quot;:&quot;スクリーンショット 2020-06-16 11.25.02.png&quot;,&quot;filesize&quot;:162206,&quot;height&quot;:276,&quot;sgid&quot;:&quot;BAh7CEkiCGdpZAY6BkVUSSIwZ2lkOi8vYXBwL0FjdGl2ZVN0b3JhZ2U6OkJsb2IvMzE_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--d3494cdca81ece9d8360b982dcef17a417fd095e&quot;,&quot;url&quot;:&quot;http://localhost:3000/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBKQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--c29f83c871e80f8bf195754fb8d5ef31dca4284f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202020-06-16%2011.25.02.png&quot;,&quot;width&quot;:310}" data-trix-content-type="image/png" data-trix-id="158" data-trix-attributes="{&quot;presentation&quot;:&quot;gallery&quot;}" class="attachment attachment--preview attachment--png" data-trix-mutable="true"><img src="http://localhost:3000/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBKQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--c29f83c871e80f8bf195754fb8d5ef31dca4284f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202020-06-16%2011.25.02.png" data-trix-mutable="true" width="310" height="276" data-trix-store-key="imageElement/158/http://localhost:3000/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBKQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--c29f83c871e80f8bf195754fb8d5ef31dca4284f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202020-06-16%2011.25.02.png/310/276"><figcaption class="attachment__caption attachment__caption--editing"><textarea placeholder="Add a caption…" data-trix-mutable="true" class="attachment__caption-editor" style="height: 14px;"></textarea><textarea placeholder="Add a caption…" data-trix-mutable="true" class="attachment__caption-editor trix-autoresize-clone" tabindex="-1"></textarea></figcaption><figcaption class="attachment__caption" style="display: none;"><span class="attachment__name">スクリーンショット 2020-06-16 11.25.02.png</span> <span class="attachment__size">158.4 KB</span></figcaption><div data-trix-mutable="true" class="attachment__toolbar"><div class="trix-button-row">
  <span class="trix-button-group trix-button-group--actions">
    <button type="button" data-trix-action="remove" class="trix-button trix-button--remove" title="Remove">Remove</button>
  </span>
</div><div class="attachment__metadata-container">
  <span class="attachment__metadata">
    <span class="attachment__name" title="スクリーンショット 2020-06-16 11.25.02.png">スクリーンショット 2020-06-16 11.25.02.png</span>
    <span class="attachment__size">158.4 KB</span>
  </span>
</div></div></figure><span data-trix-cursor-target="right" data-trix-serialize="false">&#65279;</span><br><br></div><h2><!--block-->Header 1</h2><div><!--block--><strong>bold</strong></div></trix-editor>
```

DB に保存されるのは

```
<div><action-text-attachment sgid="BAh7CEkiCGdpZAY6BkVUSSIwZ2lkOi8vYXBwL0FjdGl2ZVN0b3JhZ2U6OkJsb2IvMzE_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--d3494cdca81ece9d8360b982dcef17a417fd095e" content-type="image/png" url="http://localhost:3000/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBKQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--c29f83c871e80f8bf195754fb8d5ef31dca4284f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202020-06-16%2011.25.02.png" filename="スクリーンショット 2020-06-16 11.25.02.png" filesize="162206" width="310" height="276" presentation="gallery"></action-text-attachment><br><br></div><h2>Header 1</h2><div><strong>bold</strong></div>
```

## sgid の中身

```
<div><span style="background-color: highlight;"><action-text-attachment sgid="BAh7CEkiCGdpZAY6BkVUSSIzZ2lkOi8vYXBwL05ld3M6OllvdXR1YmUvZktSeE1kOEYybE0_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--16a2d526fd6aaf2d8a71c3a0f5660f98b43cf68a"></action-text-attachment></span><br>xxx</div>
```

のように action text ではDBに保存されている。

sgid には GlobalID::Identification でGlobalIDになったクラスがある (シリアライズされてるみたいな表現であってるか？)

これを復元するには、

```
ActionText::Attachable.from_attachable_sgid "BAh7CEkiCGdpZAY6BkVUSSIzZ2lkOi8vYXBwL05ld3M6OllvdXR1YmUvZktSeE1kOEYybE0_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--16a2d526fd6aaf2d8a71c3a0f5660f98b43cf68a"
```

でクラスとして復元できる。

ということは、クラス名なども sgid の中には入っているため、クラス名が変わったらこのデータを移行する必要がある。

## クラス名変えるときにやること

```
foo = ActionText::Attachable.from_attachable_sgid "BAh7CEkiCGdpZAY6BkVUSSIzZ2lkOi8vYXBwL05ld3M6OllvdXR1YmUvZktSeE1kOEYybE0_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--16a2d526fd6aaf2d8a71c3a0f5660f98b43cf68a"
bar = NewClass(id: foo.id).new
bar.attachable_sgid #=> sgid が出力されるので、この文字列で元のsgidを置換すれば良い
```

## 普通に埋め込みに直す

```
foo = ActionText::Attachable.from_attachable_sgid "BAh7CEkiCGdpZAY6BkVUSSIzZ2lkOi8vYXBwL05ld3M6OllvdXR1YmUvZktSeE1kOEYybE0_ZXhwaXJlc19pbgY7AFRJIgxwdXJwb3NlBjsAVEkiD2F0dGFjaGFibGUGOwBUSSIPZXhwaXJlc19hdAY7AFQw--16a2d526fd6aaf2d8a71c3a0f5660f98b43cf68a"
p "<iframe id='ytplayer' type... src='https://www.youtube.com/embed/#{foo.id}'></iframe>"
```

のようにすれば良さそうか。