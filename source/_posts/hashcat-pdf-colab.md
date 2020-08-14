---
title: GPUを使ってPDFのパスワードを解析する（Google Colab編）
thumbnail: /thumbnail/hashcat-colab.png
category:
  - Programming
  - Security
tags:
  - Hashcat
  - WSL
  - Google Colab
date: 2020-08-14 15:47:11
---
以前ローカル環境でGPUを使ってPDFのパスワード解析してみたが、
実は、Google Colabを使えば、高性能GPU搭載PCを持っていなくても、GPUを使った解析を体験できてしまう。
ローカルよりも環境構築がとても楽なので、GPGPUの速度を手っ取り早く体感したい方におすすめ。
前記事同様、悪用厳禁。暗号化PDFは自分で検証用に準備すること。

## 準備するもの
ブラウザが開けるPC(chrome bookでもok)

## 環境構築
1. ブラウザでGoogle Colabを開く。
* [ https://colab.research.google.com/](https://colab.research.google.com/)
2. ヘッダーメニューのファイル→新規作成をクリック。
3. ヘッダーメニューのランタイム→ランタイムのタイプを変更→GPUを選択。
4. 以下をコードセルにペースト、実行するとHashcatが導入できる。
```
!apt install cmake build-essential -y && apt install checkinstall git -y && git clone https://github.com/hashcat/hashcat.git && cd hashcat && git submodule update --init && make && make install
```

## 検証
今回はGPGPUの速度を手っ取り早く体感するのが目的であるため、ハッシュの抽出にWebサービスを利用する。
以下のサイトに暗号化PDFをアップロード。
* https://www.onlinehashcrack.com/tools-pdf-hash-extractor

<img src="{% post_path hashcat-pdf-colab %}/hash.png" /></br>
ハッシュをコピーしてtxt形式で保存。

<img src="{% post_path hashcat-pdf-colab %}/hash2.png" /></br></br>
ハッシュが保存してあるtxtファイルをGoogle Colabにアップロード。

<img src="{% post_path hashcat-pdf-colab %}/hash3.png" /></br></br>
新しいコードセルを作成し、Hashcatを実行する。ColabではUnixコマンドは頭に!をつける必要がある。
例えば、
```
! hashcat -m 10500 -a 3 -w 4 hash.txt '?d?d?d?d?d?d'
```
-w 4 を付けることで全力で探索することができる。以前ローカル環境で-w 4をつけたらクラッシュしたが、Colabでは正常に動いた。
詳しい使い方は以下のサイトやWikiを参照。
* https://netwiz.jp/hashcat/

<img src="{% post_path hashcat-pdf-colab %}/hash4.png" /></br></br>
Google ColabではGPUは3種類(K80 T4 V100)あり自動で割り振られる。今回は、中くらいの性能であるT4が割り当てられた。(性能順は K80 < T4 < V100)
解析速度を見てみると12000kH/s～16000kH/sであり、ローカルの2070sとほぼ変わらない速度であった...
運よくV100が割り当てられたら、更に早くなるに違いない。
この環境を無料で使えてしまうGoogle Colab恐るべし...
















