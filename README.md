soundwire_notebook
=======

SoundWire v1.2.1 Notebook
-------

#### Content ####

1. [Overview](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/1_overview.md)
    - 概述 SoundWire 的優勢及特點
2. [Physical Interface (PHY)](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/2_physical_interface.md)
    - SoundWire physical layer 相關內容
3. [Frame Structure and Control Word](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/3_frame_structure_and_control_word.md)
    - 講述 Frame 以及 Frame 內之 Control Word 的架構及欄位內容
4. [Frame Synchronization](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/4_frame_synchronization.md)
    - 講述 Slave 運用 Master 發來的靜態/動態同步碼進行同步
5. [System Control](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/5_system_control.md)
    - 講述裝置列舉、Reset，以及 Clock 的產生、Pause、Stop
6. [Commands](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/6_commands.md)
    - 講述 Bank Switching, SSPs, MBQs
7. [Registers](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/7_registers.md)
    - 概述 SoundWire 有哪些 Registers
9. [Interrupts](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/8_interrupts.md)
    - 講述 Interruptes 的用法
11. [Payload Transport](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/9_payload_transport.md)
    - 講述 Payload 的結構、參數與用法
13. [Bulk Payload Transport Protocol (BPT)](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SoundWire/10_bulk_payload_transport_protocol.md)
    - 解釋 BPT 以及 BRA 的結構及用法

SDCA v0.9 Notebook
-------

#### Content ####

1. [Overview](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SDCA/1_overview.md)
    - 快速導覽 SDCA 規格書的概略內容
2. [SDCA Functions](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SDCA/2_sdca_functions.md)
    - SDCA Functions 是個抽象概念，用來描述怎麼看到 Device 的 internal signal path 並加以控制
3. [Using SDCA with SoundWire](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SDCA/3_using_sdca_with_soundwire.md)
    - 描述 SoundWire 如何用 SDCA 來傳輸 audio payload data 和 control information
4. [Entities](https://github.com/wenyuan0806/soundwire_notebook/blob/main/SDCA/4_entities.md)
    - 說明常用的 Entities 以及每個 Entity 內的 Controls