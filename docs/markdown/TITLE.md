::: rmnote
![](images/MilkyWay_Stargate_blank.svg)
:::

\toc

# まえがき {-}

## このドキュメントは何

この本は、*ふと欲しくなった「夜でも視認性の良い時計」を作ってみ*るまでをまとめた
ドキュメントです。「GreenPAK勝手に勉強会」というTwitterフレンズの集まり（サークルではない）の協力を得て
作りました。

### Chevron（しぇぶろん）とは

Chevronは映画「スターゲイト(STARGATE)」とその後のSF TVシリーズ「スターゲイト SG-1」[^when-netflix-comes]に登場する
ワームホール装置「スターゲイト」の部品の一つで、全部で９個ついています。映画の中ではワームホールを生成するために
７個のシンボル（目的地６個、最後に始点１個）を入力します。１つめがChevron one、最後がChevron sevenです。

[^when-netflix-comes]: アメリカ本国ではNETFLIXでTVシリーズが配信開始しました。
それを聞いてすぐに契約したんですが、日本には全然降ってきませんねー。吹替版はすでにあるのになー。

### なぜ[７]{.underline}なのか

最初にこの回路を設計しだしたとき、SLG46826を６個使う計画でした。クロック源で１個、分クロックに分周するために１個、
残りの４個で時分各桁の分周と表示を担っていました。**もうおわかりですね。**
その６個をChevron１〜６にたとえて、それらを載せる時計基板を７に割り当てたのです。

その後、勝手に勉強会の一部メンバによる協力を得て、というか９割方あおいさや氏(<https://twitter.com/La_zlo>)によって、
なんと２個にまで圧縮されました[^sasusaya]。クロック源は別個にRTCを１個使うことにして、全部で３個です。

[^sasusaya]: **さす**~がですわ~**さや**~さん~

## このドキュメントのゴール

マイコンによる時計の作例は山ほどあることと、**勝手にアプリケーションノート**のネタに
ちょうどよかったことからGreenPAKを採用しました。

この本に出てくる回路は**SLG46826**のために設計されています。

# そうだ、時計、作ろう

## 機能概要

この作例「Chevron7」は「Chevron1」基板を2枚（すなわちSLG46826を２個）、RTCとトランジスタアレイを１個ずつ使っています。

![完成基板外観](images/chevron7-topview.jpg){#fig:assy-topview width=150mm}

## ブロック図

[@fig:block-diagram]にブロック図を示します。RTCによる32.768KHzをクロック源にして分周し、
4桁のBCDにエンコードするブロック（Config-1）、LEDのダイナミック点灯とＢＣＤ−７セグデコーダブロック（Config-2）
に大別され、それぞれにSLG46826を1個ずつ割り当てています（[@fig:block-diagram]）。LEDはカソードコモンタイプを使います。
カソードドライバにはTBD62083（８回路トランジスタアレイ）を使います。

<!--\newpage-->

[Block diagram](data/block-diagram-2.bob){.svgbob #fig:block-diagram height=150mm}

[Block diagram](data/mux-13.bob){.svgbob}

\newpage

## クロック生成部（RX-8581NB RTC）

最初はクロック源について考えます。この本で作ろうとしているものは時計なので、できれば高精度なクロック源が望まれます。
SLG46826は専用の水晶発振回路ブロックを持っていないので、IOピンを使って実現する[^io-oscillator-an]か、
RTC（リアルタイムクロック）などを基準クロックにする必要があります。
今回はRTC「RX-8581NB[^rx8581nb]」の32.768KHz出力（~~めんどいので~~今後は32Kと表記）を受け分周する設計にしました。
このICはFOEピンをHにするとでFOUTピンから基準クロック（デフォルトは32.768KHz）がでてきます。本来ならI^2^Cによる外部アクセスで
数種類の周波数から設定できますが、GPAKにはI^2^CやSPIなどのシリアル信号をホストする能力はありません[^would-work-but-waste]。
したがって、ロジックのみでクロック出力と周波数の設定ができる品種を選定する必要がありました。

[^io-oscillator-an]: IOピンを使うオシレータ回路の例がアプリケーションノートとして提供されています。\
<https://www.dialog-semiconductor.com/AN-CM-233>
[^rx8581nb]: https://www5.epsondevice.com/ja/products/rtc/rx8581nb.html

![RX-8581NB ピン配置（データシートより抜粋）](images/RX-8581NB_pinout.png){#fig:rtc-pinout width=100mm}

[^would-work-but-waste]: こうやって断定的に無理というのもどうかと思いますが、
少なくとも既存の品種は全部、マクロセルを全部消費しなければならない程度には小規模なのです。

:::rmnote
<https://www.dialog-semiconductor.com/sites/default/files/an-cm-233_external_oscillator_solutions_with_greenpak.pdf>

### 下調べ：32.768KHzのクロック回路は意外とめんどくさい

早速32.768KHz（~~めんどいので~~今後は32Kと表記）の水晶発振回路について調べます。
よくあるマイコンを使った作例では簡単に水晶1個と12pFくらいのコンデンサ2個でちゃちゃっと32Kを用意しちゃってますが、
アプリケーションノートによると、20M&Omega;、1M&Omega;、330pFなどの周辺部品を必要とするようです。
:::

\newpage

## 計時（分周）部（Config-1）

![分・秒・早送りクロック生成部](images/clock-divider-block.png){width=120mm #fig:clock-divider-block}

### 分・秒クロック生成部

今作の表示は時分だけで秒はコロンの明滅に使います。原発振32KHzを分周して秒クロック（1Hz）を得るために、
オシレータモジュール`OSC0`と、DFFの代替としてリセットカウンタを使います。リセットカウンタでDFFを代用して分周する方法は
時クロックのカウントにも使います（後述）。これらのリセットカウンタはマルチファンクションモジュール`MF6`と`MF7`を使っています。

`OSC0`はデフォルト設定では2.048KHzのRC発振器とその分周出力を出しますが、外部ピンからの入力をクロック源とすることもできます。
クロック源にできるピンは各モジュールで固有であり、`OSC0`はPIN2（IO0）が該当します。

`OSC0`は32Kから2Hzと128Hzに分周します。2Hzをリセットカウンタ`CNT6/DLY6`（`MF6`）で分周して秒クロック（1Hz）にして、さらに1Hzを
再度リセットカウンタ`CNT7/DLY7`（`MF7`）で60分周して分クロック（1/60Hz）を得ます。128Hzは`PGEN`で分周して32Hzを得て、
早送りモードに使います。コロンの明滅はEN3信号がHのときに有効になります(`MF6-LUT12`)。
1Hzと32Hzの切り替えはボタン入力をDFF3でラッチして`LUT13`マルチプレクサ（`MF7`）で選択します。

::: {.table width=[0.4,0.4]}
Table: `PIN 2`のコンフィグ; `IO0` "XIN"

| Property       | Value                                                 |
|:---------------|:------------------------------------------------------|
| I/O selection  | Digital input In mode Digital in with Schmitt trigger |
| Out mode       | None                                                  |
| Resistor       | Pull Down                                             |
| Resistor value | 1M                                                    |

:::
::: {.table width=[0.4,0.4]}
Table: `PIN 13`のコンフィグ; `IO8` "SW"

| Property       | Value                                                 |
|:---------------|:------------------------------------------------------|
| I/O selection  | Digital input In mode Digital in with Schmitt trigger |
| Out mode       | None                                                  |
| Resistor       | Pull Down                                             |
| Resistor value | 10K                                                   |

:::
::: {.table width=[0.4,0.4]}
Table: `OSC0`のコンフィグ {#tbl:config-osc0}

| Property                 | Value                      |
|:-------------------------|:---------------------------|
| Control pin mode         | Force on                   |
| OSC power mode           | Force Power On             |
| Clock selector           | EXT CLK (From PIN 2 (IO0)) |
| 'CLK' predivider by      | 4                          |
| 'OUT0' second divider by | 64                         |
| 'OUT1' second divider by | 24                         |

:::

\newpage

::: {.table width=[0.4,0.4]}
Table: `MF6`のコンフィグ; `CNT6/DLY6` "1sec counter"

| Property                | Value              |
|:------------------------|:-------------------|
| Mode                    | Reset counter      |
| Counter data            | 1                  |
| Output period (typical) | N/D                |
| Edge select             | Rising             |
| DLY IN init. value      | Bypass the initial |
| Output polarity         | Non-inverted (OUT) |
| Mode signal sync.       | Bypass             |
| Clock                   | OSC0 /4096         |
| Clock frequency         | N/D                |

:::
::: {.table width=[0.1,0.1,0.1,0.1]}
Table: `MF6`のコンフィグ; `3-bit LUT12`（Defined by user）

| in2 | in1 | in0 | **out** |
|:---:|:---:|:---:|:-------:|
|  0  |  0  |  0  |  **0**  |
|  0  |  0  |  1  |  **0**  |
|  0  |  1  |  0  |  **0**  |
|  0  |  1  |  1  |  **0**  |
|  1  |  0  |  0  |  **0**  |
|  1  |  0  |  1  |  **1**  |
|  1  |  1  |  0  |  **1**  |
|  1  |  1  |  1  |  **1**  |

:::

\newpage

::: {.table width=[0.4,0.4]}
Table: `MF7`のコンフィグ; `CNT7/DLY7` "1min counter"

| Property                | Value            |
|:------------------------|:-----------------|
| Mode                    | Reset counter    |
| Counter data            | 59               |
| Output period (typical) | N/D              |
| Edge select             | High level reset |
| DLY IN init. value      | Initial 0        |
| Output polarity         | Inverted (nOUT)  |
| Mode signal sync.       | Bypass           |
| Clock                   | CNT6/DLY6 (OUT)  |
| Clock frequency         | N/D              |

:::

<!--\newpage-->

::: {.table width=[0.1,0.15,0.15,0.1]}
Table: `MF7`のコンフィグ; `3-bit LUT13`（Multiplexer）

| s (SW) | a (1Hz) | b (32Hz) | z |
|:------:|:-------:|:--------:|:-:|
|   0    |  **0**  |    0     | 0 |
|   0    |  **0**  |    1     | 0 |
|   0    |  **1**  |    0     | 1 |
|   0    |  **1**  |    1     | 1 |
|   1    |    0    |  **0**   | 0 |
|   1    |    0    |  **1**   | 1 |
|   1    |    1    |  **0**   | 0 |
|   1    |    1    |  **1**   | 1 |

:::
::: {.table width=[0.4,0.4]}
Table: `PGEN`のコンフィグ; `2-bit LUT3` "32Hz"

| Property  | Value |
|:----------|:------|
| Type      | PGEN  |
| Bit range | 3 : 0 |
| Pattern   | 0011  |

:::

\newpage

### 早送りモード

今作では時刻合わせが課題でした。先述の通り、GPAKはRTCを操作できません。代わりの単純な方法として
秒クロックを切り替えて早送りする方法を採りました。

### 時・分カウント部

1桁の10進カウンタは思ったより簡単です。参考サイト[^ref-bcd-counter]によると、非同期リセットつきDFF4段と2入力NANDゲート1個で
作れます。GreenPAKのDFFはQ出力かnQかのいずれかしか選べないので、nQを選択し、次の桁のクロック入力にします。
この場合リセット信号生成はNANDではなくORゲートになります（[@fig:digit4-counter]）。

![BCD counter(inverted)](images/digit4-counter.png){width=100mm #fig:digit4-counter}

ですが、これを単純に4桁作ろうとすると、DFFマクロセルが16個必要になり1石では不足します。そこでまず、必要なDFFの桁数を減らします。
２４時間計は0000から2359までのカウントをしますが、このうち最上位桁は0/1/2で２ビット、３桁目は0~5で３ビットあれば繰り上がりができます。
これで16個が2+4+3+4=13個になります。

SLG46826には最大3ビットの任意リセットカウンタマクロセルがあるので、これを10分の桁（digit3）に使います（[@fig:digit3-counter]）。

![10分カウンタ](images/digit3-counter.png){width=100mm #fig:digit3-counter}

::: {.table width=[0.5,0.5]}
Table: `Ripple Counter`のコンフィグ; `3-bit LUT6`

| Property           | Value                  |
|:-------------------|:-----------------------|
| Type               | Ripple Counter         |
| nSET value (SV)    | 0                      |
| End value (EV)     | 5                      |
| Functionality mode | Full: 0-EV/EV-0 cycles |

:::

これで1分・10分の桁ができました。残りの二桁はマルチファンクションブロック内のカウンタ・タイマモジュールを
数珠つなぎにします。カウンタ・タイマモジュールはクロック源に他のカウンタ・タイマモジュールの出力を 指定することができます。

\newpage

![時カウンタ](images/digit21-counter.png){height=180mm #fig:digit21-counter}

\newpage

::: {#fig:multiplexers width=[0.6,0.6]}
::: {.table noheader=true}

| [C13/D13](data/mux-13.bob){.svgbob width=75mm #fig:multiplexers1} | [A24/B24/C24](data/mux-abc24.bob){.svgbob width=75mm #fig:multiplexers2} |
|:-----------------------------------------------------------------:|--------------------------------------------------------------------------|
|  [D24](data/mux-d24.bob){.svgbob width=75mm #fig:multiplexers3}   |                                                                          |

:::
MUX
:::

[^ref-bcd-counter]: <https://www.petervis.com/dictionary-of-digital-terms/bcd-counter-using-d-flip-flop/bcd-counter-using-d-flip-flop.html>

### 時分割多重出力部（Config-1）

::: {#fig:en-gen width=[0.5,0.5]}
::: {.table noheader=true}

| ![EN24/EN3生成](images/en3-en24-gen.png){height=60mm #fig:en-gen1} | ![EN24X生成](images/en24x-gen.png){height=60mm #fig:en-gen2} |
|:----------------------------------------------------------------:|:----------------------------------------------------------:|
|                                                                  |                                                            |

:::
EN24/EN24X/EN3生成部
:::

## ダイナミック点灯信号生成部（Config-2）

## ＢＣＤ-７セグデコーダ部（Config-2）

BCDカウンタの出力をもとに7セグLEDを光らせるための信号を作ります。各セグメントの位置と名前は[@fig:segments]の通りです。

[セグメント](data/segments.txt){.aafigure #fig:segments}

[真理値表](data/truth-table.csv){.table alignment=cccccccccccc
width="[0.05,0.1,0.075,0.075,0.1,0.05,0.05,0.05,0.05,0.05,0.05,0.05]"}

### セグメントごとの論理式

## まとめ

# あとがき {-}

- `{\large`{=latex} **緑のLEDを買い占めたのは誰ですか？めっちゃ困るんですけど！色変更せざるを得なかったんですけど！** `}`{=latex}
- 原稿PDFはこのQRコードからたどってください ![](images/QRcode.png){#fig:manuscript width=20mm}
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
