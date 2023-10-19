Bulk Payload Transport Protocol
=======

本章節會解釋 SoundWire Bulk Payload Transport Protocol (BPT) 和 Bulk Register Access (BRA)。

Overview
-------

BPT 使用了跟一般 Payload Streams 相同的傳輸機制，但 BPT 可以用來跟內部 client 進行通訊，尤其是 DP0 (而不是透過 DP1~14 連接的外部 audio client)。

BPT Payload Stream 的傳輸與接收端可以是任意的 Device，而且還可以同時多個 Devices 傳輸或接收