Entities
=======

Feature Unit (FU)
-------

FU 使用的符號可以看 Figure 79：

- User FUs
- Application FUs
- Class FUs
- Platform FUs
    - Platform FUs 跟其它種類 FUs 有以下相異處
        - 不是 dual-ranked
        - 通常使用 Gain Control (不是 Volumn Control)

![Alt text](image/figure79.png)

FU 可以對輸入的每個 Channel 做一些像是 mute, volumn control 等操控，而且可以一次只操控一個 Channel。

#### FU: Controls ####

User, Application, Class, Platform FUs 各自擁有的 Controls 都不同。

Table 86 為 Class FU Controls 列表：

![Alt text](image/table86.png)

Table 87 為 User FU Controls 列表：

![Alt text](image/table87.png)

Table 88 為 Application FU Controls 列表：

![Alt text](image/table88.png)

Table 89 為 Platform FU Controls 列表：

![Alt text](image/table89.png)

#### FU: Controls with Reset Values ####

Table 90 是在發生 SDCA_Reset 後指定的 FU Controls 值：

![Alt text](image/table90.png)

有些 Controls 會在 DisCo Properties 的 `mipi-sdca-control-default-value` or `mipi-sdca-control-cn-<n>-default-value` 裡面註記 SDCA_Reset 後的值要怎麼設定。

#### FU: DisCo Properties ####

沒有專門給 Feature Unit 的 DisCo Properties。

#### FU: Interrupt Sources ####

Feature Unit 並沒有任何標準化的 Interrupt Sources，這些 Interrupt Sources 的行為能被 Standard Class Software 理解。然而，在一個 Feature Unit 內部可能存在自定義的 Interrupt Sources，這些 Interrupt Sources 將由像是 Extension Driver 之類的東西來處理，而不是由 Standard Class Software 來處理。換句話說，這些特定的 Interrupt Sources 並不遵循通用的規範，而是由特定的驅動程式來管理和處理。

#### FU: Notes on Behavior of Device Class Software ####

典型的 Device Class Software 的行為是：

- 忽略 Platform FUs 的 Controls，這些 Controls 是由平 platform-initialization software 設定的
- 使用個別的 Channel Volumn Control 來控制音量

#### Platform FUs ####

- Platform FUs 是一種特殊的 FUs，通常只被 platform-initialization software 所存取
- Platform FUs 沒有 dual-ranked control，因為 platform-initialization software 不需要對運行中的系統進行同步更新
- Platform FUs 的 Control Access Mode 是 RW，因為 Software Agent 在正常操作時可能也會 Read Controls
- Platform FUs 有 Gain Control 而非 Volumn Control
- Platform FUs 的 Control 通常用於 platform-level 的 Gain 修改

Platform FUs 應該要實現以下任一項：

- A Main Gain (i.e., 0x0B.0) Control
- A Channel Gain (i.e., 0x0B.1–n) Control
- No Gain Control

Mixer Unit (MU)
-------

Mixer Unit 的符號可以參考 Figure 81：

![Alt text](image/figure81.png)

Mixer Unit 可以把多個 Input Channels 混合並輸出在一個 Output Channel 上。

![Alt text](image/table85.png)

#### Platform MUs in SDCA-Defined Functions ####

- 用於產生 unity-gain 的 Mixer Control 的實際值是 platform-specific 的，例如可能是 0 dB, −3 dB, −6 dB 等
- Mixer Control 的值也可能是 −∞ dB，也就是 muted connection
- 在某些情況下，這些 Gain 值由 Host Agent 複製 DisCo Properties `mipi-sdca-control-cn-<n>-default-value` 的值寫入 Mixer Control

#### MU: Controls ####

Table 106 為 MU Controls 列表：

![Alt text](image/table106.png)

#### MU: Mapping of Control Number to Cross-Point Mixing Matrix ####

Figure 82 解釋 MU Control 是怎麼做 mix 的：

- Output Pin 有 Q 個 Channel
    - 這些 Channels 都是由多個 Input Pin mix 出來的結果
- MU 有 P 個 Input Pin
    - 每個 Input Pin 又都自己有好幾個 Channel

![Alt text](image/figure82.png)

Figure 64 為 Control Numbers in the UAJ sidetone mixer 的範例：

![Alt text](image/figure64.png)

> 用於 MU Control 的 Q7.8dB format 代表增益範圍 −128 dB 至 +127.996 dB

#### MU: DisCo Properties ####

沒有專門針對 Mixer Unit 的 DisCo Properties。

#### MU: Interrupt Sources ####

- Mixer Unit 並沒有任何標準化的 Interrupt Sources，這些 Interrupt Sources 的行為能被 Standard Class Software 理解
- 在一個 Mixer Unit 內部可能存在自定義的 Interrupt Sources，這些 Interrupt Sources 將由像是 Extension Driver 之類的東西來處理，而不是由 Standard Class Software 來處理

#### MU: Notes on Behavior of Device Class Software ####

MU 應該要實現以下任一項：

- 具有 DisCo Properties **`mipi-sdca-control-access-layer`=Class** 的 Fixed-Function Unity Gain Mixer（例如 UAJ Function 中的 side tone mixer）
- 具有 DisCo Properties **`mipi-sdca-control-access-layer`=Platform** 的 Mixer（例如，SmartAmp Function 輸出路徑中的配套 Input Stream Mixer）

Selector Unit (SU)
-------

Figure 87 為 SU 的符號表示：

- Class Selector
- Device Selector

![Alt text](image/figure87.png)

![Alt text](image/table128.png)

Selector Unit 做的事就是從幾個 Input Pins 中選擇一個做為 Output Pin 輸出，中間不會做其它訊號處理。

#### SU: Controls ####

Table 129 是 Class SU 的 Controls 列表：

![Alt text](image/table129.png)

Table 130 是 Device SU 的 Controls 列表：

![Alt text](image/table130.png)

#### SU: Controls with Reset Values ####

在發生 SDCA_Reset 後，SU Controls 的預設值列表如 Table 131：

![Alt text](image/table131.png)

#### SU: Interrupt Sources ####

SU 沒有任何 Standardized Interrupt Sources，其行為由 Standard Class Software 來處裡。

Group Entity (GE)
-------

Figure 92 用 SDCA Audio Function Topology 來說明 GE：

![Alt text](image/figure92.png)

- GE 是一種捷徑機制，允許 Host Software 透過單一個 Control `Selected_Mode`，對 Audio Function 進行一組相關的變更
- 當軟體寫入新的 Mode 到 `Selected_Mode` 時，Audio Function 中的多個 Control 和多個 Entity 就可以同時被更新

DisCo 屬性定義了哪些 Control 和 Entity 會被 GE 的 Selected_Mode Control 所改變。Audio Function Topology 中，任何可能受到 GE 影響的 Entity（例如 Selector Unit）都會用橘色背景和紅色虛線框來標示。

#### Device Behavior for Jack Insertion ####

當 Jack 插入時，

1. Device Function 要執行自定義的 detection algorithm 來判斷插進來的東西是啥
    - 在執行 detection algorithm 時，Device Function 可能會暫時把 `GE:Detected_Mode` Control 設為 2 (代表 `Detection_in_Progress`)
2. 接下來 Device Function 要再把 `GE:Detected_Mode` Control 設定成一個非 0 or 2 的值
    - 如果 `Selected_Mode` 目前的值無法安全的用於剛偵測到的 peripheral，則 Device Function 應該把 `Selected_Mode` 變更為與 peripheral 相容的值
    - 當 `Detected_Mode` 是已知的 peripheral 時（Mode 3 ~ N），則 Device Function 應該會把 `Selected_Mode` 變更為與 `Detected_Mode` 相同的值
3. 最後將 `IntStat_NN` 位元設為 1 (`NN` 是分配給 GE:Detected_Mode Control 的 interrupt number)
    - 如果對應的 `IntEnable_NN` 位元為 1，則會在 Ping Command 中報告此 interrupt 狀態

> Function Agent 也可能會協助 Device Function 做 detection algorithm

#### Class Software Behavior for Jack Insertion ####

當 Class Software 看到 `GE:Detected_Mode` Control 改變時 (可能是透過 polling 或 interface 看到的)，它必須執行以下至少一項：

1. 若新的 `GE:Detected_Mode` value 是一個已知的 peripheral (value 不是 1)，則會把這個 value 寫到 `GE:Selected_Mode` Control
    - 與 `GE:Detected_Mode` 的值相同
    - 該值不能是 1 或 2
2. 如果新的 `GE:Detected_Mode` 值為 Jack_Unknown（Mode 1），則會寫入 `GE:Selected_Mode` Control 以選取非 1 或非 2 的 Mode value
3. 如果新的 `GE:Detected_Mode` value 為 `Detection_in_Progress`，則不會更新 `GE:Selected_Mode` Control value

> 一個 GE:Detected_Mode 的變更可能會導致 Host 重新評估其他 GE:Selected_Mode 的決策

#### Class Software Prioritization Behavior with two Jacks ####

可以參考 Table 149, 150, 151, 152

Table 153 為 Detected_Mode 與 Selected_Mode 各種 Mode 下的解釋：

![Alt text](image/table153.png)



