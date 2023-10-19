Interrupts
=======

Interrupt 流程
-------

1. New Event : Peripheral 有未處理的 Interrupt
2. Peripheral assert `PREQ` in Command Field
3. Manager 看到 `PREQ`，並發出 Ping Command 蒐集所有 Peripheral 的 `Per_Stat_NN`

當 Peripheral 已經因為有新的 `Alert` 發出 `PREQ`，後續就不會在發了。除非重新回到 Attached 才會再發第二次 `PREQ`

1. Manager 詢問有 `Alert` status 的 Peripheral (速度由 System defined，自行定義優先順序)
2. Manager 讀取 `PCP_IntStat` 來確認有哪些 Actived & Enabled Int (找出哪個 DP 有 Interrupt)
3. Manager 讀取有 Interrupt 之 DP 的 `DPx_IntStat` 與 (速度由 System defined，自行定義優先順序)

注意 `DPx_IntStat` 都沒有經過 `IntMask`，讀取者須要自行處理

1. Manager 處理對應的 `IntStat` signal (System defined 自訂優先順序)
2. Manager 處理好對應的 `IntStat` signal，並且將 `DPx_IntClear` 寫入 1，以清除對應 Interrupt
3. 回到 Step6，依序清除該 Data Port 的每個 Interrupt，直到該 DP Interrupt 皆清除
4. 回到 Step5，確認還有哪些 Data Port 和 Interrupt，直到所有 DP Interrupt 皆清除
5. 回到 Step2，確認還有哪些 Peripheral 還有 Alert，並繼續處理 Interrupt 直到所有 Peripheral 皆沒有 Interrupt