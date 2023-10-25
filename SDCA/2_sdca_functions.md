SDCA Functions
=======

Overview
-------

SDCA Functions 是個抽象概念，用來描述怎麼看到 Device 的 internal signal path 並加以控制。

- 每個 SDCA Device 最多可以有 8 個 SDCA Functions
    - 除了 `HID` 以外，其他都可以重複
    - 用 **3-bit** 的 `Function Number` 來定址 (000 to 111)

#### Funtion Type ####

可以在 `Function_Type` Control 裡面選擇要 Table 1 中的哪種 Function Type：

![Alt text](image/table1.png)

#### DisCo Properties ####

與 Peripheral Device 對應的 DisCo data 裡面會描述現在這個 Device 用了哪些 Functions。

- DisCo data 用了以下兩種 _ADR
    - **Peripheral Device `_ADR`**
        - 是一個 64-bit number，用來辨認 Peripheral Device
        - 對 SoundWire Spec 中一些 `DeviceID` 欄位進行編碼後得來的 (例如 `VendorID`)
    - **Function Device `_ADR`**
        - 是一個 `3-bit` `Function Number`，用來辨識 Device 裡的 Function

- Device Function DisCo Properties 包含了
    - `mipi-sdca-function-topology-features` Property
        - 他是一個 64-bit bitmask，用於標識該 Function 中有哪些 optional features
    - `mipi-sdca-entity-id-list` Property
        - 裡面會標示目前該 SDCA Function 中所有的 Entity IDs
    - 由 `mipi-sdca-cluster-id-list` Property 組成的 cluster library
        - 裡面會列出所有 ClusterIDs (還有描述每個 Cluster 的 subproperties)
    - 由 `mipi-sdca-behavior-set-id-list` Property 組成的 behavior set library
        - skip...
    - (For Functions that use Opaque Set Controls within UDMPUs) 由 `mipi-sdca-opaqueset-id-list` Property 組成的 opaque set library
        - 它標識了所有 opaque sets 和 subproperties (用來描述每個 Cluster)
    - (For Functions that use UMP to download Files for AE or FDL) 由 `mipi-sdca-file-set-id-list` Property 組成的 file set library
        - skip...
    - DisCo `mipi-sdca-function-initialization-table` Property 中的 function initialization table
        - 它描述了 Function Initialization 期間要執行的 byte-writes sequence
