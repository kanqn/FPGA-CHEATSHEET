### 制約ファイル（CST）の書き方
  
CSTファイルには主に2種類の命令があります。

#### IO_LOC : ピン番号の指定

**信号名はVerilogのポート名と完全一致している必要があります。**  
```
IO_LOC "信号名" ピン番号;

例:
IO_LOC "clk" 52;   // Verilogの"clk"という信号を52番ピンに接続
IO_LOC "led" 10;   // Verilogの"led"という信号を10番ピンに接続
```

#### IO_PORT : ピンの電気的設定

```
IO_PORT "信号名" オプション=値;
```

##### よく使うオプション

```
オプション     値        意味
PULL_MODE     UP        プルアップ（未接続時に1になる）
PULL_MODE     DOWN      プルダウン（未接続時に0になる）
PULL_MODE     NONE      プルなし
DRIVE        8 16 24    出力電流強度(mA)
IO_TYPE      LVCMOS33   電圧レベル（3.3V）
```

### 配列信号のピン指定
Verilog側で配列を使っている場合：  
```
// Verilog側
output wire [5:0] led;
```

```
// CST側
IO_LOC "led[0]" 10;
IO_LOC "led[1]" 11;
IO_LOC "led[2]" 13;
```

### 複数オプションを書く場合
オプションはスペース区切りで複数指定できます。  
```
IO_LOC "clk" 52;
IO_PORT "clk" PULL_MODE=UP IO_TYPE=LVCMOS33;
```
  
## Tang Nano 9Kのよく使うピン番号

```
// クロック
IO_LOC "clk" 52;          // 27MHz オンボードクロック

// LED (6個)
IO_LOC "led[0]" 10;
IO_LOC "led[1]" 11;
IO_LOC "led[2]" 13;
IO_LOC "led[3]" 14;
IO_LOC "led[4]" 15;
IO_LOC "led[5]" 16;

// ボタン (2個)
IO_LOC "btn[0]" 3;
IO_LOC "btn[1]" 4;

// UART
IO_LOC "uart_tx" 17;
IO_LOC "uart_rx" 18;
```
  






