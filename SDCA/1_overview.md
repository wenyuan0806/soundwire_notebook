Overview
=======

Features
-------

- 將 Software Interface 進行標準化，讓我們得以透過軟體來控制音訊相關功能
- 工程師可以用 Software/Firmware 來控制 Audio Components
- 用 SDCA 這套標準 Class Driver Software 就可以做到啟動系統並且傳輸音訊，而不需要額外的 Vendor-Specific Drivers
- 為一些常用的 audio components 提供框架 (例如：Analog Jacks, Amplifiers, Microphones 等)
- SDCA 是用來補充 SoundWire or SoundWire I3S 之音訊傳輸的規格

Typical SDCA System Architectures
-------

Figure 2 例子中有多個 Peripheral 連接到 bus 上，而每個 Peripheral 都有一個或多個 SDCA Functions：

![Alt text](image/figure2.png)

Figure 3 例子中有一個多功能的 Peripheral 連接到 bus 上，並且用三條 data lanes 來增加傳輸頻寬：

![Alt text](image/figure3.png)

Typical SDCA Software Architectures
-------

SDCA 沒有強制規範軟體架構，但它通常會有很典型的 Function, Device, Bus 和 Controller Driver。Figure 5 為 Host Software 的範例：

![Alt text](image/figure5.png)

Function Topology
-------

Figure 8 為 Function Topology：

![Alt text](image/figure8.png)

可以看到每個 Entity 都有一個編號，例如像 Figure 8 中的 **FU 32**，32 就該 Feature Unit 的 Entity ID (可以用來尋址)，範圍是 1~127，而 Disco data 裡面會存放 Entity 的  Label & ID。

每個 Entity 的 Input/Output Pin 會相互連結，如此一來 Software 就可以透過這些連結來遍歷所有 Entities 以建構出 Netlist Data Structure (可以對應 Function Topology)。

Terms
-------

- `Signal` : 輸入 Entity 或從 Entity 輸出的數據
    - 其中會包含一個或多個 Channels
- `Signal Path` : 透過 Entity 的 Input/Output Pin 將多個 Entity 連接起來的鏈路
- `Entity` : Signal Path 上具備特定功能的區塊
    - 每個 Entity 都會有 Entity ID，可以被尋址
- `Stream` : 使用 SoundWire 介面傳輸的取樣數據
- `Cluster` : 一個或多個 Channels 組成的 Signal
- `Cluster Descriptor` : 定義一個 Cluster 的 Channel 數量，每個 Channel 又會有自己的 Channel Descriptor
- `Channel Descriptor` : 描述 Channel 的三個屬性
    - `Channel_ID`, `Relationship`, `Purpose`

可以搭配 Figure 10 來看 Topology Terms：

![Alt text](image/figure10.png)

Audio Clocking
-------

- Audio Funtion 的 Sample Rate 沒有規範，可以根據用途自行定義
- Sample Rate 由 Output Terminal 的 Clock Source Entity 決定
    - 通常 Sample Rate 會與 DP 傳輸速率相同；但在使用 Flow Control 時可能會更低
- DisCo Property 可以區分 Clock Source 是來自 local PLL/Oscillator (internal) 還是 Bus Clock (external)
- Clock Source Entity 內有 Sample Rate 清單，而 `CS:SampleRateIndex` 是用來選擇該清單內的 Sample Rate
    - 當 DP 使用 **normal isochronous transport** 時，Host Agent 會用 `CS:SampleRateIndex` 選擇與 DP 相同速率的 Sample Rate
    - 當 DP 使用 **flow-controlled transport** 時，Host Agent 會用 `CS:SampleRateIndex` 指定實際的 Sample Rate，並且用程式控制 DP 的傳輸速率

#### Sample Rates for Transducer Terminals ####

若有一個 always-on Function (ex: SmartMic)，需要它在 bus clock stop 後還可以運作，這時候會需要 IC 內部自行產生 clock，用於 always-on sampling。

SDCA Defined Function Types
-------

SDCA 有定義了一些常用的 Audio Device Function

#### Smart Amp ####

Smart Amp Function 可以驅動一個或多個 Transducers，並提供以下功能：

- 保護並避免 Speaker Transducer 損壞
- 有 Sense Stream 可以支援 Host-Resident Protection Processing
- 有 Reference Stream 可以支援 Host-Resident Acoustic Echo Cancellation
- 有 Optional Second Render Stream，其中 Ultrasound 的內容與 Human-Oriented 的 audio band 分離
- 可以產生 simple tone 來指示設備的狀態，例如開機聲等

#### Smart Mic ####

Smart Mic Function 整合了一個或多個麥克風，並提供以下功能：

- Application-specific processing (例如聲音通訊或語音辨識)
- Acoustic event detection (例如關鍵字辨識)
- 在 low-power sleep state 下時，有 History Buffer 可以先放置捕捉到的音訊數據
    - History Buffer 位在 Smart Mic Processing Unit (SMPU) 中

#### Speaker+Mic ####

Speaker+Mic 結合的 Smart Amp 和 Smart Mic，並且可以移除從 Mic 收到的 Speaker 的聲音。

#### Universal Audio Jack (UAJ) ####

UAJ Function 可以支援單個 headset 或 headphone 接上一個 4-pole TRRS 3.5mm Jack (或兩個 TRS Jacks)。

#### Retaskable Jack (RJ) ####

RJ Function 可以支援外部的 Speakers, Headphones, Microphones 接上一組 1 ~ 6 個 TRS 3.5mm Jack(s)，用以支援多聲道 (例如 5.1/7.1 聲道輸出)。

#### Human Interface Device (HID) ####

HID Function 在 SDCA HID Entity 中使用 UMP 來跟 Host 傳送或接收 HID Reports，可以用來實現在 Audio Device 上做 Volumn Up/Down。

Entities
-------

Entities 根據功能做分類，又可以分成以下幾種：

- Terminals
- Units
- Clock Entities
- Other Entities

#### Terminals ####

- `Input Terminal (IT)`
    - Signal 輸入至 Function
- `Output Terminal (OT)`
    - Signal 從 Function 輸出

![Alt text](image/figure11.png)

#### Units ####

- `Channel Remapping Unit (CRU)`
    - 用來執行 Channel Mapping，每個 Output Channel 都是一個 Input Channel 的副本
- `Feature Unit (FU)`
    - 用於單獨處理每個 Channel
    - 例如調整音量、將某個 Channel 靜音
- `Multi-Function Processing Unit (MFPU)`
    - 使用 Processing Algorithms 對 Input Signal(s) 進行運算
- `Mixer Unit (MU)`
    - 將兩個以上的 Input Signals 混在一起後輸出
- `Posture Processing Unit (PPU)`
    - 讓設備以相對於使用者的方向變化來修改訊號
    - 例如平板轉向、筆電闔上
- `Smart Amp Processing Unit (SAPU)`
    - 使用相關的 Protection Function 來保護 Speaker 不損壞
- `Smart Mic Processing Unit (SMPU)`
    - 用來辨識特定聲音以觸發特定功能、在 clock stop 時去 buffer 住收到的音訊
- `Selector Unit (SU)`
    - 從多個 Input Signals 中選擇一個輸出
- `Up Down Mixer Processing Unit (UDMPU)`
    - 將預先定義好的 Mix 功能應用於一個或多個 Input Signal(s)
    - 例如，矩陣解碼可產生 5.1 Output，或將多個 Transducers 的通道組合成多通道的 reference stream
- `eXtension Unit (XU)`
    - 將自定義的 processing 方式應用在 Input Signal 上
    - XU 還提供 Download File（例如 Firemware），可以使用 UMP 來執行

![Alt text](image/figure12.png)

#### Clock Entities ####

Clock Entities 可以提供 Audio Function 內 Sample Clock 的控制及狀態。

- `Clock Source (CS)`
    - Sample Clock Source，可以從 SoundWire bus clock 除頻得來、或是來自內部 PLL or 振盪器等
- `Clock Selector (CX)`
    - 用來切換 Sample Clock Source

![Alt text](image/figure13.png)

#### Other Entities ####

- Group Entity (GE)
- Human Interface Device Entity (HIDE)
- Power Domain Entity (PDE)
- Security & Privacy Entity (SPE)
- Tone Generator (TG)

![Alt text](image/figure14.png)

Controls
-------

用來描述 Host Softwawre 如何改變 Peripheral Device 的 Function 行為。

#### Addressing Control ####

Host 會用 `Hierarchical Address` 來 acceess Control，該位址裡包含了
- `Function Number`
- `Entity ID`
    - `Entity ID (0)` 被用來定址 Function-Level Controls (例如做 reset Function 或判斷現在是否 busy 等操作)。
- `Control Selector`
    - 1-byte Control
    - 用 MBQ 做 multi-byte Control
    - 用 Control Number 來做 multi-byte Control (一個 byte 一個 byte 存取)
    - 由 Control Number 標識的一組 Control (例如 signal 中的每個 Channel 都有 1 個 Control) 這些 Control 可以是 1-byte 也可以是 MBQ
- `Control Number` (Optional)
    - 取決於 Control Selector 選擇的 Control 需不需要 Control Number

#### Access Mode ####

Control Access Mode 有以下六種模式：

- RW
    - Read/Write
- Dual
    - Dual-Ranked Read/Write
- RW1C
    - Read/Write with Write-1-to-Clear
- RW1S
    - Read/Write with Write-1-to-Set
- RO
    - Read Only
- DC
    - DisCo Contant
    - Peripheral Device Hardware 裡沒有這個 Control，但 Software 可以用這個 Control 從 DisCo Data 中讀出常數值

#### Access Layer ####

每個 Control 的 DisCo Properties 都包含了 Access Layer，可以用來指示預計哪些 Software Layers 或哪些 Layers 可以存取這個 Control。

- `User`
    - 直接從 user input 推導出來的
    - 例如 system-level volumn Control
- `Application`
    - 從 Class Software 上面的 application software 推導出來的
    - 例如一個 conferencing application
- `Class`
    - 在 Class Software 中算出來的
- `Platform`
    - 由 OEM / System Vendor 決定
- `Device`
    - 由 Device Function 自己控制
- `Extension`
    - 由 Extension Software 控制

#### Deferred Access to Controls ####

有一些 Controls 會很花時間，例如我們寫了某個 Control 後需要等待 DSP 做訊號處理的結果，因此有一個 Function_Status Control 裡面有一個 busy bit 來指示 Control 做完了沒。如果 busy bit 還沒放下來，那麼 Host 就先不要做其他的 Control Access。

下圖是 Function_Status Control bit 7 (busy bit) 的介紹：

![Alt text](image/busybit.png)

#### Interrupt from Controls ####

有些 Controls 會發 interrupt 告知 Host Software，Function 自己修改了 Control 的值。

> 也可以在 Control Selector 裡面加入自己定義的 Control。

Resets
-------

#### SDCA_Reset ####

Peripheral Device 在 SDCA parts 的 Reset 就叫 SDCA_Reset。

- 當發生以下事件時會激活 SDCA_Reset：
    - Hard Reset
    - Severe Reset
    - (Optional) `Function_Action` Control
        - 每個 SDCA Functions 會有 `Function_Action` Control，可以用來啟動  SDCA_Reset 來 reset 單一個 Function，而不影響到其他 Functions

#### Soft Reset ####

就是 SoundWire Spec 裡面講的 Soft Reset。

Soft Reset 會暫停 DP 的 audio streams，Designer 可以自定義 Soft Reset 時對應的動作，例如修改 Amp Output 來避免 pop 聲或保護 Amp 之類的。

#### Resets in Function_Status Control ####

每個 Function 都有 `Function_Status` Control，用於向 Software 報告各種事件(例如 reset 非同步的發生了)，不是 Software 寫入 Controls 的直接結果。

#### Effects of Reset ####

SDCA_Reset 將大多數的 Controls 變更為 Default 狀態（可以從 DisCo data 看到），包括變更 Power Domain Entity (PDE) 中的 Power State Control。

SDCA_Reset 對 memory 的內容應不會有影響，但由於 Entity 的 Effective Power State 會被變更為 PS3，所以 memory 的內容可能會遺失。

Host Agent Software
-------

可以理解成 Driver，或是 Host 這邊相關硬體的 Firmware 都算是 Host Agent。

其它章節會用其它術語更具體的表示其 Host Agent 的功能：

- `Class Software`
    - 跟 Device 互動的軟體
- `Function Agent`
    - 用來執行 Device Function 某些行為的軟體（Figure 30）
    - 例如，Host DSP 處理數據以提高音訊保真度，或將 PDM Stream 轉換為 PCM Stream

![Alt text](image/figure30.png)

#### Extension Driver ####

Host Software 架構可以自定義，但通常會包含一個 Class Driver 用來 loading Extension Drivers。

而 Extension Driver 不限於只能訪問 XUs，也可訪問任意 Entities 的 Controls、以及 Entities 以外的東西 (例如 Device Address Map 自定義的 Registers)。

#### Initialization Writes ####

Device 的 Disco data 包含的 `Function Initialization Table` 描述了 Initialization Writes 的順序。該 table 是一個 32-bit address 對 8-bit data 的列表。

Device Function 如果需要做 Initialization Writes，它可以在 Entity0:`Function_Status` Control 中設定 `Function_Needs_Initialization` bit；在 Host 執行完 Initialization Writes 後，Host Agent 就會清掉 `Function_Needs_Initialization` bit。

Initialization Writes 使用的是 **flat addressing** 而不是 structured SDCA addressing (Function, Entity, Function Selector, etc.)，因為它可以存取 Disco 裡面沒有宣告的 Controls、以及 Device Address Map 自定義的 Registers。

#### Class Software Initialization and Default Values ####

Controls 的起始值會被以下事件影響：

- Resets
- Initialization Writes
- Class Software Initialization
- Class Software Resotring User State

Class Software 初始化 Device 後，每個 Control 中的值通常優先順序為：

1. A value chosen by the user or OS.
2. A value chosen by the Class Software.
3. A platform-specific Default Value that the OEM specified in a DisCo Property.
4. A specification-defined hardware reset value. (Known from this Specification, and might be Function-specific or Entity-specific.)
5. An implementation-defined hardware reset value. (Chosen by the hardware manufacturer.)
6. A known existing value. (Persisting from before this initialization.)
7. An unknown / unpredictable value, which might be different from what was there earlier.

有些 Controls 會有 DisCo Property `mipi-sdca-control-default-value` (or `mipi-sdca-control-cn-<n>-default-value`)

#### Outline of Software Flow for Stream Start and Stop ####

Figure 17 是 Host Software Framwwork 控制 audio stream 以支援系統省電的範例。

- Stream Controls at Class Level：
    - Stream Create / Destroy
    - Stream Start
    - Stream Pause / Resume
    - Enter Low Power

- Stream Controls at Bus Controller Level：
    - Start Bus
    - Clock Start / Stop 
    - Idle Timeout

完整的 Bus Controller 框架還會把多個 Clients 的 request merge 起來，並且採用更高級別的 Power State (如果有需要的話)。

![Alt text](image/figure17.png)

Bus Interfaces
-------

#### SoundWire ####

SDCA 使用的是 SoundWire Interface，有兩個 Blocks 的 address map 是分配給 SDCA 使用的

- 64 MB for Controls
    - 用於 Control 的 Hirarchical Address
        - 3-bit for Function, 7-bit for Entity ID ... 等
- 64 MB for Memories
    - Address 的分配可以自定義

此外，Device 可能還會有其它的 memory 或 registers，而他們使用的是 SoundWire Address Map 中的自定義區域。

有些 Controls 會使用 MBQ (in SoundWire Spec)；有些 Controls 會使用 Dual-Ranked Register (in SoundWire Spec)。

SoundWire Peripheral Control Port 分配了 31 個 interrupts 給 SDCA (`IntStat`, `IntClear`, `IntMask`)。

UMP (Universal Message Passing)
-------

UMP 用來做數據交換。

- **Application to Any Function**
    - File download for Device-resident firmware in XUs
    - Algorithm Extension in MFPUs
- **Specific to Particular Functions**
    - 將 SMPUs (SmartMic) 的 history buffer 內的 audio samples 上傳
    - SAPUs (SmartAmp) 之間做 Device-to-Device 的訊息傳遞，用來做 synchronization protection behavior
    - SMPUs (SmartMic) 之間做 Device-to-Device 的訊息傳遞，用來做 synchronization triggers
    - 傳送 HID Report，用來做控制使用 (例如調整音量等)

#### Summary of UMP ####

透過在 Device 和 Host 之間傳遞共享的 device memory area 的所有權來傳輸 `Message`，而每個 `Message` 都是一個或多個 byte block(s) 組成。DisCo Properties 有定義了 Device Address Space 的區域，稱為 `buffer`。

每個 UMP 都有以下三個 standard Controls:

- `CurrentOwner`
    - 會指示哪個 Host or Device 擁有 message memory 的所有權
    - Host Software 可以啟用 interrupt 來轉移所有權
- `MessageOffset`
    - 指示 `Message` 的位置是在 `Buffer` 的哪裡
- `MessageLength`
    - 指示要傳遞的 `Message` 的長度

Host 會使用 Read/Write Operation 來存取 `buffer`，甚至是用 BRA 來存取也可以。

> 一個 UMP 只能單方向傳遞 `Message`，要雙向的話就要兩個 UMPs。

#### UMP Clients ####

Device Function 用來做 UMP 通訊的部分就稱為 **UMP Clients**。

Device Function 也可以請求 Host 用 UMP 傳遞一些 Messages，例如請求 Firmware Download (`FDL_UMP` in XU)、或是請求 Algorithm Extension Data (`AE_UMP` in MFPU) ... 等。

