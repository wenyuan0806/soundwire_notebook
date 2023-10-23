Payload Transport
=======

本章節會說明 SoundWire Data Ports 是如何傳輸 payload data，一般的 payload stream 就是一個 Sample Data 的序列，例如 audio payload。而 Payload Stream 會由一個 Data Port 發出，然後可以被多個 Data Ports 接收。

之後後面還會提到 Bulk Payload Transport (BPT)，BPT 可以用於傳輸大量的 Register Configuration，會比用 SoundWire Control Word 來配置 Slave 還要有效率。

> BPT 一般都是由 Control Port 0 來發送。

一般的設備的 **Data Port 編號是從 1 ~ 14**，這個編號可用來辨識 Register 是想要控制哪個 Data Port。而**每個 Data Port 中的 Channel 編號則是從 1 ~ 8**，每一個 channel 會同時傳輸一個 payload stream。

你可能會好奇把多個 channel 塞在一個 data port 裡面的意義是什麼，它的目的其實就是為了讓這些 channel 可以用相同的傳輸參數 (包含 Sample Rate, Sample bit width 等) 來進行傳輸。

Sample Event, Sample Intervals and Sample Windows
-------

- `Sample Event`
    - 指的是開始取樣的時間點
    - Sample Event 是由 Data Port 產生的，因為 SoundWire Spec 並沒有定義 Sample Event 要什麼時後發生，只要能正常傳輸音訊就好
- `Sample Interval`
    - 就是兩個 Sample Event 之間傳輸的 bits 數量
    - 為了讓每一個 Frame 裡面都包含整數個 Sample Interval，所以通常 Frame Shape 的大小要是 Sample Interval 的整數倍 (但其實沒有整數倍也可以 Work)
- `Sample Window`
    - 兩個 Sample Events 之間的 bitslots 集合又稱為 Sample Window

Figure 121 為 Sample Events 與 Sample Intervals 的概念圖：

![Alt text](image/figure121-1.png)

![Alt text](image/figure121-2.png)

Payload Data Window
-------

SoundWire 允許將最多 4 個 Sample Windows 組在一起，形成一個 `Payload Data Window`，而整個 Payload Data Window 裡面的 Bitslots 數量又稱為 `Payload Data Interval`。可以參考 Figure 122：

- Payload Data Window 裡面可以有 1 ~ 4 個 Sample Windows
- 可以用 `BlockGroupCount` 參數來設定

![Alt text](image/figure122-1.png)

![Alt text](image/figure122-2.png)

下圖 Figure 123 則是一個 `BlockGroupCount=1` (Payload Data Window 與 Sample Window 的大小一樣) 的範例，其 Frame Shape 共 800 bits，可以容下四個 Payload Data Interval 為 200 bits 的 Payload Data Window。

![Alt text](image/figure123.png)

> Payload Data Window 可以是任意的大小。

Transport Sub-Frame
-------

Sub-Frame 其實只是一個抽象的概念，data port 會將它用在 Frame 中，以區分哪個 bitslot 屬於哪個 stream。

Figure 126 為一個 Frame 中有兩個 Data Port Sub-Frame 的例子，其中 `HStop` 就等於 Sub-Frame 的行數。要注意多個 Sub-Frame 區塊不能相互覆蓋。一個 Sub-Frame 內的 Bitslots 就是某一個 Data Port 可能要傳輸的數據。

![Alt text](image/figure126.png)

> 每個 Data Port 配置的 Transport Sub-Frame 大小都可以不一樣。

Payload Transport Window
-------

簡單來說 Payload Transport Window 就是 Payload Data Window 與 Transport Sub-Frame 的交集區域。Data Port 也會用 Payload Transport Window 來區分哪個 bitslot 屬於哪個 stream。

Figure 128 是 Payload Transport Window 的示意圖：

![Alt text](image/figure128.png)

Figure 129 是 Payload Transport Window 在相同 Sample Interval 的情況下，不同 BlockGroupCount 的示意圖：

![Alt text](image/figure129.png)

Payload Data Container
-------

Payload Data Container 是 **channel 傳輸 audio sample 的最小單位**，`WordLength` 參數會決定了 Payload Data Container 的長度，如下圖所示，前段會放取樣數據，而後段沒放數據的部分則填充 0，並且是先傳 MSB 再傳 LSB：

![Alt text](image/payload_data_container.png)

Address 0x0003 `DPx_BlockCtrl1` 內的 `WordLength` 參數可以設定 Payload Data Container 的長度，範圍是 1~64，如下 Figure 87：

![Alt text](image/figure87.png)

> 每個 Payload Data Container 都對應了一個 channel。

Payload Data Block
-------

Payload Data Block 是由一個或多個 Payload Data Container 所組成，是 Data Port 進行傳輸的最小單位。

Payload Data Block 有一個 `BlockPackingMode` 參數，其有兩種模式可以選擇，分別為 `Block-per-Port Mode` 和 `Block-per_channel Mode`。

#### Block-per-Port Mode ####

每個 Payload Data Block 都囊括了一組 Payload Channel Samples，並且每個 Payload Channel Sample 都對應其 channel。`BlockGroupCount` 參數決定了該 Payload Data Block 有幾組 Payload Channel Samples，如下圖：

![Alt text](image/block-per-port.png)

![Alt text](image/figure124.png)

#### Block-per-Channel Mode ####

將 Payload Data Block 又分成好幾個 Payload Data Sub-Block，每個 Sub-Block 都包含了一個 Payload Channel Samples，並且每個 Payload Channel Sample 都對應其 channel。在該模式下 `BlockGroupCount` 值固定為 1，如下圖：

![Alt text](image/block-per-channel.png)

上圖的 `SubBlockOffset` 參數用來區分每個 Payload Data Sub-Block。

Payload Positioning
-------

`BlockOffset` 參數可以用來控制 Payload Transport Window 的位移。主要目的是用於辨識 Payload Transport Window 中的有效數據位置，以確保接收端能夠正確解析音訊和控制數據。

![Alt text](image/payload_positioning.png)

Multiplexing Payload Streams
-------

如果有兩個 payload stream 的 sample rate 相同，則他們可以共享一個 Payload Data Window。而如果這些 stream 來自同一個 data port，並且設定了相同的傳輸參數 (Sub-Frame)，則他們在 Payload Transport Window 中會分享相同的 bitslot。

兩個不同的 bitslots 之間可以使用 `BlockOffset` 來區分他們各自的 Payload Data Block，使其在相同的 Payload Transport Window 中不會互相覆蓋。

Figure 133 可以看到 data port A 和 B 擁有相同的取樣率和 Transport Sub-Block 的配置，所以他們可以共用一個 Payload Transport Window，並通過 `BlockOffset` 來區分開。

![Alt text](image/figure133.png)