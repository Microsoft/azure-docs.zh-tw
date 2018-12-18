---
title: 在 Azure 中的 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和共用磁碟上安裝 SAP NetWeaver HA | Microsoft Docs
description: 了解如何在 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和共用磁碟上安裝 SAP NetWeaver HA。
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 6209bcb3-5b20-4845-aa10-1475c576659f
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d5d52653d68c6ebfca7e35a134da263eee99fd3e
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34657072"
---
# <a name="install-sap-netweaver-ha-on-a-windows-failover-cluster-and-shared-disk-for-an-sap-ascsscs-instance-in-azure"></a>在 Azure 中的 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和共用磁碟上安裝 SAP NetWeaver HA

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:http://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-ha-guide-8.9]:high-availability-guide.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:high-availability-guide.md#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md



[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-2005]:./media/virtual-machines-shared-sap-high-availability-guide/2005-wsfc-sap-ha-e2e-arch-template2-on-azure.png

[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png

[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/resource-group-overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md

本文說明針對為 SAP ASCS/SCS 執行個體進行叢集處理，如何使用 Windows Server 容錯移轉和叢集共用磁碟在 Azure 中安裝及設定高可用性 SAP 系統。

## <a name="prerequisites"></a>先決條件

在開始安裝之前，請檢閱這些文章：

* [架構手冊：使用叢集共用磁碟於 Windows 容錯移轉叢集上進行 SAP ASCS/SCS 執行個體叢集處理][sap-high-availability-guide-wsfc-shared-disk]

* [使用 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和共用磁碟，為 SAP HA 準備 Azure 基礎結構][sap-high-availability-infrastructure-wsfc-shared-disk]

由於設定視使用的 DBMS 系統而異，因此我們不會在本文中說明 DBMS 設定。 我們會假設 DBMS 在高可用性方面的疑慮已藉由不同 DBMS 廠商為 Azure 提供的功能支援而獲得解決。 範例包括適用於 SQL Server 的 Always On 或資料庫鏡像，以及適用於 Oracle 資料庫的 Oracle Data Guard。 在本文使用的案例中，我們不對 DBMS 加入更多保護。

當不同的 DBMS 服務與 Azure 中的叢集 SAP ASCS/SCS 組態互動時，沒有任何特殊的考量。

> [!NOTE]
> SAP NetWeaver ABAP 系統、Java 系統及 ABAP+Java 系統的安裝程序幾乎完全相同。 最顯著的差異在於 SAP ABAP 系統有一個 ASCS 執行個體。 SAP Java 系統有一個 SCS 執行個體。 SAP ABAP+Java 系統有一個 ASCS 執行個體和一個 SCS 執行個體在相同 Microsoft 容錯移轉叢集群組中執行。 我們會明確說明每個 SAP NetWeaver 安裝堆疊的所有安裝差異。 您可以假設所有其他部分都相同。  
>
>

## <a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a>使用高可用性 ASCS/SCS 執行個體安裝 SAP

> [!IMPORTANT]
> 請不要將分頁檔放在 SIOS DataKeeper 鏡像磁碟區上。 DataKeeper 不支援鏡像磁碟區。 您可以將分頁檔留在 Azure 虛擬機器的暫存磁碟機 D 中，此為預設值。 如果還不在此磁碟機中，請將 Windows 分頁檔移至 Azure 虛擬機器的磁碟機 D。
>
>

使用高可用性 ASCS/SCS 執行個體安裝 SAP 包含下列工作︰

* 建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱。
* 安裝 SAP 的第一個叢集節點。
* 修改 ASCS/SCS 執行個體的 SAP 設定檔。
* 新增探查連接埠。
* 開啟 Windows 防火牆探查連接埠。

### <a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a>建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱

1.  在「Windows DNS 管理員」中，為 ASCS/SCS 執行個體的虛擬主機名稱建立 DNS 項目。

  > [!IMPORTANT]
  > 您指派給 ASCS/SCS 執行個體之虛擬主機名稱的 IP 位址必須與指派給 Azure Load Balancer (\<SID\>-lb-ascs) 的 IP 位址相同。  
  >
  >

  虛擬 SAP ASCS/SCS 主機名稱 (pr1-ascs-sap) 的 IP 位址與 Azure Load Balancer (pr1-lb-ascs) 的 IP 位址相同。

  ![圖 1：定義 SAP ASCS/SCS 叢集虛擬名稱和 TCP/IP 位址的 DNS 項目][sap-ha-guide-figure-3046]

  _**圖 1：** 定義 SAP ASCS/SCS 叢集虛擬名稱和 TCP/IP 位址的 DNS 項目_

2.  若要定義指派給虛擬主機名稱的 IP 位址，選取 [DNS 管理員] > [網域]。

  ![圖 2：SAP ASCS/SCS 叢集設定的新虛擬名稱和 TCP/IP 位址][sap-ha-guide-figure-3047]

  _**圖 2：** SAP ASCS/SCS 叢集設定的新虛擬名稱和 TCP/IP 位址_

### <a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> 安裝 SAP 的第一個叢集節點

1.  在叢集節點 A 上執行第一個叢集節點選項。例如，在 pr1-ascs-0* 主機上。
2.  如果要為 Azure 內部負載平衡器保留預設連接埠，請選取：

  * **ABAP 系統**：**ASCS** 執行個體號碼 **00**
  * **Java 系統**：**SCS** 執行個體號碼 **01**
  * **ABAP+Java 系統**：**ASCS** 執行個體號碼 **00** 和 **SCS** 執行個體號碼 **01**

  如果 ABAP ASCS 執行個體要使用 00 以外的執行個體編號，且 Java SCS 執行個體使用 01 以外的執行個體編號，首先，變更 Azure 內部負載平衡器預設負載平衡規則。 如需詳細資訊，請參閱[變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則][sap-ha-guide-8.9]。

接下來幾個步驟並未在標準 SAP 安裝文件中說明。

> [!NOTE]
> SAP 安裝文件說明如何安裝第一個 ASCS/SCS 叢集節點。
>
>

### <a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> 修改 ASCS/SCS 執行個體的 SAP 設定檔

首先，新增新的設定檔參數。 此設定檔參數可避免 SAP 工作程序與加入佇列伺服器之間的連線在閒置時間太長時關閉。 我們在本文中的[在 SAP ASCS/SCS 執行個體的兩個叢集節點上都新增登錄項目][sap-ha-guide-8.11]中提到問題案例。 在該部分，我們也介紹了對一些基本 TCP/IP 連線參數所做的兩項變更。 在第二個步驟中，您必須設定讓加入佇列伺服器傳送 `keep_alive` 訊號，如此連線才不會達到 Azure 內部負載平衡器的閒置臨界值。

若要修改 ASCS/SCS 執行個體的 SAP 設定檔：

1.  將此設定檔參數新增至 SAP ASCS/SCS 執行個體設定檔：

  ```
  enque/encni/set_so_keepalive = true
  ```
  在我們的範例中，路徑為：

  `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`

  例如，新增到 SAP SCS 執行個體設定檔和對應的路徑：

  `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`

2.  若要套用變更，請重新啟動 SAP ASCS/SCS 執行個體。

### <a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> 新增探查連接埠

使用內部負載平衡器探查功能，讓整個叢集組態使用 Azure Load Balancer。 Azure 內部負載平衡器通常會在參與的虛擬機器之間，平均分配內送的工作負載。

 不過，這對某些叢集組態不會產生作用，因為只有一個執行個體是主動的。 另一個執行個體是被動的，而且不接受任何工作負載。 Azure 內部負載平衡器僅將工作指派給主動的執行個體時，探查功能才可提供協助。 使用探查功能，內部負載平衡器便能夠偵測哪些執行個體是主動的，然後只以該執行個體作為工作負載的目標。

若要新增探查連接埠︰

1.  執行下列 PowerShell 命令，檢查目前的 **ProbePort** 值：

  ```PowerShell
  $SAPSID = "PR1"     # SAP <SID>

  $SAPNetworkIPClusterName = "SAP $SAPSID IP"
  Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter
  ```

   在叢集組態中的其中一個虛擬機器中執行命令。

2.  定義探查連接埠。 預設探查連接埠號碼為 0。 在我們的範例中，我們會使用探查連接埠 62000。

  ![圖 3：叢集設定探查連接埠預設為 0][sap-ha-guide-figure-3048]

  _**圖 3：** 預設叢集設定探查連接埠為 0_

  SAP Azure Resource Manager 範本中已定義連接埠號碼。 您可以在 PowerShell 中指派連接埠號碼。

  若要針對 SAP \<SID\> IP 叢集資源設定新的 ProbePort 值，請執行下列 PowerShell 指令碼來更新您環境的 PowerShell 變數：

  ```PowerShell
  $SAPSID = "PR1"      # SAP <SID>
  $ProbePort = 62000   # ProbePort of the Azure internal load balancer

  Clear-Host
  $SAPClusterRoleName = "SAP $SAPSID"
  $SAPIPresourceName = "SAP $SAPSID IP"
  $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
  $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
  $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
  $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
  $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
  $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
  $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value

  $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }

  Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
  Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter

  Write-Host
  Write-Host "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." -ForegroundColor Cyan
  Write-Host
  Write-Host "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." -ForegroundColor Cyan
  Write-Host

  $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}

  Write-Host

  $ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"

  if($ActivateChanges -eq "yes"){
  Write-Host
  Write-Host "Activating changes..." -ForegroundColor Cyan

  Write-Host
  write-host "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..." -ForegroundColor Cyan
  Stop-ClusterResource -Name $SAPIPresourceName
  sleep 5

  Write-Host "Starting SAP cluster role '$SAPClusterRoleName' ..." -ForegroundColor Cyan
  Start-ClusterGroup -Name $SAPClusterRoleName

  Write-Host "New ProbePort parameter is active." -ForegroundColor Green
  Write-Host

  Write-Host "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" -ForegroundColor Cyan
  Write-Host
  Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
  }else
  {
  Write-Host "Changes are not activated."
  }
  ```

  讓 SAP \<SID\> 叢集角色上線之後，請確認 **ProbePort** 已設定為新值。

  ```PowerShell
  $SAPSID = "PR1"     # SAP <SID>

  $SAPNetworkIPClusterName = "SAP $SAPSID IP"
  Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter

  ```
  指令碼執行之後，系統會提示您重新啟動 SAP 叢集群組，以啟動變更。

  ![圖 4：設定新值之後，探查叢集連接埠][sap-ha-guide-figure-3049]

  _**圖 4：** 設定新值之後，探查叢集連接埠_

### <a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a> 開啟 Windows 防火牆探查連接埠

開啟這兩個叢集節點上的 Windows 防火牆探查連接埠。 使用下列指令碼開啟 Windows 防火牆探查連接埠。 請更新您環境的 PowerShell 變數。

  ```PowerShell
  $ProbePort = 62000   # ProbePort of the Azure internal load balancer

  New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

**ProbePort** 已設為 **62000**。 現在您可以從其他主機存取檔案共用 \\\ascsha-clsap\sapmnt，例如從 ascsha-dbas。

## <a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> 安裝資料庫執行個體

若要安裝資料庫執行個體，請依照 SAP 安裝文件中所述的程序。

## <a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> 安裝第二個叢集節點

若要安裝第二個叢集，請依照 SAP 安裝指南中所述的步驟。

## <a name="094bc895-31d4-4471-91cc-1513b64e406a"></a> 變更 SAP ERS Windows 服務執行個體的啟動類型

將兩個叢集節點上 SAP ERS Windows 服務的啟動類型都變更為 [自動 (延遲啟動)]。

![圖 5：將 SAP ERS 執行個體的服務類型變更為延遲的自動類型][sap-ha-guide-figure-3050]

_**圖 5：** 將 SAP ERS 執行個體的服務類型變更為延遲的自動類型_

## <a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> 安裝 SAP 主要應用程式伺服器

在您指派來裝載 PAS 的虛擬機器上安裝主要應用程式伺服器 (PAS) 執行個體 \<SID\>-di-0。 在 Azure 上沒有相依性。 沒有 DataKeeper 特定設定。

## <a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a> 安裝 SAP 其他應用程式伺服器

在您指派來裝載 SAP 應用程式伺服器執行個體的所有虛擬機器上安裝 SAP 其他應用程式伺服器 (AAS)。 例如，在 \<SID\>-di-1 到 \<SID\>-di-&lt;n&gt;。

> [!NOTE]
> 這樣就完成高可用性的 SAP NetWeaver 系統的安裝。 接下來，繼續進行容錯移轉測試。
>


## <a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> 測試 SAP ASCS/SCS 執行個體容錯移轉和 SIOS 複寫
使用「容錯移轉叢集管理員」和 SIOS DataKeeper 管理和組態工具，可輕鬆地測試及監視 SAP ASCS/SCS 執行個體容錯移轉和 SIOS 磁碟複寫。

### <a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a> SAP ASCS/SCS 執行個體在叢集節點 A 上執行

SAP PR1 叢集群組在叢集節點 A 上執行，例如，在 pr1-ascs-0 上。 將屬於 SAP PR1 叢集群組的共用磁碟 S 指派給叢集節點 A。ASCS/SCS 執行個體也使用磁碟 S。 

![圖 6：容錯移轉叢集管理員：SAP \<SID\> 叢集群組在叢集結點 A 上執行][sap-ha-guide-figure-5000]

_**圖 6：** 容錯移轉叢集管理員：SAP \<SID\> 叢集群組在叢集結點 A 上執行_

在 SIOS DataKeeper 管理和組態工具中，您可以看到共用磁碟資料以同步方式從叢集節點 A 上的來源磁碟區 S 複寫到叢集節點 B 上的目標磁碟區 S。例如，從 pr1-ascs-0 [10.0.0.40] 複寫到 pr1-ascs-1 [10.0.0.41]。

![圖 7：在 SIOS DataKeeper 中，將本機磁碟區從叢集節點 A 複寫到叢集節點 B][sap-ha-guide-figure-5001]

_**圖 7：** 在 SIOS DataKeeper 中，將本機磁碟區從叢集節點 A 複寫到叢集節點 B_

### <a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a> 從節點 A 到節點 B 進行容錯移轉

1.  使用下列其中一個選項以起始將 SAP \<SID\> 叢集群組從叢集節點 A 到叢集節點 B 的容錯移轉：
  - 容錯移轉叢集管理員  
  - 容錯移轉叢集 PowerShell

  ```PowerShell
  $SAPSID = "PR1"     # SAP <SID>

  $SAPClusterGroup = "SAP $SAPSID"
  Move-ClusterGroup -Name $SAPClusterGroup

  ```
2.  重新啟動 Windows 客體作業系統內的叢集節點 A。 這會起始將 SAP \<SID\> 叢集群組從節點 A 到節點 B 的自動容錯移轉。  
3.  從 Azure 入口網站重新啟動叢集節點 A。 這會起始將 SAP \<SID\> 叢集群組從節點 A 到節點 B 的自動容錯移轉。  
4.  使用 Azure PowerShell 重新啟動叢集節點 A。 這會起始將 SAP \<SID\> 叢集群組從節點 A 到節點 B 的自動容錯移轉。

  容錯移轉之後，SAP \<SID\> 叢集群組會在叢集節點 B 上執行。例如，在 pr1-ascs-1 上執行。

  ![圖 8：在「容錯移轉叢集管理員」中，SAP \<SID\> 叢集群組在叢集節點 B 上執行][sap-ha-guide-figure-5002]

  _**圖 8**在「容錯移轉叢集管理員」中，SAP \<SID\> 叢集群組在叢集節點 B 上執行_

  共用磁碟現在已掛接在叢集節點 B 上。SIOS DataKeeper 正在將資料從叢集節點 B 上的來源磁碟區 S 複寫到叢集節點 A 上的目標磁碟區 S。例如，從 pr1-ascs-1 [10.0.0.41] 複寫到 pr1-ascs-0 [10.0.0.40]。

  ![圖 9：SIOS DataKeeper 將本機磁碟區從叢集節點 B 複寫到叢集節點 A][sap-ha-guide-figure-5003]

  _**圖 9：** SIOS DataKeeper 將本機磁碟區從叢集節點 B 複寫到叢集節點 A_
