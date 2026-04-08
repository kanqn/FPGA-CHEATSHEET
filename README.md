# FPGA-CHEATSHEET

## 各ファイルの説明

```
┌─────────────────────────────────┐
│  Verilog / VHDL                 │  ← 回路の「設計図」
│  (.v / .vhd)                    │
└─────────────────────────────────┘
┌─────────────────────────────────┐
│  Physical Constraints (.cst)    │  ← ピンの「配線図」
└─────────────────────────────────┘
┌─────────────────────────────────┐
│  Timing Constraints (.sdc)      │  ← 速度の「仕様書」
└─────────────────────────────────┘
         ↓ GowinIDEが処理
┌─────────────────────────────────┐
│  ビットストリーム (.fs)          │  ← FPGAに書き込む「バイナリ」
└─────────────────────────────────┘
```

### Verilog File (.v)
  
回路そのものを記述するファイル  
プログラミングでいう.pyや.jsに相当します。  
ただし実行される命令ではなくハードウェアの構造を表現します。  
```
// 「どんな回路を作るか」を書く
module led_blink (
    input  wire clk,
    output wire led
);
// ここに回路の動作を記述
endmodule
```

### VHDL File (.vhd)
  
VerilogとまったくおなじくFPGAの回路を記述するファイルです。  
VerilogとVHDLはどちらを使っても同じ回路が作れる別の言語です。  
現代ではVerilogが主流なのでVHDLは基本無視でOK  
```
-- Verilogと同じことをVHDLで書いた場合
entity led_blink is
    port (
        clk : in  std_logic;
        led : out std_logic
    );
end entity;
```
  
### Physical Constraints File (.cst)
  
「Verilogで書いた信号名」と「FPGAの実際の物理ピン番号」を対応付けるファイルです。  
これがないとFPGAはどのピンから信号を入出力すればいいかわかりません。  
```
# clkという信号名 → 52番ピンに接続
IO_LOC "clk" 52;

# ledという信号名 → 10番ピンに接続
IO_LOC "led" 10;
```
  
### Timing Constraints File (.sdc)
  
回路の動作速度の制約を書くファイルです。  
複雑な回路を高速動作させるときに必要になります。  
```
# clkは27MHzで動かす、という宣言
create_clock -name clk -period 37.037 [get_ports clk]
```
