::: rmnote
![](images/MilkyWay_Stargate_blank.svg)
:::

# まえがき {-}

この本は、*ふとした思いつきで夜でも視認性の良い時計が欲しくなったので作ってみ*るまでをまとめた
ドキュメントです。

マイコンによる時計の作例は山ほどあることと、**勝手にアプリケーションノート**のネタに
ちょうどよかったことからGreenPAKを採用しました。

この本に出てくる回路は**SLG46826**のために設計されています。当初はマクロセルが不足
するかと考えていましたがうまく入りました。

# そうだ、時計、作ろう
## ブロック図

[@fig:block-diagram]にブロック図を示します。RTCによる１Hzをクロック源にして、
同じ回路情報を書き込んだGPAKを5または6個、時分(秒)のカウントに用います。

[Block diagram](data/block-diagram.bob){.svgbob #fig:block-diagram}

## 全体図
## クロック生成・リセット入力部

最初はクロック源について考えます。この本で作ろうとしているものは時計なので、より高精度なクロック源が必要です。
SLG46826は専用の水晶発振回路ブロックを持っていないので、IOピンを使って実現するか、RTCから
受ける必要があります。幸いIOピンを使う例がアプリケーションノートとして提供されているので大いに参考にし、
どちらの選択も取れるようにしました。

### 下調べ：32.768KHzのクロック回路は意外とめんどくさい

早速32.768KHz（~~めんどいので~~今後は32Kと表記）の水晶発振回路について調べます。
よくあるマイコンを使った作例では簡単に水晶1個と12pFくらいのコンデンサ2個でちゃちゃっと32Kを用意しちゃってますが、
アプリケーションノートによると、20M&Omega;、1M&Omega;、330pFなどの周辺部品を必要とするようです。
:::

## ＢＣＤ（１０進）カウンタ部

10進カウンタは思ったより簡単でした。参考サイト[^ref-bcd-counter]によると、リセットつきDFF4段と2入力NANDゲートで
作れます。GreenPAKのDFFはQ出力かnQかのいずれかしか選べないので、2入力LUTを4つ消費して各桁の
nQを作ります。これらは次の桁のクロック入力にもなります。

[^ref-bcd-counter]: <https://www.petervis.com/dictionary-of-digital-terms/bcd-counter-using-d-flip-flop/bcd-counter-using-d-flip-flop.html>

## ＢＣＤ-７セグデコーダ部

BCDカウンタの出力をもとに7セグLEDを光らせるための信号を作ります。各セグメント
の位置と名前は[@fig:segments]の通りです。

[セグメント](data/segments.txt){.aafigure #fig:segments}

[真理値表](data/truth-table.csv){.table alignment=ccccccccccc width=[0.1,0.1,0.1,0.1,0.05,0.05,0.05,0.05,0.05,0.05,0.05]}

### セグメントごとの論理式
## クロック分周部
## 時刻セットをどうするか
## まとめ

# あとがき {-}

- 原稿PDFはこのQRコードからたどってください ![](images/QRcode.png){#fig:manuscript width=30%}
- `{\large`{=latex}[Stargate SG-1]{.underline}っていう海外SFドラマシリーズ知ってる人いますか？`}`{=latex}
- 表紙の画像は
  [<https://commons.wikimedia.org/wiki/File:MilkyWay_Stargate_blank.svg>]{.underline}から拝借しました。
  [[CC BY-SA 3.0 ライセンス]{.underline}](https://creativecommons.org/licenses/by-sa/3.0/deed.en)です。

## 謝辞 {-}

「Chevron1」ならびに「Chevron7」の企画・設計に協力してくださった諸氏に謝意を示します（敬称略）。

- そんそん(<https://twitter.com/sonson1919>)
- あおいさや(<https://twitter.com/La_zlo>)
- 余熱＠れすぽん(<https://twitter.com/yone2_net>)
- niszet(<https://twitter.com/niszet0>)
- Kashiken(<https://twitter.com/kashiken>)
