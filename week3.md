# Week 3: Protocol Stack

## 課程目標

- 了解核心網路中的常見協定
- 介紹每個協定的使用情境，並且以程式碼為輔助進行解說
- 學習使用 Wireshark 分析網路問題

## 協定堆疊 (Protocol Stack)

在 5G 網路中，UE（使用者設備）與網路之間的通訊以及核心網路內部不同網路功能（NF）之間的通訊，都依賴於一組定義好的協定。這些協定共同構成了 5G 的協定堆疊。本週我們將探討幾個關鍵的協定，它們是實現 5G 各種服務的基礎。

![image](https://hackmd.io/_uploads/H1bGXRcolg.png)

![image](https://hackmd.io/_uploads/HkjNQC9jlx.png)


:::info
- TS 24.501: NAS
- TS 29.244: PFCP
- TS 38.413 & TS 38.410: NGAP
- TS 29.281: GTPv1-U
:::

---

### NGAP (Next-Generation Application Protocol)

NGAP 是在 5G 基地台（gNB）和 AMF（存取與行動管理功能）之間使用的控制平面協定。它取代了 4G 中的 S1AP 協定。
NGAP 預設使用 port 38412。

- **主要功能**:
    - **Paging**：通知 RAN 尋找 UE。
    - **UE 上下文管理 (UE Context Management)**：建立、修改和釋放 UE 在 AMF 和 gNB 中的上下文資訊。
    - **PDU 會話管理 (PDU Session Management)**：管理 UE 的 PDU 會話，例如建立和釋放與 UPF 連接所需的資源。
    - **行動性管理 (Mobility Management)**：處理 UE 在不同 gNB 之間移動時的切換（Handover）程序。
    - **傳輸 NAS 訊息**：在 UE 和 AMF 之間透明地傳輸 NAS 訊息。
    - **NG Interface 管理**：管理 RAN 與 AMF 之間的 NGAP 連線。


![NGAP Interface](https://hackmd.io/_uploads/ry34RFPPgg.png)

*圖：控制平面協定堆疊，顯示 NGAP 位於 gNB 與 AMF 之間*

NGAP 提供的服務可以分成兩大類：

1\. Non UE-associated Service
這類的 NGAP Services 與 RAN 有關 (NGAP Signaling connection)，負責處理 RAN 與 AMF 之間的 N2 服務，像是：NG Setup、AMF Configuration Update、RAN Configuration Update。

2\. UE-associated services
這類的 NGAP Services 與 UE 有關，像是：Paging、Initial Context Setup、UE Context Release、Handover preparation/cancelation、PDU Session Resource Setup/Release、Uplink/Downlink NAS Transport 等流程，有關這類服務的 NGAP 訊息中會帶有 NAS Payload（裡面可能是 Registration、PDU Session Establishment、Deregistration、Service Request 等流程）。

此外，NGAP procedures 可再分為雙向回應的 class 1 以及單方面傳遞的 class 2：

![image](https://hackmd.io/_uploads/rktb0Xuogg.png)
*圖：NGAP Class 1 procedures，來源：TS 38.413*

![image](https://hackmd.io/_uploads/Syv4AQusgg.png)
*圖：NGAP Class 2 procedures，來源：TS 38.413*

![image](https://hackmd.io/_uploads/HJ0L_Sajel.png)
*圖：NGAP 處理流程，來源：https://www.researchgate.net/publication/356891201_Tutorial_on_communication_between_access_networks_and_the_5G_core*

- 第三步驟之前是 Non UE-associated
    - 這個 flow 最開始的時候 RAN 和 AMF 沒有任何連線，所以首先要先建立 NGAP 連線，這時後會進行 NG Setup 的流程，讓 AMF 和 RAN 能交換 NGAP application 層的資訊。 像是在 NGSetupRequest，RAN 端會提供它的 ID 、名字、Support 的 Tracking Area 給 AMF，Response 的時候 AMF 也會提供它自己相關的資訊給 RAN。
- 後面都是 UE-associated
    - 四以及五的 NGAP Message 用來交換 NAS 訊息，NAS PDU 就是帶 NAS Message 的 IE，AMFUENGAPID 和 RANUENGAPID 則是識別這個 NGAP Message 是屬於 UE 的 （因為這邊是 UE-associated services，所以需要一個區分不同 UE 的方法），RANUENGAPID 是 RAN 端給的，AMFUENGAPID 是 AMF 給的。
    - 三也可以帶 NAS 訊息，但不會包含 AMFUENGAPID，因為 AMFUENGAPID 是步驟四分配的，兩邊要完成 NGAP ID 的交換整個 UE-associated NGAP connection 才算建立完成。
    - 6a & 8a：當 AMF 和 UE 開始交換一些資訊之後，AMF 可能也會視情況需要去跟 RAN 更新 UE 相關的 Context，這邊 AMF 就會透過 Initial Context Setup Request 去通知 NGAP，NGAP 收到後會把 UE Context 存起來然後回 Response 給 AMF。此外，要注意 AMFUENGAPID 和 RANUENGAPID 在這邊都是 mandatory IE（因為是 UE-associated NGAP Service），然後因為是第一次要 Setup UE Context，像是 UE 的 Security Capability 或是 AMF 這邊算出來的 Security Key 都是必須要提供的資訊。

:::spoiler
![image](https://hackmd.io/_uploads/ryDytB6jxg.png)
*圖：Initial UE Message，來源：TS 38.413*

![image](https://hackmd.io/_uploads/ryrz5r6ixl.png)
*圖：UE Context Management Procedures，來源：TS 38.413*

![image](https://hackmd.io/_uploads/HJKu9Baoxe.png)
*圖：PDU Session Management Procedures，來源：TS 38.413*
:::

---

### NAS (Non-Access Stratum)

NAS 是 UE 和 AMF 之間直接通訊的協定，它「透明地」穿過基地台（gNB）。"Non-Access Stratum" 的意思是它與存取技術（如無線電）無關。

- **主要功能**:
    - **註冊管理 (Registration Management)**: UE 開機後向網路註冊。
    - **連線管理 (Connection Management)**: 建立和釋放與網路的信令連線。
    - **行動性管理 (Mobility Management)**: 追蹤 UE 的位置（Tracking Area Update）。
    - **會話管理 (Session Management)**: 建立、修改和釋放 PDU 會話。

NAS 訊息分為兩大類：
1.  **5GMM (5G Mobility Management)**: 負責行動性相關程序，另外也包含像是註冊、身分認證跟 Security 的進行。
2.  **5GSM (5G Session Management)**: 負責 PDU 會話相關程序。

![4](https://hackmd.io/_uploads/B1odRYPPge.png)


*圖：控制平面協定堆疊，顯示 NAS 位於 gNB、AMF 與 SMF 之間*

NAS Message Type 可分為 MM（Mobility Management）與 SM（Session Management）兩種，所有類型可參考：https://www.tech-invite.com/3m24/toc/tinv-3gpp-24-501_zu.html#e-9-7

![image](https://hackmd.io/_uploads/rJokaS6olg.png)
*圖：NAS 封包範例*

AMF 會先處理 NAS-MM 的訊息，確認 MM Context 與 Security Context 並且正確的解密 ciphered nas message 後，才能將 NAS-SM 交給 SMF 處理。

![image](https://hackmd.io/_uploads/SkH5pBasle.png)
*圖：NAS 封包與 Protocol Stack 對照圖*


![image](https://hackmd.io/_uploads/SkeT6rTsll.png)
*圖：Non-Access Stratum message flows，來源：https://www.researchgate.net/publication/356891201_Tutorial_on_communication_between_access_networks_and_the_5G_core*


AMF 中 NasMM 的狀態機：
```go
var transitions = fsm.Transitions{
	{Event: GmmMessageEvent, From: context.Deregistered, To: context.Deregistered},
	{Event: GmmMessageEvent, From: context.Authentication, To: context.Authentication},
	{Event: GmmMessageEvent, From: context.SecurityMode, To: context.SecurityMode},
	{Event: GmmMessageEvent, From: context.ContextSetup, To: context.ContextSetup},
	{Event: GmmMessageEvent, From: context.Registered, To: context.Registered},
	{Event: StartAuthEvent, From: context.Deregistered, To: context.Authentication},
	{Event: StartAuthEvent, From: context.Registered, To: context.Authentication},
	{Event: AuthRestartEvent, From: context.Authentication, To: context.Authentication},
	{Event: AuthSuccessEvent, From: context.Authentication, To: context.SecurityMode},
	{Event: AuthFailEvent, From: context.Authentication, To: context.Deregistered},
	{Event: AuthErrorEvent, From: context.Authentication, To: context.Deregistered},
	{Event: SecurityModeSuccessEvent, From: context.SecurityMode, To: context.ContextSetup},
	{Event: SecurityModeFailEvent, From: context.SecurityMode, To: context.Deregistered},
	{Event: ContextSetupSuccessEvent, From: context.ContextSetup, To: context.Registered},
	{Event: ContextSetupFailEvent, From: context.ContextSetup, To: context.Deregistered},
	{Event: InitDeregistrationEvent, From: context.Registered, To: context.DeregistrationInitiated},
	{Event: DeregistrationAcceptEvent, From: context.DeregistrationInitiated, To: context.Deregistered},
}

var callbacks = fsm.Callbacks{
	context.Deregistered:            DeRegistered,
	context.Authentication:          Authentication,
	context.SecurityMode:            SecurityMode,
	context.ContextSetup:            ContextSetup,
	context.Registered:              Registered,
	context.DeregistrationInitiated: DeregisteredInitiated,
}
```

---

### PFCP (Packet Forwarding Control Protocol)

PFCP 是 5G 核心網路中一個非常重要的協定，它在 SMF（控制平面）和 UPF（使用者平面）之間運作，實現了 CUPS（Control and User Plane Separation）架構。
PFCP 預設使用 8805 port。

- **主要功能**:
    - Session Level
        - **建立/修改/刪除 PDU 會話**: SMF 透過 PFCP 協定，指示 UPF 如何處理特定 PDU 會話的數據流。
        - **安裝轉發規則 (Forwarding Rules)**: SMF 在 UPF 上安裝 Packet Detection Rules (PDRs)、Forwarding Action Rules (FARs) 等規則，告訴 UPF 如何識別、轉發、阻擋或緩存用戶數據。
        - **報告使用情況**: UPF 可以向 SMF 報告數據使用量，用於計費。
    - Node Level
        - **管理 PFCP Node 之間的關聯**：PFCP Node 可以是 SMF 或是 UPF，PFCP Node 僅處理來自於與之建立關聯的 PFCP Node 的訊息。
        - **管理 PFCP Node 之間的狀態**：PFCP 提供 heartbeat 功能，用於偵測與之關聯的 PFCP 節點的工作狀態。此外，我們更可以透過 heartbeat 訊息之中的 timestamp IE 來檢測目標 PFCP 節點是否發生重啟，進一步判斷是否需要執行還原流程。

這種分離使得使用者平面（UPF）可以根據流量需求靈活地部署在網路邊緣，以減少延遲，而控制平面（SMF）則可以集中部署。

#### Node 管理

:::spoiler
參考 TS 29.244 可以得知：PFCP Node（即 SMF 或是 UPF 這類支援 PFCP 協定的網路元件），只會受理與之建立 Association 節點的訊息。
為此，TS 29.244 定義了一系列用於管理 PFCP Node 的流程：

![image](https://hackmd.io/_uploads/ryKO5avigx.png)


:::

#### Session 管理

:::spoiler
參考 TS 29.244 的 5.6.2 Session Endpoint Identifier Handling 一節：
> The PFCP endpoint locally assigns the SEID value the peer PFCP side
has to use when transmitting message. The SEID values are exchanged between PFCP endpoints using PFCP messages. 

可以得知 PFCP Node 之間用來識別 PFCP Session 的 SEID 是由對方分配的，假設現在有 SMF-1 與 UPF-1，並且 UE 註冊了 PDU Session 產生了 PFCP Session。那麼 SMF-1 會在發送 PFCP Session Establishment Request 時為 PFCP Session 分配一個 SEID，並將其帶入送往 UPF-1 的 PFCP Session Establishment Request，當 UPF-1 處理 PFCP Session Establishment Request 時會將 Remote SEID 儲存（SMF-1 分配的 SEID），同時分配 Local SEID（UPF-1 分配的 SEID）。
讓我們參考 UPF 的實作：
```go=
func (s *PfcpServer) handleSessionEstablishmentRequest(
	req *message.SessionEstablishmentRequest,
	addr net.Addr,
) {
    
    // ... 省略了一部分程式碼

	ies := make([]*ie.IE, 0)
	ies = append(ies, CreatedPDRList...)
	ies = append(ies,
		newIeNodeID(s.nodeID),
		ie.NewCause(ie.CauseRequestAccepted),
		ie.NewFSEID(sess.LocalID, v4, v6)) // UPF 分配的 SEID（Local SEID）

	rsp := message.NewSessionEstablishmentResponse(
		0,             // mp
		0,             // fo
		sess.RemoteID, // SMF 分配的 SEID（Remote SEID）
		req.Header.SequenceNumber,
		0, // pri
		ies...,
	)

	err = s.sendRspTo(rsp, addr)
	if err != nil {
		s.log.Errorln(err)
		return
	}
}
```

上方程式碼的 `rsp := message.NewSessionEstablishmentResponse()` 函式的第三個傳入參數會被放置於 PFCP message header，方便 SMF-1 處理 Session Establishment Response 時辨別該訊息屬於哪一個 PFCP Session。同時也儲存 UPF 分配的 SEID（參考 `ie.NewFSEID(sess.LocalID, v4, v6))`），後續有關於該 PFCP Session 的訊息需要發送至 UPF 時，就會將 UPF 分配的 SEID 放置於 PFCP message header。
:::

---


### GTP (GPRS Tunneling Protocol)

![](https://github.com/ianchen0119/cndi-md/blob/main/assets/2-3.png?raw=true)
*圖：參與 PDU Session 流程的網路元件關係，圖片來源：https://docs.magmaindia.org/Free5gc_5gCore/upf/upf.html*

GTP（GPRS Tunneling Protocol）是一種用於在移動網路中傳輸用戶數據的協定。它主要用於 4G 和 5G 網路中資料層的傳輸。
UPF 使用的是 GTP-U v1 協定，N3、N6 和 N9 接口皆使用 GTP-U v1 協定來傳輸用戶數據。GTP-U 協定允許在 UPF 和其他網路元件之間建立隧道，以便在不同的 PDU 會話之間轉發數據。
GTP-U 預設使用 2152 port。

![image](https://hackmd.io/_uploads/BJK0A65slg.png)
*圖：GTP 封包範例，來源：https://github.com/free5gc/free5gc/issues/101*

- T-PDU 表示該封包為 GPRS 隧道中 PDU 的 payload
- TEID（Tunnel Endpoint Identifier）用於識別封包屬於哪一個 GPRS tunnel，SMF 會為 UL/DL 個別分配一個 TEID，UL TEID 通過 NGAP 交給 RAN，UL/DL TEID 通過 PFCP 交給 UPF。
- 上行：UPF 在匹配 TEID 後，會通過 PDI 之中的 **UE IP** 與 **SDF** 找出符合條件的 PDR，使用找到的 PDR 對封包進行處理。
- 下行：通過封包的 DST 是否與 PDI 之中的 **UE IP** 相同 & **SDF** 找出 PDR，使用找到的 PDR 對封包進行處理。

![image](https://hackmd.io/_uploads/SJXBMCcsgg.png)
*圖：GTP 封包處理流程，來源：TS 29.244*


---

### SBI (Service-Based Interface)

5G 核心網路採用了服務化架構（Service-Based Architecture, SBA），其中每個網路功能（NF）都提供一組服務給其他 NF 使用。SBI 就是這些 NF 之間通訊的介面。

- **技術基礎**:
    - 基於 RESTful API，使用 HTTP/2 協定。
    - 使用 JSON (JavaScript Object Notation) 作為數據格式。
    - 每個 NF 都有一個 Profile，儲存在 NRF（NF Repository Function）中，包含了它提供的服務資訊。

- **運作方式**:
    1.  **NF 發現 (NF Discovery)**: 當一個 NF（例如 AMF）需要使用另一個 NF（例如 SMF）的服務時，它會先向 NRF 查詢，找到可用的 SMF 實例。
    2.  **服務請求 (Service Request)**: AMF 向 SMF 發送一個 HTTP/2 請求（如 POST, GET, PUT）來觸發一項服務（例如，建立 PDU 會話）。
    3.  **服務回應 (Service Response)**: SMF 處理請求後，回傳一個 HTTP/2 回應。

SBA 和 SBI 的設計大大提高了網路的靈活性、可擴展性和可維護性。

## 使用 Wireshark 分析封包

### SCTP (Stream Control Transmission Protocol)

SCTP 定義在 RFC 4960 中，是一種面向連接的協定，主要用於在 IP 網路上傳輸訊息。它支援多個流（Streams）和多宿主（Multi-homing），適合用於需要高可靠性和順序保證的應用。
- 不是使用資料報（datagrams）或區段（segments），SCTP 使用的是「區塊」（Chunks）。
- 區塊(Chunk)：SCTP 封包中的資訊單元。區塊可以包含用戶資料或 SCTP 控制資料。
- 多個區塊可以被打包在一個 SCTP 封包內，直到達到 MTU 大小為止，INIT、INIT ACK 和 SHUTDOWN COMPLETE 區塊除外。
- 端點(Endpoint)：SCTP 封包的邏輯發送者/接收者。在多宿主主機上，SCTP 端點對其對等端表現為一組可用的目的地傳輸位址（可發送 SCTP 封包的位址）和一組可用的來源傳輸位址（可接收 SCTP 封包的位址）的組合。
- SCTP 使用「串流」(stream) 作為傳送有序應用訊息的邏輯通道。串流是單向的。

![alt text](https://github.com/ianchen0119/cndi-md/blob/main/assets/2-1.png?raw=true)

- 多宿主（Multihoming）可以在 INIT ACK 階段建立。
- 這就是我們如何從基地台（Cell Site）到 AMF/MME 動態建立備援路徑。
- 備援路徑會啟動心跳（heart beat）機制。

#### 選擇性確認（Selective Ack's）

![image](https://hackmd.io/_uploads/rywn-S0iee.png)
![image](https://hackmd.io/_uploads/rkbT-B0oxl.png)




- 確認訊息會攜帶一方已接收到的所有傳輸序列號碼（Transmission Sequence Number, TSN）。
- 也就是說，有一個累積 TSN 確認值（Cumulative TSN Ack value），表示在接收端已成功重組的所有資料。
- 還有間隙區塊（Gap Blocks），用來指示哪些資料區塊段已到達，而中間有些資料區塊遺失。

#### 路徑監控（Path Monitoring）

![image](https://hackmd.io/_uploads/SJ5FWrRjee.png)
![image](https://hackmd.io/_uploads/BkH5bBRoxl.png)

- HEARTBEAT 區塊會在所有路徑上發送。每個 HEARTBEAT 區塊都必須由 HEARTBEAT-ACK 區塊進行確認。
- 每個路徑都被分配一個狀態：主動（active）或非主動（inactive）。
- 當心跳在特定時間內未被確認的事件數量，或重傳事件數量超過某個可配置的限制時，對等端點會被視為不可達，且關聯將透過 ABORT 區塊終止。


#### Init & Term

![image](https://hackmd.io/_uploads/rJOsoVtsxl.png)
*圖：SCTP 初始化流程，圖片來源：https://docs.oracle.com/cd/E80921_01/html/esbc_ecz740_configuration/GUID-E6214D44-39E0-4B00-A491-06A5194CB820.htm*

1. 客戶端向伺服器發送INIT訊號以啟動關聯。
2. 收到 INIT 訊號後，伺服器會向客戶端發送 INIT-ACK 回應。此 INIT-ACK 訊號包含一個狀態 Cookie。此狀態 Cookie 必須包含訊息認證碼 (MAC)、與 Cookie 建立時間對應的時間戳記、狀態 Cookie 的有效期限以及建立關聯所需的資訊。 MAC 由伺服器根據只有自己知道的金鑰計算得出。
![image](https://hackmd.io/_uploads/BkDoTNRilg.png)
3. 收到此 INIT-ACK 訊號後，用戶端會發送 COOKIE-ECHO 回應，該回應僅回應狀態 cookie。
![image](https://hackmd.io/_uploads/SJXaaNRsel.png)

4. 在使用金鑰驗證狀態 cookie 的真實性後，伺服器將為關聯分配資源，發送確認 COOKIE- ECHO 訊號的 COOKIE-ACK 回應，並將關聯移至 ESTABLISHED 狀態。

:::info
在 SCTP 中，狀態 cookie 是建立關聯時使用的安全機制，用於防止拒絕服務 (DoS) 攻擊，例如 SYN 洪水。 此 cookie 由伺服器透過INIT-ACK訊息傳送，包含訊息認證碼 (MAC)、時間戳記、有效期限以及關聯相關資料。 用戶端隨後會透過訊息將此 cookie 回顯給伺服器 COOKIE-ECHO，以便伺服器驗證並分配資源，而不是過早地將資源分配給攻擊者。 
:::

![image](https://hackmd.io/_uploads/B1b3i4Yieg.png)
*圖：SCTP 連線終止流程，圖片來源：https://docs.oracle.com/cd/E80921_01/html/esbc_ecz740_configuration/GUID-E6214D44-39E0-4B00-A491-06A5194CB820.htm*


1. 客戶端向伺服器發送 SHUTDOWN 訊號，告訴伺服器客戶端已準備好關閉連線。
2. 伺服器透過發送 SHUTDOWN-ACK 確認進行回應。
3. 然後客戶端向伺服器發送 SHUTDOWN-COMPLETE 訊號。

https://www.ibm.com/docs/en/aix/7.2.0?topic=protocol-sctp-association-startup-shutdown

### 使用 Wireshark 觀察 SBI 封包

![image](https://hackmd.io/_uploads/By2x40cjlx.png)
![image](https://hackmd.io/_uploads/SyZW4A9oel.png)

### 使用 Wireshark 觀察 NAS 封包

free5GC 預設選用 NEA0（null ciphering）來加密 NAS 封包。
如果在不更改 Wireshark 設定的情況下，我們無法看見 Plain NAS message。

進入 **Edit → Preferences → Protocols → NAS-5GS** 後，將 **Try to detect and decode 5G-EA0 ciphered messages** 勾選。

### 常用表達式

1. 表達 NOT：`not ...==...`
2. 表達 AND：`and`
3. 表達 OR：`or`
4. IP
    - `ip.addr`
    - `ip.src`
    - `ip.dst`
5. Protocols
    - `icmp`
    - `sctp`
    - `ngap`（NAS 也是看 ngap）
    - `gtp`
    - `pfcp`
6. Port
    - `tcp.srcport`
    - `tcp.port`
    - `tcp.dstport`

:::info
範例：
過濾出 TCP 端口不為 3389 & IP 位址不為 10.135.71.54 & 端口不為 80：`not tcp.port == 3389 and not ip.addr == 10.135.71.54 and not tcp.port == 80`
:::

### TCPDUMP

![image](https://hackmd.io/_uploads/BkOzPCqsgg.png)
*圖：TCPDUMP 常用命令，圖片來源：https://www.templateroller.com/template/2638761/tcpdump-cheat-sheet-jeremy-stretch.html#google_vignette*

### MacOS 多開 Wireshark

請參考：https://zhuanlan.zhihu.com/p/676586313

---

## 課堂練習

本週公告 Project 1，沒有額外的課堂練習。
