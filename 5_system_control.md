System Control
=======

本章節主要在介紹 SoundWire 的底層功能，包括 Resets, Clocks, Data Drive Collision, PHYs Selection 等。

Resets
-------

SoundWire 有三種 reset 都會讓 device 狀態轉為 unattached，從而需要重新 attached 到 bus 來等待列舉 (enumeration)，下面根據對系統的影響從小排到大，這三種 reset 分別是：

1. Soft Reset
    - Soft Reset 一般發生在設備發生同步錯誤的時候，當設備同步碼出現太多錯誤 bit 時就會發出 Soft Reset，這種 reset 對設備的影響最小，僅僅是用來防止設備在錯誤的時間往 bus 發送數據。當設備重新 attach 上 bus 後並且完成列舉之後，Master 就可以讀取設備的 register 來進行 debug。

2. Hard Reset
    - 當發生 `clock stop` 或是往 SCP_Ctrl Register 寫 `ForceReset` 時就會觸發 Hard Reset。Hard Reset 會對 Slave 內部多個(或全部)狀態有影響，影響程度取決於 designer 怎麼設計。

3. Severe Reset
    - 對設備影響最大的就是 Severe Reset，例如發生 POR 或 Bus Reset 時，其影響範圍也涵蓋了 Hard Reset 影響的部分。甚至是一些從初始化之後就不會更改的部分 (例如 PHY Output Control) 也會被 Severe Reset 影響到。

#### Bus Reset ####

如 Figure 30 所示，如果設備連續接收了超過 4096 個 Logic 1 時 (也就是 2048 個 clock cycle)，此時就會在第 4096 個 Logic 1 時觸發 Bus Reset。而之後只要保持發送 Logic 1，則設備就會一直 keep 在 reset 狀態。

![Alt text](image/figure30.png)

Clock
-------

#### Clock Generation ####

Master 通過產生 clock 來實現數據傳輸。

#### Clock Scaling

Master 可以動態調整 Clock Frequncy，以增加傳輸效率或節省功耗。Figure 31 就示範了 Master 是如何在還有 audio payload 在傳輸的期間改變 clock frequency。

![Alt text](image/figure31.png)