---
title: AviUtil で GPUを使ったエンコード
date: 2020-08-05T10:15:49.304Z
image: img/1.png
draft: false
---
AviUtil で h.264 のエンコードをする

## 必要なもの

- [AviUtl](http://spring-fragrance.mints.ne.jp/aviutl/)
- [拡張編集Plugin](http://spring-fragrance.mints.ne.jp/aviutl/)
- [NeroAACCodec](https://www.free-codecs.com/nero_aac_codec_download.htm?f=nero_aac_codec_download)
- [L-SMASH](https://pop.4-bit.jp/)
- [L-SMASH Works](https://pop.4-bit.jp/)
- Geforceの場合 [NVEnc](https://rigaya34589.blog.fc2.com/blog-category-17.html)
- Radeonの場合 [VCEEnc](https://rigaya34589.blog.fc2.com/blog-category-12.html)

## 各種インストールする
### AviUtl をインストール

これが、動画編集ソフト本体になる

1. ダウンロードしたものを解凍するだけ

### 拡張編集Pluginをインストール

aviutl で、複数の動画や音楽ファイルを読み込んだり、文字を挿入したりなどできるようになる。
(エンコードに必須ではない)

1. aviutl.exe のあるディレクトリに `Plugins` ディレクトリを作成する
2. 拡張編集Plugin を解凍した中にある `exedit.txt` に従って、解凍したファイルすべてを `Plugins` ディレクトリにコピーする

## NeroAACCodec をインストール

音声のエンコードをする。一応これがなくてもできるはずだが、デフォルトがこれ

1. aviutl.exe のあるディレクトリに `exe_files` ディレクトリを作成する
1. NeroAACCodecを解凍し、解凍したディレクトリの中の `win` ディレクトリの中のファイルをすべて `exe_files` にコピーする

### L-SMASH

音声のエンコードをしたものを動画のエンコードの中にいれるもの (多分)

1. 解凍したディレクトリの中身をすべて `exe_files` にコピーする

### L-SMASH Works

MP4 などの動画を aviutl で読み込めるようにする

1. 解凍して、 lwcolor.auc、lwdumper.auf、lwinput.aui、lwmuxer.auf 各ファイルを `Plugins` ディレクトリにコピーする

### Geforceの場合 NVEnc をインストールする

Geforce を使ってる場合こちらのプラグインを使います。

1. 解凍したディレクトリにある `auto_setup.exe` を起動します。
2. aviutl.exe のあるディレクトリを聞かれるので、選択して実行します。

`NVENCが利用可能か確認 [ダブルクリック].bat` で使えるかチェックできる。

### Radeonの場合 VCEEnc をインストールする

Radeon を使ってる場合こちらのプラグインを使います。

1. 解凍したディレクトリにある `auto_setup.exe` を起動します。
2. aviutl.exe のあるディレクトリを聞かれるので、選択して実行します。

## 動画をエンコードする

1. aviutl.exe で起動する
2. 設定から、拡張編集の設定を選ぶ
3. 出てきたwindowに動画をDrag&Dropする (読み込むファイルに合わせるにチェックを入れる)
4. 1ファイルにエンコードしたい動画をすべて同様の手順でいれる
![](1.png)
5. ファイル -> プラグイン出力 -> 拡張 VCE 出力を選択する (Geforceの場合は NVEEnc を選んでください)
6. ビデオ圧縮で圧縮の設定をする
7. 動画エンコードの設定を適当にする(画像サイズや画像の綺麗さに応じて)
8. 音声の `外部エンコーダを使用する` にチェックを入れる
  - NeroAacEnc を指定して
  - NeroAacEnc.exe の指定に `.\exe_files\neroAacEnc.exe` を入れる
9. 右下の mp4 タグの `外部muxerを使用` にチェックをする
  - remuxer.exe の指定に、 `.\exe_files\remuxer.exe`
  - timelineeditor.exe の指定に、`.\exe_files\timelineeditor.exe`
  - muxer.exe の指定に、`.\exe_files\muxer.exe`
![](2.png)
10. OK を押して、動画を保存する。しばらく待てば完成！