---
title: WSLで「12ステップで作る組み込みOS自作入門」の環境構築をしてHello World!してみる
thumbnail: /thumbnail/serister.png
category:
  - Programming
  - 12stepOS
tags:
  - kozos
  - Environment
  - WSL
date: 2020-04-16 17:18:52
---

暇なので積読していた「12ステップで作る組み込みOS自作入門」に手を出してみました。
書籍に紹介されていたwindowsでの開発環境ではCygwinの使用が推奨されていましたが、今はWSLがあるのでこれを使ってみようと思いました。
WSLはおろかLinux自体まともに使ったことがなかったので、環境構築に丸一日かかった...

## モノの準備
とりあえず、自分の環境の紹介から。</br>
* デスクトップPC(Win10pro 1903 64bitに WSLでUbuntu 18.04.4LTS導入済み) </br>

次にマイコン関連</br>

* [12ステップで作る組み込みOS自作入門](https://www.amazon.co.jp/dp/4877832394/ref=cm_sw_r_tw_dp_U_x_mPbMEb1ZAQ8RF)
* [H8/3069F マイコンボード](http://akizukidenshi.com/catalog/g/gK-01271/)
* [スイッチングＡＣアダプター　５Ｖ２Ａ](http://akizukidenshi.com/catalog/g/gM-11996/)
* [ＵＳＢシリアル変換ケーブル](http://akizukidenshi.com/catalog/g/gM-02746/)

ボードのシリアルポートとケーブルを固定する金属部分が、邪魔で接続できないので外しました。</br>
<img src="{% post_path 12step-wsl-environment %}/port.jpg" /></br>
私は、問題なく書き込みできましたが、公式ページでは延長ケーブルを買って、ねじ止めして接続することが推奨されています。</br>
* [公式ページ-シリアルコネクタのねじについて](http://kozos.jp/books/makeos/#serial_screw)
* [RS-232C延長ケーブル](https://www.amazon.co.jp/dp/B00A6GIVM2/ref=cm_sw_r_tw_dp_U_x_u6bMEbHH9SEZ9)
</br>

## 環境構築
WSL導入済みという前提のもとで書きます。
折角、ネイティブLinuxではなくWSLを使うので、ファイル編集とシリアル通信テストはWindows側から行ってみます。
エディタは[VSCode](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)、シリアル通信テスタは[Serister](https://www.vector.co.jp/soft/win95/hardware/se423507.html)を使用します。</br>

* 補足
  VSCodeは「右クリックでvscodeを開く」を有効にしておくことをお勧めします。
  * [Windowsの右クリックメニューに「VSCodeで開く」を追加する](https://qiita.com/kaityo256/items/7fefd1d1463184ae1420)</br>

一応、自分が構築したディレクトリ構造を記します。(gccなどのコンパイラとコードはディレクトリ分けてもいいかも)
```
home
|---user
|   |---12step
|       |---src
|           |---binutils-2.19.1
|           |---gcc-3.4.6
|           |---kz_h8write-v0.2.1
|           |---01 (以下書籍のコード)
|           |---02
|           |---
```
今回、gcc、binutilsのconfigureスクリプト実行時に、ファイル場所を指定しません。その場合、h8マイコンのツール類(h8300-elf-hoge)は、make install時に以下にインストールされます。</br>
```
Ubuntu
|---bin
|---boot
|---dev
|---etc
|---home
|   |---user
|       |---12step
|           |---src
|---usr
|   |---local
|       |---bin
|           |---h8300-elf-gcc
|           |---h8300-elf-as
|           |---
|           |---
```
</br>

### binutils
初めにbinutilsを導入します。</br>
バージョンは2.19.1を選択。最新のバージョン入れたらビルド時エラーはいたので書籍に掲載されているバージョンをお勧めします。

```
$ cd ~/ 
$ mkdir 12step     
$ cd 12step
$ mkdir src
$ cd src 
$ wget http://core.ring.gr.jp/pub/GNU/binutils/binutils-2.19.1.tar.gz
$ tar zxvf binutils-2.19.1.tar.gz
$ cd binutils-2.19.1
$ mkdir build
$ cd build
$ ../configure --target=h8300-elf --disable-nls --disable-werror
$ make
$ sudo make install
```

* 補足
  makeコマンドを打つとき、オプション```-j 5```(数字はcpuのコア数+1?)をつけるとビルドを高速化出来ます。実行時間が早くなるのでお勧めです。
  * [makeの並列オプションは何を指定するべきか](http://lpha-z.hatenablog.com/entry/2018/12/30/231500)

</br>

### gcc
次にgccを導入します。
バージョンは3.4.6を選択。こちらも書籍と同じバージョンをお勧めします。</br>
今回、64bit環境ですので、パッチの適応が必要です。
* [64ビットマシンの利用について](http://kozos.jp/books/makeos/#pc64bit)

```
$ cd ~/12step/src
$ wget http://core.ring.gr.jp/pub/GNU/gcc/gcc-3.4.6/gcc-3.4.6.tar.gz
$ tar zxvf gcc-3.4.6.tar.gz
$ cd gcc-3.4.6
$ vi gcc/collect2.c /* 本で解説されている修正を行う */
$ wget http://kozos.jp/books/makeos/patch-gcc-3.4.6-x64-h8300.txt
$ patch -p0 < patch-gcc-3.4.6-x64-h8300.txt /* 64bit環境用のパッチを適用 */
$ mkdir build
$ cd build
$ ../configure --target=h8300-elf --disable-nls --disable-threads --disable-shared --enable-languages=c --disable-werror
$ make -j 5
$ sudo make install
```

</br>
書籍p17の修正は上記のようにviを使用してもいいですが、Windows側から編集してみましょう。</br></br></br>
エクスプローラーを開き、アドレスバーに\\wsl$と入力</br></br>
<img src="{% post_path 12step-wsl-environment %}/explorer.jpg" /></br></br>
WSLにアクセスできました。</br></br></br>
gccディレクトリで右クリックしてCodeで開くを選択</br></br>
<img src="{% post_path 12step-wsl-environment %}/code.png" /></br></br></br>
vscodeで編集ができます。</br></br>
<img src="{% post_path 12step-wsl-environment %}/code_edit.png" /></br></br>

### kz_h8write
シリアル通信でH8/3069FのフラッシュROMに書き込みを行うツールです。h8writeの改良版とのことでこちらを使用します。
* [H8/3069F writer for KOZOS (kz_h8write)](https://ja.osdn.net/projects/kz-h8write/)
zip形式での配布のため上記サイトからだとwgetコマンドが使えません。試してみたところhtmlがダウンロードされてしまいました。
そこで、Windows側でブラウザからkz_h8write-v0.2.1.zipをダウンロードし、解凍、エクスプローラーを開きWSLの該当ディレクトリにコピーします。</br>
コピー出来たら以下のようにコマンドを打ちます。
```
$ cd ~/12step/src/kz_h8write-v0.2.1/PackageFiles/src
$ make
```
kz_h8write というバイナリがビルドされブートローダーが生成されます。
これは書き込み時に、プロジェクトのMakefileから直接参照させるのでmake installする必要はないです。</br>
とりあえず、環境構築は出来ました。
次にHello World!してみます。</br></br>

## Hello World!
まず、公式サイトからソースコードをダウンロードします。
* [「12ステップで作る 組込みＯＳ自作入門」のソースコード](http://kozos.jp/kozos/osbook_03.html)

01というフォルダが、Hello World!のプロジェクトだと思います。
このフォルダを先ほどと同じように、エクスプローラーからWSLにアクセスして、作業ディレクトリにコピーします。</br>
次に、プロジェクト内のMakefileを編集します。</br>
<img src="{% post_path 12step-wsl-environment %}/code_edit2.png" /></br></br>
16行目20行目を書き換えます。</br>
<img src="{% post_path 12step-wsl-environment %}/code_edit3.png" /></br>
16行目は上記のように書き換えればいいですが、20行目は、シリアルポート接続先なので、環境によって異なります。
Windows 10 のアップデートBuild 16176でWSLから直接シリアルポートを利用できるようになりました。

* [Serial Support on the Windows Subsystem for Linux](https://docs.microsoft.com/ja-jp/archive/blogs/wsl/serial-support-on-the-windows-subsystem-for-linux)</br>
デバイスマネージャーからマイコンで使用しているシリアルポートを確認します。</br>
<img src="{% post_path 12step-wsl-environment %}/device.png" /></br>
COM1なら/dev/ttyS1
COM2なら/dev/ttyS2
と指定します。</br>
上記の場合COM3なので/dev/ttyS3と指定します。</br>
後は書籍通り
```
$ cd ~/12step/src/01/bootload
$ make
$ make image
$ make write
```
と打てば書き込めると思います。書き込めない場合、マイコンのスイッチをチェックしてみてください。</br>
最後にseristerで監視してみようと思います。</br>
マイコンのスイッチを切り替えた後、書籍に書いてありますが、ボーレートを9600、バリティなし、バイトサイズ8、ストップビット1と設定します。
ポート番号は先ほどデバイスマネージャーで確認したポート番号を入力してください。</br>
監視開始ボタンを押し、マイコンボードのリセットボタンを押すと以下のように、Hello World!が出力されると思います。</br>
<img src="{% post_path 12step-wsl-environment %}/serister.png" /></br></br>

* 注意
  Seristerの監視をオンにしたままマイコンに書き込むとエラーが出るので、書き込む際にはSeristerの監視をオフにしましょう。
