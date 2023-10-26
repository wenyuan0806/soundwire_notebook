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

![Alt text](image/figure87.png)

![Alt text](image/table128.png)