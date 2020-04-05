---
title: processingでマンデルブロ集合を書く
date: 2020-04-05 16:38:38
thumbnail: /thumbnail/mandelplot.jpg
category:
    - Programming
    - Math
tags:
    - Art
    - Mandelbrot
    - Complex_function
---

春休みがクソ長くなったので、とりあえず何かしようと思った。
みんな大好きマンデルブロ集合をprocessingで書いてみる。

## マンデルブロ集合とは何か

説明しようと思ったけど、なんかhexoでtex書くとバグるのでやめた。

※参考リンク

[	マンデルブロー集合 ——2次関数の複素力学系入門——: 川平　友規 名古屋大学大学院多元数理科学研究科](http://www.math.titech.ac.jp/~kawahira/courses/mandel.pdf)</br>

[	マンデルブロ集合の不思議な世界](http://azisava.sakura.ne.jp/mandelbrot/definition.html)


### ソースコード
``` Java
//漸化式計算回数 
int N = 255;
// 拡大率
float SCALE = 3.0;

void setup() {
  size(512, 512);
}

void draw() {
  
  //中心に原点配置
  translate(width/2, height/2);
  background(0);

  for (int i = -width/2; i <= width/2; i++ ) {
    for (int j = -height/2; j <= height/2; j++ ) {    
      // 複素数c=x+yiを定める 
      float x = SCALE * i / width;
      float y = SCALE * j / height;
      //収束判定
      int r = calc(x, y);
      
      //収束したら黒
      if(r == 0){
        stroke(0, 0, 0);
        //点を描写
        rect(i, j, 1, 1);
      }
      else{ 
      // 発散の速度に応じて色を指定  
      stroke(r % 256 + 1, r % 16 * 16 + 1, (r % 32) * 8 + 1);
      //点を描写
      rect(i, j, 1, 1);
      }       
    }
```

### 結果

<img src="{% post_path Mandelbrot %}/mandelplot.jpg" />