# Yosys manual 日本語版 <!-- omit in toc -->

[Yosys manual](https://yosyshq.readthedocs.io/projects/yosys/en/latest/)

# 概要 <!-- omit in toc -->

今日のデジタル設計のほとんどは、HDL コード (主に Verilog または VHDL) で、HDL 合成ツールの助けを借りて行われています。

粗粒セル ライブラリの合成や新しい合成アルゴリズムのテストなどの特殊なケースでは、カスタム HDL 合成ツールを作成したり、既存のツールに新しい機能を追加したりする必要がある場合があります。このような場合、カスタム ツールのベースとして使用できる フリー アンド オープン ソース (FOSS) 合成ツールが利用できると便利です。

このようなツールがなかったため、Yosys Open SYnthesis Suite (Yosys) が開発されました。このドキュメントでは、このツールの設計と実装について説明します。現時点では、Yosys の主な焦点はデジタル合成の高レベルの側面にあります。Yosys は、既存の FOSS 論理合成ツール ABC を使用して、高度なゲートレベルの最適化を実行します。

実際の設計に基づいた Yosys の評価が含まれています。Yosys をそのまま使用してそのような設計を合成できることが示されています。このテストで Yosys によって生成された結果は、形式的検証を使用して正常に検証されており、商用合成ツールによって生成された結果と品質が同等です。

この文書はもともとウィーン工科大学の学士論文として出版されたものです[Wol13]。

- [1. はじめに](#1-はじめに)
  - [1.1. Yosysの歴史](#11-yosysの歴史)
  - [1.2. 文書の構成](#12-文書の構成)
- [2. 基本原理](#2-基本原理)
  - [2.1. 抽象化のレベル](#21-抽象化のレベル)
    - [2.1.1. システムレベル](#211-システムレベル)
    - [2.1.2. 高レベル](#212-高レベル)
    - [2.1.3. 動作レベル](#213-動作レベル)
    - [2.1.4. レジスタ転送レベル (RTL)](#214-レジスタ転送レベル-rtl)
    - [2.1.5. 論理ゲートレベル](#215-論理ゲートレベル)
    - [2.1.6. 物理ゲートレベル](#216-物理ゲートレベル)
    - [2.1.7. Yosys](#217-yosys)
  - [2.2. 合成可能なVerilogの特徴](#22-合成可能なverilogの特徴)
    - [Structural Verilog](#structural-verilog)
    - [Verilog の式](#verilog-の式)
    - [動作モデリング](#動作モデリング)
    - [関数とタスク](#関数とタスク)
    - [条件文、ループ、generate文](#条件文ループgenerate文)
    - [配列とメモリ](#配列とメモリ)
  - [2.3. デジタル回路合成における課題](#23-デジタル回路合成における課題)
    - [規格への準拠](#規格への準拠)
    - [最適化](#最適化)
    - [テクノロジーマッピング](#テクノロジーマッピング)
  - [2.4. スクリプトベースの合成フロー](#24-スクリプトベースの合成フロー)
  - [2.5. コンパイラ設計の方法](#25-コンパイラ設計の方法)
- [3. アプローチ](#3-アプローチ)
  - [3.1. データおよび制御フロー](#31-データおよび制御フロー)
  - [3.2. Yosys の内部フォーマット](#32-yosys-の内部フォーマット)
  - [3.3. 典型的な使用例](#33-典型的な使用例)
- [4. 実装の概要](#4-実装の概要)
  - [4.1. 簡素化されたデータフロー](#41-簡素化されたデータフロー)
  - [4.2. RTL 中間言語 (RTLIL)](#42-rtl-中間言語-rtlil)
  - [4.3. コマンドインターフェイスと合成スクリプト](#43-コマンドインターフェイスと合成スクリプト)
  - [4.4. ソースツリーとビルドシステム](#44-ソースツリーとビルドシステム)
- [5. 内部セルライブラリ](#5-内部セルライブラリ)
  - [5.1. RTLセル](#51-rtlセル)
  - [5.2. ゲート](#52-ゲート)
- [6. Yosys 拡張機能のプログラミング](#6-yosys-拡張機能のプログラミング)
  - [6.1. ガイドライン](#61-ガイドライン)
  - [6.2. 「stubsnets」サンプルモジュール](#62-stubsnetsサンプルモジュール)
- [7. Verilog および AST フロントエンド](#7-verilog-および-ast-フロントエンド)
  - [7.1. Verilog から AST への変換](#71-verilog-から-ast-への変換)
  - [7.2. AST から RTLIL への変換](#72-ast-から-rtlil-への変換)
  - [7.3. always の合成](#73-always-の合成)
  - [7.4. array の合成](#74-array-の合成)
  - [7.5. parameter の合成](#75-parameter-の合成)
- [8. 最適化](#8-最適化)
  - [8.1. 単純な最適化](#81-単純な最適化)
  - [8.2. FSM の抽出とエンコード](#82-fsm-の抽出とエンコード)
  - [8.3. ロジックの最適化](#83-ロジックの最適化)
- [9. テクノロジーマッピング](#9-テクノロジーマッピング)
  - [9.1. セルの置換](#91-セルの置換)
  - [9.2. 部分回路の置換](#92-部分回路の置換)
  - [9.3. ゲートレベルのテクノロジーマッピング](#93-ゲートレベルのテクノロジーマッピング)

# 1. はじめに

## 1.1. Yosysの歴史

## 1.2. 文書の構成

# 2. 基本原理

この章では、デジタル回路合成の基本原理について簡単に説明します。

## 2.1. 抽象化のレベル

デジタル回路は、さまざまな抽象度で表現できます。設計プロセスの上流において、回路は高い抽象度で表現されます。実装とは、より低い抽象度で機能的に同等の表現をすることであるといえます。これがソフトウェアを使用して自動的に行われる場合、合成と言います。

したがって、合成とは、回路の高レベル表現を、機能的に等価な回路の低レベル表現に自動的に変換することです。図 2.1 は、さまざまなレベルの抽象化と、それらがさまざまな種類の合成にどのように関係するかを示しています。

![](img/basics_abstractions.svg)

図 2.1さまざまなレベルの抽象化と合成。

回路の下位レベル表現を取得する方法 (合成または手動設計) に関係なく、下位レベル表現は通常、下位レベル表現と上位レベル表現のシミュレーション結果を比較することによって検証されます[1]。したがって、合成が使用されない場合でも、設計の検証を可能にするために、すべてのレベルで回路のシミュレーション可能な表現が存在する必要があります。

注: 「高レベル」などの用語の正確な意味は、もちろん時間が経っても固定されません。たとえば、HDL「ABEL」は 1985 年に「プログラマブル ロジック デバイス用の高水準設計言語」[LHBB85]として初めて導入されましたが、今日では「高水準言語」とみなされません。

### 2.1.1. システムレベル

システムレベルの抽象化では、CPU やコンピューティングコアなどの最大の構成要素のみが考慮されます。このレベルでは、回路は通常、C/C++ や Matlab などの従来のプログラミング言語を使用して記述されます。時として、SystemC など、システムレベルのシミュレーションを目的とした特別なソフトウェアライブラリが使用される場合があります。

通常、回路のシステムレベル表現を低レベル表現に自動的に変換する合成ツールは使用されません。ただし、システムレベルのブロックを接続するために使用できるシステムレベルの設計ツールが存在します。

IEEE 1685-2009 は、システム レベルの設計を表すために使用できる IP-XACT ファイル形式と、そのようなシステム レベルの設計で使用できるビルディングブロックを定義します。[ A+10 ]

### 2.1.2. 高レベル

システムの高レベルの抽象化 (アルゴリズムレベルと呼ばれることもあります) も、従来のプログラミング言語を使用して表現されることがよくありますが、機能が制限されています。たとえば、C 言語で高レベルの抽象化で設計を表現する場合、ポインタはメモリ インターフェイスなどのハードウェアに見られる概念を模倣するためにのみ使用できます。完全な動的メモリ管理機能は、デジタル回路に対応する概念がないため許可されません。

高レベル コード (通常は追加のメタデータを含む C/C++/SystemC コードの形式) を動作 HDL コード (通常は Verilog または VHDL コードの形式) に合成するツールが存在します。高位合成用の多くの商用ツールの他に、高位合成用の FOSS ツールも多数あります。

### 2.1.3. 動作レベル

動作レベルでは、回路の記述にはVerilogやVHDLなどのハードウェア記述を目的とした言語が使用されますが、回路記述の少なくとも一部にはいわゆる動作モデリングが使用される。動作モデリングでは、命令型プログラミングを使用してデータ パスとレジスタを記述することを可能にする言語機能が必要です。これは、Verilog の always ブロックと VHDL の process ブロックです。

動作モデリングでは、コードフラグメントが機密性リストとともに提供されます。信号と条件のリスト。シミュレーションでは、感度リスト内の信号の値が変更されるか、感度リスト内の条件がトリガーされるたびに、コード フラグメントが実行されます。合成ツールは、この表現を適切なデータパスに転送し、その後に適切なタイプのレジスタを転送できなければなりません。

たとえば、次の Verilog コードを考えてみましょう。

```
always @(posedge clk)
    y <= a + b;
```

シミュレーションでは、`clk` 信号のポジティブエッジが検出されるたびに `y <= a + b` が実行されます。ただし、合成結果には常に `a+b` を計算する加算器が含まれ、その後に加算器出力を D 入力に、信号を Q 出力に持つ D型フリップフロップ `y` が接続されます。

通常、動作モデリングで使用される命令型コードには、ループを完全に展開できる限り、条件文 (Verilog の `if` および `case`) とループを含めることができます。

興味深いことに、Verilog または VHDL の動作合成を実行できる FOSS ツールは Yosys 以外にないようです。

### 2.1.4. レジスタ転送レベル (RTL) 

レジスタ転送レベルでは、回路は組み合わせデータパスとレジスタ (通常は D型フリップフロップ) によって表されます。次の Verilog コードは、前の Verilog の例と同等ですが、RTL 表現になっています。

```
assign tmp = a + b;       // combinatorial data path

always @(posedge clk)     // register
    y <= tmp;
```

RTL 表現のデザインは通常、Verilog や VHDL などの HDL を使用して保存されます。ただし、使用される機能は非常に限られており、使用されるレジスタタイプとデータパス ロジックの無条件割り当てをモデル化する最小限の Always ブロック (Verilog) またはプロセスブロック (VHDL) のみです。このレベルで HDL を使用すると、RTL 表現でデザインをシミュレーションするために追加のツールが必要ないため、シミュレーションが簡素化されます。

多くの最適化と分析は、RTL レベルで最適に実行できます。例には、FSM の検出と最適化、メモリまたはその他のより大きな構成要素の識別、共有可能なリソースの識別などが含まれます。

RTL は、回路が回路要素 (レジスタおよび組合回路) と信号のグラフとして表現される最初の抽象化レベルであることに注意してください。このようなグラフをセルと接続のリストとしてエンコードすると、ネットリストと呼ばれます。

ネットリスト内の各回路ノード要素を等価なゲートレベルの回路に置き換えるだけで済むため、RTL 合成は簡単です。ただし、通常、RTL 合成という用語は、RTL ネットリストをゲートレベルのネットリストに合成することだけを指すのではなく、上に挙げた例のように、RTL 表現内で多くの高度な最適化を実行することも指します。

RTL 合成ステップのドメイン内で個別のタスクを実行できる FOSS ツールが多数存在します。しかし、広範囲の RTL 合成操作をカバーする FOSS ツールはないようです

###  2.1.5. 論理ゲートレベル

論理ゲート レベルでは、デザインは、基本的な論理ゲート (AND、OR、NOT、XOR など) やレジスタ (通常は D タイプ) などの少数のシングル ビット セルのセルのみを使用するネットリストによって表されます。ビーチサンダル）。

電子設計交換フォーマット (EDIF) など、このレベルで使用できるネットリスト フォーマットが多数存在しますが、シミュレーションを容易にするために、HDL ネットリストが使用されることがよくあります。後者は、セルのインスタンス化と接続に最も基本的な言語構造のみを使用する HDL ファイル (Verilog または VHDL) です。

論理合成には 2 つの課題があります。1 つはゲート レベルのネットリスト内で最適化の機会を見つけること、2 つ目は論理ゲート ネットリストを物理的に利用可能なゲート タイプの同等のネットリストに最適 (または少なくとも良好な) マッピングすることです。

論理合成への最も単純なアプローチは 2 レベル論理合成です。この場合、論理関数は、たとえばカルノー図を使用して積和表現に変換されます。これは単純なアプローチですが、最悪の場合の労力が指数関数的に増加し、AND/NAND、OR/NOR、および NOT ゲート以外の物理ゲートを効率的に使用できません。

したがって、最新の論理合成ツールは、より複雑なマルチレベル論理合成アルゴリズム[ BHSV90 ]を利用しています。これらのアルゴリズムのほとんどは、論理関数を Binary-Decision-Diagram (BDD) または And-Inverter-Graph (AIG) に変換し、その表現に基づいて動作します。前者には、独自の正規化された形式があるという利点があります。後者の方が最悪の場合のパフォーマンスがはるかに優れているため、大規模な論理関数の合成に適しています。

マルチレベル論理合成用の優れた FOSS ツールが存在します。

Yosys には基本的な論理合成機能が含まれていますが、論理合成ステップに ABC を使用することもできます。ABCの使用をお勧めします。

### 2.1.6. 物理ゲートレベル

物理ゲートレベルでは、ターゲットアーキテクチャで物理的に利用可能なゲートのみが使用されます。
場合によっては、これは D型レジスタだけでなく、NAND、NOR、NOT ゲートのみである場合もあります。
他の場合には、論理ゲートレベルで使用されるセルよりも複雑なセル (完全な半加算器など) が含まれる場合があります。
FPGA ベースのデザインの場合、物理ゲートレベルの表現は、オプショナルな出力レジスタを備えた LUT のネットリストです。
これは、これらがFPGAロジックセルの基本構成要素であるためです。

合成ツールチェーンの場合、この抽象化は通常、最低レベルになります。
ASICベースの設計の場合、セルライブラリには、物理​​セルが個々のスイッチ (トランジスタ) にどのようにマッピングされるかに関する詳細情報が含まれる場合があります。

### 2.1.7. Yosys

Yosys は Verilog HDL 合成ツールです。
これは、動作デザイン記述を入力として受け取り、デザインの RTL、論理ゲート、または物理ゲート レベルの記述を出力として生成することを意味します。
Yosys の主な強みは動作と RTL 合成です。
Yosys 内には広範囲のコマンド (合成パス) が存在し、動作合成、rtl 合成、および論理合成の領域内で幅広い合成タスクを実行するために使用できます。
Yosys は拡張可能に設計されているため、特殊なタスク用のカスタム合成ツールを実装するための優れた基盤となります。

## 2.2. 合成可能なVerilogの特徴

合成可能な Verilog [ A+06 ]のサブセットは、別の IEEE 標準文書である IEEE 標準 1364.1-2002 [ A+02 ]で指定されています。
この標準は、特定の言語構造が合成の範囲内でどのように解釈されるかについても説明します。

このセクションでは、合成可能な Verilog の最も重要な機能の概要を、複雑さの順に構造化して説明します。

### Structural Verilog

Structural Verilog (Verilog ネットリストとも呼ばれる) は、Verilog 構文のネットリストです。
この場合、次の言語構造のみが使用されます。

- 定数値
- ワイヤーとポートの宣言
- 信号の他の信号への静的な割り当て
- セルのインスタンス化

多くのツール (特に合成チェーンのバックエンド) は、入力としてStructual Verilog のみをサポートします。
ABC はそのようなツールの一例です。
残念ながら、Structural Verilog が実際に何であるかを指定する標準は存在しないため、
属性やマルチビット信号などの機能に関して、どのような構文構造が構造 Verilog でサポートされるかについて混乱が生じています。

### Verilog の式

Verilog が定数値、信号名、 演算子（`+` `-` `*` などの算術演算、`&` `|` `^` などのブール演算、比較演算、単項演算子など）を用いた式を受け付ける場合、それらを使用でき*ます。

合成中に、これらの演算子は、それぞれの関数を実装するセルに置き換えられます。

Verilog を処理できると主張する多くの FOSS ツールは、実際には基本的な Structual Verilog と単純な式のみをサポートしています。Yosys を使用すると、フル機能の合成可能な Verilog をこの単純なサブセットに変換できるため、そのようなアプリケーションをより豊富な Verilog 機能セットで使用できるようになります。

### 動作モデリング

Verilog always ステートメントを利用するコードは、動作モデリングを使用しています。
動作モデリングでは、回路は特定のイベント、つまり信号の変化、立ち上がりエッジ、または立ち下がりエッジで実行される命令型プログラム コードによって記述されます。
これはシミュレーション中は非常に柔軟な構造ですが、次のいずれかがモデル化されている場合にのみ合成可能です。

- 非同期またはラッチされたロジック
- 
この場合、sensitivity リストには、always ブロック内で使用されるすべての式が含まれている必要があります。
`@*`はこのような場合に使用できます。
この種の例には次のようなものがあります。

```
// asynchronous
always @* begin
    if (add_mode)
        y <= a + b;
    else
        y <= a - b;
end

// latched
always @* begin
    if (!hold)
        y <= a + b;
end
```

ラッチされたロジックは、多くの場合、悪いスタイルであると考えられており、多くの場合、単にずさんな HDL 設計の結果であることに注意してください。
したがって、多くの合成ツールは、ラッチされたロジックが生成されるたびに警告を生成します。

- 同期ロジック (オプションの同期リセット付き)

これは、出力に DFF を備えたロジックです。この場合、sensitivity　リストにはそれぞれのクロックエッジのみが含まれている必要があります。

```
// counter with synchronous reset
always @(posedge clk) begin
    if (reset)
        y <= 0;
    else
        y <= y + 1;
end
```

- 非同期リセットを備えた同期ロジック

これは、出力に非同期リセットを備えた D タイプ フリップフロップを備えたロジックです。
この場合、sensitivity リストにはそれぞれのクロックエッジとリセットエッジのみが含まれている必要があります。
リセット分岐に割り当てられる値は定数である必要があります。

```
// counter with asynchronous reset
always @(posedge clk, posedge reset) begin
    if (reset)
        y <= 0;
    else
        y <= y + 1;
end
```

Yosys を含む多くの合成ツールは、always ステートメントを使用してモデル化できるフリップフロップの幅広いサブセットをサポートしています。
ただし、Verilog 合成の標準でカバーされるのは上記に挙げたものだけであり、新しいデザインを作成する場合はこれらのケースに限定する必要があります。

動作モデリングでは、ブロッキング代入 `=` と非ブロッキング代入 `<=` を使用できます。
ブロッキング代入と非ブロッキング代入の概念は、Verilog [CI00]で最も誤解されている構造の 1 つです。

ブロッキング代入は、命令型プログラミング言語の代入とまったく同じように動作しますが、
非ブロッキング代入では、代入の右側はすぐに評価されますが、左側のレジスタの実際の更新はタイムステップの終わりまで遅延されます。
たとえば、次のVerilog コードは2 つのレジスタの値を交換します。

```
a <= b;
b <= a;
```

### 関数とタスク

Verilog は、複数の場所で使用されるステートメントをバンドルする関数とタスクをサポートしています (命令型プログラミングのプロシージャと同様)。
どちらの構成も、関数/タスク呼び出しを関数またはタスクの本体に置き換えることで簡単に実装できます。

### 条件文、ループ、generate文

Verilog は、`if-else`-statements と `always` 文内の `for` ループをサポートします。

また、`generate`文中で両方の機能もサポートされます。
if-elseこれを使用して、モジュール パラメータに基づいてモジュールの一部を選択的に有効または無効にしたり ( )、または同様のサブ回路のセットを生成したり ( for)することができます。

if-elsealways ブロック内の -statement は動作モデリングの一部ですが、
他の 3 つのケースは (少なくとも合成ツールの場合) 組み込みマクロ プロセッサの一部です。
したがって、合成ツールはすべてのループを完全に展開し、const-folding を使用して -statementsif-else内のすべての -statementの条件を評価できる必要があります。generate

### 配列とメモリ

Verilog は配列をサポートしています。
これは一般に、合成可能な言語の機能です。
ほとんどの場合、配列はアドレス指定可能なメモリを生成することで合成できます。
ただし、複雑なアクセス パターンや非同期アクセス パターンが使用される場合、配列をメモリとしてモデル化することはできません。
このような場合、配列はワードごとに個別の信号を使用してモデル化する必要があり、配列へのすべてのアクセスは大規模なマルチプレクサを使用して実装する必要があります。

場合によっては、メモリを使用して配列をモデル化することも可能ですが、それは望ましくありません。次の遅延回路を考えてみましょう。

```
module (clk, in_data, out_data);

parameter BITS = 8;
parameter STAGES = 4;

input clk;
input [BITS-1:0] in_data;
output [BITS-1:0] out_data;
reg [BITS-1:0] ffs [STAGES-1:0];

integer i;
always @(posedge clk) begin
    ffs[0] <= in_data;
    for (i = 1; i < STAGES; i = i+1)
        ffs[i] <= ffs[i-1];
end

assign out_data = ffs[STAGES-1];

endmodule
```

これは、STAGES 入力ポートと出力ポートを備えたアドレス指定可能なメモリを使用して実装できます。
より良い実装は、フリップフロップの単純なチェーン (いわゆるシフト レジスタ) を使用することです。
このより優れた実装は、最初にメモリベースの実装を作成し、次にすべてのポートの静的アドレス信号に基づいて最適化するか、
言語フロントエンドでそのような状況を直接識別し、すべてのメモリアクセスを正しい信号への直接アクセスに変換することによって実現できます。 

## 2.3. デジタル回路合成における課題

このセクションでは、デジタル回路合成における最も重要な課題を要約します。
ツールは、これらのトピックにどれだけうまく対処できるかによって特徴づけられます。

### 規格への準拠

最も重要な課題は、当該の HDL 標準規格 (Verilog の場合、IEEE 標準 1364.1-2002 および 1364-2005) への準拠です。
これは次の 2 つの項目に分類できます。

- 標準の実装の完全性
- 標準の実装の正確さ

完全性は、既存の HDL コードとの互換性を保証するために最も重要です。
デザインが検証され、テストされると、HDL 設計者は、たとえ新しい合成ツールで欠落している機能を回避するためのわずかな変更が数件程度であっても、デザインの変更には非常に消極的になります。

正確さは非常に重要です。
一部の領域では、これは明らかです (基本的な動作モデルの正しい合成など)。
しかし、これは、HDL コードが別の合成ツールを対象としていない場合でも、符号付き式を処理するための正確なルールなど、規格の細かい詳細に関係する領域にとっても重要です。
これは、(コンパイラによってのみ処理されるソフトウェア ソース コードとは異なり) ほとんどのデザイン フローでは、
HDL コードが合成ツールによって処理されるだけでなく、1 つ以上のシミュレータによって処理され、
場合によっては正式な検証ツールによっても処理されるためです。
この検証プロセスでは、これらすべてのツールが HDL コードに対して同じ解釈を使用することが重要です。

### 最適化

一般に、合成ツールがデザインをどの程度最適化するかを一次元で説明することは困難です。
まず第一に、すべての最適化がすべてのデザインやすべての合成タスクに適用できるわけではないからです。
最適化の中には、粗粒度レベル (加算器や乗算器などの複雑なセルを使用) で (最適に) 機能するものと、
粒度の細かいレベル (シングル ビット ゲート) で (最適に) 機能するものがあります。
サイズをターゲットとする最適化もあれば、速度をターゲットにする最適化もあります。
大規模なデザインでうまく機能するものもありますが、拡張性が低く、小さなデザインにしか適用できないものもあります。

優れたツールは、さまざまな抽象化レベルで幅広い最適化を適用でき、
どの最適化を実行する (またはスキップする) か、最適化の目標を設計者が制御できます。

### テクノロジーマッピング

テクノロジー マッピングは、デザインをターゲット アーキテクチャで使用可能なセルのネットリストに変換するプロセスです。
ASIC フローでは、これは工場によって提供されるプロセス固有のセル ライブラリである可能性があります。
FPGA フローでは、これは LUT セルや専用乗算器などの特殊な機能ユニットである場合があります。
Coarse-grain のフローでは、これはさらに複雑な特殊機能ユニットになる可能性があります。

オープンでベンダーに依存しないツールは、さまざまなタイプのターゲットアーキテクチャを広範囲にサポートする場合に特に興味深いものになります。

## 2.4. スクリプトベースの合成フロー

デジタル設計は通常、目的の機能の高レベルまたはシステムレベルのシミュレーションを実装することによって開始されます。
次に、この記述は、合成可能な下位レベルの記述 (通常は動作レベル) に手動で変換 (または再実装) され、
両方をシミュレーションしてシミュレーション結果を比較することによって、2 つの表現の等価性が検証されます。

次に、一連のツールを使用して合成可能な記述が低レベルの表現に変換され、その結果がシミュレーションを使用して再度検証されます。このプロセスを図 2.2に示します。


図 2.2 一般的な設計フロー。緑色のボックスは手動で作成されたモデルを表します。オレンジ色のボックスは、合成ツールによって生成されたモデルを表します。

この例では、システム レベル モデルと動作モデルは両方とも手動で作成された設計ファイルです。
システム レベル モデルと動作モデルの同等性が検証された後、合成ツールを使用してデザインの下位レベルの表現を生成できます。
最後に、RTL モデルとゲートレベル モデルが検証され、設計プロセスは終了します。

ただし、実際の設計作業では、この設計プロセスを複数回繰り返すことになります。
この理由としては、設計要件の変更が遅かったこと、または低抽象化モデルの解析 (ゲート レベルのタイミング解析など) により、
設計要件 (最大値など) を満たすために設計変更が必要であることが判明したことが考えられます。可能なクロック速度）。

動作モデルまたはシステム レベル モデルが変更されるたびに、
シミュレーションを再実行して結果を比較することによって、
それらの等価性を再検証する必要があります。
動作モデルが変更されるたびに、合成を再実行し、合成結果を再検証する必要があります。

再現性を保証するには、固定の設定セットを使用して設計プロジェクトのすべての自動ステップを簡単に再実行できることが重要です。
このため、通常、合成フローで使用されるすべてのプログラムはスクリプトを使用して制御できます。
これは、すべての機能がテキスト コマンド経由で利用できることを意味します。
このようなツールが GUI を提供する場合、これはコマンド ライン インターフェイスを補完するものであり、
コマンド ライン インターフェイスの代わりとなるものではありません。

通常、UNIX/Linux 環境の合成フローは、必要なすべてのツール (この例では合成およびシミュレーション/検証) を正しい順序で呼び出すシェル スクリプトによって制御されます。
これらの各ツールは、それぞれのツールのコマンドを含むスクリプト ファイルを使用して呼び出されます。
ツールに必要なすべての設定はこれらのスクリプト ファイルによって提供されるため、手動による操作は必要ありません。
これらのスクリプト ファイルは設計ソースとみなされ、システム レベルのソース コードや動作モデルと同様にバージョン管理下に保管される必要があります。

## 2.5. コンパイラ設計の方法

合成ツールの一部には、コンパイラ設計から伝統的に知られている問題領域が含まれます。
このセクションでは、これらのドメインのいくつかについて説明します。

【略】

# 3. アプローチ

Yosys は、ターゲット アーキテクチャ ネットリストに対して Verilog HDL コードを合成 (動作) するためのツールです。
Yosys は幅広いアプリケーション ドメインを対象としているため、新しいタスクに柔軟に簡単に適応できる必要があります。
この章では、このツールを実装するために採用される一般的なアプローチについて説明します。

## 3.1. データおよび制御フロー

一般的な合成ツールのデータおよび制御フローは、一般的なコンパイラのデータおよび制御フローと非常に似ています。
さまざまなサブシステムが所定の順序で呼び出され、
それぞれが最後のサブシステムによって生成されたデータを消費して、
データを生成します。
次のサブシステム用です (図 3.1を参照)。



## 3.2. Yosys の内部フォーマット

## 3.3. 典型的な使用例

# 4. 実装の概要

## 4.1. 簡素化されたデータフロー

## 4.2. RTL 中間言語 (RTLIL)

## 4.3. コマンドインターフェイスと合成スクリプト

## 4.4. ソースツリーとビルドシステム

# 5. 内部セルライブラリ

## 5.1. RTLセル

## 5.2. ゲート

# 6. Yosys 拡張機能のプログラミング

## 6.1. ガイドライン

## 6.2. 「stubsnets」サンプルモジュール

# 7. Verilog および AST フロントエンド

## 7.1. Verilog から AST への変換

## 7.2. AST から RTLIL への変換

## 7.3. always の合成

## 7.4. array の合成

## 7.5. parameter の合成

# 8. 最適化

## 8.1. 単純な最適化

## 8.2. FSM の抽出とエンコード

## 8.3. ロジックの最適化

# 9. テクノロジーマッピング

## 9.1. セルの置換

## 9.2. 部分回路の置換

## 9.3. ゲートレベルのテクノロジーマッピング
