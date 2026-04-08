## UARTモジュールの実装
### UARTとは
**UARTは1bitずつ順番にデータを送信する通信方式**  

### UARTの仕組み
```
1文字（8bit）を送る場合の波形：
アイドル  スタート  D0 D1 D2 D3 D4 D5 D6 D7  ストップ
  1    ─┐   0   ┌─b0─b1─b2─b3─b4─b5─b6─b7─┐  1
        └───────┘                           └────

部分              値      意味
アイドル           1      送信待ち状態
スタートビット      0      送信開始の合図
データビット       8bit    実際のデータ
ストップビット     1       送信終了の合図
```

**UARTはLSB（低いbit）から順番に送ります。**
```
例えば tx_buf = 8'b01000001（'A'の文字コード）の場合：
bit_index:  0  1  2  3  4  5  6  7
tx_buf[i]:  1  0  0  0  0  0  1  0
                                 ↑
                             8bit送ったらSTOPへ
```

## 送信モジュール
  
### 状態推移の流れ
```
IDLE ──tx_startが1になる──→ START
       スタートビット送信
START ──234クロック経過──→ DATA
       8bit分データ送信
DATA ──8bit送り終わる──→ STOP
       ストップビット送信
STOP ──234クロック経過──→ IDLE
```
  
```
module uart_tx (
    input  wire       clk,      // 27MHz
    input  wire       tx_start, // 1にすると送信開始
    input  wire [7:0] tx_data,  // 送信したい8bitデータ
    output reg        tx,       // UARTのTXピン
    output reg        tx_busy   // 送信中は1になる
);

// 115200bps @ 27MHz
parameter CLKS_PER_BIT = 234;

reg [7:0]  clk_count;  // ボーレートカウンタ
reg [3:0]  bit_index;  // 今何bit目を送っているか
reg [7:0]  tx_buf;     // 送信バッファ

// 状態定義
parameter IDLE  = 2'd0;
parameter START = 2'd1;
parameter DATA  = 2'd2;
parameter STOP  = 2'd3;

reg [1:0] state;

always @(posedge clk) begin
    case (state)

        IDLE: begin
            tx      <= 1;  // アイドルは1
            tx_busy <= 0;
            clk_count <= 0;
            bit_index <= 0;

            if (tx_start) begin
                tx_buf <= tx_data;
                state  <= START;
                tx_busy <= 1;
            end
        end

        START: begin
            tx <= 0;  // スタートビット
            if (clk_count == CLKS_PER_BIT - 1) begin
                clk_count <= 0;
                state     <= DATA;
            end else begin
                clk_count <= clk_count + 1;
            end
        end

        DATA: begin
            tx <= tx_buf[bit_index];  // LSBから順に送信
            if (clk_count == CLKS_PER_BIT - 1) begin
                clk_count <= 0;
                if (bit_index == 7) begin
                    bit_index <= 0;
                    state     <= STOP;
                end else begin
                    bit_index <= bit_index + 1;
                end
            end else begin
                clk_count <= clk_count + 1;
            end
        end

        STOP: begin
            tx <= 1;  // ストップビット
            if (clk_count == CLKS_PER_BIT - 1) begin
                clk_count <= 0;
                state     <= IDLE;
                tx_busy   <= 0;
            end else begin
                clk_count <= clk_count + 1;
            end
        end

    endcase
end

endmodule
```

### 送信モジュールの疑問
  
#### clk_count == CLKS_PER_BIT - 1はなぜ-1にするのかわかりません
カウンタは0からスタートするから  
0から数えると 0,1,2,3 で4クロック分になります。  
もし == CLKS_PER_BIT にすると 0,1,2,3,4 で5クロック分になってしまいます。  
```
CLKS_PER_BIT = 4 の場合の例:

clk_count:  0 → 1 → 2 → 3 → 次の状態へ
            ↑              ↑
          最初           ここで検出

// 正しい（234クロック待つ）
if (clk_count == CLKS_PER_BIT - 1)  // 0〜233 = 234個

// 間違い（235クロック待つ）
if (clk_count == CLKS_PER_BIT)      // 0〜234 = 235個
```
  
#### end else begin clk_count <= clk_count + 1;

```
if (clk_count == CLKS_PER_BIT - 1) begin
    // 234クロック経過した → 次の状態へ進む
    clk_count <= 0;
    state     <= DATA;
end else begin
    // まだ経過していない → カウントを増やす
    clk_count <= clk_count + 1;
end

毎クロック:
  clk_countが233になった？
    YES → リセットして次の状態へ
    NO  → clk_countを+1して待ち続ける
```

#### reg [1:0] state のbit数
  
2bitです。[1:0] の意味は：  
```
[上位bit : 下位bit]
[  1    :   0   ] → 2bit

表記       bit数        扱える範囲
[1:0]      2bit           0〜3
[3:0]      4bit           0〜15（0〜F）
[7:0]      8bit           0〜255
[15:0]     16bit          0〜65535

今回 state は IDLE/START/DATA/STOP の4種類（0〜3）
なので [1:0] の2bitで足ります。
```







