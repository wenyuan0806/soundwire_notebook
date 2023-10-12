1 Overview
=======

SoundWire 是由 MIPI 提出的協議，他與 I2S, HDA 一樣都是 Audio Interface Protocol，以下是 SoundWire 的特點：
- 僅需兩條信號線 (clock & data) 就可以傳輸音訊數據、控制命令等
  - 每條 data line 可支援最高 40Mbps bit rate
  - 可以做 Audio Streaming, Device Control, Data Transport
- (Optional) 可以選擇使用更多條 data lane 訊號線來提供更大的傳輸頻寬
  - 最多使用 8 條 data lines
- 具備 clock scaling 能力，可以動態調節 clock rate 來優化數位音訊傳輸的效能或功耗
- 使用 DDR (double data rate) 數據傳輸方式，因此可以使用更低的 clock rate 和功耗
- 一個 Master 最多可以連接 11 個 Slave
  - 所有 Slave 共用 clock line 和 data lines
- 支援 Slave-to-Slave 的數據傳輸方式
- 支援 Multi-Payload Transport Mechanisms，其中包括同步和異步的 audio stream
- 可以自行靈活配置 Frame Shape
- 支援 PCM, PDM 格式的 audio streams
  - 支援的取樣頻率有 8 / 11.025 / 16 / 22.05 / 32 / 44.1 / 48 / 88.2 / 96 / 192 ... KHz
- SoundWire 比起 SLIMbus，用來傳輸 PDM devices 會更有利，其 silicon 面積可縮小 40%