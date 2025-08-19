# Week 6: 5G Deployment - 1

## 課程目標

- 了解容器化技術的基本概念
- 學習 Docker 的基本操作與管理
- 掌握 Docker Compose 的使用方法
- 將 5G 核心網路元件容器化部署
- 理解容器化部署的優勢與挑戰

## 為什麼需要容器化？

在前面的課程中，我們學習了 5G 核心網路的各種網路元件以及它們之間的協作關係。當我們要在實際環境中部署這些網路元件時，會面臨許多挑戰：

### 傳統部署方式的問題

- **環境依賴複雜**：每個網路元件可能需要不同版本的執行環境、函式庫
- **部署一致性**：開發環境與正式環境的差異可能導致「在我的機器上可以跑」的問題
- **擴展困難**：當需要增加 UPF 實例以支援更多流量時，傳統方式需要手動配置多台機器
- **資源利用率低**：每個服務獨佔一台實體機器，造成資源浪費
- **維護成本高**：系統更新、安全修補需要逐台處理

### 容器化技術的優勢

正如我們在[認識核心網路](#認識核心網路)中提到的，營運商需要核心網路提供：
- **承載大量 UE 的能力**
- **動態擴展的能力** 
- **部署在分散式架構的能力**

容器化技術完美地解決了這些需求：

1. **環境一致性**：「Build once, run anywhere」
2. **快速部署**：秒級啟動，相比虛擬機器的分鐘級啟動
3. **資源效率**：共享主機核心，比虛擬機器更輕量
4. **易於擴展**：可以快速啟動多個相同的容器實例
5. **微服務架構支援**：每個網路元件可以獨立部署和更新

## Docker 基礎概念

### 什麼是 Docker？

Docker 是一個開源的容器化平台，它使用 Linux 核心功能（如 cgroups 和 namespaces）來實現應用程式的隔離執行。

```
傳統虛擬機器 vs 容器
┌─────────────────────┐    ┌─────────────────────┐
│    應用程式 A        │    │    應用程式 A        │
├─────────────────────┤    ├─────────────────────┤
│    Guest OS         │    │    容器執行環境      │
├─────────────────────┤    ├─────────────────────┤
│    Hypervisor       │    │    Docker Engine    │
├─────────────────────┤    ├─────────────────────┤
│    Host OS          │    │    Host OS          │
└─────────────────────┘    └─────────────────────┘
```

### 核心概念

#### Image（映像檔）
- **定義**：容器的模板，包含了應用程式執行所需的所有內容
- **特性**：唯讀、分層儲存、可版本控制
- **類比**：就像是 VM 的 ISO 檔案，但更輕量

#### Container（容器）
- **定義**：Image 的執行實例
- **特性**：可讀寫、隔離執行、可以啟動/停止/刪除
- **類比**：就像是從 ISO 檔案安裝並執行的虛擬機器

#### Dockerfile
- **定義**：定義如何建置 Image 的文字檔案
- **用途**：自動化建置過程，確保環境一致性

### Docker 基本操作

#### 1. 映像檔管理

```bash
# 拉取映像檔
docker pull ubuntu:20.04

# 列出本地映像檔
docker images

# 刪除映像檔
docker rmi ubuntu:20.04

# 建置映像檔
docker build -t my-app:latest .
```

#### 2. 容器管理

```bash
# 執行容器
docker run -it ubuntu:20.04 /bin/bash

# 在背景執行容器
docker run -d --name my-container nginx

# 列出執行中的容器
docker ps

# 列出所有容器（包含已停止的）
docker ps -a

# 停止容器
docker stop my-container

# 啟動已停止的容器
docker start my-container

# 進入執行中的容器
docker exec -it my-container /bin/bash

# 刪除容器
docker rm my-container
```

#### 3. 網路與儲存

```bash
# 建立自定義網路
docker network create my-network

# 使用自定義網路執行容器
docker run -d --name app --network my-network my-app

# 掛載本地目錄到容器
docker run -v /host/path:/container/path my-app

# 建立 Volume
docker volume create my-volume
docker run -v my-volume:/data my-app
```

### 撰寫 Dockerfile

以 free5GC 的 AMF 為例，一個基本的 Dockerfile 可能長這樣：

```dockerfile
# 使用官方 Golang 映像檔作為建置環境
FROM golang:1.21-alpine AS builder

# 設定工作目錄
WORKDIR /app

# 複製 go.mod 和 go.sum
COPY go.mod go.sum ./

# 下載依賴套件
RUN go mod download

# 複製原始碼
COPY . .

# 建置應用程式
RUN go build -o amf ./cmd/amf

# 使用輕量的 Alpine Linux 作為執行環境
FROM alpine:latest

# 安裝必要的工具
RUN apk --no-cache add ca-certificates

# 建立應用程式使用者
RUN adduser -D -s /bin/sh amf

# 複製建置好的執行檔
COPY --from=builder /app/amf /usr/local/bin/amf

# 複製配置檔案
COPY --from=builder /app/config /etc/free5gc/

# 切換到應用程式使用者
USER amf

# 暴露埠號
EXPOSE 8000

# 設定啟動命令
CMD ["amf", "-config", "/etc/free5gc/amfcfg.yaml"]
```

## Docker Compose 簡介

### 為什麼需要 Docker Compose？

當我們要部署完整的 5G 核心網路時，需要同時執行多個容器：
- AMF
- SMF  
- UPF
- AUSF
- UDM
- UDR
- NRF
- CHF
- NEF
- MongoDB（作為資料庫）

手動使用 `docker run` 命令來啟動這麼多容器會很麻煩，而且容器之間的網路配置和依賴關係也很複雜。Docker Compose 就是為了解決這個問題而生的。

### Docker Compose 的優勢

1. **宣告式配置**：使用 YAML 檔案定義整個應用程式的架構
2. **一鍵部署**：`docker-compose up` 就能啟動整個系統
3. **依賴管理**：自動處理容器間的啟動順序
4. **網路管理**：自動建立專用網路，容器間可以用服務名稱互相通訊
5. **環境隔離**：不同的專案可以有獨立的環境

### Docker Compose 基本操作

```bash
# 啟動所有服務
docker-compose up

# 在背景啟動所有服務
docker-compose up -d

# 檢視服務狀態
docker-compose ps

# 檢視服務日誌
docker-compose logs

# 檢視特定服務的日誌
docker-compose logs amf

# 停止所有服務
docker-compose stop

# 停止並刪除所有容器
docker-compose down

# 重新建置並啟動服務
docker-compose up --build

# 執行一次性命令
docker-compose exec amf /bin/sh
```

### 撰寫 docker-compose.yml

以下是一個簡化的 free5GC 部署範例：

```yaml
version: '3.8'

services:
  # 資料庫服務
  mongodb:
    image: mongo:5.0
    container_name: mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongodb_data:/data/db
    networks:
      - free5gc-net
    ports:
      - "27017:27017"

  # NRF - 服務註冊與發現
  nrf:
    build:
      context: ./nf_repository
      dockerfile: Dockerfile
    container_name: nrf
    environment:
      - DB_URI=mongodb://admin:password@mongodb:27017/free5gc?authSource=admin
    depends_on:
      - mongodb
    networks:
      - free5gc-net
    ports:
      - "8000:8000"

  # AMF - 存取與行動管理
  amf:
    build:
      context: ./access_mobility_mgmt
      dockerfile: Dockerfile
    container_name: amf
    environment:
      - NRF_URI=http://nrf:8000
    depends_on:
      - nrf
    networks:
      - free5gc-net
    ports:
      - "8001:8000"

  # SMF - 會話管理
  smf:
    build:
      context: ./session_mgmt
      dockerfile: Dockerfile
    container_name: smf
    environment:
      - NRF_URI=http://nrf:8000
    depends_on:
      - nrf
    networks:
      - free5gc-net
    ports:
      - "8002:8000"

  # UPF - 用戶平面
  upf:
    build:
      context: ./user_plane
      dockerfile: Dockerfile
    container_name: upf
    environment:
      - SMF_URI=http://smf:8000
    depends_on:
      - smf
    networks:
      - free5gc-net
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun
    ports:
      - "8805:8805/udp"

networks:
  free5gc-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16

volumes:
  mongodb_data:
```

### 重要配置說明

#### 1. 網路配置
```yaml
networks:
  free5gc-net:
    driver: bridge
```
- 建立專用網路，容器間可以使用服務名稱互相通訊
- 例如：AMF 可以透過 `http://nrf:8000` 存取 NRF 服務

#### 2. 依賴關係
```yaml
depends_on:
  - nrf
```
- 確保 NRF 先啟動，再啟動依賴它的服務

#### 3. 環境變數
```yaml
environment:
  - NRF_URI=http://nrf:8000
```
- 設定容器內的環境變數，應用程式可以讀取這些變數進行配置

#### 4. 持久化儲存
```yaml
volumes:
  - mongodb_data:/data/db
```
- 確保資料庫資料不會因為容器重啟而遺失

#### 5. 特殊權限
```yaml
cap_add:
  - NET_ADMIN
devices:
  - /dev/net/tun
```
- UPF 需要網路管理權限來建立 TUN/TAP 介面

## 實際部署 free5GC

### 1. 準備工作

首先確保系統已安裝 Docker 和 Docker Compose：

```bash
# 檢查 Docker 版本
docker --version

# 檢查 Docker Compose 版本  
docker-compose --version

# 確保 Docker 服務正在執行
sudo systemctl status docker
```

### 2. 設定組態檔案

每個網路元件都需要相應的配置檔案，以 AMF 為例：

```yaml
# amfcfg.yaml
info:
  version: 1.0.9
  description: AMF initial local configuration

configuration:
  amfName: AMF
  ngapIpList:
    - 172.20.0.10  # AMF 在容器網路中的 IP
  ngapPort: 38412
  sbi:
    scheme: http
    registerIPv4: amf  # 使用服務名稱註冊到 NRF
    bindingIPv4: 0.0.0.0  # 綁定到所有網路介面
    port: 8000
  serviceNameList:
    - namf-comm
    - namf-evts
    - namf-mt
    - namf-loc
    - namf-oam
  nrfUri: http://nrf:8000  # NRF 服務位址
```

### 3. 啟動核心網路

```bash
# 啟動所有服務
docker-compose up -d

# 檢查服務狀態
docker-compose ps

# 檢視 AMF 日誌
docker-compose logs -f amf
```

### 4. 驗證部署

```bash
# 檢查 NRF 中註冊的服務
curl http://localhost:8000/nnrf-nfm/v1/nf-instances

# 檢查 AMF 狀態
curl http://localhost:8001/namf-comm/v1/status
```

## 容器化部署的優勢

### 1. 開發與正式環境一致性
```bash
# 開發環境
docker-compose -f docker-compose.dev.yml up

# 正式環境  
docker-compose -f docker-compose.prod.yml up
```

### 2. 輕鬆的水平擴展
```yaml
# 擴展 UPF 實例
upf:
  # ... 其他配置
  deploy:
    replicas: 3  # 啟動 3 個 UPF 實例
```

### 3. 版本管理與回滾
```bash
# 更新到新版本
docker-compose pull
docker-compose up -d

# 回滾到舊版本
docker tag myapp:latest myapp:backup
docker-compose down
docker-compose up -d
```

### 4. 監控與日誌集中化
```yaml
# 整合監控工具
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

## 生產環境部署考量

當我們要將 5G 核心網路部署到生產環境時，除了基本的容器化技術外，還需要考慮許多重要的議題。這些議題關係到系統的穩定性、安全性和可維護性。

### 資源管理

#### 記憶體管理

在生產環境中，5G 核心網路需要處理大量的 UE 連接和資料流量，記憶體管理變得非常重要：

```yaml
# docker-compose.yml 中設定記憶體限制
services:
  amf:
    # ... 其他配置
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
    # 設定重啟策略
    restart: unless-stopped
```

**OOM (Out of Memory) 問題排查**：
```bash
# 檢查容器的 exit code
docker inspect amf | grep -A 5 "State"

# 使用 dmesg 檢查系統日誌
dmesg | grep -i "killed process"

# 監控容器資源使用狀況
docker stats amf
```

**記憶體洩漏檢測**：
- 定期監控容器的記憶體使用量
- 使用 Prometheus + Grafana 建立監控儀表板
- 設定記憶體使用量警告閾值

#### 磁碟空間管理

5G 核心網路會產生大量的日誌和暫存檔案，需要定期清理以避免磁碟空間不足：

```bash
# 建立自動清理腳本
#!/bin/bash
# cleanup-docker.sh

# 清理無用的容器
docker container prune -f

# 清理無用的映像檔
docker image prune -a -f

# 清理無用的 volumes
docker volume prune -f

# 清理無用的網路
docker network prune -f

# 清理 build cache
docker builder prune -a -f
```

設定定期執行：
```bash
# 設定 crontab，每日凌晨 2 點執行清理
0 2 * * * /path/to/cleanup-docker.sh
```

### 安全性考量

#### 避免使用 root 使用者

```dockerfile
# 在 Dockerfile 中建立專用使用者
FROM golang:1.21-alpine AS builder
# ... 建置步驟

FROM alpine:latest

# 建立應用程式專用群組和使用者
RUN addgroup -g 1001 -S free5gc && \
    adduser -u 1001 -S free5gc -G free5gc

# 設定檔案權限
COPY --from=builder --chown=free5gc:free5gc /app/amf /usr/local/bin/amf
COPY --from=builder --chown=free5gc:free5gc /app/config /etc/free5gc/

# 切換到非特權使用者
USER free5gc

CMD ["amf", "-config", "/etc/free5gc/amfcfg.yaml"]
```

對於需要特殊權限的服務（如 UPF），使用 Linux capabilities：
```yaml
# docker-compose.yml
services:
  upf:
    # ... 其他配置
    cap_add:
      - NET_ADMIN       # 僅授予網路管理權限
      - NET_RAW         # 原始套接字權限
    cap_drop:
      - ALL             # 移除所有其他權限
    user: "1001:1001"   # 使用非 root 使用者
```

#### 映像檔安全

**固定版本號碼**：
```yaml
# 錯誤示範 - 不要使用 latest
mongodb:
  image: mongo:latest  # ❌ 避免使用

# 正確示範 - 使用特定版本
mongodb:
  image: mongo:5.0.15  # ✅ 使用固定版本
```

**映像檔掃描**：
```bash
# 使用 syft 掃描映像檔漏洞
syft mongo:5.0.15

# 使用 grype 掃描安全漏洞
grype mongo:5.0.15

# 整合到 CI/CD pipeline
docker build -t my-amf:v1.0.0 .
grype my-amf:v1.0.0 --fail-on medium
```

**使用精簡化映像檔**：
```dockerfile
# 使用 distroless 映像檔提升安全性
FROM gcr.io/distroless/base-debian11

COPY --from=builder /app/amf /amf
COPY --from=builder /app/config /config/

EXPOSE 8000
ENTRYPOINT ["/amf"]
```

#### Docker Hub Rate Limit 應對策略

```yaml
# 使用私有 registry 或設定登入認證
services:
  base-image-cache:
    image: registry:2
    ports:
      - "5000:5000"
    volumes:
      - registry_data:/var/lib/registry

# 在 CI/CD 中登入 Docker Hub
# docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
```

### 網路安全議題

#### 防火牆配置

Docker 會自動修改 iptables 規則，這可能與系統防火牆產生衝突：

```bash
# 檢查 Docker 相關的 iptables 規則
sudo iptables -L DOCKER-USER -n -v

# 為特定服務設定防火牆規則
sudo iptables -I DOCKER-USER -p tcp --dport 38412 -j ACCEPT  # AMF NGAP
sudo iptables -I DOCKER-USER -p udp --dport 8805 -j ACCEPT   # UPF PFCP
```

建立專用的防火牆規則：
```bash
#!/bin/bash
# docker-firewall.sh

# 允許內部網路通訊
iptables -I DOCKER-USER -s 172.20.0.0/16 -d 172.20.0.0/16 -j ACCEPT

# 僅允許必要的外部連接
iptables -I DOCKER-USER -p tcp --dport 8000 -j ACCEPT  # NRF
iptables -I DOCKER-USER -p tcp --dport 38412 -j ACCEPT # AMF NGAP

# 拒絕其他外部連接
iptables -I DOCKER-USER -j DROP
```

#### 網路隔離

```yaml
# 使用多個網路進行服務隔離
networks:
  # 前端網路 - 對外服務
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.21.0.0/16

  # 後端網路 - 內部服務
  backend:
    driver: bridge
    internal: true  # 內部網路，無法存取外部
    ipam:
      config:
        - subnet: 172.22.0.0/16

services:
  nrf:
    networks:
      - frontend
      - backend

  mongodb:
    networks:
      - backend  # 資料庫僅在內部網路
```

### 監控與日誌管理

#### 健康檢查

```yaml
services:
  amf:
    # ... 其他配置
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/namf-comm/v1/status"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
```

#### 日誌管理

```yaml
# 設定日誌輪替，避免日誌檔過大
logging:
  driver: "json-file"
  options:
    max-size: "100m"
    max-file: "5"
    
# 或使用 syslog 驅動
logging:
  driver: "syslog"
  options:
    syslog-address: "tcp://localhost:514"
    tag: "5gc-{{.Name}}"
```

#### 效能監控

```yaml
# 整合 Prometheus 監控
services:
  prometheus:
    image: prom/prometheus:v2.40.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:9.3.0
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
```

### Dockerfile 最佳實踐

#### 多階段建置優化

```dockerfile
# 建置階段
FROM golang:1.21-alpine AS builder

# 使用 build cache 優化
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

# 分層複製，提高 cache 命中率
COPY cmd/ cmd/
COPY internal/ internal/
COPY pkg/ pkg/

# 建置優化
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-w -s" \
    -o amf ./cmd/amf

# 執行階段
FROM gcr.io/distroless/static:nonroot

# 複製必要檔案
COPY --from=builder /app/amf /amf
COPY --from=builder /app/config /config/

# 使用非 root 使用者
USER nonroot:nonroot

EXPOSE 8000
ENTRYPOINT ["/amf"]
```

#### .dockerignore 優化

```dockerignore
# .dockerignore
.git/
.gitignore
README.md
Dockerfile*
docker-compose*.yml

# 開發相關檔案
.vscode/
*.test
coverage.out

# 建置產物
bin/
dist/
```

### 高可用性部署

```yaml
# 實現服務的高可用性
services:
  amf:
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      placement:
        constraints:
          - node.role == worker
```

## 課堂/課後練習

### 基礎練習
- [ ] 安裝 Docker 和 Docker Compose，確保能正常執行基本命令
- [ ] 撰寫一個簡單的 Dockerfile，將 "Hello World" Go 程式容器化
- [ ] 使用 Docker Compose 部署 free5GC，並確保所有網路元件能正常註冊到 NRF

### 進階練習  
- [ ] 修改 docker-compose.yml，使 UE Simulator 也能透過容器執行
- [ ] 實作容器的健康檢查機制，確保服務異常時能自動重啟
- [ ] 撰寫安全強化的 Dockerfile，使用非 root 使用者執行應用程式

### 生產環境練習
- [ ] 建立自動清理 Docker 資源的腳本，並設定定期執行
- [ ] 使用 syft 或 grype 掃描映像檔的安全漏洞
- [ ] 設定 Docker 容器的資源限制（CPU、記憶體）
- [ ] 實作多網路隔離，將資料庫與對外服務分離
- [ ] 整合 Prometheus 監控系統，監控容器資源使用狀況
- [ ] 設定適當的日誌輪替策略，避免日誌檔佔用過多磁碟空間

### 挑戰練習
- [ ] 建立 CI/CD pipeline，整合映像檔安全掃描
- [ ] 實作 Docker Registry 快取，減少對 Docker Hub 的依賴
- [ ] 設定防火牆規則，確保只有必要的埠號對外開放
- [ ] 使用 distroless 映像檔重新建置 5G 網路元件

## 下週預告

下週我們將深入探討 Kubernetes 的使用，學習如何在生產環境中大規模部署和管理 5G 核心網路，包括：
- Kubernetes 基本概念與架構
- Pod、Service、Deployment 等核心物件
- 使用 Helm 管理複雜的應用程式部署
- 實現 5G 核心網路的高可用性與自動擴展

## Reference

### 官方文件
- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [Docker Network Security](https://docs.docker.com/network/packet-filtering-firewalls/)

### 5G 相關資源
- [free5GC Documentation](https://free5gc.org/)
- [5G Mobile Core Network: Design, Deployment, Automation, and Testing Strategies](https://www.tenlong.com.tw/products/9781484264720)

### 安全性工具
- [Syft - Container Image SBOM Generator](https://github.com/anchore/syft)
- [Grype - Container Image Vulnerability Scanner](https://github.com/anchore/grype)
- [Distroless Images](https://github.com/GoogleContainerTools/distroless)

### 生產環境最佳實踐
- [Docker Production Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Container Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [Docker Logging Best Practices](https://docs.docker.com/config/containers/logging/)

### 監控與觀測
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Docker Metrics Collection](https://docs.docker.com/config/containers/runmetrics/)
