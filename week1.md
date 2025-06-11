# 5G Network Functions 

:::info
Syllabus: https://timetable.nycu.edu.tw/?r=main/crsoutline&Acy=114&Sem=1&CrsNo=535607&lang=en
:::

## 課程目標

- 認識核心網路
- 從需求出發，了解網路元件的用途
- 認識網路元件
- 認識以服務為導向架構，並且了解網路元件之間如何協作


## 認識核心網路

![image](https://hackmd.io/_uploads/rkSQAiRMll.png)
> Ref: https://www.scimonth.com.tw/archives/4130#google_vignette

核心網路是通訊網路的大腦，他負責管理用戶設備與基地台的連線狀態（這部分相當複雜，透過後續的章節我們會慢慢了解）以及用戶層資料的傳輸。

通訊網路由三大部分組成：

- UE (User Equipment)
- RAN (Radio Access Network)
- CN (Core Network)

其中 UE 並不會直接觸碰到核心網路，而是透過基地台的控制訊號封裝後才能與核心網路溝通。

## 用戶（營運商）對核心網路的需求

![image](https://hackmd.io/_uploads/SkkRkhCGxg.png)
> Ref: https://www.geeksforgeeks.org/5g-network-architecture/

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

### SMF

### UPF

### AUSF

### UDM

### UDR

### NRF

### NEF

### NSSF

:::info
網路切片是 5G 基礎設施的一項全新功能，在部署多樣化網路服務和應用時，能夠帶來高度的部署靈活性和高效的資源利用率。邏輯端到端網路切片具有預先確定的功能、流量特性和服務等級協定 (SLA)。它包含滿足行動虛擬網路營運商 (MVNO) 或一組使用者需求所需的虛擬化資源，包括專用使用者平面功能 (UPF)、會話管理功能 (SMF) 和策略控制功能 (PCF)。 
服務用戶設備 (UE) 的存取和移動性管理功能 (AMF) 實例，對於 UE 所屬的所有網路切片而言都是通用的。網路切片的識別是透過單一網路切片選擇輔助資訊 (S-NSSAI) 識別碼完成的。網路切片實例選擇由第一個收到 UE 註冊請求的 AMF 觸發，該 AMF 從統一資料管理 (UDM) 元素中檢索允許的切片，然後從 NSSF 請求適當的網路切片實例。 
更多資訊可參考 [5G Identifier & Network Slicing](https://ithelp.ithome.com.tw/articles/10290629) 一文。
:::

### CHF

### PCF


## SBA (Service-Based Architecture)

以服務為導向架構（Service-Based Architecture）提供模組化的框架，使網路元件透過共通的方式協作（只要遵照標準的規範，即使是來自不同供應商的網路元件也能正確的溝通）。
在以服務為導向架構中，網路元件（生產者）會向其他網路元件（消費者）提供自身特有的服務，消費者可以透過生產者的特有介面存取需要的服務。

![image](https://hackmd.io/_uploads/S1gzYsLQel.png)
> Ref: https://www.researchgate.net/figure/G-Architecture-in-SBA-representation_fig2_328233142

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

