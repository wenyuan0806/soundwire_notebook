Physical Interface (PHY)
=======

PHY
-------

SoundWire 接口使用兩根信號線來做設備之間的連接：
- Clock : 從 Master 發送到全部 Slave 的 clock
- Data : 可以被所有設備驅動的 data 信號，並且可以配置沒有人驅動時的空載值

而訊號線電壓通常訂為 `1.2V` or `1.8V`。

Figure 8 為 SoundWire Master/Slave Interface，其內部的 digital interface 負責傳輸 payload, command 等；而 PHY 負責與其他 SoundWire 設備的物理連接。

![image](https://github.com/wenyuan0806/soundwire_notebook/assets/133325842/defa4bd4-657a-4bed-9fff-b7be24bc24f4)

Data Signaling and Bitslots
-------

#### Bitslots ####

SoundWire Frame 內的每一個 bit 就叫做 bitslot，frame 內的第一個 bitslot 就叫 `bitslot[0]`、第二個叫 `bitslot[1]`，後面以此類推。而一個 bitslot 內的值可能同時由多個 SoundWire 設備所提供，而提供了該 bitslot 值的設備，我們就將其稱為 the **owner** of the Bitslot。

#### DDR (Double Data Rate) ####

SoundWire 採用 DDR 進行數據傳輸，也就是可以在一個 clock cycle 內塞兩個 bits 的數據。一個 bit 在 clock rising edge 被驅動，並且在 clock falling edge 被取樣；下一個 bit 則在 clock falling edge 被驅動，並且在 clock rising edge 被取樣。換句話說，SoundWire 可以同時在 clock rising/falling edge 被驅動或取樣，因此數據傳輸的頻率是 clock 的兩倍。

#### Modified NRZI Data Encoding ####

Figure 11 是一般傳統 NRZI 的編碼方式，當數據為 1 時則改變電位 (High-to-Low or Low-to-High)，當數據為 0 時則 keep 電位，可參考下圖：

![image](https://github.com/wenyuan0806/soundwire_notebook/assets/133325842/4cb0e3a7-db6d-4f40-a03f-087f917046d0)

Figure 12 則是 SoundWire 採用的 Modified NRZI，跟傳統 NRZI 差異為，當數據為 0 時是透過一個高阻抗 bus-keeper 來 keep 電位，因此這時候 data line 電位是可以被別人拉動。

![image](https://github.com/wenyuan0806/soundwire_notebook/assets/133325842/b37163e4-4857-49be-bdd7-3761f2d16063)

Modified NRZI 有以下兩個優點：
1. 如果當前 bitslot 沒有 owner，或是因為某些原因導致 data 沒有被驅動，這時 Master 的高阻抗 bus-keeper 也會讓 data 保持在 Logic 0 狀態，這樣有利於檢查出斷線的設備或錯誤的配置。
2. 允許同一個 bitslot 擁有多個 owner，而多個 owner 之間的關係是 wire-OR，因此輸出 1 的 owner 會比輸出 0 的 owner 擁有更高的優先權 (因為是 wire-OR，因此輸出 0 的 owner 會被 1 的 owner 蓋過去)

