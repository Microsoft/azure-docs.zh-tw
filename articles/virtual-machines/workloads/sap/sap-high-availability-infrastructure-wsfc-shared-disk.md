---
title: 使用 SAP ASCS/SCS 的 Windows 容錯移轉叢集和共用磁碟，為 SAP HA 準備 Azure 基礎結構 | Microsoft Docs
description: 了解如何使用 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和共用磁碟，為 SAP HA 準備 Azure 基礎結構。
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ec976257-396b-42a0-8ea1-01c97f820fa6
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 223c038155d16f41f1599aa76081560739cd7095
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34657371"
---
# <a name="prepare-the-azure-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster-and-shared-disk-for-sap-ascsscs"></a>使用 SAP ASCS/SCS 的 Windows 容錯移轉叢集和共用磁碟，為 SAP HA 準備 Azure 基礎結構

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides
[tuning-failover-cluster-network-thresholds]:https://blogs.msdn.microsoft.com/clustering/2012/11/21/tuning-failover-cluster-network-thresholds/

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
[sap-ha-guide-9.1.1]:high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f


[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-shared-disk-vpn]:sap-high-availability-infrastructure-wsfc-shared-disk.md#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-high-availability-infrastructure-wsfc-shared-disk-change-def-ilb]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-wsfc]:sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-high-availability-infrastructure-wsfc-shared-disk-add-dot-net]:sap-high-availability-infrastructure-wsfc-shared-disk.md#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios-both-nodes]:sap-high-availability-infrastructure-wsfc-shared-disk.md#dd41d5a2-8083-415b-9878-839652812102
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (SAP 多重 SID 高可用性設定)

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png


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


> ![Windows][Logo_Windows] Windows
>

本文說明使用*叢集共用的磁碟*作為叢集 SAP ASCS 執行個體的選項，針對在 Windows 容錯移轉叢集上安裝及設定高可用性 SAP 系統，準備 Azure 基礎結構的步驟。

## <a name="prerequisites"></a>先決條件

在開始安裝之前，請檢閱這篇文章：

* [架構手冊：使用叢集共用磁碟於 Windows 容錯移轉叢集上進行 SAP ASCS/SCS 執行個體叢集處理][sap-high-availability-guide-wsfc-shared-disk]

## <a name="prepare-the-infrastructure-for-architectural-template-1"></a>準備架構範本 1 的基礎結構
適用於 SAP 的 Azure Resource Manager 範本有助於簡化部署所需的資源。

Azure Resource Manager 中的三層式範本也支援高可用性案例。 例如架構範本 1 有兩個叢集。 每個叢集是適用於 SAP ASCS/SCS 和 DBMS 的 SAP 單一失敗點。

以下是可取得我們在本文中說明之範例案例的 Azure Resource Manager 範本︰

* [Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image) \(英文\)  
* [使用 Azure 受控磁碟的 Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md) \(英文\)  
* [自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image)
* [使用受控磁碟的自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md) \(英文\)

若要準備架構範本 1 的基礎結構：

- 在 Azure 入口網站中，在 [參數] 窗格的 [SYSTEMAVAILABILITY] 方塊中，選取 [HA]。

  ![圖 1：設定 SAP 高可用性 Azure Resource Manager 參數][sap-ha-guide-figure-3000]

_**圖 1：** 設定 SAP 高可用性 Azure Resource Manager 參數_


  範本會建立：

  * **虛擬機器**：
    * SAP 應用程式伺服器虛擬機器︰\<SAPSystemSID\>-di-\<Number\>
    * ASCS/SCS 叢集虛擬機器︰\<SAPSystemSID\>-ascs-\<Number\>
    * DBMS 叢集：\<SAPSystemSID\>-db-\<Number\>

  * **所有虛擬機器的網路卡，包含關聯的 IP 位址**：
    * \<SAPSystemSID\>-nic-di-\<Number\>
    * \<SAPSystemSID\>-nic-ascs-\<Number\>
    * \<SAPSystemSID\>-nic-db-\<Number\>

  * **Azure 儲存體帳戶 (僅限非受控磁碟)**：

  * 下列項目的**可用性群組**：
    * SAP 應用程式伺服器虛擬機器︰\<SAPSystemSID\>-avset-di
    * SAP ASCS/SCS 叢集虛擬機器︰\<SAPSystemSID\>-avset-ascs
    * DBMS 叢集虛擬機器︰\<SAPSystemSID\>-avset-db

  * **Azure 內部負載平衡器**：
    * 包含 ASCS/SCS 執行個體的所有連接埠與 IP 位址 \<SAPSystemSID\>-lb-ascs
    * 包含 SQL Server DBMS 的所有連接埠與 IP 位址 \<SAPSystemSID\>-lb-db

  * **網路安全性群組**：\<SAPSystemSID\>-nsg-ascs-0  
    * 包含向 \<SAPSystemSID\>-ascs-0 虛擬機器開放的外部遠端桌面通訊協定 (RDP) 連接埠

> [!NOTE]
> 網路卡的所有 IP 位址和 Azure 內部負載平衡器依預設為動態。 將其變更為靜態 IP 位址。 本文稍後會描述做法。
>
>

## <a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a> 使用公司網路連線能力 (跨單位) 部署虛擬機器以用於生產環境中
針對生產環境 SAP 系統，使用 Azure VPN 閘道或 Azure ExpressRoute，部署具有 [公司網路連線能力 (跨單位)][planning-guide-2.2] 的 Azure 虛擬機器。

> [!NOTE]
> 您可以使用您的 Azure 虛擬網路執行個體。 虛擬網路和子網路已建立並備妥。
>
>

1.  在 Azure 入口網站中，在 [參數] 窗格的 [NEWOREXISTINGSUBNET] 方塊中，選取 [存在]。
2.  在 [SUBNETID] 方塊中，新增已備妥的 Azure 網路子網路識別碼的完整字串，這是您打算用來部署 Azure 虛擬機器的位置。
3.  若要取得所有 Azure 網路子網路的清單，請執行此 PowerShell 命令：

  ```PowerShell
  (Get-AzureRmVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
  ```

  [ID] 欄位顯示子網路識別碼的值。
4. 若要取得所有子網路識別碼值的清單，請執行此 PowerShell 命令：

  ```PowerShell
  (Get-AzureRmVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
  ```

  子網路識別碼看起來像這樣：

  ```
  /subscriptions/<subscription ID>/resourceGroups/<VPN name>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<subnet name>
  ```

## <a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a> 用於測試和示範的 SAP 執行個體僅限部署雲端
您可以在僅限雲端部署模型中部署您的高可用性 SAP 系統。 這種部署主要對於示範和測試的使用案例相當有用。 但不適用於生產環境使用案例。

- 在 Azure 入口網站中，在 [參數] 窗格的 [NEWOREXISTINGSUBNET] 方塊中，選取 [新增]。 將 [SUBNETID] 欄位保留空白。

  SAP Azure Resource Manager 範本將會自動建立 Azure 虛擬網路和子網路。

> [!NOTE]
> 您也需要在相同的 Azure 虛擬網路執行個體中，針對 Active Directory 和 DNS 服務部署至少一個專用的虛擬機器。 此範本不會建立這些虛擬機器。
>
>


## <a name="prepare-the-infrastructure-for-architectural-template-2"></a>準備架構範本 2 的基礎結構

您可以使用這個適用於 SAP 的 Azure Resource Manager 範本，以協助簡化部署必要的 SAP 架構範本 2 基礎結構資源。

您可以在這裡取得此部署案例的 Azure Resource Manager 範本：

* [Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged) \(英文\)  
* [使用受控磁碟的 Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md) \(英文\)  
* [自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged)
* [使用受控磁碟的自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md) \(英文\)


## <a name="prepare-the-infrastructure-for-architectural-template-3"></a>準備架構範本 3 的基礎結構

您可以準備基礎結構，並將 SAP 設定為多 SID。 例如，您可以將額外的 SAP ASCS/SCS 執行個體新增至現有的叢集組態以建立 SAP 多 SID 組態。 如需詳細資訊，請參閱[為現有的叢集組態設定額外的 SAP ASCS/SCS 執行個體以在 Azure Resource Manager 中建立 SAP 多 SID 組態][sap-ha-multi-sid-guide]。

如果您想要建立新的多 SID 叢集，可以使用多 SID [GitHub 上的快速入門範本](https://github.com/Azure/azure-quickstart-templates)。

若要建立新的多 SID 叢集，您必須部署下列三個範本：

* [ASCS/SCS 範本](#ASCS-SCS-template)
* [資料庫範本](#database-template)
* [應用程式伺服器範本](#application-servers-template)

下列各節有更多關於您必須在範本中提供之範本和參數的詳細資料。

### <a name="ASCS-SCS-template"></a> ASCS/SCS 範本

ASCS/SCS 範本會部署兩部虛擬機器，可讓您用來建立裝載多個 ASCS/SCS 執行個體的 Windows Server 容錯移轉叢集。

若要設定 ASCS/SCS 多 SID 範本，請在 [ASCS/SCS 多 SID 範本][sap-templates-3-tier-multisid-xscs-marketplace-image]或[使用受控磁碟的 ASCS/SCS 多 SID 範本][sap-templates-3-tier-multisid-xscs-marketplace-image-md]中輸入下列參數的值︰

  - **資源前置詞**：設定資源前置詞，其用來前置部署期間建立的所有資源。 因為資源不只屬於一個 SAP 系統，資源的前置詞不是一個 SAP 系統的 SID。  前置詞必須介於三到六個字元。
  - **堆疊類型**：選取 SAP 系統的堆疊類型。 根據堆疊類型而定，Azure Load Balancer 每個 SAP 系統會有一個 (ABAP 或僅 Java) 或兩個 (ABAP + Java) 私人 IP 位址。
  -  **OS 類型**：選取虛擬機器的作業系統。
  -  **SAP 系統計數**：選取您想要安裝在此叢集中的 SAP 系統數目。
  -  **系統可用性**選取 [HA]。
  -  **管理員使用者名稱和管理員密碼**：建立可用來登入電腦的新使用者。
  -  **新的或現有的子網路**︰設定是否要建立新的虛擬網路和子網路，或是使用現有的子網路。 如果您已經有連接到內部部署網路的虛擬網路，請選取**現有**虛擬網路。
  -  **子網路識別碼**：設定虛擬機器應該連接的子網路識別碼。 選取要將虛擬機器連線到內部部署網路之 VPN 或 ExpressRoute 虛擬網路的子網路。 識別碼通常看起來像這樣︰

   /subscriptions/\<訂用帳戶識別碼\>/resourceGroups/\<資源群組名稱\>/providers/Microsoft.Network/virtualNetworks/\<虛擬網路名稱\>/subnets/\<子網路名稱\>

範本會部署一個 Azure Load Balancer 執行個體，可支援多個 SAP 系統：

- 針對執行個體號碼 00、10、20... 設定的 ASCS 執行個體
- 針對執行個體號碼 01、11、21... 設定的 ASCS 執行個體
- 針對執行個體號碼 02、12、22... 設定的 ASCS 加入佇列複寫伺服器 (ERS) (僅限 Linux) 執行個體
- 針對執行個體號碼 03、13、23... 設定的 SCS ERS (僅限 Linux) 執行個體

負載平衡器包含 1 個 VIP (2 個適用於 Linux)、1 個 VIP 適用於 ASCS/SCS 和1 個 VIP 適用於 ERS (僅 Linux)。

#### <a name="0f3ee255-b31e-4b8a-a95a-d9ed6200468b"></a> SAP ASCS/SCS 連接埠
下列清單包含所有的負載平衡規則 (其中 x 是 SAP 系統的數目，例如 1、2、3...)︰
- 每個 SAP 系統的 Windows 特定連接埠︰445、5985
- ASCS 連接埠 (執行個體數目 x0)︰32x0、36x0、39x0、81x0、5x013、5x014、5x016
- SCS 連接埠 (執行個體數目 x1)︰32x1、33x1、39x1、81x1、5x113、5x114、5x116
- Linux 上的 ASCS ERS 連接埠 (執行個體數目 x2)︰33x2、5x213、5x214、5x216
- Linux 上的 SCS ERS 連接埠 (執行個體數目 x3)︰33x3、5x313、5x314、5x316

負載平衡器設定為使用下列的探查連接埠 (其中 x 是 SAP 系統的數目，例如 1、2、3...)︰
- ASCS/SCS 內部負載平衡器探查連接埠︰620x0
- ERS 內部負載平衡器探查連接埠 (僅 Linux)︰621x2

### <a name="database-template"></a> 資料庫範本

資料庫範本會部署一個或兩個虛擬機器，可讓您用來安裝一個 SAP 系統的關聯式資料庫管理系統 (RDBMS)。 例如，如果您部署五個 SAP 系統的 ASCS/SCS 範本，您需要部署此範本五次。

若要設定資料庫多 SID 範本，請在[資料庫多 SID 範本][sap-templates-3-tier-multisid-db-marketplace-image]或[使用受控磁碟的資料庫多 SID 範本][sap-templates-3-tier-multisid-db-marketplace-image-md]中輸入下列參數的值︰

  -  **SAP 系統識別碼**：輸入您想要安裝之 SAP 系統的 SAP 系統識別碼。 該識別碼會作為所部署之資源的前置詞。
  -  **OS 類型**：選取虛擬機器的作業系統。
  -  **Dbtype**：選取您想要在叢集上安裝的資料庫類型。 如果您想要安裝 Microsoft SQL Server，選取 **SQL**。 如果您打算在虛擬機器上安裝 SAP HANA，選取 **HANA**。 請確認您選取正確的作業系統類型。 針對 SQL 選取 [Windows]，並之對 HANA 選取 Linux 散發套件。 連線到虛擬機器的 Azure Load Balancer 會設定為支援所選取的資料庫類型︰
    * **SQL**：負載平衡器會負載平衡連接埠 1433。 請務必針對您的 SQL Server AlwaysOn 設定使用此連接埠。
    * **HANA**負載平衡器會負載平衡連接埠 35015 和 35017。 請務必以執行個體數目 **50** 安裝 SAP HANA。
    負載平衡器會使用探查連接埠 62550。
  -  **SAP 系統大小**：設定新系統所提供的 SAP 數量。 如果您不確定系統需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
  -  **系統可用性**選取 [HA]。
  -  **管理員使用者名稱和管理員密碼**：建立可用來登入電腦的新使用者。
  -  **子網路識別碼**：輸入您在部署 ASCS/SCS 範本期間使用的子網路識別碼，或做為 ASCS/SCS 範本部署的一部分建立之子網路識別碼。

### <a name="application-servers-template"></a> 應用程式伺服器範本

應用程式伺服器範本會部署兩個或多個虛擬機器，可用來做為一個 SAP 系統的 SAP 應用程式伺服器執行個體。 例如，如果您部署五個 SAP 系統的 ASCS/SCS 範本，您需要部署此範本五次。

若要設定應用程式伺服器多 SID 範本，請在[應用程式伺服器多 SID 範本][sap-templates-3-tier-multisid-apps-marketplace-image]或[使用受控磁碟的應用程式伺服器多 SID 範本][sap-templates-3-tier-multisid-apps-marketplace-image-md]中輸入下列參數的值︰

  -  **SAP 系統識別碼**：輸入您想要安裝之 SAP 系統的 SAP 系統識別碼。 該識別碼會作為所部署之資源的前置詞。
  -  **OS 類型**：選取虛擬機器的作業系統。
  -  **SAP 系統大小**：新系統所提供的 SAP 數量。 如果您不確定系統需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
  -  **系統可用性**選取 [HA]。
  -  **管理員使用者名稱和管理員密碼**：建立可用來登入電腦的新使用者。
  -  **子網路識別碼**：輸入您在部署 ASCS/SCS 範本期間使用的子網路識別碼，或做為 ASCS/SCS 範本部署的一部分建立之子網路識別碼。


## <a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Azure 虛擬網路
在我們的範例中，Azure 虛擬網路執行個體的位址空間是 10.0.0.0/16。 有一個名為 Subnet、網址範圍為 10.0.0.0/24 的子網路。 所有虛擬機器和內部負載平衡器會部署在此虛擬網路中。

> [!IMPORTANT]
> 請勿對客體作業系統內的網路設定進行任何變更。 包括 IP 位址、DNS 伺服器和子網路。 在 Azure 中設定所有網路設定。 動態主機設定通訊協定 (DHCP) 服務會散佈您的設定。
>
>

## <a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a> DNS IP 位址

若要設定所需的 DNS IP 位址，請完成下列步驟：

1.  在 Azure 入口網站的 [DNS 伺服器] 窗格中，確定您的虛擬網路 [DNS 伺服器] 選項是設定為 [自訂 DNS]。
2.  根據您擁有的網路類型選取設定。 如需詳細資訊，請參閱下列資源：
    * [公司網路連線能力 (跨單位)][planning-guide-2.2]︰請新增內部部署 DNS 伺服器的 IP 位址。  
    您可以將內部部署 DNS 伺服器擴充至 Azure 中執行的虛擬機器。 在這種情況下，您可以加入執行 DNS 服務之 Azure 虛擬機器的 IP 位址。
    * [僅限雲端部署][planning-guide-2.1]：在做為 DNS 伺服器的相同虛擬網路執行個體中，部署其他虛擬機器。 新增您已設定執行 DNS 服務之 Azure 虛擬機器的 IP 位址。

    ![圖 2：設定 Azure 虛擬網路的 DNS 伺服器][sap-ha-guide-figure-3001]

    _**圖 2：** 設定 Azure 虛擬網路的 DNS 伺服器_

  > [!NOTE]
  > 如果您變更 DNS 伺服器的 IP 位址，您就必須重新啟動 Azure 虛擬機器，才能套用變更並傳播新的 DNS 伺服器。
  >
  >

在我們的範例中，是在下列 Windows 虛擬機器上安裝並設定 DNS 服務：

| 虛擬機器角色 | 虛擬機器主機名稱 | 網路卡名稱 | 靜態 IP 位址 |
| --- | --- | --- | --- |
| 第一部 DNS 伺服器 |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| 第二部 DNS 伺服器 |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

## <a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a> SAP ASCS/SCS 叢集執行個體及 DBMS 叢集執行個體的主機名稱和靜態 IP 位址

對於內部部署，您需要下列保留的主機名稱和 IP 位址：

| 虛擬主機名稱角色 | 虛擬主機名稱 | 虛擬靜態 IP 位址 |
| --- | --- | --- |
| SAP ASCS/SCS 第一個叢集虛擬主機名稱 (用於叢集管理) |pr1-ascs-vir |10.0.0.42 |
| SAP ASCS/SCS 執行個體虛擬主機名稱 |pr1-ascs-sap |10.0.0.43 |
| SAP DBMS 第二個叢集虛擬主機名稱 (叢集管理) |pr1-dbms-vir |10.0.0.32 |

當您建立叢集時，請建立虛擬主機名稱 pr1-ascs-vir 和 pr1-dbms-vir，以及管理叢集本身的相關 IP 位址。 如需如何執行這項操作的詳細資訊，請參閱[在叢集組態中收集叢集節點][sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]。

您可以在 DNS 伺服器上手動建立其他兩個虛擬主機名稱，pr1-ascs-sap 和 pr1-dbms-sap，以及相關的 IP 位址。 叢集 SAP ASCS/SCS 執行個體和叢集 DBMS 執行個體使用這些資源。 如需如何執行這項操作的詳細資訊，請參閱[建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱][sap-ha-guide-9.1.1]。

## <a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a> 設定 SAP 虛擬機器的靜態 IP 位址
部署要在叢集中使用的虛擬機器之後，您需要設定所有虛擬機器的靜態 IP 位址。 在 Azure 虛擬網路組態中執行此動作，而非在客體作業系統中執行此動作。

1.  在 Azure 入口網站中選取 [資源群組] > [網路卡] > [設定] > [IP 位址]。
2.  在 [IP 位址] 窗格的 [指派] 下，選取 [靜態]。 在 [IP 位址] 方塊中，輸入您要使用的 IP 位址。

  > [!NOTE]
  > 如果您變更網路卡的 IP 位址，您就必須重新啟動 Azure 虛擬機器，才能套用變更。  
  >
  >

  ![圖 3：為每個虛擬機器的網路卡設定靜態 IP 位址][sap-ha-guide-figure-3002]

  _**圖 3：** 為每個虛擬機器的網路卡設定靜態 IP 位址_

  為所有網路介面重複此步驟，亦即為所有虛擬機器，包括您要用於 Active Directory 或 DNS 服務的虛擬機器。

在我們的範例中，有下列虛擬機器和靜態 IP 位址：

| 虛擬機器角色 | 虛擬機器主機名稱 | 網路卡名稱 | 靜態 IP 位址 |
| --- | --- | --- | --- |
| 第一部 SAP 應用程式伺服器執行個體 |pr1-di-0 |pr1-nic-di-0 |10.0.0.50 |
| 第二部 SAP 應用程式伺服器執行個體 |pr1-di-1 |pr1-nic-di-1 |10.0.0.51 |
| ... |... |... |... |
| 最後一部 SAP 應用程式伺服器執行個體 |pr1-di-5 |pr1-nic-di-5 |10.0.0.55 |
| ASCS/SCS 執行個體的第一個叢集節點 |pr1-ascs-0 |pr1-nic-ascs-0 |10.0.0.40 |
| ASCS/SCS 執行個體的第二個叢集節點 |pr1-ascs-1 |pr1-nic-ascs-1 |10.0.0.41 |
| DBMS 執行個體的第一個叢集節點 |pr1-db-0 |pr1-nic-db-0 |10.0.0.30 |
| DBMS 執行個體的第二個叢集節點 |pr1-db-1 |pr1-nic-db-1 |10.0.0.31 |

## <a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a> 為 Azure 內部負載平衡器設定靜態 IP 位址

SAP Azure Resource Manager 範本會建立用於 SAP ASCS/SCS 執行個體叢集和用於 DBMS 叢集的 Azure 內部負載平衡器。

> [!IMPORTANT]
> SAP ASCS/SCS 之虛擬主機名稱的 IP 位址與 SAP ASCS/SCS 內部負載平衡器：pr1-lb-ascs 的 IP 位址相同。
> DBMS 之虛擬名稱的 IP 位址與 DBMS 內部負載平衡器：pr1-lb-dbms 的 IP 位址相同。
>
>

若要為 Azure 內部負載平衡器設定靜態 IP 位址：

1.  初始部署會將內部負載平衡器的 IP 位址設定為 [動態]。 在 Azure 入口網站 [IP 位址] 窗格的 [指派] 下，選取 [靜態]。
2.  請將「內部負載平衡器」**pr1-lb-ascs** 的 IP 位址設定為 SAP ASCS/SCS 執行個體的虛擬主機名稱 IP 位址。
3.  請將「內部負載平衡器」**pr1-lb-dbms** 的 IP 位址設定為 DBMS 執行個體的虛擬主機名稱 IP 位址。

  ![圖 4：為 SAP ASCS/SCS 執行個體的內部負載平衡器設定靜態 IP 位址][sap-ha-guide-figure-3003]

  _**圖 4：** 為 SAP ASCS/SCS 執行個體的內部負載平衡器設定靜態 IP 位址_

在我們的範例中，有兩個 Azure 內部負載平衡器具備這些靜態 IP 位址：

| Azure 內部負載平衡器角色 | Azure 內部負載平衡器名稱 | 靜態 IP 位址 |
| --- | --- | --- |
| SAP ASCS/SCS 執行個體內部負載平衡器 |pr1-lb-ascs |10.0.0.43 |
| SAP DBMS 內部負載平衡器 |pr1-lb-dbms |10.0.0.33 |


## <a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a> Azure 內部負載平衡器的預設 ASCS/SCS 負載平衡規則

SAP Azure Resource Manager 範本會建立所需的連接埠：
* ABAP ASCS 執行個體預設的執行個體號碼為 00
* Java SCS 執行個體預設的執行個體號碼為 01

當您安裝 SAP ASCS/SCS 執行個體時，必須為 ABAP ASCS 執行個體使用預設的執行個體號碼 00，並為 Java SCS 執行個體使用預設的執行個體號碼 01。

接著，為 SAP NetWeaver 連接埠建立下列必要的內部負載平衡端點。

若要建立必要的內部負載平衡端點，首先，為 SAP NetWeaver ABAP ASCS 連接埠建立下列必要的內部負載平衡端點：

| 服務/負載平衡規則名稱 | 預設連接埠號碼 | (執行個體號碼為 00 的 ASCS 執行個體) (ERS 為 10) 的具體連接埠 |
| --- | --- | --- |
| 加入佇列伺服器 / *lbrule3200* |32\<InstanceNumber\> |3200 |
| ABAP 訊息伺服器 / *lbrule3600* |36\<InstanceNumber\> |3600 |
| 內部 ABAP 訊息 / *lbrule3900* |39\<InstanceNumber\> |3900 |
| 訊息伺服器 HTTP / *Lbrule8100* |81\<InstanceNumber\> |8100 |
| SAP 啟動服務 ASCS HTTP / *Lbrule50013* |5\<InstanceNumber\>13 |50013 |
| SAP 啟動服務 ASCS HTTPS / *Lbrule50014* |5\<InstanceNumber\>14 |50014 |
| 加入佇列複寫 / *Lbrule50016* |5\<InstanceNumber\>16 |50016 |
| SAP 啟動服務 ERS HTTP *Lbrule51013* |5\<InstanceNumber\>13 |51013 |
| SAP 啟動服務 ERS HTTP *Lbrule51014* |5\<InstanceNumber\>14 |51014 |
| Windows 遠端管理 (WinRM) *Lbrule5985* | |5985 |
| 檔案共用 *Lbrule445* | |445 |

表 1：SAP NetWeaver ABAP ASCS 執行個體的連接埠號碼

接著，為 SAP NetWeaver Java SCS 連接埠建立下列負載平衡端點：

| 服務/負載平衡規則名稱 | 預設連接埠號碼 | (執行個體號碼為 01 的 SCS 執行個體) (ERS 為 11) 的具體連接埠 |
| --- | --- | --- |
| 加入佇列伺服器 / *lbrule3201* |32\<InstanceNumber\> |3201 |
| 閘道伺服器 / *lbrule3301* |33\<InstanceNumber\> |3301 |
| Java 訊息伺服器 / *lbrule3900* |39\<InstanceNumber\> |3901 |
| 訊息伺服器 HTTP / *Lbrule8101* |81\<InstanceNumber\> |8101 |
| SAP 啟動服務 SCS HTTP / *Lbrule50113* |5\<InstanceNumber\>13 |50113 |
| SAP 啟動服務 SCS HTTPS / *Lbrule50114* |5\<InstanceNumber\>14 |50114 |
| 加入佇列複寫 / *Lbrule50116* |5\<InstanceNumber\>16 |50116 |
| SAP 啟動服務 ERS HTTP *Lbrule51113* |5\<InstanceNumber\>13 |51113 |
| SAP 啟動服務 ERS HTTP *Lbrule51114* |5\<InstanceNumber\>14 |51114 |
| WinRM *Lbrule5985* | |5985 |
| 檔案共用 *Lbrule445* | |445 |

**表 2：** SAP NetWeaver Java SCS 執行個體的連接埠號碼

![圖 5：Azure 內部負載平衡器的預設 ASCS/SCS 負載平衡規則][sap-ha-guide-figure-3004]

_**圖 5：** Azure 內部負載平衡器的預設 ASCS/SCS 負載平衡規則_

將負載平衡器 pr1-lb-dbms 的 IP 位址設定為 DBMS 執行個體之虛擬主機名稱的 IP 位址。

### <a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> 變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則

如果您想要將其他號碼用於 SAP ASCS 或 SCS 執行個體，您就必須從預設值變更其連接埠的名稱和值。

1.  在 Azure 入口網站中，選取 **\<SID\>-lb-ascs 負載平衡器** > **負載平衡規則**。
2.  針對屬於 SAP ASCS 或 SCS 執行個體的所有負載平衡規則，變更下列值：

  * Name
  * Port
  * 後端連接埠

  例如，如果您要將預設 ASCS 執行個體號碼從 00 變更為 31，您需要針對表 1 中所列的所有通訊埠進行變更。

  以下是連接埠 *lbrule3200*的更新範例。

  ![圖 6：變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則][sap-ha-guide-figure-3005]

  _**圖 6：** 變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則_

## <a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> 將 Windows 虛擬機器新增至網域

將靜態 IP 位址指派給虛擬機器之後，請將虛擬機器新增至網域。

![圖 7：將虛擬機器新增至網域][sap-ha-guide-figure-3006]

_**圖 7：** 將虛擬機器新增至網域_

## <a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> 在 SAP ASCS/SCS 執行個體的兩個叢集節點上都新增登錄項目

Azure Load Balancer 具有內部負載平衡器，會在連線閒置一段時間 (閒置逾時) 時關閉連線。 對話方塊執行個體中的 SAP 工作程序則會在需要傳送第一個加入佇列/清除佇列要求時，立即開啟對SAP 加入佇列程序的連線。 這些連線通常會持續維持已建立狀態，直到工作程序或加入佇列程序重新啟動為止。 不過，如果連線閒置一段時間，Azure 內部負載平衡器就會關閉連線。 這不是問題，因為如果連線不存在，SAP 工作程序會對加入佇列程序重新建立連線。 這些活動會記載於 SAP 程序的開發人員追蹤，但它們會對這些追蹤建立大量額外的內容。 因此，最好先變更兩個叢集節點上的 TCP/IP `KeepAliveTime` 和 `KeepAliveInterval`。 結合這些 TCP/IP 參數中的變更與 SAP 設定檔參數中的變更，稍後在本文中會加以描述。

若要在 SAP ASCS/SCS 執行個體的兩個叢集節點上新增登錄項目，首先，在 SAP ASCS/SCS 的兩個 Windows 叢集節點上新增這些 Windows 登錄項目︰

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 變數名稱 |`KeepAliveTime` |
| 變數類型 |REG_DWORD (十進位) |
| 值 |120000 |
| 文件連結 |[https://technet.microsoft.com/library/cc957549.aspx](https://technet.microsoft.com/library/cc957549.aspx) |

表 3：變更第一個 TCP/IP 參數

然後，在 SAP ASCS/SCS 的兩個 Windows 叢集節點上都新增下列 Windows 登錄項目：

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 變數名稱 |`KeepAliveInterval` |
| 變數類型 |REG_DWORD (十進位) |
| 值 |120000 |
| 文件連結 |[https://technet.microsoft.com/library/cc957548.aspx](https://technet.microsoft.com/library/cc957548.aspx) |

表 4：變更第二個 TCP/IP 參數

若要套用變更，請重新啟動這兩個叢集節點。

## <a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> 設定 SAP ASCS/SCS 執行個體的 Windows Server 容錯移轉叢集

設定 SAP ASCS/SCS 執行個體的 Windows Server 容錯移轉叢集包含下列工作︰

- 收集叢集組態中的叢集節點。
- 設定叢集檔案共用見證。

### <a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a> 收集叢集組態中的叢集節點

1.  在新增角色及功能精靈中，將容錯移轉叢集新增至這兩個叢集節點。
2.  使用 [容錯移轉叢集管理員] 來設定容錯移轉叢集。 在 [容錯移轉叢集管理員] 中，按一下 [建立叢集]，然後只新增第一個叢集 (節點 A) 的名稱。 還不要新增第二個節點；您會在稍後步驟中新增第二個節點。

  ![圖 8：加入第一個叢集節點的伺服器或虛擬機器名稱][sap-ha-guide-figure-3007]

  _**圖 8：** 加入第一個叢集節點的伺服器或虛擬機器名稱_

3.  輸入叢集的網路名稱 (虛擬主機名稱)。

  ![圖 9︰輸入叢集名稱][sap-ha-guide-figure-3008]

  _**圖 9︰** 輸入叢集名稱_

4.  建立叢集之後，會執行叢集驗證測試。

  ![圖 10：執行叢集驗證檢查][sap-ha-guide-figure-3009]

  _**圖 10：** 執行叢集驗證檢查_

  您可以忽略此時程序中有關磁碟的任何警告。 稍後您將新增檔案共用見證與 SIOS 共用磁碟。 在這個階段，您不必擔心有仲裁。

  ![圖 11：找不到仲裁磁碟][sap-ha-guide-figure-3010]

  _**圖 11：** 找不到仲裁磁碟_

  ![圖 12：核心叢集資源需要新的 IP 位址][sap-ha-guide-figure-3011]

  _**圖 12：** 核心叢集資源需要新的 IP 位址_

5.  變更核心叢集服務的 IP 位址。 直到您變更核心叢集服務的 IP 位址之前皆無法啟動叢集，因為伺服器的 IP 位址指向其中一個虛擬機器節點。 在核心叢集服務 IP 資源的**屬性**頁面上執行此動作。

  例如，我們需要為叢集虛擬主機名稱 pr1-ascs-vir 指派 IP 位址 (在我們的範例中為 10.0.0.42)。

  ![圖 13：在 [屬性] 對話方塊中，變更 IP 位址][sap-ha-guide-figure-3012]

  _**圖 13：** 在 [屬性] 對話方塊中，變更 IP 位址_

  ![圖 14：指派為叢集保留的 IP 位址][sap-ha-guide-figure-3013]

  _**圖 14：** 指派為叢集保留的 IP 位址_

6.  將叢集虛擬主機名稱上線。

  ![圖 15：叢集核心服務以正確的 IP 位址啟動及執行][sap-ha-guide-figure-3014]

  _**圖 15：** 叢集核心服務以正確的 IP 位址啟動及執行_

7.  新增第二個叢集節點。

  既然核心叢集服務已啟動並正常執行，您便可以新增第二個叢集節點。

  ![圖 16：新增第二個叢集節點][sap-ha-guide-figure-3015]

  _**圖 16：** 新增第二個叢集節點_

8.  輸入第二個叢集節點主機的名稱。

  ![圖 17︰輸入第二個叢集節點主機名稱][sap-ha-guide-figure-3016]

  _**圖 17︰** 輸入第二個叢集節點主機名稱_

  > [!IMPORTANT]
  > 請確定*沒有*選取 [新增適合的儲存裝置到叢集] 核取方塊。  
  >
  >

  ![圖 18：請勿選取核取方塊][sap-ha-guide-figure-3017]

  _**圖 18：** 請勿選取核取方塊_

  您可以忽略有關仲裁和磁碟的警告。 您將在稍後設定仲裁和共用磁碟，如[為 SAP ASCS/SCS 叢集共用磁碟安裝 SIOS DataKeeper Cluster Edition][sap-high-availability-infrastructure-wsfc-shared-disk-install-sios] 中所述。

  ![圖 19：忽略有關磁碟仲裁的警告][sap-ha-guide-figure-3018]

  _**圖 19：** 忽略有關磁碟仲裁的警告_


#### <a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a> 設定叢集檔案共用見證

設定叢集檔案共用見證包含下列工作︰

- 建立檔案共用。
- 在容錯移轉叢集管理員中設定檔案共用見證仲裁。

#### <a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a> 建立檔案共用

1.  選取檔案共用見證，而不是仲裁磁碟。 SIOS DataKeeper 支援此選項。

  在本文的範例中，檔案共用見證是在 Azure 中執行的 Active Directory 或 DNS 伺服器上。 檔案共用見證稱為 domcontr-0。 因為您已設定 Azure 的 VPN 連線 (透過 VPN 閘道或 Azure ExpressRoute)，所以您的 Active Directory 或 DNS 服務是內部部署，而且不適用於執行檔案共用見證。

  > [!NOTE]
  > 如果您的 Active Directory 或 DNS 服務僅在內部部署上執行，請勿在執行內部部署的 Active Directory 或 DNS Windows 作業系統上設定檔案共用見證。 在 Azure 和 Active Directory 或 DNS 內部部署中執行之叢集節點之間的網路延遲可能太大，而且會造成連線問題。 請務必在執行位置靠近叢集節點的 Azure 虛擬機器上設定檔案共用見證。  
  >
  >

  仲裁磁碟機至少需要 1,024 MB 的可用空間。 我們建議使用 2,048 MB 的仲裁磁碟機可用空間。

2.  新增叢集名稱物件。

  ![圖 20：為叢集名稱物件指派共用上的權限][sap-ha-guide-figure-3019]

  _**圖 20：** 為叢集名稱物件指派共用上的權限_

  請確定權限能夠為叢集名稱物件 (在我們的範例中為 pr1-ascs-vir$) 變更共用中的資料。

3.  若要將叢集名稱物件新增至清單中，請選取 [新增]。 變更篩選，以檢查電腦物件，除了圖 22 所示的項目以外。

  ![圖 21：變更物件類型以包含電腦][sap-ha-guide-figure-3020]

  _**圖 21：** 變更**物件類型**以包含電腦_

  ![圖 22︰選取 [電腦] 核取方塊][sap-ha-guide-figure-3021]

  _**圖 22︰** 選取 [電腦] 核取方塊_

4.  輸入叢集名稱物件，如圖 21 所示。 由於已經建立記錄，因此您可以變更權限，如圖 20 所示。

5.  選取共用的 [安全性] 索引標籤，然後為叢集名稱物件設定更詳細的權限。

  ![圖 23：為叢集名稱物件設定檔案共用仲裁上的安全性屬性][sap-ha-guide-figure-3022]

  _**圖 23：** 為叢集名稱物件設定檔案共用仲裁上的安全性屬性_

#### <a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a> 在容錯移轉叢集管理員中設定檔案共用見證仲裁

1.  開啟設定仲裁設定精靈。

  ![圖 24：啟動設定叢集仲裁設定精靈][sap-ha-guide-figure-3023]

  _**圖 24：** 啟動設定叢集仲裁設定精靈_

2.  在 [選取仲裁設定選項] 頁面上，選取 [選取仲裁見證]。

  ![圖 25：您可以選擇的仲裁設定][sap-ha-guide-figure-3024]

  _**圖 25：** 您可以選擇的仲裁設定_

3.  在 [選取仲裁見證] 頁面上，選取 [設定檔案共用見證]。

  ![圖 26︰選取檔案共用見證][sap-ha-guide-figure-3025]

  _**圖 26︰** 選取檔案共用見證_

4.  輸入檔案共用的 UNC 路徑 (在我們的範例中為 \\domcontr-0\FSW)。 若要查看您可進行變更的清單，選取 [下一步]。

  ![圖 27：定義見證共用的檔案共用位置][sap-ha-guide-figure-3026]

  _**圖 27：** 定義見證共用的檔案共用位置_

5.  選取您要的變更，然後選取 [下一步]。 您必須成功重新設定叢集組態，如圖 28 中所示：  

  ![圖 28：確認您已重新設定叢集][sap-ha-guide-figure-3027]

  _**圖 28：** 確認您已重新設定叢集_

當您安裝 Windows 容錯移轉叢集之後，必須變更某些閾值，讓它們能將容錯移轉偵測適應 Azure 中的條件。 需要變更的參數記載於[調整容錯移轉叢集網路閥值][tuning-failover-cluster-network-thresholds]中。 假設為 ASCS/SCS 建置 Windows 叢集組態的兩部 VM 位於相同子網路中，請將下列參數變更為這些值︰

- SameSubNetDelay = 2
- SameSubNetThreshold = 15

這些設定已與客戶進行測試，並提供很好的折衷方案。 它們具有足夠的彈性，但也提供在 SAP 軟體的真實錯誤條件或節點或 VM 失敗中夠快的容錯移轉。

### <a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> 為 SAP ASCS/SCS 叢集共用磁碟安裝 SIOS DataKeeper Cluster Edition

您現在在 Azure 中有運作中的 Windows Server 容錯移轉叢集組態。 若要安裝 SAP ASCS/SCS 執行個體，您需要有一個共用磁碟資源。 您無法在 Azure 中建立所需的共用磁碟資源。 SIOS DataKeeper Cluster Edition 是一種可用來建立共用磁碟資源的協力廠商解決方案。

為 SAP ASCS/SCS 叢集共用磁碟安裝 SIOS DataKeeper Cluster Edition 包含下列工作︰

- 新增 Microsoft .NET Framework 3.5。
- 安裝 SIOS DataKeeper。
- 設定 SIOS DataKeeper。

### <a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a> 新增 .NET Framework 3.5
.NET Framework 3.5 不會自動啟用或安裝在 Windows Server 2012 R2。 因為 SIOS DataKeeper 需要 .NET 位於您安裝 DataKeeper 的所有節點上，您必須在叢集中所有虛擬機器的客體作業系統上安裝 .NET Framework 3.5。

有兩種方式可以新增 .Net Framework 3.5：

- 在 Windows 中使用 [新增角色及功能精靈]，如圖 29 所示：

  ![圖 29：使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5][sap-ha-guide-figure-3028]

  _**圖 29：** 使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5_

  ![圖 30︰當您使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5 時的安裝進度列][sap-ha-guide-figure-3029]

  _**圖 30︰** 當您使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5 時的安裝進度列_

- 使用 dism.exe 命令列工具。 針對這類型的安裝，您需要存取 Windows 安裝媒體上的 SxS 目錄。 在提高權限的命令提示字元上，輸入此命令：

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

### <a name="dd41d5a2-8083-415b-9878-839652812102"></a> 安裝 SIOS DataKeeper

在叢集中的每個節點上安裝 SIOS DataKeeper Cluster Edition。 若要使用 SIOS DataKeeper 建立虛擬共用儲存體，請建立同步處理的鏡像，然後模擬叢集共用儲存體。

安裝 SIOS 軟體之前，請先建立 DataKeeperSvc 網域使用者。

> [!NOTE]
> 將 DataKeeperSvc 往欲使用者同時新增到兩個叢集節點上的 [本機管理員] 群組中。
>
>

若要安裝 SIOS DataKeeper：

1.  在兩個叢集節點上都安裝 SIOS 軟體。

  ![SIOS 安裝程式][sap-ha-guide-figure-3030]

  ![圖 31：第一個 SIOS DataKeeper 安裝頁面][sap-ha-guide-figure-3031]

  _**圖 31：** 第一個 SIOS DataKeeper 安裝頁面_

2.  在對話方塊中，選取 [是]。

  ![圖 32：DataKeeper 通知將停用某個服務][sap-ha-guide-figure-3032]

  _**圖 32：** DataKeeper 通知將停用某個服務_

3.  在對話方塊中，建議您選取 [網域或伺服器帳戶]。

  ![圖 33：SIOS DataKeeper 的使用者選取項目][sap-ha-guide-figure-3033]

  _**圖 33：** SIOS DataKeeper 的使用者選取項目_

4.  輸入您為 SIOS DataKeeper 建立的網域帳戶使用者名稱和密碼。

  ![圖 34：為 SIOS DataKeeper 安裝輸入網域使用者名稱和密碼][sap-ha-guide-figure-3034]

  _**圖 34：** 為 SIOS DataKeeper 安裝輸入網域使用者名稱和密碼_

5.  為您的 SIOS DataKeeper 執行個體安裝授權金鑰，如圖 35 所示。

  ![圖 35：輸入您的 SIOS DataKeeper 授權金鑰][sap-ha-guide-figure-3035]

  _**圖 35：** 輸入您的 SIOS DataKeeper 授權金鑰_

6.  出現提示時，重新啟動虛擬機器。

### <a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a> 設定 SIOS DataKeeper

在兩個節點上都安裝 SIOS DataKeeper 之後，開始設定組態。 組態的目標是要在連接至每個虛擬機器的額外磁碟之間進行同步資料複寫。

1.  啟動 DataKeeper 管理與組態工具，然後選取 [連線伺服器]。

  ![圖 36：SIOS DataKeeper 管理和組態工具][sap-ha-guide-figure-3036]

  _**圖 36：** SIOS DataKeeper 管理和組態工具_

2.  輸入管理和組態工具應連接之第一個節點的名稱或 TCP/IP 位址，然後在第二個步驟中，插入第二個節點。

  ![圖 37：插入管理和組態工具應連接之第一個節點的名稱或 TCP/IP 位址，然後在第二個步驟中，插入第二個節點][sap-ha-guide-figure-3037]

  _**圖 37：** 插入管理和組態工具應連接之第一個節點的名稱或 TCP/IP 位址，然後在第二個步驟中，插入第二個節點_

3.  在兩個節點之間建立複寫作業。

  ![圖 38：建立複寫作業][sap-ha-guide-figure-3038]

  _**圖 38：** 建立複寫作業_

  精靈會引導您完成建立複寫作業的程序。

4.  定義複寫作業的名稱。

  ![圖 39：定義複寫作業的名稱][sap-ha-guide-figure-3039]

  _**圖 39：** 定義複寫作業的名稱_

  ![圖 40：定義節點 (應該是目前的來源節點) 的基底資料][sap-ha-guide-figure-3040]

  _**圖 40：** 定義節點 (應該是目前的來源節點) 的基底資料_

5.  定義名稱、TCP/IP 位址和目標節點的磁碟區。

  ![圖 41：定義名稱、TCP/IP 位址和目前目標節點的磁碟區][sap-ha-guide-figure-3041]

  _**圖 41：** 定義名稱、TCP/IP 位址和目前目標節點的磁碟區。_

6.  定義壓縮演算法。 在我們的範例中，建議壓縮複寫資料流。 尤其是在重新同步處理的情況下，壓縮複寫資料流可大幅縮短重新同步處理時間。 壓縮會使用虛擬機器的 CPU 和 RAM 資源。 隨著壓縮率增加，使用的 CPU 資源數量也跟著增加。 您可以稍後調整此設定。

7.  另一個您需要檢查的設定是複寫是以非同步方式還是同步方式進行。 當您保護 SAP ASCS/SCS 組態時，必須使用同步複寫。  

  ![圖 42︰定義複寫詳細資料][sap-ha-guide-figure-3042]

  _**圖 42︰** 定義複寫詳細資料_

8.  定義是否應該向 Windows Server 容錯移轉叢集的叢集組態指出複寫作業所複寫的磁碟區是共用磁碟。 就 SAP ASCS/SCS 組態而言，選取 [是] ，如此 Windows 叢集才會將複寫的磁碟區視為可用來作為叢集磁碟區的共用磁碟。

  ![圖 43：選取 [是] 以將複寫的磁碟區設定為叢集磁碟區][sap-ha-guide-figure-3043]

  _**圖 43：** 選取 [是] 以將複寫的磁碟區設定為叢集磁碟區_

  建立磁碟區之後，DataKeeper 管理和組態工具會顯示複寫作業是使用中。

  ![圖 44：SAP ASCS/SCS 共用磁碟的 DataKeeper 同步鏡像處於作用中狀態][sap-ha-guide-figure-3044]

  _**圖 44：** SAP ASCS/SCS 共用磁碟的 DataKeeper 同步鏡像處於作用中狀態_

  「容錯移轉叢集管理員」現在會顯示磁碟為 DataKeeper 磁碟，如圖 45 所示：

  ![圖 45：「容錯移轉叢集管理員」顯示 DataKeeper 複寫的磁碟][sap-ha-guide-figure-3045]

  _**圖 45：**「容錯移轉叢集管理員」顯示 DataKeeper 複寫的磁碟_

## <a name="next-steps"></a>後續步驟

* [使用 Windows 容錯移轉叢集和共用磁碟為 SAP ASCS/SCS 執行個體安裝 SAP NetWeaver HA][sap-high-availability-installation-wsfc-shared-disk]
