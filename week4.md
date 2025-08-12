# Week 4: 5G Procedures

## 課程目標

- 了解核心網路中的常見協定
- 介紹每個協定的使用情境，並且以程式碼為輔助進行解說
- 學習使用 Wireshark 分析網路問題

## UERANSIM

### nr-cli

nr-cli 為 UERANSIM 提供的 CLI tool，供使用者控制 UERANSIM 與核心網路互動。

#### 0. General

顯示所有裝置 (UE & gNb)

```shell
$ nr-cli --dump
```

針對指定裝置執行命令

```shell
nr-cli <node-name> [option...]

Options

  -d, --dump            List all UE and gNBs in the environment
  -e, --exec <command>  Execute the given command directly without an interactive shell
  -h, --help            Show this help message and exit
  -v, --version         Show version information and exit
```

#### 1. gNb

相關操作如下：

```shell
amf-info | Show some status information about the given AMF
amf-list | List all AMFs associated with the gNB
info     | Show some information about the gNB
status   | Show some status information about the gNB
ue-count | Print the total number of UEs connected the this gNB
ue-list  | List all UEs associated with the gNB
```

#### 2. UE

相關操作如下：
```shell
info           | Show some information about the UE
status         | Show some status information about the UE
timers         | Dump current status of the timers in the UE
rls-state      | Show status information about RLS
coverage       | Dump available cells and PLMNs in the coverage
ps-establish   | Trigger a PDU session establishment procedure
ps-list        | List all PDU sessions
ps-release     | Trigger a PDU session release procedure
ps-release-all | Trigger PDU session release procedures for all active sessions
deregister     | Perform a de-registration by the UE
```

#### 3. Scenarios

*使用 UE 建立 PDU Session*

:::spoiler
在嘗試建立 PDU Session 之前，請先確定已經啟動 gNb：

```shell
$ cd ~/UERANSIM
$ build/nr-gnb -c config/free5gc-gnb.yaml
```

以及 UE：

```shell
$ cd ~/UERANSIM
$ sudo build/nr-ue -c config/free5gc-ue.yaml
```
:::

進入 UERANSIM/build：

```shell
$ cd UERANSIM/build
```

查詢已建立裝置（包含 gNb 以及 UE）：

```shell
$ ./nr-cli --dump
```

選擇已建立的 UE，進入互動模式：

```shell
$ ./nr-cli imsi-208930000000003
```

進入互動模式後，使用以下命令即可建立 PDU Session：

```shell
$ ps-establish IPv4 --sst 1 --sd 66051 --dnn internet
```

:::info
需要注意的是，這邊的 `--sd` 會將參數當作 10 進制解讀，所以在使用時必須將 `sd: 0x010203` 轉為 10 進制。
:::

*查詢已建立的 PDU Session*

```shell
$ ps-list
```

*釋放 PDU Session*

```shell
$ ps-release 1 #釋放指定的 Session ID
$ ps-release-all #釋放所有處於 active 狀態的 PDU Session
```

:::info
補充
如果不需要互動模式，可以利用 --exec 執行單一命令：

./nr-cli imsi-208930000000003 --exec "ps-establish IPv4 --sst 1 --sd 66051 --dnn internet"
./nr-cli imsi-208930000000003 --exec "ps-release-all"
./nr-cli imsi-208930000000003 --exec "ps-list"
:::

*查詢與指定 gNb 相關的 UE*

進入 UERANSIM/build：

```shell
$ cd UERANSIM/build
```

查詢已建立裝置（包含 gNb 以及 UE）：

```shell
$ ./nr-cli --dump
```

選擇已建立的 gNb，進入互動模式：

```shell
$ ./nr-cli UERANSIM-gnb-001-01-1
```

查詢已連線的 UE 總數

```shell
$ ue-count
```

查詢所有已連線的 UE

```shell
$ ue-list
```

*測試 downlink paging and service request 相關 command*

Establish additional PDU session

```shell
$ ./nr-cli imsi-208930000000003 -e "ps-establish IPv4 --sst 1 --sd 66051 --dnn internet"
```

Switch UE to CM-IDLE waiting for paging

```shell
$ ./nr-cli UERANSIM-gnb-208-93-1 -e "ue-release 1"
```

After UE is in the cm-idle, 

Service Request: 可以從 ue 端 ping upf 的 ip 來 trigger service request 的流程

```shell
$ ping <UPF_IP>
```
Paging: 可以從 free5gc (UPF) ping UE 的 ip

## References

- https://github.com/aligungr/UERANSIM/wiki/Usage
- https://github.com/aligungr/UERANSIM/discussions/325