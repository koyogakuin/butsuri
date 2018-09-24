---
title: フーリエ変換でお絵かきしてみよう〜実践編〜
slug: fourier3
date: '2018-09-15'
tags:
    - "数学"
    - "フーリエ変換"
    - "processing"
categories:
    - "作ってみた"
image: "/img/fourier/9.png"
description: "フーリエ変換を視覚的に理解できるお絵かきプログラムを作ってみましょう！"
author: H﨑
draft: false
comments: true
math: true
---

## はじめに
前回、フーリエ変換について詳しく見てきましたが、「いまいちピンとこない」と言う方も多かったのではないでしょうか。  
そんな方に、「なんとしてでもフーリエ変換の面白さを伝えたい！！」ということで作ったプログラムを今回の記事で説明していきたいと思います。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">フーリエ変換でピーポくん書くやつの進化版!?作った<br>ペンの軌跡をXY方向でそれぞれに離散フーリエ変換(DFT)かけて、円の回転に置き換えるというもの<br>ソースコードは、じきに公開するつもり <a href="https://t.co/4buibaN8Ri">pic.twitter.com/4buibaN8Ri</a></p>&mdash; 甲陽學院髙等學挍 物理部 (@koyobutsuri) <a href="https://twitter.com/koyobutsuri/status/1033577694498238464?ref_src=twsrc%5Etfw">2018年8月26日</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

上の動画を見ていただければわかるように、複数の円がグルグル回っていたのがわかると思います。とても複雑な動きをしているように見えますが、実は円の回転速度や大きさは一定でただ単純に円の周りを円が回っているだけです。ニョロニョロとして気持ち悪いですが、どんな絵でも円の回転運動で描画することができます。  
どのような仕組みなのか順番にご説明したいと思います。

## 仕組み
大まかな流れとしては  

1. ペンの移動をXY方向で別々にサンプリング  
2. 波形をフーリエ変換でsinとcosの成分に分割  
3. sinとcosを合成して、sinだけの関数に直す  
4. それらを元に円を描画  

sinとcosはそもそも単位円周上を回る点のXY方向の座標を表すものであるため、円運動に変換することができます。

## プログラム解説
説明が必要とされると思われる箇所を解説していきたいと思います。  

### 波形の表示
```
//ピクセルをシフトする
loadPixels();
for (int i=0; i<winY-37; i++) {
  for (int j=winX-20; j<winX+winS+20; j++) {
    pixels[i*width+j] = pixels[(i+2)*width+j];
  }
}
for (int i=0; i<winX-37; i++) {
    for (int j=winY-20; j<winY+winS+20; j++) {
    pixels[j*width+i] = pixels[j*width+i+2];
  }
}
updatePixels();
```
現在のピクセル情報をloadPixels()関数で読み取り、それを左と上に順に配列をシフトし、updatePixels()で更新することで、まるでオシロスコープのように波形を移動させています。


### 離散フーリエ変換(DFT)
```
void DFT(int n, float[] real, float[] imag)
{
  float[] tmpReal, tmpImag;
  tmpReal = new float[n];
  tmpImag = new float[n];

  for (int i = 0; i < n; i++) {
    tmpReal[i] = 0.0;
    tmpImag[i] = 0.0;
    float d = 2.0 * PI * i / n;
    for (int j = 0; j < n; j++) {
      float phase = d * j;

      tmpReal[i] += real[j] * cos(phase);
      tmpImag[i] -= real[j] * sin(phase);
    }
  }
  for (int i = 0; i < n; i++) {
    real[i] = tmpReal[i];
    imag[i] = tmpImag[i];
  }
}
```
引数として渡された波形データを下の数式を元に離散フーリエ変換を行い、配列に代入します。realは実数部分をimagは虚数部分の配列です。渡されるimagのデータは全て0であるため、計算の際は使用していません。

$$\begin{eqnarray}
F\_n &=&\frac{1}{N}\sum\_{k=0}^{N-1} f\_ke^{-i\frac{2n\pi}{N}k}\\\\\
     &=&\frac{1}{N}\sum\_{k=0}^{N-1} f\_k\Bigl(\cos(\frac{2n\pi}{N}k)-i\sin(\frac{2n\pi}{N}k)\Bigl)
\end{eqnarray}$$

### 円の座標に変換

```
  //Xについて計算
  //データ代入
  for (int i = 0; i < drawCount; i++) {
    real[i] = waveX[i];
    imag[i] = 0.0;
  }
  //離散フーリエ変換
  DFT(drawCount, real, imag);
  //円の座標に逆変換
  for (int j=0; j<drawCount; j++) {
    amplitudeX[j] = sqrt(real[j] * real[j] + imag[j] * imag[j])/drawCount*2;

    angleX[j] = atan2(real[j], -1*imag[j]);
  }
  for (int i=0; i < drawCount; i++) {
    float d = 2.0 * PI * i / drawCount;
    deltaXX[i][0] = winX+winS/2-amplitudeX[0]*sin(angleX[0])/2;
    deltaXY[i][0] = winY/2-200;
    for (int j=0; j<drawCount/2; j++) {
      float phase = d * j;
      deltaXX[i][j+1] = deltaXX[i][j] + amplitudeX[j]*sin(phase+angleX[j]);
      deltaXY[i][j+1] = deltaXY[i][j] + amplitudeX[j]*cos(phase+angleX[j]);
    }
  }
```
DFT関数でsinとcosの係数を求めることができましたが、sinとcosの和をsin単体にまとめることができれば、円の運動として表すことができるようになります。そのために使うのが**三角関数の合成公式**です。
$$a\sin\theta+b\cos\theta=\sqrt{a^2+b^2}\sin(\theta+\alpha)$$
$$\tan\alpha=\frac{b}{a}$$
問題はどうやって$\alpha$を求めるかということですが、processingには便利な関数が用意されています。  
それは**atan2関数**です。これは$\tan$の逆関数で、aとbを代入することでαの値を求めることができます。

### お待ちかねの全プログラム公開！
processingで書いたコードですので、ご自身のパソコンにprocessingをインストールして、お試しください。  
コードに関しては「動けばいいやん」という精神で記述しているので、見にくいところもあると思いますが、お許し下さい。
```
float[]  real, imag, waveX, waveY, amplitudeX, angleX, amplitudeY, angleY;
float[][] deltaXX, deltaXY, deltaYX, deltaYY;

int winX = 850;//お絵かきスペースのX座標
int winY = 600;//お絵かきスペースのY座標
int winS = 400;//お絵かきスペースの幅
int sampling = 2000;//最大サンプリング数
int delayCount = 6000;//小さくすると描画速度が速くなる

int mouseXA, mouseYA, mouseXB, mouseYB, mouseXBB, mouseYBB, drawFlag, drawCount, ii, jj;

void setup() {
  fullScreen();
  frameRate(60);
  background(30);

  //初期設定
  rect(winX, winY, winS, winS);
  line(winX+winS/2, winY+winS/2+15, winX+winS/2, winY+winS/2-15);
  line(winX+winS/2+15, winY+winS/2, winX+winS/2-15, winY+winS/2);
  mouseXB = winX+winS/2;
  mouseYB = winY+winS/2;
  mouseXBB = mouseXB;
  mouseYBB = mouseYB;

  deltaXX = new float[sampling][sampling];
  deltaXY = new float[sampling][sampling];
  deltaYX = new float[sampling][sampling];
  deltaYY = new float[sampling][sampling];
  amplitudeX = new float[sampling];
  amplitudeY = new float[sampling];
  angleX = new float[sampling];
  angleY = new float[sampling];

  real = new float[sampling];
  imag = new float[sampling];
  waveX = new float[sampling];
  waveY = new float[sampling];
}

void mousePressed() {
  //マウス位置初期化
  mouseXA = mouseX;
  mouseYA = mouseY;
  if (mouseXA < winX) {
    mouseXA = winX;
  } else if (mouseXA > winX+winS) {
    mouseXA = winX+winS;
  }
  if (mouseYA < winY) {
    mouseYA = winY;
  } else if (mouseYA > winY+winS) {
    mouseYA = winY+winS;
  }
  mouseXB = mouseXA;
  mouseYB = mouseYA;
  mouseXBB = mouseXA;
  mouseYBB = mouseYB;
}

void draw() {
  if (drawFlag == 2) {
    if (ii<drawCount) {
      noStroke();
      fill(30, 30, 30);
      rect(0, 0, width, winY);
      rect(0, 0, winX, height);

      for (int jj=0; jj<drawCount/2; jj++) {
        stroke(250, 250, 250);
        strokeWeight(1);
        fill(0, 0, 0, 0);
        ellipse(deltaXX[ii][jj+1], deltaXY[ii][jj+1], amplitudeX[jj+1]*2, amplitudeX[jj+1]*2);
        ellipse(deltaYY[ii][jj+1], deltaYX[ii][jj+1], amplitudeY[jj+1]*2, amplitudeY[jj+1]*2);
        stroke(0, 255, 0);
        strokeWeight(4);
        line(deltaXX[ii][jj], deltaXY[ii][jj], deltaXX[ii][jj+1], deltaXY[ii][jj+1]);
        line(deltaYY[ii][jj], deltaYX[ii][jj], deltaYY[ii][jj+1], deltaYX[ii][jj+1]);
      }
      stroke(0, 255, 255);
      strokeWeight(3);
      line(deltaXX[ii][drawCount/2], deltaXY[ii][drawCount/2], deltaXX[ii][drawCount/2], winY-10);
      line(deltaYY[ii][drawCount/2], deltaYX[ii][drawCount/2], winX-10, deltaYX[ii][drawCount/2]);
      stroke(0, 255, 0);
      strokeWeight(6);
      if (ii < drawCount-1) {
        line(deltaXX[ii][drawCount/2], deltaYX[ii][drawCount/2], deltaXX[ii+1][drawCount/2], deltaYX[ii+1][drawCount/2]);
      }
      delay(delayCount/drawCount);

      ii++;
    } else {
      delay(2000);
      background(30, 30, 30);
      noStroke();
      fill(30, 30, 30);
      rect(winX-100, 0, width, winY);
      rect(0, winY-100, winX, height);
      ii=0;
      drawFlag = 0;
      drawCount = 0;
      fill(255, 255, 255);
      strokeWeight(1);
      rect(winX, winY, winS, winS);
      stroke(30, 30, 30);
      line(winX+winS/2, winY+winS/2+15, winX+winS/2, winY+winS/2-15);
      line(winX+winS/2+15, winY+winS/2, winX+winS/2-15, winY+winS/2);
    }
  } else {
    //ピクセルをシフトする
    loadPixels();
    for (int i=0; i<winY-37; i++) {
      for (int j=winX-20; j<winX+winS+20; j++) {
        pixels[i*width+j] = pixels[(i+2)*width+j];
      }
    }
    for (int i=0; i<winX-37; i++) {
      for (int j=winY-20; j<winY+winS+20; j++) {
        pixels[j*width+i] = pixels[j*width+i+2];
      }
    }
    updatePixels();

    mouseXA = mouseX;
    mouseYA = mouseY;
    if (mouseXA < winX) {
      mouseXA = winX;
    } else if (mouseXA > winX+winS) {
      mouseXA = winX+winS;
    }
    if (mouseYA < winY) {
      mouseYA = winY;
    } else if (mouseYA > winY+winS) {
      mouseYA = winY+winS;
    }

    noStroke();
    fill(30, 30, 30);
    ellipse(winX-30, mouseYB, 11, 11);
    ellipse(mouseXB, winY-30, 11, 11);

    fill(0, 250, 0);
    ellipse(winX-30, mouseYA, 10, 10);
    ellipse(mouseXA, winY-30, 10, 10);

    mouseXB = mouseXA;
    mouseYB = mouseYA;

    if((mousePressed == false && drawFlag == 1) || drawCount >= sampling){
      fourier();
    }else if (mousePressed == true) {
      drawFlag = 1;
      fill(0, 250, 0);
      stroke(0, 250, 0);
      strokeWeight(6);
      line(mouseXA, winY-42, mouseXBB, winY-42);
      line(winX-42, mouseYA, winX-42, mouseYBB);
      stroke(30, 30, 30);
      line(mouseXA, mouseYA, mouseXBB, mouseYBB);
      mouseXBB = mouseXA;
      mouseYBB = mouseYB;
      waveX[drawCount] = float(mouseXA-winX-winS/2);
      waveY[drawCount] = float(mouseYA-winY-winS/2);
      drawCount++;
    }
  }

}

void fourier() {
  //Xについて計算
  //データ代入
  for (int i = 0; i < drawCount; i++) {
    real[i] = waveX[i];
    imag[i] = 0.0;
  }
  //離散フーリエ変換
  DFT(drawCount, real, imag);
  //円の座標に逆変換
  for (int j=0; j<drawCount; j++) {
    amplitudeX[j] = sqrt(real[j] * real[j] + imag[j] * imag[j])/drawCount*2;

    angleX[j] = atan2(real[j], -1*imag[j]);
  }
  for (int i=0; i < drawCount; i++) {
    float d = 2.0 * PI * i / drawCount;
    deltaXX[i][0] = winX+winS/2-amplitudeX[0]*sin(angleX[0])/2;
    deltaXY[i][0] = winY/2-200;
    for (int j=0; j<drawCount/2; j++) {
      float phase = d * j;
      deltaXX[i][j+1] = deltaXX[i][j] + amplitudeX[j]*sin(phase+angleX[j]);
      deltaXY[i][j+1] = deltaXY[i][j] + amplitudeX[j]*cos(phase+angleX[j]);
    }
  }

  //Yについて計算
  //データ代入
  for (int i = 0; i < drawCount; i++) {
    real[i] = waveY[i];
    imag[i] = 0.0;
  }
  //離散フーリエ変換
  DFT(drawCount, real, imag);
  //円の座標に逆変換
  for (int j=0; j<drawCount; j++) {
    amplitudeY[j] = sqrt(real[j] * real[j] + imag[j] * imag[j])/drawCount*2;//振幅(円の半径)
    angleY[j] = atan2(real[j], -1*imag[j]);
  }
  for (int i=0; i < drawCount; i++) {
    float d = 2.0 * PI * i / drawCount;
    deltaYX[i][0] = winY+winS/2-amplitudeY[0]*sin(angleY[0])/2;
    deltaYY[i][0] = winX/2-100;
    for (int j=0; j<drawCount/2; j++) {
      float phase = d * j;
      deltaYX[i][j+1] = deltaYX[i][j] + amplitudeY[j]*sin(phase+angleY[j]);
      deltaYY[i][j+1] = deltaYY[i][j] + amplitudeY[j]*cos(phase+angleY[j]);
    }
  }

  //円描画モードに移行
  drawFlag = 2;
  strokeWeight(8);
  stroke(255, 255, 255);
}

void DFT(int n, float[] real, float[] imag)
{
  float[] tmpReal, tmpImag;
  tmpReal = new float[n];
  tmpImag = new float[n];

  for (int i = 0; i < n; i++) {
    tmpReal[i] = 0.0;
    tmpImag[i] = 0.0;
    float d = 2.0 * PI * i / n;
    for (int j = 0; j < n; j++) {
      float phase = d * j;

      tmpReal[i] += real[j] * cos(phase);
      tmpImag[i] -= real[j] * sin(phase);
    }
  }
  for (int i = 0; i < n; i++) {
    real[i] = tmpReal[i];
    imag[i] = tmpImag[i];
  }
}
```
```
int winX = 850;//お絵かきスペースのX座標
int winY = 600;//お絵かきスペースのY座標
int winS = 400;//お絵かきスペースの幅
int sampling = 800;//最大サンプリング数
int delayCount = 6000;//小さくすると描画速度が速くなる
```
ここの箇所はご自身の環境に合わせて調節して見て下さい。
