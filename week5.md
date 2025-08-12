# Week5: 5GC Deployment

free5GC 起初是為了打破 4G 時代所有軟硬體皆整合在特定硬體（Propriety Hardware）的窘境，特定硬體意味著市場壟斷，市場壟斷使研究機構、中小型企業難以取得核心網路、基地台這類通訊設備用於學術研究或商業化行為。

為了解決這個問題，Jyh-Cheng Chen 教授在 4G 時代就開始進行核心網路軟體化與虛擬化的研究，發表了 [Poster: RECO: A Reconfigurable Core Network for Future 5G Communication Systems](https://dl.acm.org/doi/10.1145/3117811.3131257)、[Service Level Virtualization (SLV): A Preliminary Implementation of 3GPP Service Based Architecture (SBA)](https://dl.acm.org/doi/abs/10.1145/3241539.3267749) 等研究。

![image](https://hackmd.io/_uploads/r1UmjxNUxx.png)
> RECO

![image](https://hackmd.io/_uploads/Hy4djx4Uel.png)
> SLV

Jyh-Cheng Chen 教授用獨到的遠見與在通訊網路多年的經驗帶領著國立交通大學的師生們在 2019 年 1 月發表了第一代的 free5GC（Non-Stand Alone 的 5G 核心網路），也是世界首套開源的 5G 核心網路專案，free5GC 更是在當年度 10 月正式支援 Stand Alone 5GC。

隨著時間的推演，free5GC 陸續地推出 3GPP 定義的新功能，包含但不限於：Convergent Charging、Non 3GPP Access、Traffic Influence、QoS support。
現在，free5GC 在Linux 基金會專案進行治理，主要貢獻者來自國立陽明交通大學 (NYCU)，其他貢獻者請參考 [CONTRIBUTORS.md](https://github.com/free5gc/governance/blob/main/CONTRIBUTORS.md)。

free5GC 的出現讓研究機構、中小型企業不再需要藉由數據模擬的方式，只要使用開源的基地台模擬器就能夠以低成本的方式實現他們的 PoC。