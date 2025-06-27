# Week 2: Protocol Stack

## 協定堆疊 (Protocol Stack)

在 5G 網路中，UE（使用者設備）與網路之間的通訊以及核心網路內部不同網路功能（NF）之間的通訊，都依賴於一組定義好的協定。這些協定共同構成了 5G 的協定堆疊。本週我們將探討幾個關鍵的協定，它們是實現 5G 各種服務的基礎。

---

### NGAP (Next-Generation Application Protocol)

NGAP 是在 5G 基地台（gNB）和 AMF（存取與行動管理功能）之間使用的控制平面協定。它取代了 4G 中的 S1AP 協定。

- **主要功能**:
    - **UE 上下文管理 (UE Context Management)**: 建立、修改和釋放 UE 在 AMF 和 gNB 中的上下文資訊。
    - **PDU 會話管理 (PDU Session Management)**: 管理 UE 的 PDU 會話，例如建立和釋放與 UPF 連接所需的資源。
    - **行動性管理 (Mobility Management)**: 處理 UE 在不同 gNB 之間移動時的切換（Handover）程序。
    - **傳輸 NAS 訊息**: 在 UE 和 AMF 之間透明地傳輸 NAS 訊息。

![NGAP Interface](https://www.researchgate.net/profile/Walter-Feess/publication/338822566/figure/fig2/AS:851882912841728@1580117925433/5G-System-control-plane-and-user-plane-protocol-stacks-The-control-plane-protocol-stack.jpg)
*圖：控制平面協定堆疊，顯示 NGAP 位於 gNB 與 AMF 之間*

---

### NAS (Non-Access Stratum)

NAS 是 UE 和 AMF 之間直接通訊的協定，它「透明地」穿過基地台（gNB）。"Non-Access Stratum" 的意思是它與存取技術（如無線電）無關。

- **主要功能**:
    - **註冊管理 (Registration Management)**: UE 開機後向網路註冊。
    - **連線管理 (Connection Management)**: 建立和釋放與網路的信令連線。
    - **行動性管理 (Mobility Management)**: 追蹤 UE 的位置（Tracking Area Update）。
    - **會話管理 (Session Management)**: 建立、修改和釋放 PDU 會話。

NAS 訊息分為兩大類：
1.  **5GMM (5G Mobility Management)**: 負責行動性相關程序。
2.  **5GSM (5G Session Management)**: 負責 PDU 會話相關程序。

---

### PFCP (Packet Forwarding Control Protocol)

PFCP 是 5G 核心網路中一個非常重要的協定，它在 SMF（控制平面）和 UPF（使用者平面）之間運作，實現了 CUPS（Control and User Plane Separation）架構。

- **主要功能**:
    - **建立/修改/刪除 PDU 會話**: SMF 透過 PFCP 協定，指示 UPF 如何處理特定 PDU 會話的數據流。
    - **安裝轉發規則 (Forwarding Rules)**: SMF 在 UPF 上安裝 Packet Detection Rules (PDRs)、Forwarding Action Rules (FARs) 等規則，告訴 UPF 如何識別、轉發、阻擋或緩存用戶數據。
    - **報告使用情況**: UPF 可以向 SMF 報告數據使用量，用於計費。

這種分離使得使用者平面（UPF）可以根據流量需求靈活地部署在網路邊緣，以減少延遲，而控制平面（SMF）則可以集中部署。

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

---

## 本週作業

1.  **問題**:
    - 請解釋 NGAP 和 NAS 協定之間的主要區別是什麼？為什麼需要這兩種不同的協定？
    - CUPS 架構為什麼對 5G 網路很重要？PFCP 在其中扮演了什麼角色？
    - 什麼是服務化架構 (SBA)？與傳統的點對點（Point-to-Point）介面相比，它有什麼優點？

2.  **實作**:
    - (可選) 使用 Wireshark 捕獲一個 5G 裝置的網路流量（如果條件允許），嘗試過濾並識別出 NAS 訊息。觀察註冊（Registration）或 PDU 會話建立（PDU Session Establishment）過程中的訊息交換。
