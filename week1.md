# Week 1: 5G Network Functions 
<!-- 
:::info
Syllabus: https://timetable.nycu.edu.tw/?r=main/crsoutline&Acy=114&Sem=1&CrsNo=535607&lang=en
:::
 -->
## 課程目標

- 認識核心網路
- 從需求出發，了解網路元件的用途
- 認識網路元件
- 認識以服務為導向架構，並且了解網路元件之間如何協作


## 認識核心網路

![image](https://hackmd.io/_uploads/rkSQAiRMll.png)
> Ref: https://www.scimonth.com.tw/archives/4130#google_vignette

*圖：通訊網路架構，展示從手機接入一路至骨幹網路*

核心網路是通訊網路的大腦，他負責管理用戶設備與基地台的連線狀態（這部分相當複雜，透過後續的章節我們會慢慢了解）以及用戶層資料的傳輸。

通訊網路由三大部分組成：

- UE (User Equipment)
- RAN (Radio Access Network)
- CN (Core Network)

其中 UE 並不會直接觸碰到核心網路，而是透過基地台的控制訊號封裝後才能與核心網路溝通。

## 用戶（營運商）對核心網路的需求

![image](https://hackmd.io/_uploads/SkkRkhCGxg.png)
> Ref: https://www.geeksforgeeks.org/5g-network-architecture/

*圖：核心網路架構，可細分為控制平面與傳輸平面*

對於公網來說，其實我們並不會察覺到核心網路的存在。人們在乎大概就是「有沒有吃到飽」、「基地台數量（覆蓋率）」、「我家附近的訊號強度」。
但對於營運商（電信業者）來說，核心網路對他們就非常有感了，我們可以想像一下自己家是電信業者的話，我們會期待核心網路為我們提供什麼？
- 管理基地台連線的能力
- 承載大量 UE 的能力
- 訊號傳輸的安全性
- 如果基地台離機房真的太遠，有沒有辦法讓資料層的延遲降低？
- 計費管理

我們就從這些角度出發，看看核心網路該如何設計吧！

### 管理基地台連線的能力

感謝基地台幫我們處理了控制訊號以及用戶層資料的無線傳輸，所以開發核心網路不用探討無線傳輸技術。
到目前為止，我們對基地台的已知是：
- 它會與核心網路建立連線
- 它會與 UE 建立連線（接入層，Access Stratum），並且保證連線的安全性
- 它會接收 UE 傳來的控制訊號以及用戶層資料
- 它會接收核心網路傳來的控制訊號以及用戶層資料

也就是說，核心網路需要為上述幾點分別提供對應的能力：
- 它會與核心網路建立連線：**核心網路要能夠管理每一筆基地台連線背後的狀態，進而得知每一筆訊息源自於哪一個基地台。**
- 它會與 UE 建立連線（接入層，Access Stratum），並且保證連線的安全性：**基地台安全的收到 UE 的訊號後，會透過後傳網路送往核心網路，核心網路需要有能力保護它與 UE 之間訊號的安全性（非接入層，Non-Access Stratum）**。
- 它會接收 UE 傳來的控制訊號以及用戶層資料：**核心網路需要管理 UE 的連線、註冊、會議（Session）狀態。也因為這些狀態，核心網路中需要有網路元件提供明確的策略（Policy）來處理 UE 在接入、會議上的需求。**
- 它會接收核心網路傳來的控制訊號以及用戶層資料：**在某些情況下，核心網路會需要干涉基地台與 UE 的連線（如：Handover、Paging、Network-initiated deregistration 等。）不止如此，由於每一個會議的資料層需要依靠基地台轉送，因此核心網路必須要管理基地台的資料層資源。**

### 承載大量 UE 的能力

一個電信商的訂閱用戶通常以百萬為單位，核心網路中的網路元件們相互合作完成 UE 的每一個請求。[C10k](https://en.wikipedia.org/wiki/C10k_problem) 問題是我們在開發網路服務時需要考量的重點之一，這個問題同樣存在於通訊網路中。不僅如此，通訊網路對於[可用性](https://zh.wikipedia.org/zh-tw/%E9%AB%98%E5%8F%AF%E7%94%A8%E6%80%A7)有著更高的要求。

由此可知，核心網路在設計時須考慮：
- **動態擴展的能力**
- **部署在分散式架構的能力**

### 訊號傳輸的安全性

在[管理基地台連線的能力](#管理基地台連線的能力)一節有提到：**核心網路需要有能力保護它與 UE 之間訊號的安全性（非接入層，Non-Access Stratum）**，這部分在 3GPP 的標準中都有明確定義，在後面的課程中我們也會詳細介紹。
對於安全性，我們可以預期核心網路之中必須有網路元件能夠：
- **管理加解密的必要元素**
- **驗證（Authentication）**
- **加解密**

### 如果基地台離機房真的太遠，有沒有辦法讓資料層的延遲降低？

降低資料層的延遲最好的做法就是讓負責處理資料層轉發的網路元件距離使用者近一點（也就是離基地台近一點）。所以**網路元件在設計時必須確保管理會議與處理資料層流量的網路元件必能夠分離**，管理會議的網路元件甚至有可能要一次管理大量的資料層元件。

```
+------+
| SMF1 |
+------+ N4 (192.168.1.10/24)
|      |
|      +-----------------+
| N4 (192.168.1.11/24)   | N4 (192.168.1.12/24)
+------+                 +------+
| UPF1 |                 | UPF2 |
+------+                 +------+
```


### 計費管理

對於營運商來說，最重要的事情除了提供用戶服務以外，再來就是**收錢**了，所以核心網路勢必要能夠：
- **根據會議使用的資料層流量提供計費資訊**
- **儲存、管理這些已產生的計費資訊**
- **每一個會議背後的用途可能不同（如：Internet、IMS、SMS 等。），不同的用途對應的費率必須能夠單獨設定**

## 認識網路元件

到目前為止，我們已經了解營運商的需求層面以及核心網路必須提供的能力。接下來就讓我們逐一認識常見的（free5GC 有實作的）網路元件：

### AMF (Access and Mobility Management Function)

AMF 負責的工作在 EPC 中原本是由 MME 負責，但是在 EPS 的設計中，MME 需要負責更多 Control Plane 的處理。在 5GS 中，AMF 僅需要注重在訪問與移動性管理，具體功能包含:
- NAS（Non-Access Stratum）傳輸過程的加密與完整性保護
- 為 UE 的 PDU Session 選擇合適的 SMF（可以基於 TAC、切片或是 DNN）
- 參與 Authentication 流程
- Connection & Registration Management
- 處理與 UE、RAN 之間的 Control Plane message
- 處理 4G-5G or 5G-5G 的 Handover
- UE reachability
- 網路切片的選擇（與 NSSF 互動選擇出最佳的 Slice）

### SMF (Session Management Function)

SMF 負責 UE 的會議管理相關功能，它是 5G 核心網路中負責管理 PDU Session 的關鍵元件。在 4G EPS 中，這些功能主要由 P-GW-C 和 S-GW-C 負責。SMF 的主要功能包含：

- **PDU Session 管理**：建立、修改、刪除 PDU Session
- **UE IP 位址分配與管理**：為 UE 分配 IP 位址並管理 IP 位址資源池
- **UPF 選擇與控制**：選擇適當的 UPF 並透過 N4 介面控制 UPF 的封包轉送行為
- **流量路由控制**：配置用戶平面的流量路由規則
- **QoS 控制**：實施 QoS 流（QoS Flow）和服務資料流（Service Data Flow）的映射
- **計費控制**：向 CHF 提供計費相關資訊
- **策略執行**：執行來自 PCF 的策略決定
- **漫遊功能**：在漫遊場景中與 Home SMF 協作

SMF 在 SBA 架構中扮演重要角色，它需要與多個網路元件協作，包括 AMF（會議建立請求）、UPF（用戶平面控制）、PCF（策略控制）、UDM（會議相關資料）和 CHF（計費資訊）。

### UPF (User Plane Function)

UPF 是 5G 核心網路中負責用戶平面功能的網路元件，相當於 4G EPS 中的 S-GW-U 和 P-GW-U 的功能組合。UPF 是唯一處理用戶資料流量的核心網路元件，其主要功能包含：

- **封包路由與轉送**：根據 SMF 的控制規則轉送用戶資料封包
- **封包檢測**：深度封包檢測（Deep Packet Inspection, DPI）以支援服務資料流的識別
- **QoS 處理**：執行流量整型（Traffic Shaping）、流量管制（Traffic Policing）和封包標記
- **使用量報告**：向 SMF 報告用戶資料使用量以支援計費和策略控制
- **上行流量分流**：支援本地分流（Local Breakout）功能，將特定流量就近處理
- **下行封包緩衝**：在 UE 處於閒置狀態時暂存下行封包
- **多歸屬支援**：支援 PDU Session 的多路徑傳輸
- **邊緣運算支援**：可部署在網路邊緣以降低延遲

UPF 透過 N4 介面接受 SMF 的控制，透過 N3 介面與 RAN 連接，透過 N6 介面與外部資料網路（如網際網路）連接。UPF 的分散式部署特性使其成為支援邊緣運算和超低延遲應用的關鍵元件。

### AUSF (Authentication Server Function)

AUSF 是 5G 核心網路中負責用戶認證的網路元件，對應於 4G EPS 中的 HSS/AuC（Home Subscriber Server/Authentication Center）的認證功能。AUSF 的主要功能包含：

- **用戶認證**：執行 5G-AKA（5G Authentication and Key Agreement）認證程序
- **金鑰產生與管理**：產生並管理認證過程中需要的各種金鑰
- **認證向量產生**：產生認證向量（Authentication Vector）供 AMF 使用
- **EAP 認證支援**：支援 EAP-AKA'（Extensible Authentication Protocol - Authentication and Key Agreement Prime）認證方法
- **認證狀態管理**：維護用戶的認證狀態資訊
- **安全錨點功能**：作為 UE 與網路之間的信任錨點
- **漫遊認證**：在漫遊場景中與 Home AUSF 協作進行認證

AUSF 與 UDM 緊密協作，從 UDM 獲取用戶的長期金鑰和認證資料。在認證過程中，AUSF 透過 Nausf 介面為 AMF 提供認證服務，確保只有合法的用戶能夠存取網路服務。

```
UE -> AMF -> AUSF -> UDM
   認證請求   取得用戶
            認證資料
```

### UDM (Unified Data Management)

UDM 是 5G 核心網路中的統一資料管理元件，整合了 4G EPS 中 HSS（Home Subscriber Server）的功能。UDM 負責管理用戶相關的各種資料，其主要功能包含：

- **用戶身份管理**：管理 SUPI（Subscription Permanent Identifier）和 GPSI（Generic Public Subscription Identifier）
- **訂閱資料管理**：儲存和管理用戶的訂閱資料，包括服務權限、QoS 設定等
- **認證資料管理**：管理用戶的長期金鑰（K）和認證相關參數
- **存取與移動性管理資料**：維護用戶的註冊狀態、位置資訊和 AMF 資訊
- **會議管理資料**：管理 PDU Session 相關的訂閱資料和 SMF 選擇資訊
- **隱私保護**：支援用戶身份隱私保護機制（SUPI 保護）
- **服務授權**：為其他網路元件提供用戶資料的授權存取
- **事件暴露**：向 NEF 暴露用戶狀態變化事件

UDM 與 UDR（Unified Data Repository）配合工作，UDM 提供業務邏輯處理，而 UDR 負責實際的資料儲存。UDM 透過標準化的 SBI 介面為其他網路元件提供資料服務，是 5G 核心網路中的重要資料中心。

### UDR (Unified Data Repository)

UDR 是 5G 核心網路中的統一資料倉庫，負責儲存各種網路和用戶資料。UDR 是一個純粹的資料儲存層，不包含業務邏輯處理，其主要功能包含：

- **用戶訂閱資料儲存**：儲存用戶的訂閱資料、服務配置檔案等
- **策略資料儲存**：儲存策略控制相關的資料和規則
- **暴露資料儲存**：儲存透過 NEF 暴露的資料
- **應用資料儲存**：儲存應用相關的資料和配置
- **結構化資料儲存**：提供結構化的資料儲存和檢索功能
- **資料一致性保證**：確保資料的一致性和完整性
- **高可用性支援**：支援資料備份和災難恢復
- **存取控制**：提供細粒度的資料存取控制

UDR 採用 RESTful API 介面，支援標準的 HTTP 方法（GET、POST、PUT、DELETE、PATCH）進行資料操作。主要的資料存取者包括：
- **UDM**：存取用戶訂閱資料和認證資料
- **PCF**：存取策略資料和用戶策略設定
- **NEF**：存取暴露資料和應用資料
- **UDSF**：在某些部署中可能透過 UDSF 存取結構化資料

UDR 的設計遵循資料與邏輯分離的原則，使得資料管理更加靈活和可擴展。

### NRF (NF Repository Function)

NRF 是 5G 核心網路中的網路功能資源庫，在 SBA 架構中扮演服務註冊與發現的核心角色。NRF 沒有對應的 4G EPS 元件，是 5G 架構中的全新設計。其主要功能包含：

- **NF 註冊管理**：接受其他網路元件的註冊資訊並維護 NF Profile
- **NF 發現服務**：根據查詢條件為消費者提供合適的生產者資訊
- **NF 狀態監控**：監控已註冊 NF 的健康狀態和可用性
- **負載均衡支援**：支援多個相同類型 NF 實例之間的負載均衡
- **服務授權**：管理 NF 之間的服務存取授權
- **NF 去註冊**：處理 NF 的去註冊請求
- **通知服務**：向訂閱者通知 NF 狀態變化
- **存取令牌管理**：在某些部署中支援 OAuth 2.0 存取令牌管理

NRF 的核心服務包括：
- **Nnrf_NFManagement**：提供 NF 註冊、更新、去註冊功能
- **Nnrf_NFDiscovery**：提供 NF 發現和查詢功能
- **Nnrf_AccessToken**：提供存取令牌管理功能（可選）

```
     註冊 NF Profile
SMF -----------------> NRF
                        |
                        | 查詢 SMF 資訊
AMF <------------------+
     回傳 SMF 的服務資訊
```

NRF 是實現 SBA 架構的關鍵元件，使得網路元件能夠動態發現和存取彼此的服務。

### NEF (Network Exposure Function)

NEF 是 5G 核心網路中的網路暴露功能元件，負責安全地將核心網路的能力和資訊暴露給外部應用和第三方服務。NEF 是 5G 新增的網路元件，在 4G 中沒有直接對應的元件。其主要功能包含：

- **API 暴露**：將核心網路功能封裝成標準化的 API 供外部使用
- **資料暴露**：安全地暴露網路分析資訊和用戶相關資料
- **事件暴露**：將網路事件（如 UE 位置變化、連接狀態變化）暴露給應用
- **外部參數提供**：將外部應用的參數和策略傳遞給核心網路
- **安全管理**：確保外部存取的安全性，包括認證、授權和加密
- **速率限制**：控制外部應用的 API 調用頻率
- **資料翻譯**：在外部 API 和內部 SBI 之間進行資料格式轉換
- **計費支援**：支援對 API 使用進行計費

NEF 支援多種暴露場景：
- **第三方應用**：為第三方開發者提供網路能力 API
- **邊緣運算**：為邊緣運算應用提供網路資訊
- **企業應用**：為企業私有網路應用提供客製化服務
- **垂直行業**：為特定行業應用（如智慧製造、自動駕駛）提供專用 API

NEF 透過標準化的 Capif（Common API Framework）介面對外提供服務，同時與 UDM、PCF、AMF 等內部元件互動獲取相關資訊。

### CHF (Charging Function)

CHF 是 5G 核心網路中的計費功能元件，負責收集和處理網路服務使用的計費資訊。CHF 整合了 4G EPS 中 OCS（Online Charging System）和 OFCS（Offline Charging System）的功能。其主要功能包含：

- **線上計費**：即時監控用戶的服務使用情況並進行額度控制
- **離線計費**：收集用戶服務使用記錄，生成計費資料記錄（CDR）
- **配額管理**：管理預付費用戶的餘額和配額
- **計費事件處理**：處理來自各網路元件的計費事件
- **費率管理**：根據不同的服務類型和用戶套餐計算費用
- **計費資料聚合**：將多個計費事件聚合成完整的計費記錄
- **計費策略執行**：執行運營商定義的計費策略和規則
- **第三方計費系統整合**：與外部的計費和帳務系統整合

CHF 支援多種計費模式：
- **基於時間的計費**：根據服務使用時間計費
- **基於流量的計費**：根據資料使用量計費
- **基於事件的計費**：根據特定事件（如語音通話、SMS）計費
- **基於服務的計費**：根據特定服務類型計費

計費流程中的關鍵互動：
```
SMF/AMF --> CHF: 計費請求
CHF --> UDR: 儲存計費記錄  
CHF --> 外部系統: 傳送 CDR
```

CHF 透過 Nchf 介面為其他網路元件提供計費服務，確保營運商能夠準確地對用戶的網路服務使用進行計費。


## SBA (Service-Based Architecture)

以服務為導向架構（Service-Based Architecture）提供模組化的框架，使網路元件透過共通的方式協作（只要遵照標準的規範，即使是來自不同供應商的網路元件也能正確的溝通）。
在以服務為導向架構中，網路元件（生產者）會向其他網路元件（消費者）提供自身特有的服務，消費者可以透過生產者的特有介面存取需要的服務。

![image](https://hackmd.io/_uploads/S1gzYsLQel.png)
> Ref: https://www.researchgate.net/figure/G-Architecture-in-SBA-representation_fig2_328233142

*圖：以服務為導向架構*

:::info
網路元件可以扮演服務消費者或服務生產者的角色。每個網路功能服務都透過基於服務的介面 (SBI) 公開其功能，該介面採用基於 HTTP/2 的、定義明確的 REST 介面。為了緩解 TCP 隊頭 (HOL) 阻塞問題，未來可能會使用快速 UDP 網際網路連線 (QUIC) 協定。
此外，國外學者也有研究基於 gRPC 的 SBI 服務，詳細資訊可參考 [SBA-gRPC-5G](https://github.com/iithnewslab/SBA-gRPC-5G/blob/master/Presentation_Netsoft19_gRPC_5G.pdf)。
:::

### 實際案例：AMF 尋找 SMF

若 UE 透過基地台向 AMF 請求建立會議（PDU Session），AMF 要如何在不知道 SMF 確切位址的情況下找到它呢？
在先前的章節中，我們有提到 NRF 會提供 NF Management 以及 NF Discovery 的服務，前者讓所有的網路元件都能向 NRF 進行註冊，使 NRF 能夠管理 SBA 網路中所有網路元件的資訊。接著，若某個網路元件需要得知特定網路元件的資訊，就能使用 NF Discovery 找到對方。

為此，具體的流程如下：
1. SMF 啟動時向 NRF 發送 NF Registration request
2. NRF 接受 SMF 的 NF Registration
3. AMF 收到 UE 的 PDU Session Establishment Request
4. AMF 向 NRF 詢問 SMF 的位址（消費 NF Discovery 服務）
5. AMF 成功取得 SMF 的資訊，消費 SMF 提供的 SMContext 服務

:::info
每個 NF 提供的 SBI 服務可參考：https://3gpp.sungwoonsong.com/
3GPP 提供的 OpenAPI 文見可參考：https://github.com/jdegre/5GC_APIs/tree/Rel-18 
:::

## 課堂/課後練習

- [ ] 閱讀[官方文件](https://free5gc.org/guide/3-install-free5gc/)安裝 free5GC。
- [ ] 閱讀[官方文件](https://free5gc.org/guide/5-install-ueransim/)，使 UE Simulator 能夠透過核心網路 ping 到 `8.8.8.8`。
- [ ] 試著修改 SMF 與 UPF 的組態（Configuration）檔案，使 UE 分配到的 IP 從 `10.60.0.0/16` 改為 `10.61.0.0/16`。

