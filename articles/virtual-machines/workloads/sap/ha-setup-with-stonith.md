---
title: 使用 STONITH 為 SAP HANA on Azure (大型執行個體) 進行高可用性設定 | Microsoft Docs
description: 使用 STONITH 在 SUSE 中為 Azure 上的 SAP HANA (大型執行個體) 建立高可用性
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/21/2017
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 982c6112a19654e268c9c50fec35d65fbc1766c2
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37062015"
---
# <a name="high-availability-set-up-in-suse-using-the-stonith"></a>使用 STONITH 在 SUSE 中進行高可用性設定
本文件提供使用 STONITH 裝置在 SUSE 作業系統上進行高可用性設定的詳細逐步指示。

**免責聲明：***本指南為透過在可成功運作的 Microsoft HANA 大型執行個體環境中測試設定所衍生。由於 HANA 大型執行個體的 Microsoft 服務管理小組不支援作業系統，您可能需要連絡 SUSE 以針對作業系統層進行進一步疑難排解或釐清。Microsoft 服務管理小組會設定 STONITH 裝置並傾力支援，而且會參與對於 STONITH 裝置問題的疑難排解。*
## <a name="overview"></a>概觀
若要使用 SUSE 叢集進行高可用性設定，必須符合下列先決條件。
### <a name="pre-requisites"></a>先決條件
- 已佈建 HANA 大型執行個體
- 作業系統已註冊
- HANA 大型執行個體伺服器已連線到 SMT 伺服器以取得修補程式/套件
- 作業系統已安裝最新修補程式
- NTP (時間伺服器) 已設定
- 閱讀並了解有關 HA 設定的最新 SUSE 文件版本

### <a name="setup-details"></a>設定詳細資料
本指南使用下列設定：
- 作業系統：適用於 SAP 的 SLES 12 SP1
- HANA 大型執行個體：2xS192 (4 個插槽，2 TB)
- HANA 版本：HANA 2.0 SP1
- 伺服器名稱：sapprdhdb95 (node1) 和 sapprdhdb96 (node2)
- STONITH 裝置：iSCSI 型 STONITH 裝置
- 在其中一個 HANA 大型執行個體節點上設定 NTP

當您使用 HSR 設定 HANA 大型執行個體時，可以要求 Microsoft 服務管理小組設定 STONITH。 如果您是已佈建 HANA 大型執行個體的現有客戶，並需要為現有的刀鋒視窗設定 STONITH 裝置，您需要在服務要求表單 (SRF) 中向 Microsoft 服務管理小組提供下列資訊。 您可以透過技術支援專案經理或 Microsoft 連絡人索取 HANA 大型執行個體上架 SRF 表單。 新客戶可以在佈建期間要求設定 STONITH 裝置。 輸入可在佈建要求表單中取得。

- 伺服器名稱和伺服器 IP 位址 (例如 myhanaserver1，10.35.0.1)
- 位置 (例如美國東部)
- 客戶名稱 (例如 Microsoft)
- SID - HANA 系統識別碼 (例如 H11)

設定 STONITH 裝置之後，Microsoft 服務管理小組會提供 SBD 裝置名稱與 iSCSI 儲存裝置的 IP 位址，您可以用來設定 STONITH 設定。 

若要使用 STONITH 設定端對端 HA，必須依照下列步驟執行：

1.  識別 SBD 裝置
2.  初始化 SBD 裝置
3.  設定叢集
4.  設定 Softdog 監視程式
5.  將節點加入叢集
6.  驗證叢集
7.  設定叢集資源
8.  測試容錯移轉程序

## <a name="1---identify-the-sbd-device"></a>1. 識別 SBD 裝置
本節說明在 Microsoft 服務管理小組設定 STONITH 之後，如何為您的設定判斷 SBD 裝置。 **本節僅適用於現有客戶**。 如果您是新客戶，Microsoft 服務管理小組會提供 SBD 裝置名稱給您，而您可以略過本節。

1.1 將 */etc/iscsi/initiatorname.isci* 修改為 
``` 
iqn.1996-04.de.suse:01:<Tenant><Location><SID><NodeNumber> 
```

Microsoft 服務管理小組會提供此字串。 修改這**兩個**節點上的檔案，但是每個節點的節點編號不同。

![initiatorname.png](media/HowToHLI/HASetupWithStonith/initiatorname.png)

1.2 修改 */etc/iscsi/iscsid.conf*：設定 *node.session.timeo.replacement_timeout=5* 與 *node.startup = automatic*。 修改這**兩個**節點上的檔案。

1.3 執行探索命令，它會顯示四個工作階段。 請在這兩個節點上執行。

```
iscsiadm -m discovery -t st -p <IP address provided by Service Management>:3260
```

![iSCSIadmDiscovery.png](media/HowToHLI/HASetupWithStonith/iSCSIadmDiscovery.png)

1.4 執行命令以登入 iSCSI 裝置，它會顯示四個工作階段。 在這**兩個**節點上執行。

```
iscsiadm -m node -l
```
![iSCSIadmLogin.png](media/HowToHLI/HASetupWithStonith/iSCSIadmLogin.png)

1.5 執行重新掃描指令碼：*rescan-scsi-bus.sh*。此指令碼會顯示為您建立的新磁碟。  請在這兩個節點上執行。 您應會看到大於零的 LUN (例如 1、2 等)。

```
rescan-scsi-bus.sh
```
![rescanscsibus.png](media/HowToHLI/HASetupWithStonith/rescanscsibus.png)

1.6 執行命令 *fdisk –l* 以取得裝置名稱。 請在這兩個節點上執行。 挑選大小為 **178 MiB** 的裝置。

```
  fdisk –l
```

![fdisk-l.png](media/HowToHLI/HASetupWithStonith/fdisk-l.png)

## <a name="2---initialize-the-sbd-device"></a>2. 初始化 SBD 裝置

2.1 在這**兩個**節點上初始化 SBD 裝置

```
sbd -d <SBD Device Name> create
```
![sbdcreate.png](media/HowToHLI/HASetupWithStonith/sbdcreate.png)

2.2 檢查寫入至裝置的項目。 請在這**兩個**節點上執行

```
sbd -d <SBD Device Name> dump
```

## <a name="3---configuring-the-cluster"></a>3. 設定叢集
本節說明設定 SUSE HA 叢集的步驟。
### <a name="31-package-installation"></a>3.1 套件安裝
3.1.1   請確定已安裝 ha_sles 和 SAPHanaSR-doc 模式。 如果未安裝，請安裝它們。 在這**兩個**節點上安裝。
```
zypper in -t pattern ha_sles
zypper in SAPHanaSR SAPHanaSR-doc
```
![zypperpatternha_sles.png](media/HowToHLI/HASetupWithStonith/zypperpatternha_sles.png)
![zypperpatternSAPHANASR-doc.png](media/HowToHLI/HASetupWithStonith/zypperpatternSAPHANASR-doc.png)

### <a name="32-setting-up-the-cluster"></a>3.2 設定叢集
3.2.1   您可以使用 *ha-cluster-init* 命令或使用 yast2 精靈來設定叢集。 在此案例中，使用 yast2 精靈。 您**只能在主要節點上**執行這個步驟。

依序選取 [yast2] > [High Availability] > [Cluster] \([yast2] > [ 高可用性] > [叢集]\) ![yast-control-center.png](media/HowToHLI/HASetupWithStonith/yast-control-center.png)
![yast-hawk-install.png](media/HowToHLI/HASetupWithStonith/yast-hawk-install.png)

因為已經安裝 halk2 套件，請按一下 [取消]。

![yast-hawk-continue.png](media/HowToHLI/HASetupWithStonith/yast-hawk-continue.png)

按一下 [繼續]

預期值=已部署的節點數 (在此案例中為 2) ![yast-Cluster-Security.png](media/HowToHLI/HASetupWithStonith/yast-Cluster-Security.png) 按一下 [下一步]
![yast-cluster-configure-csync2.png](media/HowToHLI/HASetupWithStonith/yast-cluster-configure-csync2.png) 新增節點名稱，然後按一下 [Add suggested files] \(新增建議的檔案\)

按一下 [Turn csync2 ON] \(開啟 csync2\)

按一下 [Generate Pre-Shared-Keys] \(產生預先共用金鑰\)，隨即顯示下面的快顯視窗

![yast-key-file.png](media/HowToHLI/HASetupWithStonith/yast-key-file.png)

按一下 [檔案] &gt; [新增] &gt; [專案] 

驗證會使用 IP 位址和 Csync2 中的預先共用金鑰執行。 金鑰檔案是使用 csync2 -k /etc/csync2/key_hagroup 產生。 檔案 key_hagroup 應在建立之後手動複製到叢集的所有成員。 **務必將檔案從 node1 複製到 node2**。

![yast-cluster-conntrackd.png](media/HowToHLI/HASetupWithStonith/yast-cluster-conntrackd.png)

按一下 [下一步]
![yast-cluster-service.png](media/HowToHLI/HASetupWithStonith/yast-cluster-service.png)

在預設選項中，[Booting] \(開機\) 處於關閉狀態，請將它變更為 [on] \(開啟\)，以在開機時啟動 Pacemaker。 您可以根據自己的設定需求來選擇。
按一下 [下一步]，叢集設定就完成了。

## <a name="4---setting-up-the-softdog-watchdog"></a>4. 設定 Softdog 監視程式
本節說明監視程式 (softdog) 的設定。

4.1 在這**兩個**節點上將下行加入至 /etc/init.d/boot.local。
```
modprobe softdog
```
![modprobe-softdog.png](media/HowToHLI/HASetupWithStonith/modprobe-softdog.png)

4.2 更新這**兩個**節點上的檔案 */etc/sysconfig/sbd*，如下所示：
```
SBD_DEVICE="<SBD Device Name>"
```
![sbd-device.png](media/HowToHLI/HASetupWithStonith/sbd-device.png)

4.3 執行下列命令以在這**兩個**節點上載入核心模組
```
modprobe softdog
```
![modprobe-softdog-command.png](media/HowToHLI/HASetupWithStonith/modprobe-softdog-command.png)

4.4 檢查並確定 softdog 正在這**兩個**節點上執行，如下所示：
```
lsmod | grep dog
```
![lsmod-grep-dog.png](media/HowToHLI/HASetupWithStonith/lsmod-grep-dog.png)

4.5 在這**兩個**節點上啟動 SBD 裝置
```
/usr/share/sbd/sbd.sh start
```
![sbd-sh-start.png](media/HowToHLI/HASetupWithStonith/sbd-sh-start.png)

4.6 在這**兩個**節點上測試 SBD 精靈。 在這**兩個**節點上設定它之後，您會看到兩個項目
```
sbd -d <SBD Device Name> list
```
![sbd-list.png](media/HowToHLI/HASetupWithStonith/sbd-list.png)

4.7 將測試訊息傳送至其中**一個**節點
```
sbd  -d <SBD Device Name> message <node2> <message>
```
![sbd-list.png](media/HowToHLI/HASetupWithStonith/sbd-list.png)

4.8 在**第二個**節點 (node2) 上，您可以檢查訊息狀態
```
sbd  -d <SBD Device Name> list
```
![sbd-list-message.png](media/HowToHLI/HASetupWithStonith/sbd-list-message.png)

4.9 若要採用 sbd 設定，請更新檔案 */etc/sysconfig/sbd*，如下所示。 更新這**兩個**節點上的檔案
```
SBD_DEVICE=" <SBD Device Name>" 
SBD_WATCHDOG="yes" 
SBD_PACEMAKER="yes" 
SBD_STARTMODE="clean" 
SBD_OPTS=""
```
4.10    在**主要節點** (node1) 上，啟動 Pacemaker 服務
```
systemctl start pacemaker
```
![start-pacemaker.png](media/HowToHLI/HASetupWithStonith/start-pacemaker.png)

如果 Pacemaker 服務*失敗*，請參閱「案例 5：Pacemaker 服務失敗」

## <a name="5---joining-the-cluster"></a>5. 加入叢集
本節說明如何將節點加入叢集。

### <a name="51-add-the-node"></a>5.1 新增節點
在 **node2** 上執行下列命令，讓 node2 加入叢集。
```
ha-cluster-join
```
如果您在加入叢集期間收到*錯誤*，請參閱「案例 6：節點 2 無法加入叢集」。

## <a name="6---validating-the-cluster"></a>6. 驗證叢集

### <a name="61-start-the-cluster-service"></a>6.1 啟動叢集服務
檢查並選擇性地在第一次在這**兩者**節點上啟動叢集。
```
systemctl status pacemaker
systemctl start pacemaker
```
![systemctl-status-pacemaker.png](media/HowToHLI/HASetupWithStonith/systemctl-status-pacemaker.png)
### <a name="62-monitor-the-status"></a>6.2 監視狀態
執行命令 *crm_mon* 以確保這**兩個**節點已連線。 您可以在叢集的**任一節點**上執行它
```
crm_mon
```
![crm-mon.png](media/HowToHLI/HASetupWithStonith/crm-mon.png) 您也可以登入hawk 來檢查叢集狀態 *https://<node IP>:7630*。 預設使用者為 hacluster，而密碼為 linux。 如有需要，您可以使用 *passwd* 命令來變更密碼。

## <a name="7-configure-cluster-properties-and-resources"></a>7.設定叢集屬性及資源 
本節說明設定叢集資源的步驟。
在此範例中，設定下列資源，其他部分可以 (視需要) 參考 SUSE HA 指南來設定。 只需要在**其中一個節點**執行此設定。 請在主要節點上執行。

- 叢集啟動程序
- STONITH 裝置
- 虛擬 IP 位址


### <a name="71-cluster-bootstrap-and-more"></a>7.1 叢集啟動程序以及其他
新增叢集啟動程序。 建立檔案並新增文字如下：
```
sapprdhdb95:~ # vi crm-bs.txt
# enter the following to crm-bs.txt
property $id="cib-bootstrap-options" \
no-quorum-policy="ignore" \
stonith-enabled="true" \
stonith-action="reboot" \
stonith-timeout="150s"
rsc_defaults $id="rsc-options" \
resource-stickiness="1000" \
migration-threshold="5000"
op_defaults $id="op-options" \
timeout="600"
```
將設定新增至叢集。
```
crm configure load update crm-bs.txt
```
![crm-configure-crmbs.png](media/HowToHLI/HASetupWithStonith/crm-configure-crmbs.png)

### <a name="72-stonith-device"></a>7.2 STONITH 裝置
新增資源 STONITH。 建立檔案並新增文字如下。
```
# vi crm-sbd.txt
# enter the following to crm-sbd.txt
primitive stonith-sbd stonith:external/sbd \
params pcmk_delay_max="15" \
op monitor interval="15" timeout="15"
```
將設定新增至叢集。
```
crm configure load update crm-sbd.txt
```

### <a name="73-the-virtual-ip-address"></a>7.3 虛擬 IP 位址
新增資源虛擬 IP。 建立檔案並新增文字如下。
```
# vi crm-vip.txt
primitive rsc_ip_HA1_HDB10 ocf:heartbeat:IPaddr2 \
operations $id="rsc_ip_HA1_HDB10-operations" \
op monitor interval="10s" timeout="20s" \
params ip="10.35.0.197"
```
將設定新增至叢集。
```
crm configure load update crm-vip.txt
```

### <a name="74-validate-the-resources"></a>7.4 驗證資源

當您執行命令 *crm_mon* 時，您可以看到有兩個資源。
![crm_mon_command.png](media/HowToHLI/HASetupWithStonith/crm_mon_command.png)

此外，您可以在 *https://<node IP address>:7630/cib/live/state* 查看狀態

![hawlk-status-page.png](media/HowToHLI/HASetupWithStonith/hawlk-status-page.png)

## <a name="8-testing-the-failover-process"></a>8.測試容錯移轉程序
若要測試容錯移轉程序，請停止 node1 上的 Pacemaker 服務，並將資源容錯移轉至 node2。
```
Service pacemaker stop
```
現在，停止 **node2** 上的 Pacemaker 服務，然後將資源容錯移轉至 **node1**

**容錯移轉之前**
![Before-failover.png](media/HowToHLI/HASetupWithStonith/Before-failover.png)
**容錯移轉之後**
![after-failover.png](media/HowToHLI/HASetupWithStonith/after-failover.png)
![crm-mon-after-failover.png](media/HowToHLI/HASetupWithStonith/crm-mon-after-failover.png)


## <a name="9-troubleshooting"></a>9.疑難排解
本節說明可能會在設定期間遇到的一些失敗案例。 您不一定會遇到這些問題。

### <a name="scenario-1-cluster-node-not-online"></a>案例 1：叢集節點未連線
如果任一節點未在叢集管理員中顯示為連線，您可以嘗試下列執行動作使其連線。

啟動 iSCSI 服務
```
service iscsid start
```

現在您應能夠登入該 iSCSI 節點
```
iscsiadm -m node -l
```
預期的輸出看起來如下
```
sapprdhdb45:~ # iscsiadm -m node -l
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.11,3260] (multiple)
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.12,3260] (multiple)
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.22,3260] (multiple)
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.21,3260] (multiple)
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.11,3260] successful.
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.12,3260] successful.
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.22,3260] successful.
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.21,3260] successful.
```
### <a name="scenario-2-yast2-does-not-show-graphical-view"></a>案例 2：yast2 未顯示圖形檢視
此文件使用 yast2 圖形畫面來設定高可用性叢集。 如果 yast2 並未如圖所示以圖形視窗開啟，並擲回 Qt 錯誤，請依照下列步驟執行。 如果它以圖形視窗開啟，您可以略過步驟。

**錯誤**

![yast2-qt-gui-error.png](media/HowToHLI/HASetupWithStonith/yast2-qt-gui-error.png)

**預期的輸出**

![yast-control-center.png](media/HowToHLI/HASetupWithStonith/yast-control-center.png)

如果 yast2 並未以圖形檢視開啟，請依照下列步驟執行。

安裝必要的套件。 您必須以使用者 “root” 的身分登入，並讓 SMT 安裝程式下載/安裝套件。

若要安裝套件，請使用 [yast] > [Software] > [Software Management] > [Dependencies] \([yast] > [軟體] > [軟體管理] > [相依項目]>\) 選項 [Install recommended packages…]\(安裝建議的套件\)。 下列螢幕擷取畫面顯示預期的畫面。
>[!NOTE]
>請務必在這兩個節點上執行所述步驟，才能從這兩個節點存取 yast2 圖形檢視。

![yast-sofwaremanagement.png](media/HowToHLI/HASetupWithStonith/yast-sofwaremanagement.png)

在 [Dependencies] \(相依項目\) 下，選取 [Install recommended packages…] \(安裝建議的套件\) ![yast-dependencies.png](media/HowToHLI/HASetupWithStonith/yast-dependencies.png)

檢閱所做的變更，然後按 [確定]

![yast](media/HowToHLI/HASetupWithStonith/yast-automatic-changes.png)

套件安裝繼續進行 ![yast-performing-installation.png](media/HowToHLI/HASetupWithStonith/yast-performing-installation.png)

按一下 [下一步]

![yast-installation-report.png](media/HowToHLI/HASetupWithStonith/yast-installation-report.png)

按一下 [完成]

您也需要安裝 libqt4 和 libyui qt 套件。
```
zypper -n install libqt4
```
![zypper-install-libqt4.png](media/HowToHLI/HASetupWithStonith/zypper-install-libqt4.png)
```
zypper -n install libyui-qt
```
![zypper-install-ligyui.png](media/HowToHLI/HASetupWithStonith/zypper-install-ligyui.png)
![zypper-install-ligyui_part2.png](media/HowToHLI/HASetupWithStonith/zypper-install-ligyui_part2.png) Yast2 現在應能夠開啟圖形檢視，如這裡所示。
![yast2-control-center.png](media/HowToHLI/HASetupWithStonith/yast2-control-center.png)

### <a name="scenario-3-yast2-does-not-high-availability-option"></a>案例 3：yast2 沒有高可用性選項
您需要安裝額外的套件，才能在 yast2 控制中心上顯示 [High Availability] \(高可用性\) 選項。

使用 [Yast2] > [Software] > [Software management] \([Yast2] > [軟體] > [軟體管理]\) 並選取下列模式

- SAP HANA 伺服器基底
- C/C++ 編譯器和工具
- 高可用性
- SAP 應用程式伺服器基底

下列畫面顯示安裝模式的步驟。

使用 [yast2] > [Software] > [Software management] \([yast2] > [軟體] > [軟體管理]\)

![yast2-control-center.png](media/HowToHLI/HASetupWithStonith/yast2-control-center.png)

選取模式

![yast-pattern1.png](media/HowToHLI/HASetupWithStonith/yast-pattern1.png)
![yast-pattern2.png](media/HowToHLI/HASetupWithStonith/yast-pattern2.png)

按一下 [接受]

![yast-changed-packages.png](media/HowToHLI/HASetupWithStonith/yast-changed-packages.png)

按一下 [繼續]

![yast2-performing-installation.png](media/HowToHLI/HASetupWithStonith/yast2-performing-installation.png)

安裝完成時，按 [下一步]

![yast2-installation-report.png](media/HowToHLI/HASetupWithStonith/yast2-installation-report.png)

### <a name="scenario-4-hana-installation-fails-with-gcc-assemblies-error"></a>案例 4：HANA 安裝失敗並顯示 gcc 組件錯誤
HANA 安裝失敗並顯示下列錯誤。

![Hana-installation-error.png](media/HowToHLI/HASetupWithStonith/Hana-installation-error.png)

若要修正此問題，您需要安裝程式庫 (libgcc_sl 和 libstdc++6)，如下所示。

![zypper-install-lib.png](media/HowToHLI/HASetupWithStonith/zypper-install-lib.png)

### <a name="scenario-5-pacemaker-service-fails"></a>案例 5：Pacemaker 服務失敗

Pacemaker 服務啟動期間，發生下列問題。

```
sapprdhdb95:/ # systemctl start pacemaker
A dependency job for pacemaker.service failed. See 'journalctl -xn' for details.
```
```
sapprdhdb95:/ # journalctl -xn
-- Logs begin at Thu 2017-09-28 09:28:14 EDT, end at Thu 2017-09-28 21:48:27 EDT. --
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync configuration map
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [QB    ] withdrawing server sockets
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync configuration ser
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [QB    ] withdrawing server sockets
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync cluster closed pr
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [QB    ] withdrawing server sockets
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync cluster quorum se
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync profile loading s
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [MAIN  ] Corosync Cluster Engine exiting normally
Sep 28 21:48:27 sapprdhdb95 systemd[1]: Dependency failed for Pacemaker High Availability Cluster Manager
-- Subject: Unit pacemaker.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit pacemaker.service has failed.
--
-- The result is dependency.
```
```
sapprdhdb95:/ # tail -f /var/log/messages
2017-09-28T18:44:29.675814-04:00 sapprdhdb95 corosync[57600]:   [QB    ] withdrawing server sockets
2017-09-28T18:44:29.676023-04:00 sapprdhdb95 corosync[57600]:   [SERV  ] Service engine unloaded: corosync cluster closed process group service v1.01
2017-09-28T18:44:29.725885-04:00 sapprdhdb95 corosync[57600]:   [QB    ] withdrawing server sockets
2017-09-28T18:44:29.726069-04:00 sapprdhdb95 corosync[57600]:   [SERV  ] Service engine unloaded: corosync cluster quorum service v0.1
2017-09-28T18:44:29.726164-04:00 sapprdhdb95 corosync[57600]:   [SERV  ] Service engine unloaded: corosync profile loading service
2017-09-28T18:44:29.776349-04:00 sapprdhdb95 corosync[57600]:   [MAIN  ] Corosync Cluster Engine exiting normally
2017-09-28T18:44:29.778177-04:00 sapprdhdb95 systemd[1]: Dependency failed for Pacemaker High Availability Cluster Manager.
2017-09-28T18:44:40.141030-04:00 sapprdhdb95 systemd[1]: [/usr/lib/systemd/system/fstrim.timer:8] Unknown lvalue 'Persistent' in section 'Timer'
2017-09-28T18:45:01.275038-04:00 sapprdhdb95 cron[57995]: pam_unix(crond:session): session opened for user root by (uid=0)
2017-09-28T18:45:01.308066-04:00 sapprdhdb95 CRON[57995]: pam_unix(crond:session): session closed for user root
```

若要修正此問題，請從檔案 */usr/lib/systemd/system/fstrim.timer* 刪除下行

```
Persistent=true
```

![Persistent.png](media/HowToHLI/HASetupWithStonith/Persistent.png)

### <a name="scenario-6-node-2-unable-to-join-the-cluster"></a>案例 6：節點 2 無法加入叢集

使用 *ha-cluster-join* 命令將 node2 加入現有叢集時，發生下列錯誤。

```
ERROR: Can’t retrieve SSH keys from <Primary Node>
```

![ha-cluster-join-error.png](media/HowToHLI/HASetupWithStonith/ha-cluster-join-error.png)

若要修正，請在這兩個節點上執行下列命令

```
ssh-keygen -q -f /root/.ssh/id_rsa -C 'Cluster Internal' -N ''
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
```

![ssh-keygen-node1.PNG](media/HowToHLI/HASetupWithStonith/ssh-keygen-node1.PNG)

![ssh-keygen-node2.PNG](media/HowToHLI/HASetupWithStonith/ssh-keygen-node2.PNG)

進行先前的修正之後，應可以將 node2 加入至叢集

![ha-cluster-join-fix.png](media/HowToHLI/HASetupWithStonith/ha-cluster-join-fix.png)

## <a name="10-general-documentation"></a>10.一般文件
您可以在下列文章中找到有關 SUSE HA 設定的詳細資訊： 

- [SAP HANA SR 效能最佳化案例](https://www.suse.com/docrep/documents/ir8w88iwu7/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf ) \(英文\)
- [以儲存體為基礎的隔離](https://www.suse.com/documentation/sle_ha/book_sleha/data/sec_ha_storage_protect_fencing.html) \(英文\)
- [部落格 - 針對 SAP HANA 使用 Pacemaker 叢集 - 第 1 部分](https://blogs.sap.com/2017/11/19/be-prepared-for-using-pacemaker-cluster-for-sap-hana-part-1-basics/) \(英文\)
- [部落格 - 針對 SAP HANA 使用 Pacemaker 叢集 - 第 2 部分](https://blogs.sap.com/2017/11/19/be-prepared-for-using-pacemaker-cluster-for-sap-hana-part-2-failure-of-both-nodes/) \(英文\)