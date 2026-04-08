## wire変数とreg変数
Verilogの変数には、wire（ワイヤ）とreg（レジスタ）の２つの型みたいなものがあります  
  
### wire変数
  
wire変数は、モジュール（部品）同士をつなぐ配線をあらわします。  
C言語の変数と違い、値を保持する入れ物ではなく**値が流れている線**のイメージです。  
たとえば、32bitのwire変数は以下のように宣言します。  

```
wire [31:0] hoge;
```
  
ハードウェア記述なので**何bitなのかを宣言しなければなりません。**  
wire変数は、常に値の更新が起こりうる変数です。  
  
### reg変数
  
reg変数は、C言語などの変数と同様、値を保持しておく入れ物です。  
たとえば、32bitのreg変数は以下のように宣言します。  

```
reg [31:0] hoge;
```
  
reg変数はwire変数とは違って、値が流れているわけではないので  
**値を代入するタイミングを指定する必要があります。**  
  
## assign文とalways文
ワイヤ変数への代入はassign文  
レジスタ変数への代入はalways文中でおこなう  
(※実際には、レジスタ変数はalways文以外での書き換えられます)  
  
### assign文
ワイヤ同士をつなげたい場合assign文を使用します。  
たとえば、32bitの符号付きワイヤaとbの和をワイヤcに流したい場合、以下のように書きます。  
```
wire signed [31:0]　a, b, c;
assign c = a + b;

または、
wire [31:0] a, b;
wire [31:0] c = a + b;
```

**Verilogでは、なにも宣言しないと符号なしの変数になる**  
  
### always文

reg変数はwire変数とは違って、**値が流れているわけではないので値を代入するタイミングを指定する必要があります。**  
このタイミングの指定をハードウェアではクロックで行います。  
→そのタイミングの指定方法がalways文です。  
**always文は、あるタイミングで起こしたい処理を書くことができます。**  
  
```
//クロックの立ち上がり時に行いたい処理は以下のブロックのなかに書きます（以下、クロックはCLKというreg変数で表します）。
//ブロックの指定にはC言語の{, }の代わりにbegin, endを使用します。

reg [31:0] a;
always @(posedge CLK) begin
    a <= 0; //CLKの立ち上がり時に、aに0を代入
end

```

#### posedgeとは
positive edge(立ち上がりエッジ)の略  
CLKの波形で見ると以下の部分  
```
CLK:  0 ─┐      ┌─────┐      ┌─────
          └─────┘     └─────┘
                ↑            ↑
           posedge     posedge
           ここで処理    ここで処理
```
**つまり CLKが0→1に変わった瞬間だけ、begin〜endの中が実行されます。**  
  
#### negedge
立下りエッジ・・・1 → 0 の瞬間  
```
CLK:  0 ─┐     ┌─────┐     ┌─────
          └────┘     └─────┘
               ↑     ↓     ↑     ↓
           posedge negedge posedge negedge
           立ち上がり 立ち下がり
```

**ほぼ全ての回路で posedge を使う**  
**negedge はUARTやSPIなど一部の通信プロトコルで稀に使う程度**  
  
#### <= とは
<= はノンブロッキング代入と呼ばれ、=（ブロッキング代入）と区別されます。  

```
<=  ノンブロッキング代入  always @(posedge CLK) の中  
=   ブロッキング代入  always @(*) や関数の中
```
  
**always @(posedge CLK) の中では必ず <= を使うとまず覚えておけばOK**
  
  
### ブロッキング代入とノンブロッキング代入
  
Verilogの代入には2種類の代入法があります。  
**違いとしては、同時に代入を行うのか、順番に行うのかということです。**  
なんで2種類必要かというと、ハードウェアなのでC言語みたいに 
**コードを上から読んでいくわけではない**です。  
なので、あるタイミングで、この代入とこの代入を同時に行いたい、という場合があります。  

#### ブロッキング代入
  
順番に行う代入です。=を使用します。  
```
reg [31:0] a, b, c;
c = a+ b;
```
  
#### ノンブロッキング代入
  
同時に行う代入  
**C言語では必要なスワップ用の一時変数が不要になっています。**  
たとえば、レジスタaとbの値を入れ替えたいときは以下のようになります。  
```
reg [31:0] a, b;
a <= b;
b <= a;
```
  
### moduleの作成
**モジュールとは、回路を構成している部品**  
**最初にinputとoutputで入力と出力を指定します。**  
基本的に入出力はワイヤで行います。なお、mem[1023:0]は配列を表します。  

メモリはレジスタとして値を保持していて、それをワイヤに流すという構図になっています。  
出力のdataはワイヤなのでassignを使用していることに注意してください。  
```
modeule MEM(addr, data) ;
	input [9:0] addr;
	output [31:0] data;
	
	reg [31:0] mem[1023:0]; //4 Kbyte memory
	assign data = mem[addr];
endmodule
```
  
### function文
function文はC言語の関数に相当します。  
**ただし、関数名と返り値の名前が一致していなくてはなりません。**  
```
wire [31:0] a,result;
function [31:0] calctest;
	input [31:0] inp;
	calctest = inp + 1;
endfunction
//呼び出すとき
assign result = calctest(a);
```
  
### if文とswitch文
**if文やswitch文はfunctionおよびalways文中でしか使用できません。**  

#### if文
```
function calcif;
	input [31:0] inp;
	if(inp == 1) begin
		calcif = 0;
	end else if (inp == 2) begin
		calcif = 1;
	end else begin
		calcif = 2;
	end
endfunction
```
  
#### switch文
```
function calccase;
	input [31:0] inp;
	case(inp)
		32'd1: calccase = 0;
		32'd2: calccase = 1;
		defaule: calccase = 2;
	endcase
endfunction
```

参考  
https://qiita.com/thtitech/items/8cc898dda7a10780f495  










