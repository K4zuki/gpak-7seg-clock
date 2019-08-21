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
## クロック・リセット入力部
## ＢＣＤ（１０進）カウンタ部

10進カウンタは思ったより簡単でした。参考サイト[^ref-bcd-counter]によると、リセットつきDFF4段と2入力NANDゲートで
作れます。GreenPAKのDFFはQ出力かnQかのいずれかしか選べないので、2入力LUTを4つ消費して各桁の
nQを作ります。これらは次の桁のクロック入力にもなります。

[^ref-bcd-counter]: <https://www.petervis.com/dictionary-of-digital-terms/bcd-counter-using-d-flip-flop/bcd-counter-using-d-flip-flop.html>

## ＢＣＤ-７セグデコーダ部

[セグメント](data/segments.txt){.aafigure}

[真理値表](data/truth-table.csv){.table alignment=ccccccccccc}

### セグメントごとの論理式
## クロック分周部
## 時刻セットをどうするか
## まとめ
