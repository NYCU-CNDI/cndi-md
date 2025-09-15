# Week 2: 5G Network Concepts & Network Identifier

## 回顧：Week 1 課堂練習

- [x] 修改 Subscription Data，讓 UE 分配到我們預先設定的 IP（Static IP Pool）。
- [x] 試著修改 SMF 與 UPF 的組態（Configuration）檔案，使 UE 分配到的 IP 從 `10.60.0.0/16` 改為 `10.60.1.0/24`。


![image](https://hackmd.io/_uploads/r1gxZBljxx.png)

### Downlink

- UPF 根據 DL PDR 的 Packet Filter Set，按優先權對傳入封包進行分類。
- UPF 使用 QFI 進行 N3（和 N9）使用者平面標記，傳達屬於 QoS 流的使用者平面流量的分類。RAN 將 QoS Flow 綁定到 AN 資源（即 3GPP RAN 中的 Data Radio Bearers）。
- QoS Flow 和 AN 資源之間沒有嚴格的 1:1 關係。
- RAN 負責建立 QoS 流可對應的必要 AN 資源，並負責釋放這些資源。 - RAN 應向 SMF 指示何時釋放 QoS 流所對應到的 AN 資源。
- 如果未找到符合的 DL PDR，UPF 應丟棄該 DL 封包。

### Uplink

- 對於 IP 或乙太網路類型的 PDU Session，UE 會根據 QoS 規則中的 Packet Filter Set 的 UL packet filter，依照優先權從高到低的順序評估 UL 封包，直到找到相符的 QoS 規則（即，其封包過濾器與 UL 封包相符）。
- 如果未找到符合的 QoS 規則，則 UE 應丟棄該 UL 封包。


### 流程解說

1\. UPF 啟動，設定 [ip rule](https://man7.org/linux/man-pages/man8/ip-rule.8.html)，新增 virtual device `upfgtp`，同時建立 udp socket 與 `upfgtp` 綁定。

:::info
- upfcfg.yaml 設定的 N3 IP 用來建立 UDP Socket，這個動作直接決定了 UPF 的 N3 IP(s)。
- UPF N6 IP 則由 routing table 決定。
:::

```
# ip rule ls
0:      from all lookup local
500:    from 10.60.0.0/16 lookup 1200
500:    from 10.61.0.0/16 lookup 1200
1000:   from all lookup [l3mdev-table]
32766:  from all lookup main
32767:  from all lookup default

# ip r show table 1200
default via 10.10.2.1 dev n6 proto static metric 1 
10.10.2.0/24 dev n6 proto kernel scope link src 10.10.2.60 
local 10.10.2.60 dev n6 proto kernel scope host src 10.10.2.60 
broadcast 10.10.2.255 dev n6 proto kernel scope link src 10.10.2.60 
10.60.0.0/16 dev upfgtp proto static 
10.61.0.0/16 dev upfgtp proto static 
```

2\. SMF 處理 PDU Session 建立請求，成功後會告訴 **RAN** UPF 的 N3 IP。

:::info
- smfcfg.yaml 設定的 N3 IP 決定了 UPLINK GTP packets 的 Dst IP。
:::

![image](https://hackmd.io/_uploads/Syec0OEoex.png)
> GTP 封裝示意圖，圖片來源：https://www.sharetechnote.com/html/Handbook_LTE_GTP.html

3\. GTP5G 進行 GTP 解封裝

4\. GTP5G 進行 FIB (Forwarding Information Base) lookup

5\. GTP5G 進行 S-NAT

Src: https://docs.google.com/document/d/1D0Jzc7cL9qY6h8Jb5HGH9UgL1CuX-0smJsaUCZPWplo/edit?tab=t.0#heading=h.wwxnd5wzpy50

## 課程目標

- 從需求面出發了解網路切片
- 認識常見的 5GS Identifier
- 認識 5G 核網的新功能（ULCL、Traffic Influence）

## 網路切片（Network Slicing）

營運商可以藉由網路切片，在同一個公共陸地移動網路（Public Land Mobile Network，PLMN）中提供不同類型的邏輯網路以滿足不同客戶在個別場景中的需求。

![](https://lh3.googleusercontent.com/proxy/Xp1w9FybTH3KtR2ucEslBH_d8JtM9-BYOHcBREgOAdnX_LersVljVPfNMznA-N8ebvfAcPd8xid87EkPNJle5X_qRw)
*圖一：5G 的三大應用場景*

ITU 在 2015 年 9 月時正式定義了 5G 的三大應用場景：
- eMBB
- eMTC
- URLLC


為了因應複雜的使用場景，5G 網路在設計時導入了「網路切片」的概念，用白話文來說就是將一個實體網路切分成多個邏輯網路，為這些邏輯網路賦予不同的定位以及給予不同的資源，使 5G 網路有能力處理不同的業務需求。

![](https://www.tech-invite.com/3m23/img/tinv-23-003-28.4.2-1.gif)
*圖二：S-NSSAI，出處 [tech-invite](https://www.tech-invite.com/3m23/toc/tinv-3gpp-23-003_zi.html)。*

- SST
    - 1: eMMB
    - 2: URLLC
    - 3: eMTC
    - 5 ~ 127: TBD
    - 128 ~ 255: Operator specific
- SD (optional)


![](https://user-images.githubusercontent.com/42661015/179144604-4f5b1f17-1d46-4621-bec5-ce62df22ed24.png)
*圖三：網路切片架構*

3GPP 定義了 5G MANO（Management and Network Orchestration）相關規範，讓營運商能夠使用 TOSCA 樣板（Topology and Orchestration Specification for Cloud Applications，其規格描述在 ETSI GS NFV-SOL 001 規格書）來描述 5G 系統的部署。

TOSCA 主要由三個部分組成：
- The Virtualised Network Function Descriptor (VNFD)：用於描述 VNF 的部署和運行行為要求，並包含連接、介面和虛擬化資源要求。
- The Network Service Descriptor (NSD)：是一個模板文件，其參數遵循 ETSI MANO 規範，NFV 編排器 (NFVO) 使用它來部署網路服務（作為多個 VNF 的組合）。它包含 NFV 編排器 (NFVO) 用於 NS 生命週期管理的資訊。
- The Physical Network Function Descriptor (PNFD)：用於描述 Physical Network Function 的模板文件。

然而，TOSCA 只是由 3GPP 規範用來描述 Infra、Network、Service 的樣板，實際上我們仍需要使用支援 TOSCA 的管理平台，才能夠用標準定義的方式部署網路切片：

- free5GMano 部署 free5GC 的範例： 
    - https://github.com/free5gmano/kube5gnfvo/tree/for-K8S-1.2X/example/free5gcv3.2.1-cni-nodeport

- OpenStack [Tacker](https://wiki.openstack.org/wiki/Tacker) 部署 free5GC 的範例：
    - https://free5gc.org/blog/20230726/network_slice/?h=ope#mano-architecture
    - https://github.com/openstack/tacker/tree/master/samples/free5gc/cnf/sample_free5gc_cnf_package

:::spoiler
縮寫對照表：
- EMS: Element Management System
- TN: Transport Network
- Mgmt: Management
- CP: Control Plane
- UP: User Plane
- DN: Data Network
- OSS: Operations Support System
:::

![](https://user-images.githubusercontent.com/42661015/179154292-c631635b-5bd9-4a8d-852a-0a5bdd5e4ca8.png)

*圖四：參與網路切片之網路功能*

:::spoiler
NSSAI 為 S-NSSAI 的集合，可以再細分成 5 種:

- Default S-NSSAI
如果 UE 在 Registration Request 沒有攜帶 Allowed NSSAI，核心網路會使用 Default S-NSSAI 來為 UE 提供服務。

- Requested NSSAI
請求夾帶的 NSSAI，也就是 UE 在 Registration Request 攜帶的 Allowed NSSAI。

- Allowed NSSAI
表示 UE 請求的 NSSAI 中，哪些 S-NSSAI (切片功能) 被核心網路允許了，核心網路會利用 Registration Accept 之中的 Allowed NSSAI IE 將資訊帶給 UE。

- Rejected NSSAI
被拒絕的 NSSAI，表示 UE 請求的 NSSAI 中，哪些 S-NSSAI 被核心網路拒絕了，核心網路會利用 Registration Accept 之中的 Rejected NSSAI IE 將資訊帶給 UE。

- Configured NSSAI
核心網路配置給 UE 使用的 NSSAI，UE 會知道核心網路有哪些 S-NSSAI 可用。
核心網路會利用 Registration Accept 之中的 Configured NSSAI IE 將資訊帶給 UE。如果註冊後 UE 的配置有變化，則核心網路可通過 Configuration update command 通知 UE 更新。
:::




## 5GS 常見的 Identifier

### Global Identifier

#### PLMN (Public Land Mobile Network) ID

公用陸上行動網路（PLMN）是一電子通訊術語，為基於陸地、必須裝備無線電發射台與基地台的行動電話設施的總稱。

PLMN ID 由 MCC 以及 MNC 組成，每個電信營運商都會有自己專屬的 PLMN。
以台灣這邊的業者來說，每個業者使用的 PLMN 都可以在 NCC 的[網站](https://www.ncc.gov.tw/chinese/files/14050/%E8%A1%8C%E5%8B%95%E7%B6%B2%E8%B7%AF%E8%AD%98%E5%88%A5%E7%A2%BC%E6%A0%B8%E9%85%8D%E7%8F%BE%E6%B3%81.pdf)上面找到。

:::info

#### VPLMN & HPLMN

V-PLMN（Visited PLMN）與 H-PLMN（Home PLMN），它們主要是用於漫遊的場景（你的電信商的 PLMN 是 H-PLMN，當地的網路提供商是 V-PLMN），在 3GPP TS 23.501 也可以看到漫遊的架構圖：
![](https://i.imgur.com/vheV36N.png)
*圖五：5G 漫遊架構圖*

以上圖來說，我們的手機（UE）會透過基地台（RAN）接入到當地營運商的核心網路，而當地營運商會透過 N32 Interface 取得門號對應的訂閱用戶資料（Subscriber Data）以及相關的策略資料（AM/SM Policy）。

:::

#### MCC (Mobile Country Code)
MCC 長度為三碼，用來表示國家。

#### MNC (Mobile Network Code)
MNC 長度為二或三碼，用來表示不同的電信業者。

#### IMSI (International Mobile Subscriber Identity)
在 5G 系統當中又稱為 PEI（Permanent Equipment Identifier），由 PLMN ID + MSIN 組成：
```
[PLMN ID][MSIN]
```
其中，MSIN（Mobile Subscriber Identification Number）作為在一個 PLMN 下的識別號碼，每個訂閱用戶都有屬於自己的 MSIM。



### UE Identifier

#### GUAMI (Globally Unique AMF ID)
GUAMI 可以幫助我們識別全球的 AMF，每一個 AMF 持有的 GUAMI 都是獨一無二的。 
```
[MCC][MNC][AMF Region ID][AMF Set ID][AMF Pointer]
```
- MCC (Mobile Country Code)
- MNC (Mobile Network Code)
- AMF Region ID
- AMF Set ID
- AMF Pointer

#### 5G-TMSI (5G Temporary Mobile Subscriber Identity)

5G-TMSI 是由 AMF 產生的，可以幫助我們識別 AMF 中的 UE。
如果是用於 paging，因為已知 AMF，可以只使用 5G-TMSI 提高傳輸效率。

#### SUPI (Subscription Permanent Identifier)
是 5G 用戶的永久身份，在 2G - 4G 採用的是 IMSI (International Mobile Subscriber Identity)。


#### SUCI (Subscription Concealed Identifier)
使用 PLMN 的 Public Key 對 SUPI 加密產生 SUCI。

#### 5G-GUTI
當 UE 向核心網路完成註冊時，核心網路中的 AMF 會為 UE 分配 5G-GUTI，5G-GUTI 是核心網路分配給 UE 的臨時識別證。
```
[GUAMI][5G-TMSI]
```

:::spoiler
AMF 在執行時期需要記載許多 N1/N2 相關的上下文，如：
- AmfRan Context 紀錄 AMF 與 RAN 之間的狀態（NGAP），如：SupportedTAList、RanId、AnType、Connection。
- RanUe Context 紀錄 RAN 與 UE 之間的狀態，如：Handover Context（Type、T-GNB、S-GNB）、RAN-UE-NGAP-ID、AMF-UE-NGAP-ID、TAI、AmfRan、AmfUe
- AmfUe Context 紀錄 AMF 與 UE 之間的狀態（NAS），如：Security Context、Timer、UE Identifier、TimeZone、ServingNFs、SMContextList、RanUE Context

在 free5GC/amf 的 source code 當中可以看見 AMF 如何處理當 UE 使用 GUTI 註冊的請求：
```go
func handleInitialUEMessageMain(ran *context.AmfRan,
	message *ngapType.NGAPPDU,
	rANUENGAPID *ngapType.RANUENGAPID,
	nASPDU *ngapType.NASPDU,
	userLocationInformation *ngapType.UserLocationInformation,
	rRCEstablishmentCause *ngapType.RRCEstablishmentCause,
	fiveGSTMSI *ngapType.FiveGSTMSI,
	uEContextRequest *ngapType.UEContextRequest,
) {
    // ignore ...

    // If id type is GUTI, since MAC can't be checked here (no amfUe context), the GUTI may not direct to the right amfUe.
	// In this case, create a new amfUe to handle the following registration procedure.
	isInvalidGUTI := (idType == "5G-GUTI")
	amfUe, ok := findAmfUe(ran, id, idType)
	if ok && !isInvalidGUTI {
		// TODO: invoke Namf_Communication_UEContextTransfer if serving AMF has changed since
		// last Registration Request procedure
		// Described in TS 23.502 4.2.2.2.2 step 4 (without UDSF deployment)
		ranUe.Log.Infof("find AmfUe [%q:%q]", idType, id)
		ranUe.Log.Debugf("AmfUe Attach RanUe [RanUeNgapID: %d]", ranUe.RanUeNgapId)
		ranUe.HoldingAmfUe = amfUe
	}
```
:::

#### MSISDN

MSISDN（Mobile Subscriber ISDN Number）就是我們最常使用的手機門號。值得一提的是：對於核心網路來說，手機門號並不是必要的 IE（Information Element），在手機向核心網路進行註冊時，通常是使用 SUPI 或是核心網路分配的 5G-GUTI。


<!-- https://leozzmc.github.io/posts/a05f1769.html#SUPI -->


## ULCL (Uplink Classifier)

ULCL 是 5G 核心網路中的一個重要功能，它允許在 UPF（User Plane Function）中對上行流量進行分類和路由。這項技術主要用於實現本地分流（Local Breakout）和邊緣運算（Edge Computing）等應用場景。

### 主要功能

- **流量分類（Traffic Classification）**：根據預定義的規則（如目的地 IP、應用類型等）對上行流量進行分類
- **本地分流（Local Breakout）**：將特定流量直接從本地 UPF 分流到本地網路，而不需要回傳到中央的 UPF
- **邊緣運算支援**：支援 MEC（Multi-access Edge Computing）場景，降低延遲並提高服務品質

### 應用場景

1. **企業專網**：企業內部流量可以直接在本地處理，提高效率
2. **內容分發**：將熱門內容的請求分流到就近的 CDN 節點
3. **IoT 應用**：物聯網設備的數據可以在邊緣節點進行處理


![ULCL 架構示意圖](https://hackmd.io/_uploads/H1bAES6cll.png)
*圖六：ULCL 在 5G 網路中的部署架構，來源：https://vocus.cc/article/61c68837fd89780001cee23b*

### Session and Service Continuity (SSC)

![image](https://hackmd.io/_uploads/HyREtJXjgg.png)
*圖七：SSC Modes 比較圖，來源：http://4g5gworld.com/blog/session-and-service-continuity-evolution-5g-networks*

- SSC Mode 1：不管 UE 如何移動，當 PDU Session 建立後，該 PDU Session 選擇的 PDU Session Anchor UPF（PSA UPF）都不會改變。這使得 IP 也不會隨 UE 移動而發生變化，適合對 IP 改變敏感的服務，如語音或視訊服務。這類似傳統 4G 網路的作法。
- SSC Mode 2：放棄原本的 PSA UPF，並且向 UE 要求重新建立一個 PDU Session，適合 IP 可暫時丟棄的服務，如網頁瀏覽，Email，或 VR/AR 這種對頻寬有較高要求但可接受服務暫時中斷的服務。
- SSC Mode 3: 允許新的 PSA UPF 向同個 DN 建立 PDU Session，在釋放原來的 PSA UPF。適合 IP 可以改變，但不能容忍封包丟失的服務。這是用在像自動駕駛或物聯網這種對延遲跟可靠性有較高的要求的服務。

## Traffic Influence

Traffic Influence 是 5G 核心網路中用於最佳化流量路由和提升用戶體驗的功能。它允許應用服務提供商（ASP）或第三方應用程式影響用戶流量的路由決策。

![image](https://hackmd.io/_uploads/SJGAEHNill.png)
*圖八：Traffic Influence 流程圖，來源：TS 23.501*

這個功能對 5GC 與 MEC（Multi-access Edge Computing）的整合而言至關重要，MEC controller 能夠藉由 Traffic Influence 動態的控制 UE(s) 於 Data Plane 流量的路由。



![image](https://hackmd.io/_uploads/HJsoHVEjxe.png)
*圖九：OpenNESS 與 5GC 整合架構圖，來源：Intel smart-edge-open*

:::spoiler
CNCA: Core Network Configuration Agent
:::

以 Intel 維護的 OpenNESS（Open Network Edge Services Software）為例，OpenNESS 提供 Application Function，使用者能夠透過 UI/CLI 管理：
- Traffic Influence Subscription
- Packet Flow Description: A set of information enabling the detection of application traffic provided by a 3rd party service provider.

### 案例分析：free5GC-compose

- [5G 网络中 DNAI（Data Network Access Identifier）与 DNN 的区别和用途是什么？](https://www.zhihu.com/question/573896691/answer/3049417714)

```yaml=
# src: https://github.com/free5gc/free5gc-compose/blob/master/config/ULCL/smfcfg.yaml
info:
  version: 1.0.7
  description: SMF initial local configuration

configuration:
  smfName: SMF # the name of this SMF
  sbi: # Service-based interface information
    scheme: http # the protocol for sbi (http or https)
    registerIPv4: smf.free5gc.org # IP used to register to NRF
    bindingIPv4: smf.free5gc.org # IP used to bind the service
    port: 8000 # Port used to bind the service
    tls: # the local path of TLS key
      key: cert/smf.key # SMF TLS Certificate
      pem: cert/smf.pem # SMF TLS Private key
  serviceNameList: # the SBI services provided by this SMF, refer to TS 29.502
    - nsmf-pdusession # Nsmf_PDUSession service
    - nsmf-event-exposure # Nsmf_EventExposure service
    - nsmf-oam # OAM service
  snssaiInfos: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
      dnnInfos: # DNN information list
        - dnn: internet # Data Network Name
          dnaiList:
            - mec
          dns: # the IP address of DNS
            ipv4: 8.8.8.8
            ipv6: 2001:4860:4860::8888
  plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
  pfcp: # the IP address of N4 interface on this SMF (PFCP)
    # addr config is deprecated in smf config v1.0.3, please use the following config
    nodeID: smf.free5gc.org # the Node ID of this SMF
    listenAddr: smf.free5gc.org # the IP/FQDN of N4 interface on this SMF (PFCP)
    externalAddr: smf.free5gc.org # the IP/FQDN of N4 interface on this SMF (PFCP)
    heartbeatInterval: 5s
  userplaneInformation: # list of userplane information
    upNodes: # information of userplane node (AN or UPF)
      gNB1: # the name of the node
        type: AN # the type of the node (AN or UPF)
        nodeID: gnb.free5gc.org # the Node ID of this gNB
      I-UPF: # the name of the node
        type: UPF # the type of the node (AN or UPF)
        nodeID: i-upf.free5gc.org # the Node ID of this UPF
        sNssaiUpfInfos: # S-NSSAI information list for this UPF
          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
            dnnUpfInfoList: # DNN information list for this S-NSSAI
              - dnn: internet
                dnaiList:
                  - mec
        interfaces: # Interface list for this UPF
          - interfaceType: N3 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - i-upf.free5gc.org
            networkInstances: # Data Network Name (DNN)
              - internet
          - interfaceType: N9 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - i-upf.free5gc.org
            networkInstances: # Data Network Name (DNN)
              - internet
      PSA-UPF: # the name of the node
        type: UPF # the type of the node (AN or UPF)
        nodeID: psa-upf.free5gc.org # the Node ID of this UPF
        sNssaiUpfInfos: # S-NSSAI information list for this UPF
          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
            dnnUpfInfoList: # DNN information list for this S-NSSAI
              - dnn: internet
                pools:
                  - cidr: 10.60.0.0/16
        interfaces: # Interface list for this UPF
          - interfaceType: N9 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - psa-upf.free5gc.org
            networkInstances: # Data Network Name (DNN)
              - internet
    links: # the topology graph of userplane, A and B represent the two nodes of each link
      - A: gNB1
        B: I-UPF
      - A: I-UPF
        B: PSA-UPF
  # retransmission timer for pdu session modification command
  t3591:
    enable: true # true or false
    expireTime: 16s # default is 6 seconds
    maxRetryTimes: 3 # the max number of retransmission
  # retransmission timer for pdu session release command
  t3592:
    enable: true # true or false
    expireTime: 16s # default is 6 seconds
    maxRetryTimes: 3 # the max number of retransmission
  nrfUri: http://nrf.free5gc.org:8000 # a valid URI of NRF
  nrfCertPem: cert/nrf.pem # NRF Certificate
  urrPeriod: 10 # default usage report period in seconds
  urrThreshold: 1000 # default usage report threshold in bytes
  requestedUnit: 1000
  ulcl: true
logger: # log output setting
  enable: true # true or false
  level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
  reportCaller: false # enable the caller report or not, value: true or false
```

新增 TI Subs：

```
set -e

curl -X POST -H "Content-Type: application/json" --data @./nef_ti_anyUE.json \
	http://nef.free5gc.org:8000/3gpp-traffic-influence/v1/af001/subscriptions

exit 0
```

`nef_ti_anyUE.json` 的內容：
```
{
	"afServiceId": "Service1",
	"dnn": "internet",
	"snssai": {
		"sst": 1,
		"sd": "010203"
	},
	"anyUeInd": true,
	"notificationDestination": "http://af:8000/test123",
	"trafficFilters": [{
		"flowId": 1,
		"flowDescriptions": [
			"permit out ip from 1.0.0.1/32 to 10.60.0.0/16"
		]
	}],
	"trafficRoutes": [
		{
			"dnai": "mec"
		}
	]
}
```

## 課堂/課後練習

- [ ] 使用 free5GC 完成 ULCL 部署。
- [ ] 使用 UERANSIM 或 free-ran-ue 啟動兩個 UE，並且為這兩個 UE 配置不同的路由設定，藉此觀察 UE 的上行封包會如何被 UPF 處理。
    - [ ] UERANSIM版本：https://github.com/free5gc/free5gc-compose?tab=readme-ov-file#ulcl-configuration
    - [ ] free-ran-ue版本：https://alonza0314.github.io/free-ran-ue/doc-user-guide/11-docker-ulcl/


<!-- https://medium.com/@jessica.chchuang/5g%E4%BC%81%E6%A5%AD%E5%B0%88%E7%B6%B2%E7%B6%B2%E7%AE%A1%E8%88%87%E7%B6%AD%E9%81%8B%E5%B9%B3%E5%8F%B0%E4%B9%8B%E5%8A%9F%E8%83%BD%E8%88%87%E4%BB%8B%E9%9D%A2-9c1d14e06304 -->

<!-- https://learn.microsoft.com/en-us/azure/private-5g-core/ran-insights-create-resource -->