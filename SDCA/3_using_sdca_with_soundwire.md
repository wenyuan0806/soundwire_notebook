Using SDCA with SoundWire
=======

描述 SoundWire 如何用 SDCA 來傳輸 audio payload data 和 control information。

Addressing of SDCA Resources
-------

#### Hirarchical Addressing of SDCA Controls ####

Controls 可以用 26-bit Hirarchical Address 來辨別，由以下元素組成 (Figure 151)：

- `Function Number` (3 bits)
    - 每個裝置最多可以有 **1 ~ 8** 個 SDCA Functions (例如麥克風、放大器等)
- `Entity ID` (7 bits)
    - 每個 Function 最多可以有 **1 ~ 127** 個 Entities
    - 其中 `Entity ID #0` 保留給 Function-Level Controls 使用
- `Control Selector` (6 bits)
    - 用來識別 **1 ~ 48** 個標準化參數、或者 **1 ~ 16** 個自定義參數
- `Control Number` (6 bits)
    - 用來識別一個 Control 中 **1 ~ 64** 種的 instance
    - 用於識別 Entity 中對 per-Channel 的控制
    - `Control Number #0` 為 Main Control，可以一次影響 Cluster 中所有的 Channel
- `nMBQ_Atomic` (1 bit)
    - 0 : Access the primary byte
    - 1 : Access the queued byte
- `CN_Attribute` (1 bit)
    - 0 : Select Current Value Register (CVR)
    - 1 : Select Next Value Register (NVR)
- Reserved (2 bits)
    - Host / Manage 要將 Reserved Bits 設為 0，而 Peripheral Device 則可以直接忽略 Reserved Bits

![Alt text](image/figure151.png)

#### Addressing for SDCA Memory Blocks ####

SDCA Device 可以有一個或多個 Memory Blocks for 一種或多種用途，例如：

- Firmware in XUs
- History Buffer in SMPUs
- UMP Message Buffer in several types of Entity

> 可以自定義要怎麼劃分記憶體 for 這些不同的用途。

Addressing SDCA Resources When Using a SoundWire Interface
-------

#### Accessing SDCA Controls with SoundWire ####

SoundWire 分配了 **64 MB** for SDCA Controls (`0x40000000` - `0x43FFFFFF`)。

設計 SDCA Hirarchical Address (映射到 SoundWire Address) 的目的是減少一直切換 Page 的次數，以增加 access Controls 的效率。

Device 用 Read/Write Command 時，26 bits Hirarchical Address 如下：

- bit[25:23] (3 bits) 是由第一個 Paging Register (`PCP_AddrPage1`) 提供
- bit[22:15] (8 bits) 是由第二個 Paging Register (`PCP_AddrPage2`) 提供

![Alt text](image/figure152.png)

從上圖可以看到基本的 SDCA Controls Access 都用 16A 而已，很少會去修改 bit 15~25 的 Paging Register 區域，以節省存取時間。

- `0x40000000` – `0x43FFFFFF`
    - SoundWire 分配給 SDCA for Audio Controls

![Alt text](image/address_for_sdca_control.png)

#### Accessing SDCA Memory Blocks with SoundWire ####

SoundWire 分配了 **64 MB** for SDCA Memory (`0x44000000` - `0x47FFFFFF`)。

定址方式跟 SDCA Controls Access 一樣，可以直接看 Figure 153：

![Alt text](image/figure153.png)

也可以直接用 BRA 來存取 Memory Blocks，BRA 還可以在不用 Paging Registers 的情況下提供 32-bit 的定址。

#### Allocating Address Space for Virtualized Memory in a Function Agent ####

某些 SDCA Memory（例如，用於 HID Messaging 的自定義 Buffer）實際上可能位於 Function Agent 中，例如 DSP。

分配給 SDCA Memory 的 64 MB SoundWire 位址空間中的前 1M 被保留用於此用途，而不是在 Device Function 本身實現的 Memory。這 1M 位址區塊對應於 Figure 153 中位址的 bit 25 ~ 20 (0b11111)：

- `0x44000000` – `0x47EFFFFF`
    - Memories implemented in Device Function (hardware)
- `0x47F00000` – `0x47FFFFFF`
    - Memories implemented in Function Agent (software/firmware)

![Alt text](image/address_for_sdca_memory.png)

#### Accessing Dual-Ranked Registers with SoundWire ####

當 Host 要用 Dual-Ranked Access Mode 來存取 SDCA Control(s)，它需要做以下事：

- 分配一個 commit group 給 Function
- Write the next Controls
- 寫一個 bitmask 1 到 PCP_Commit Register (不需要自己清零)
    - 寫入後會發生 commit operation，一次更新一群 Controls

#### Accessing Multi-Byte Quantities with SoundWire ####

用 SoundWire 學到的 MBQ 讀寫機制來讀寫 SDCA Controls。

#### Accessing Unimplemented SDCA Resources with SoundWire ####

Unimplementation SDCA Resources:

- A Function Number that is not implemented
- A Entity ID that is not implemented
- A Control Selector that is not implemented, or is Reserved
- A Current_Next bit that indicates a Next Value Register for a Control that is not dual-ranked
- A Control Number that is not implemented (e.g., the addressed Control Selector is specified as5673
having only Control Number 0, but the Control Number field contains a non-zero number)
- A Memory address that is not populated with memory

有發生以上 Unimplementation Resources 例子時通常是軟體發生錯誤了，此時 SDCA Device 硬體可能會用的回覆有：

- `Command_OK`
- `Command_Ignored`

Response for Writes 在各種情況的對應動作 :

![Alt text](image/unimplementation_for_writes.png)

Response for Reads 在各種情況的對應動作 : 

![Alt text](image/unimplementation_for_reads.png)

#### Immediate Access to SDCA Controls with SoundWire v1.2 ####

看一下每個 SDCA Control Access Mode 對應到 SoundWire 那邊有哪些限制就好。

![Alt text](image/table204.png)

> Host Agent 不會對 DC Access Mode 執行 bus access。

