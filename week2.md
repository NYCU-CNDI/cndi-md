# Week 2: 5G Network Concepts & Network Identifier

## 課程目標

- 從需求面出發了解網路切片
- 認識常見的 Identifier
- 認識 5G 核網的新功能（ULCL、Traffic Influence）

## 網路切片（Network Slicing）

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
PLMN Id 由 MCC 以及 MNC 組成，每個電信營運商都會有自己專屬的 PLMN。
以台灣這邊的業者來說，每個業者使用的 PLMN 都可以在 NCC 的[網站](https://www.ncc.gov.tw/chinese/files/14050/%E8%A1%8C%E5%8B%95%E7%B6%B2%E8%B7%AF%E8%AD%98%E5%88%A5%E7%A2%BC%E6%A0%B8%E9%85%8D%E7%8F%BE%E6%B3%81.pdf)上面找到。

#### VPLMN & HPLMN

V-PLMN（Visited PLMN）與 H-PLMN（Home PLMN），它們主要是用於漫遊的場景（你的電信商的 PLMN 是 H-PLMN，當地的網路提供商是 V-PLMN），在 3GPP TS 23.501 也可以看到漫遊的架構圖：
![](https://i.imgur.com/vheV36N.png)
*圖五：5G 漫遊架構圖*

以上圖來說，我們的手機（UE）會透過基地台（RAN）接入到當地營運商的核心網路，而當地營運商會透過 N32 Interface 取得門號對應的訂閱用戶資料（Subscriber Data）以及相關的策略資料（AM/SM Policy）。

#### MCC (Mobile Country Code)
MCC 長度為三碼，用來表示國家。

#### MNC (Mobile Network Code)
MNC 長度為二或三碼，用來表示不同的電信業者。

#### IMSI (International Mobile Subscriber Identity)
在 5G 系統當中又稱為 PEI，由 PLMN ID + MSIN 組成：
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

#### 5G-TSMI (5G Temporary Mobile Subscriber Identity)

5G-TMSI 是由 AMF 產生的，可以幫助我們識別 AMF 中的 UE。
如果是用於 paging，因為已知 AMF，可以只使用 5G-TMSI 提高傳輸效率。

#### SUPI (Subscription Permanent Identifier)
是 5G 用戶的永久身份，在 2G - 4G 採用的是 IMSI (International Mobile Subscriber Identity)。


#### SUCI (Subscription Concealed Identifier)
使用 PLMN 的 Public Key 對 SUPI 加密產生 SUCI。

#### 5G-GUTI
當 UE 向核心網路完成註冊時，核心網路中的 AMF 會為 UE 分配 5G-GUTI，5G-GUTI 是核心網路分配給 UE 的臨時識別證。
```
[GUAMI][5G-TSMI]
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


https://leozzmc.github.io/posts/a05f1769.html#SUPI