---
title: Processingでライフゲーム
slug: lifegame
date: '2018-05-03'
tags:
    - "ライフゲーム"
    - "Processing"
categories:
    - "作ってみた"
image: "/img/3DP/1.png"
description: "ライフゲームってご存知ですか？まるで生きているかのような振る舞いをする、とても興味深いプログラムです。"
author: 濱﨑 拓
draft: true
comments: true
---




## きっかけ
いつものように、物理部で数学の勉強してたのですが(ｵｲ)、そこにいた部員の一人が「ライフゲームって知ってる？」と発言したことが発端です。  
僕自身、全く知らなかったので、即興でProcessingでコードを書くことに。  
残念ながら謎のバグのせいで、その日の物理部では完成しませんでしたが、家に帰って完成させたので紹介します。

## ライフゲームとは？
<div class="movie-wrap">
  <iframe width="854" height="480" src="https://www.youtube.com/embed/ZOkm867AleM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

ルールはいたって簡単。  

> 誕生  
> 死んでいるセルに隣接する生きたセルがちょうど3つあれば、次の世代が誕生する。  
> 生存  
> 生きているセルに隣接する生きたセルが2つか3つならば、次の世代でも生存する。  
> 過疎  
> 生きているセルに隣接する生きたセルが1つ以下ならば、過疎により死滅する。  
> 過密  
> 生きているセルに隣接する生きたセルが4つ以上ならば、過密により死滅する。  
> (Wikipedia引用)

こんだけです。
一つずつのセルに対して、８近傍の状態を調べて次の世代を決定するというプログラムを組めば良いわけです。

## ソースコード
```
import controlP5.*;

int gw, gh;
int block = 7;
int delayTime = 1;
int s_count = 0;

int weflag = 0; //0-normal 1-write 2-erace

boolean[][] flag;
boolean[][] flag_new;

ControlP5 cp5;
Textlabel label;
Button Button1, Button2, Button3;

int[][] charac = new int[][]{
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1},
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1},
  {0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}};

void setup() {
  background(0, 0, 0);
  size(displayWidth, displayHeight);
  gw = displayWidth/block;
  gh = displayHeight/block;
  flag = new boolean[gw][gh];
  flag_new = new boolean[gw][gh];

  PFont pfont = createFont("Arial", 20, true); // use true/false for smooth/no-smooth
  ControlFont font = new ControlFont(pfont, 20);
  cp5 = new ControlP5(this);
  Button1 = cp5.addButton("init")
    .setLabel("初期化")//テキスト
    .setPosition(displayWidth-200, 40)
    .setSize(95, 40)
    .setFont(font);
  Button2 = cp5.addButton("erace")
    .setLabel("消去")//テキスト
    .setPosition(displayWidth-300, 40)
    .setSize(95, 40)
    .setFont(font);
  Button3 = cp5.addButton("write")
    .setLabel("書込")//テキスト
    .setPosition(displayWidth-400, 40)
    .setSize(95, 40)
    .setFont(font);

  label = cp5.addTextlabel("label");
  label.setText("");
  label.setPosition(displayWidth-325, 10);
  label.setColorValue(0xffffffff);
  label.setFont(createFont("Arial", 20, true));

  cp5.addSlider("delayTime")
    //.setLabel("bbb")
    .setRange(0, 1000)
    .setValue(0)//初期値
    .setPosition(displayWidth-400, 90)//位置
    .setSize(295, 20);//大きさ

  strokeWeight(0);
  frameRate(50);
  //reset();
}


void draw() {
  if (weflag != 0 && mousePressed == true) {
    int X = mouseX/block;
    int Y = mouseY/block;
    if (X >= 0 && Y >= 0  && X < gw && Y < gh) {
      if (weflag == 1) {
        flag_new[X][Y] = true;
      } else {
        flag_new[X][Y] = false;
      }
      display();
    }
  }else if ((millis()-s_count) > delayTime && weflag == 0) {
    s_count = millis();
    for (int i=1; i < gw-1; i++) {
      for (int j=1; j < gh-1; j++) {
        if (flag[i][j] == false) {
          if (count(i, j) == 3) {
            flag_new[i][j] = true;
          } else {
            flag_new[i][j] = false;
          }
        } else {
          if (count(i, j) <= 1 || count(i, j) >= 4) {
            flag_new[i][j] = false;
          } else {
            flag_new[i][j] = true;
          }
        }
      }
    }
  }
  display();
}

void display() {//更新を表示
  for (int i=0; i < gw; i++) {
    for (int j=0; j< gh; j++) {
      if (flag[i][j] == true && flag_new[i][j] == false) {
        fill(0, 0, 0);
        rect(block*i, block*j, block, block);
        flag[i][j] = flag_new[i][j];
      } else if (flag[i][j] == false && flag_new[i][j] == true) {
        fill(int(random(50, 255)), 255, 0);
        rect(block*i, block*j, block, block);
        flag[i][j] = flag_new[i][j];
      }
    }
  }
}

void reset() {//設定を初期化
  for (int i=0; i < gw; i++) {
    for (int j=0; j< gh; j++) {
      flag[i][j] = true;
      flag_new[i][j] = false;
    }
  }
  for (int j=0; j<9; j++) {
    for (int i=0; i<37; i++) {
      if (charac[j][i] == 1) {
        flag_new[i+20][j+5] = true;
      }
    }
  }
  for (int j=0; j<9; j++) {
    for (int i=0; i<37; i++) {
      if (charac[j][i] == 1) {
        flag_new[i+20][j+30] = true;
      }
    }
  }
  for (int j=0; j<9; j++) {
    for (int i=0; i<37; i++) {
      if (charac[j][i] == 1) {
        flag_new[i+20][j+55] = true;
      }
    }
  }
  display();
}

int count(int x, int y) {//周辺をカウント
  int count = 0;
  for (int a=0; a < 3; a++) {
    for (int b=0; b < 3; b++) {
      if (flag[x+a-1][y+b-1] == true) {
        count++;
      }
    }
  }
  if (flag[x][y] == true) {
    count--;
  }
  return count;
}

void init() {
  rect(displayWidth-325, 10, 100, 40);
  label.setText("");
  weflag = 0;
  reset();
}

void write() {
  if (weflag == 1) {
    fill(0, 0, 0);
    rect(displayWidth-325, 10, 110, 40);
    label.setText("");
    weflag = 0;
  } else {
    fill(0, 0, 0);
    rect(displayWidth-325, 10, 110, 40);
    label.setText("書込モード");
    weflag = 1;
  }
}
void erace() {
  if (weflag == 2) {
    fill(0, 0, 0);
    rect(displayWidth-325, 10, 110, 40);
    label.setText("");
    weflag = 0;
  } else {
    fill(0, 0, 0);
    rect(displayWidth-325, 10, 110, 40);
    label.setText("消去モード");
    weflag = 2;
  }
}
```

ライフゲーム単体だけであれば、もっと短くて済みますが、他の機能として、

- 消去、書き込み機能
- スピード変更機能
- グライダー銃をプリセット

を追加。ちょっと長くなってしまいました。（頑張ればもうちょっと短くなると思いますが）

仕組みとしてはとても単純なのですが、
