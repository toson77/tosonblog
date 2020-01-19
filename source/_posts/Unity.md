---
title: UnityとKinectで作るジェネレーティブアート:Part1
thumbnail: /thumbnail/unity1.jpg
date: 2020-01-09 01:01:15
category:
  - programming
tags:
  - Kinect
  - Unity
  - Art
---

今更感あるけど、
今年の学祭(去年になっちゃったね)で展示したものの紹介。<br/>

サークルの備品でほこりかぶってた Kinect をどうにか活用できないかなと思って、手を付けたが、初代 Kinect(v1)であったため情報が少なく、レポート等で忙しく(言い訳）、結局 2 週間程度で無理やり形に持って行った。完成度は高いとはあまり言えないのであしからず。<br/>

最初ゲームを作ろうと思ったが、なぜか当たり判定が死んで、半日程度格闘したが原因がわからなかったので別のものを作ろうと考えた。サークルの出し物がデザイン寄りなのも考えて、ジェネラティブアート的なのを作ろうと思った。<br/>

「Unity ジェネラティブアート」でググると一番上にこのサイトが出てくる。</br>

https://ics.media/entry/19479/

これを Kinect で制御したらまあまあ面白いものができるのではと思った。</br>
描写に関しては上のサイトとほとんど同じ...だとつまらないので、いくつかモードを追加した。</br></br>

Kinect の前に人がいないとき、キャンパスに説明などを描写する待機モード。
<img src="{% post_path Unity %}/taiki.png" />
</br>
上のサイトをもとに作った、描写モード 1。
<img src="{% post_path Unity %}/play1.png" />
<img src="{% post_path Unity %}/play12.png" />
</br>
自作した、花火みたいなエフェクトを表示させる描写モード 2。
<img src="{% post_path Unity %}/play21.png" />
<img src="{% post_path Unity %}/play22.png" />

簡単に説明すると、Kinect が人を認識していないとき待機モードになる。

人を認識すると描写モードに遷移する。現在時刻が 0-19 分の時、描写モード 1 を表示。現在時刻が 20-59 分の時、描写モード 2 を表示する。</br>例えば 17:09 の時、描写モード 1 を表示。10:34 の時、描写モード 2 を表示する。</br>

描写モード 1 では、左腕の肘より左手の位置が高い時、時計回りに玉が回転する。逆に左腕の肘より左手の位置が低い時、逆時計回りに玉が回転する。</br>

描写モード 2 では、両手を左右に動かすと玉の生成位置が移動する。X 軸(横方向)しか取得していないため上下は反応しない。

動画が残っていたため以下に添付する。</br>(描写モード 1 の動画はどこかに無くした...)

<div style="width:100%;height:0px;position:relative;padding-bottom:56.250%;"><iframe src="https://streamable.com/s/nphfo/kljf" frameborder="0" width="100%" height="100%" allowfullscreen style="width:100%;height:100%;position:absolute;left:0px;top:0px;overflow:hidden;"></iframe></div>
</br>

補足説明として、
待機モードから描写モードに移るときに来場者人数をカウントする。その際、カウントした値は Unity 内で保管されるわけではなく、localhost の PHP サーバー上に保管される。相方が実装したのでよく分からないが、インターネット環境下で外部に PHP サーバーを置けば、外部から今来場者が何人来たか確認できるらしい。サーバー上に保管することによって、一旦アプリケーションを閉じて、再度実行しても来場者数はリセットされずにカウントを続けることができる。

描写モード 1 では、ほとんどスクリプトを書いていないので、描写モード 2 について書こうと思うが、長くなりそうなので物体生成のスクリプトだけ説明する。

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FlowerCircle : MonoBehaviour
{
// このコルーチンの処理を待たせる時間
public float WaitTime = 0.5f;

    public float AllWaitTime = 1.0f;

    public float WaitobjectTime = 0.05f;
    // 周りに生成するオブジェクト
    public GameObject[] objects;
    // 周りに生成するオブジェクト (その子供)
    public GameObject[] childobjects;
    // 関節の番号 (自分で振る)
    public int JointNumber = 0;

    // オブジェクトを削除する時間
    public float ObjDeleteTime;
    // 子供のオブジェクトを削除する時間
    public float childObjDeleteTime;
    // 周りに生成するオブジェクト数
    public int circleObjMax = 12;
    // 座標の中心の object
    public GameObject BaseObj;
    // copy する object
    private GameObject CloneObj;
    private Rigidbody[] rb;


    // Use this for initialization
    void Start()
    {

        objects = new GameObject[50];
        rb = new Rigidbody[50];
        childobjects = new GameObject[50];
        // オブジェクトを削除する時間
        ObjDeleteTime = WaitTime * 11f;
        childObjDeleteTime = WaitTime * 10.05f;

        // circle 生成処理
        StartCoroutine("OnCreateCircle");
    }

    IEnumerator OnCreateCircle()
    {
        //円の大きさ
        int CircleDouble = 2;

        while (true)
        {
            if (DataCenter.IsDetected[JointNumber] && DataCenter.GameMode == 1)
            {
                // プレイヤーの座標取得 (更新)
                Vector3 basePos = BaseObj.gameObject.transform.position;
                // 周りのオブジェクトを生成
                for (int circleObjIdx = 0; circleObjIdx < circleObjMax; circleObjIdx++)
                {
                    CloneObj = (GameObject)Resources.Load("MovingCreate");

                    // 正規化されたベクトル
                    Vector3 objVec = new Vector3(
                        CircleX(circleObjIdx, circleObjMax, CircleDouble),
                        CircleY(circleObjIdx, circleObjMax, CircleDouble),
                        0
                    );

                    // 周りの円の位置を計算(1回目)
                    Vector3 objPos = basePos + objVec;
                    // オブジェクトを生成
                    objects[circleObjIdx] = Instantiate(CloneObj, objPos, Quaternion.identity);

                    //rigidbody取得
                    Rigidbody rb = objects[circleObjIdx].GetComponent<Rigidbody>();
                    //オブジェクトに放射状に力を加える
                    rb.AddForce(objVec * 100);

                    // 作ったオブジェクトを一定時間後に消す
                    Destroy(objects[circleObjIdx], ObjDeleteTime);

                    // fade outを行う
                    StartCoroutine("FadeOut", objects[circleObjIdx]);

                }

                yield return new WaitForSeconds(WaitobjectTime);

                // 一回目に生成した円中心に再度描写
                for (int circleObjIdx = 0; circleObjIdx < circleObjMax; circleObjIdx++)
                {
                    for (int circleChildObjIdx = 0; circleChildObjIdx < circleObjMax; circleChildObjIdx++)
                    {
                        // 正規化されたベクトル
                        Vector3 objChildVec = new Vector3(
                            CirclechildX(circleChildObjIdx, circleObjMax, CircleDouble),
                            CirclechildY(circleChildObjIdx, circleObjMax, CircleDouble),
                            0
                        );

                        // 周りの円の位置を計算
                        Vector3 objChildPos = objects[circleObjIdx].gameObject.transform.position + objChildVec;

                        // オブジェクトを生成
                        childobjects[circleChildObjIdx] = Instantiate(CloneObj, objChildPos, Quaternion.identity);
                        //rigidbody取得
                        Rigidbody rbc = childobjects[circleChildObjIdx].GetComponent<Rigidbody>();

                        //オブジェクトに放射状に力を加える
                        rbc.AddForce(objChildVec * 100);

                        Destroy(childobjects[circleChildObjIdx], childObjDeleteTime);
                        // fade outを行う
                        StartCoroutine("FadeOut", childobjects[circleChildObjIdx]);

                        yield return new WaitForSeconds(WaitobjectTime / circleObjMax);
                    }
                }
            }

            yield return new WaitForSeconds(AllWaitTime); // n 秒処理を待つ
        }
    }

    IEnumerator FadeOut(GameObject obj)
    {
        Vector3 objVecSub = new Vector3(0.2f, 0.2f, 0);

        for (int i = 0; i < 5; i++)
        {
            obj.transform.localScale = obj.transform.localScale - objVecSub;

            yield return new WaitForSeconds(1);
        }
    }

    //一回目に円状に生成される玉のX座標
    float CircleX(int circleObjNum, int circleObjMax, int Double)
    {
        float angle = circleObjNum * 360 / circleObjMax;
        float x = Mathf.Sin(angle * (Mathf.PI / 180));
        return x * Double;
    }

//一回目に円状に生成される玉の Y 座標
float CircleY(int circleObjNum, int circleObjMax, int Double)
{
float angle = circleObjNum _ 360 / circleObjMax;
float y = Mathf.Cos(angle _ (Mathf.PI / 180));
return y \* Double;
}

//一回目に生成された円を中心にして再度円状に生成される玉の X 座標
float CirclechildX(int circleObjNum, int circleObjMax, int Double)
{
float angle = circleObjNum _ 360 / circleObjMax + 360 / circleObjMax / 2;
float x = Mathf.Sin(angle _ (Mathf.PI / 180));
return x \* Double;
}

//一回目に生成された円を中心にして再度円状に生成される玉の Y 座標
float CirclechildY(int circleObjNum, int circleObjMax, int Double)
{
float angle = circleObjNum _ 360 / circleObjMax + 360 / circleObjMax / 2;
float y = Mathf.Cos(angle _ (Mathf.PI / 180));
return y \* Double;
}
}

```

玉が一斉に表示されてグチャグチャにならないように、コルーチンを使って玉が生成される時間に差をつけることでアニメーションを作った。(コルーチンの変数名が適当なのは申し訳ない...)

</br>
<div style="text-align: center;">
プレイヤーの座標取得
</div>
<div style="text-align: center;">
↓
</div>
<div style="text-align: center;">
プレイヤーの取得座標を中心に円状に玉を配置して放射状に力を加える
</div>
<div style="text-align: center;">
↓
</div>
<div style="text-align: center;">
WaitobjectTime秒待つ
</div>
<div style="text-align: center;">
↓
</div>
<div style="text-align: center;">
最初に生成された玉1つを中心に玉を円状に玉を配置して放射状に力を加える。</br>
(玉1つ生成毎にWaitobjectTime / circleObjMax秒待つ)
</div>
<div style="text-align: center;">
↓
</div>
<div style="text-align: center;">
再度、最初に生成された玉1つを中心に玉を円状に玉を配置して放射状に力を加える。</br>
(玉1つ生成毎にWaitobjectTime / circleObjMax秒待つ)</br>
(最初に生成された玉の数(circleObjMax)回繰り返す。)
</div>
<div style="text-align: center;">
↓
</div>
<div style="text-align: center;">
AllWaitTime秒待つ
</div>
<div style="text-align: center;">
↓
</div>
<div style="text-align: center;">
最初に戻る
</div>

</br></br>

文章だとわかりにくいので図で簡潔に説明してみる。

circleObjMax=3 のとき

最初、プレイヤーの取得座標を原点と仮定すると下図のようになる。
<img src="{% post_path Unity %}/object.png" />

次に最初に生成された玉を中心に再度下図のように描写。
<img src="{% post_path Unity %}/childobject.png" />

みたいな感じ...</br>
circleObjMax(玉の数)を変えても均等に配置される。

今回初めて github を使って共同制作した。複数人で 1 つの作品を制作-公開するにはすごく便利なツールだなと思った。(小並感)

他の説明は相方に任せます。</br>
<a href="https://over-hk.net/articles/kinect-unity">Unity と Kinect で作るジェネレーティブアート:Part2</a></br>
スクリプトは相方の github 上で公開してます。</br>
<a href="https://github.com/H37kouya/kinect-unity">github:H37kouya/kinect-unity</a>
