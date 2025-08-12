# Syllabus

:::info
Syllabus: https://timetable.nycu.edu.tw/?r=main/crsoutline&Acy=114&Sem=1&CrsNo=535607&lang=en
:::

## 課程終極目標
- 讓學員有能力貢獻到開源的 5G 專案，不管是 5G CORE、5G RAN、5G Infra、5G Tools
- 有能力成為 free5GC 專案的 Commiter

## 課程進度

https://hackmd.io/@4GQQ/ry89gJjTd

| 週 | 主題 | 教材 | 說明 |
| -- | -------- | -------- | -------- |
| 1 | Syllabus + Introduction | -- | JC 授課 |
| 2 | 5G Network Functions     | [[1]](https://hackmd.io/@cndi2025/B1CuTo0Gll)     | 介紹 5G 核心網路與常見的網路元件。     |
| 3 | 5G Network Concepts & Network Identifier | [[1]](https://hackmd.io/@cndi2025/rkyMMm88xg) | 介紹核心網路常見功能的觀念，同時介紹常見的 network identifier。 |
| 4 | Protocol Stack | [[1]](https://hackmd.io/@cndi2025/S1BM-mULgg) | 介紹 5G 核心網路的各種通訊協定與用途。 |
| 5 | 5G Procedures |  | 使用 UERANSIM 觸發常用場景，並且使用 wireshark 解說。 |
| 6 | 5G Deployment - 1 | [[1]](https://hackmd.io/@cndi2025/H1XDHR28gx) [[2]](https://github.com/ianchen0119/introduction-to-k8s) | 使用 Docker 與 Kubernetes 部署核心網路，以及討論最佳實踐相關議題。 |
| 7 | 5G Deployment - 2 | [[1]](https://hackmd.io/@cndi2025/H1XDHR28gx) [[2]](https://github.com/ianchen0119/introduction-to-k8s) | 使用 Docker 與 Kubernetes 部署核心網路，以及討論最佳實踐相關議題。 |
| 8 | 5GC development - 1 |  |  |
| 9 | 5GC development - 2 |  |  |
| 10 | 5GC development - 3 |  |  |
| 11 | 5GC development - 4 |  |  |
| 12 | 5GC development - 5 |  |  |
| 13 | Use Case sharing |  |  |
| 14 | Use Case sharing |  |  |
| 15 | Final Project |  |  |
| 16 | Final Project |  |  |

## 參考教材

1. [5G Mobile Core Network: Design, Deployment, Automation, and Testing Strategies](https://www.tenlong.com.tw/products/9781484264720)
2. [EN 帶你入門 5G 核心網路](https://www.books.com.tw/products/0010970849?srsltid=AfmBOooPp8AyGCq4LX8M9ByPcjcSVHvmUsl8Q_N4xIW7C4j_dphDs7Y4)

:::info
- 參考教材 1 可以在交大圖書館借閱、或是透過圖書館系統下載電子版，不需要購買。
- 參考教材 2 可以在交大圖書館借閱，不需要購買。
:::

## 推薦教材/參考資料

1. [為你自己學 git](https://gitbook.tw/)

## 作業與專案

本課程會有 4 個專案需要大家完成：
- Project 1 (15%) ：free5GC CTF *2025-09-22 公布、2025-10-6(一) 23:59 繳交截止。*
- Project 2 (20%) ：Simple NF *2025-10-13 公布、2025-10-27(一) 23:59 繳交截止。*
- Project 3 (20%)：DNN Gateway
- Final Project (35%) *2025-10-20 公布、2025-11-10(一) 23:59 proposal 繳交截止、2025-12-7(日) 23:59 繳交截止。*
- 自我評量（10%）：自己的評分自己給！請說明你在課堂學到了什麼，並且說明為了給自己這個期望分數。我們在審查時會參考每堂課課後練習的答題狀況，以及期末專案的表現決定自我評量分數是否與表現相符。

### 注意事項
- 作業基本上會需要使用 Golang 進行開發，我們會在第八周開始探討核心網路的開發議題，為了避免時程安排與作業衝突，請從現在開始自學 Golang。
- 每次課堂中或課堂後都會提供練習題目（不計分），但如果提出 good first issue 會酌情加分。
- **嚴禁抄襲行為**