---
title: GPUを使ってPDFのパスワードを解析する
thumbnail: /thumbnail/hashcat.png
category:
  - Programming
  - Security
tags:
  - Hashcat
  - WSL
date: 2020-08-12 10:45:40
---
オンライン授業やリモートワークでパスワード付きのPDFを扱うことが増えてきたと思う。
適切な強いパスワードを設定する人もいると思うが、大多数はパスワードを適当に設定しているだろう。
今回パスワード解析ツールHashcatとGPUを用いてPDFのパスワードを解析してみる。
実際に解析することで、どのようなパスワードが強いか、検証する。
悪用厳禁。

## 準備するもの
  * グラボ付きPC
    * 検証で使用したPCのスペック
      * CPU: i7 8700
      * GPU: 2070 super
      * RAM: DDR4-2400 24GB
      
## 環境構築
今回、Windows 10 home (1909)環境下で行った。一部WSLを使用している。 
Hashcatをソースコードからビルドしないといけないため、パッケージ提供のあるUbuntuや、デフォルトツールとして導入済みのKali Linuxを用いた方が楽かも。
Windows ver2004だとWSL2上でCUDAが使えるので、Windows環境下であればWSL2上でKaliを使うのが一番楽かな？

### CUDA Toolkitの導入
CUDAToolkit 9.0以降を導入する。
古いCUDAToolkitが入っている人は、一度アンインストールしてから、新しいToolkitを導入しよう。
異なるバージョンのToolkitが複数あると正常に動かなかった。
* [CUDA Toolkit 11.0](https://developer.nvidia.com/cuda-toolkit)

### Jown the ripperの導入
PDFからハッシュを抜き出すために使う。
64-bit Windows binariesをDLして解凍。pdf2john.plを用いる。
* [John the Ripper](https://www.openwall.com/john/)

### Hashcatの導入
ハッシュ文(暗号文)が分かれば、PDF MSOffice ZIP ありとあらゆるファイルのパスワードを解析できるちょっと怖いツール。
使い慣れている人が脆弱性を利用して、計算量を削減して実行すると、22.7ゼッタ(1垓)ハッシュ毎秒というとんでもないスピードで解析も可能。
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Support for PKZIP Master Key added to <a href="https://twitter.com/hashtag/hashcat?src=hash&amp;ref_src=twsrc%5Etfw">#hashcat</a> with an insane guessing rate of 22.7 ZettaHash/s on a single RTX 2080Ti. All passwords up to length 15 in less than 15 hours with only 4 GPUs! Excellent contribution from <a href="https://twitter.com/s3inlc?ref_src=twsrc%5Etfw">@s3inlc</a> and <a href="https://twitter.com/EU_ScienceHub?ref_src=twsrc%5Etfw">@EU_ScienceHub</a> <a href="https://t.co/kVUDBrQWM3">https://t.co/kVUDBrQWM3</a> <a href="https://t.co/iqVoszaqEi">pic.twitter.com/iqVoszaqEi</a></p>&mdash; hashcat (@hashcat) <a href="https://twitter.com/hashcat/status/1129441728761610242?ref_src=twsrc%5Etfw">May 17, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</br>

今回Windows環境下で使うので、公式サイト若しくはgithubからソースコードをDLしてビルドする。gitcloneしてもいい。
* [hashcat(github)](https://github.com/hashcat/hashcat)
* [hashcat(公式)](https://hashcat.net/hashcat/)

WSL上でビルドするためMinGWを導入。
```
$ sudo apt install  mingw-w64 win-iconv-mingw-w64-dev
```
エディタでHashcatフォルダ内のsrc/Makefileを開く。
<img src="{% post_path hashcat-pdf %}/code.png" /></br>
573行目のopt/win-iconv-64を/usr/x86_64-w64-mingw32にパス変更。
ビルドする。
```
$ make win -j 5
```
hashcat.exeができる。

## 解析作業
環境構築が終わったので、実際に解析してみよう。

### ハッシュの抽出
John the Ripperのpdf2john.plを使う。
WSLにはperl実行環境はデフォルトでインストールされていると思うので、WSL上で実行。
```
$ Perl john-1.9.0-jumbo-1-win64/run/pdf2john.pl MyPDF.pdf > MyPDF-Hash.txt
```
MyPDF.pdfには解析したいPDFのパスを入力、MyPDF-Hash.txtに抽出したハッシュが保存される。

John the Ripperの代わりにハッシュ抽出サイトを使ってもよいが、大切なファイルはアップロードしない方がいいかも。
* [PDF HASH EXTRACTOR](https://www.onlinehashcrack.com/tools-pdf-hash-extractor.php)

### Hashcatで解析
WSLではなくcmdでhashcat.exeを実行する。
例えば
```
C:/Users/user/hashcat>hashcat.exe -m 10500 -a 3 -w 4 C:/Users/user/Desktop/Mypdf-Hash.txt
```
のように入力。

-mオプションで、適切なハッシュを選択。

| 番号 | 対象 |
|  --- | --- |
| 10400 | PDF 1.1 - 1.3 (Acrobat 2 - 4) |
| 10500 | PDF 1.4 - 1.6 (Acrobat 5 - 8) |
| 10600 | PDF 1.7 Level 3 (Acrobat 9) |
| 10700 | PDF 1.7 Level 8 (Acrobat 10 - 11) |

-aオプションでアタックモード選択。

|オプション|モード|
|---|---|
|0|辞書攻撃|
|1|辞書組み合わせ|
|3|総当たり攻撃|
|6|辞書+マスク(ハイブリッド攻撃)|
|7|マスク+辞書(ハイブリッド攻撃)|

詳しい使い方は、wikiや下のサイトを参照。
* [https://netwiz.jp/hashcat/](https://netwiz.jp/hashcat/)

## 結果

今回Acrobatの有料版を持っていなかったため、MSWordからパスワード付きpdfを作成した。
ハッシュを見た感じ128bitAESであったため、PDF1.6だと思われる。
最新のPDF、Acrobat9以降(PDF1.7)では256bitAESが標準であるため、PDF1.6よりも解析難易度は高いと思われる。

### 数字6桁総当たり攻撃
pass:324913
数字6桁だと100万通り。
1秒以下で解析できた。
速度が14687.6kH/s=1468.7万回/sなので当たり前かな...
数字8桁でも1億通りなので、10秒かからないと思われる。

<img src="{% post_path hashcat-pdf %}/crack1.png" /></br>

### 小文字英語数字6桁総当たり攻撃
pass:gewr39
今度は(26+10)^6=2176782336通り
2分くらいだと思われる。
実際は30秒で解析できた。

<img src="{% post_path hashcat-pdf %}/crack2.png" /></br>

### 小文字英語数字8桁総当たり攻撃
pass:hoge1200
36^8=2.28兆通り。
ちょっと厳しいかなー
全て探索が終わるまで1日と13時間かかるが...
まさかの1分45秒で解析できてしまった...
総当たりなので、前半で当たれば解析は可能。

<img src="{% post_path hashcat-pdf %}/crack3.png" /></br>

### 大文字小文字英語数字8桁総当たり攻撃
pass:Hksg1532
62^8=218兆通り。
全て探索が終わるまで117日かかるので、少し絞ってみる。
大文字小文字英語と数字をつかって8桁というのはよく使われるパスワードだが、1文字目を大文字にする人が多いだろう。
1文字目のみ大文字、2~8文字目は小文字英語数字と絞り込んでみると117日を27時間まで減らすことができた。
さらに絞り込んで、1文字目は大文字英語、2～4文字目小文字英語、5～8文字目を数字と仮定して総当たりしてみる。これにより、3分まで減らすことができた。
実際、30秒程度で解析できた。

<img src="{% post_path hashcat-pdf %}/crack4.png" /></br>

### 大文字小文字英語数字記号8桁総当たり攻撃
6634兆通り。

<img src="{% post_path hashcat-pdf %}/crack5.png" /></br>
全て探索が終わるまで10年47日かかるらしい。現実的な時間では総当たりでの解析は不可能。
最近記号を含めないとパスワード登録できないサービスが増えたが、脆弱性がない限り、理にかなっていることが分かる。

## まとめ
今回誰でも手軽に手に入れることができる2070superで解析してみたが、小文字英語数字8桁程度なら、総当たりでも現実的な時間で解析できることが分かった。
従ってtesla v100等、最新のgpgpu専用gpuを複数台使用したら、更に早く解析できると思われる。
また、hashcatには、辞書列攻撃と総当たり攻撃を組み合わせたハイブリット攻撃が可能である。これを利用すると8文字以上のパスワードの解析可能性が大幅に向上すると思われる。
例えば、power1342というパスワードで、powerという単語が辞書に含まれていたら、数字4桁総当たり攻撃とほぼ変わらない速度で解析ができる。従って、8文字以上でも単語と適当な数字の組み合わせでは強いパスワードとは言えない。
辞書列攻撃が不可能なランダム文字列で、大文字小文字英語数字記号を含む8桁以上がやはり強いパスワードと言える。












