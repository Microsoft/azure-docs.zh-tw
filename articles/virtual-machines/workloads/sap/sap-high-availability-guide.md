---
title: SAP NetWeaver 的 Azure 虛擬機器高可用性 | Microsoft Docs
description: Azure 虛擬機器上的 SAP NetWeaver 高可用性指南
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 353caf2bca68513a1153fe7b569c85606f72e753
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45631033"
---
# <a name="azure-virtual-machines-high-availability-for-sap-netweaver"></a>SAP NetWeaver 的 Azure 虛擬機器高可用性

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

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-ha-guide]:sap-high-availability-guide.md
[sap-ha-guide-2]:#42b8f600-7ba3-4606-b8a5-53c4f026da08
[sap-ha-guide-4]:#8ecf3ba0-67c0-4495-9c14-feec1a2255b7
[sap-ha-guide-8]:#78092dbe-165b-454c-92f5-4972bdbef9bf
[sap-ha-guide-8.1]:#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-ha-guide-8.9]:#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-ha-guide-8.12]:#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-ha-guide-8.12.1]:#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-ha-guide-8.12.3]:#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-ha-guide-8.12.3.1]:#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-ha-guide-8.12.3.2]:#dd41d5a2-8083-415b-9878-839652812102
[sap-ha-guide-8.12.3.3]:#d9c1fc8e-8710-4dff-bec2-1f535db7b006
[sap-ha-guide-9]:#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-ha-guide-9.1.1]:#a97ad604-9094-44fe-a364-f89cb39bf097

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (SAP 多重 SID 高可用性設定)


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



對於需要在最短的時間內處理計算、儲存和網路資源，而不需要漫長的採購週期的組織而言，「Azure 虛擬機器」是適用的解決方案。 您可以使用「Azure 虛擬機器」以部署傳統的應用程式，例如 SAP NetWeaver 架構 ABAP、Java，以及 ABAP+Java 堆疊。 擴充的可靠性與可用性，無須其他內部部署資源。 「Azure 虛擬機器」支援跨單位連線能力，貴公司可將「Azure 虛擬機器」整合到其內部部署網域、私人雲端，以及 SAP 系統環境中。

在本文中，我們將探討使用 Azure Resource Manager 部署模型，在 Azure 中部署高可用性 SAP 系統時可採取的步驟。 我們會引導您完成這些主要工作：

* 尋找正確的 SAP 附註和安裝指南 (在[資源][sap-ha-guide-2]一節中列出)。 本文補充 SAP 安裝文件及 SAP 附註，其為有助於在特定平台上安裝及部署 SAP 軟體的主要資源。
* 了解 Azure Resource Manager 部署模型與 Azure 傳統部署模型之間的差異。
* 了解「Windows Server 容錯移轉叢集」仲裁模式，以便選取適合您 Azure 部署的模型。
* 了解 Azure 服務中的「Windows Server 容錯移轉叢集」共用儲存體。
* 了解如何在 Azure 中協助保護單一失敗點元件，例如進階商業應用程式程式設計 (ABAP) SAP Central Services (ASCS)/SAP Central Services (SCS) 和資料庫管理系統 (DBMS)，以及備援元件，例如 SAP 應用程式伺服器。
* 藉由使用 Azure Resource Manager，遵循在 Windows Server 容錯移轉叢集的叢集中安裝及設定高可用性 SAP 系統的逐步說明範例。
* 深入了解在 Azure 中使用 Windows Server 容錯移轉叢集所需的步驟，但內部部署不需要這些步驟。

為了簡化部署和設定，在本文中，我們使用新的 SAP 三層式高可用性 Resource Manager 範本。 此範本自動部署高可用性 SAP 系統所需的整個基礎結構。 基礎結構也支援 SAP 應用程式效能標準 (SAPS) 調整您 SAP 系統的大小。

## <a name="217c5479-5595-4cd8-870d-15ab00d4f84c"></a> 必要條件
開始之前，請確定符合下列各節所述的必要條件。 此外，請務必檢查[資源][sap-ha-guide-2]一節中列出的所有資源。

在本文中，我們針對[使用受控磁碟的三層式 SAP NetWeaver](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md/) 使用 Azure Resource Manager 範本。 如需範本的實用概觀，請參閱 [SAP Azure Resource Manager 範本](https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/16/azure-quickstart-templates-for-sap/)。

## <a name="42b8f600-7ba3-4606-b8a5-53c4f026da08"></a> 資源
這些文章涵蓋 Azure 中的 SAP 部署：

* [SAP NetWeaver 的 Azure 虛擬機器規劃和實作指南][planning-guide]
* [SAP NetWeaver 的 Azure 虛擬機器部署][deployment-guide]
* [SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* [SAP NetWeaver 的 Azure 虛擬機器高可用性 (本指南)][sap-ha-guide]

> [!NOTE]
> 文件中會儘可能提供所參考 SAP 安裝指南的連結 (請參閱 [SAP 安裝指南][sap-installation-guides])。 如需安裝程序的必要條件和相關資訊，建議您詳細閱讀 SAP NetWeaver 安裝指南。 本文僅涵蓋可與 Azure 虛擬機器搭配使用之 SAP NetWeaver 架構系統的特定工作。
>
>

下列 SAP 附註與 Azure 上的 SAP 主題相關：

| 附註編號 | 標題 |
| --- | --- |
| [1928533] |Azure 上的 SAP 應用程式︰支援的產品和大小 |
| [2015553] |Microsoft Azure 上的 SAP：支援的必要條件 |
| [1999351] |適用於 SAP 的增強型 Azure 監視功能 |
| [2178632] |Microsoft Azure 上的 SAP 主要監視度量 |
| [1999351] |Windows 上的虛擬化︰增強型監視功能 |
| [2243692] |針對 SAP DBMS 執行個體使用 Azure 進階 SSD 儲存體 |

深入了解 [Azure 訂用帳戶限制][azure-subscription-service-limits-subscription]，包含一般預設限制與最大限制。

## <a name="42156640c6-01cf-45a9-b225-4baa678b24f1"></a>使用 Azure Resource Manager 的高可用性 SAP 與 Azure 傳統部署模型
Azure Resource Manager 與 Azure 傳統部署模型在下列方面有所不同：

- 資源群組
- Azure 資源群組上的 Azure 內部負載平衡器相依性
- 支援 SAP 多 SID 案例

### <a name="f76af273-1993-4d83-b12d-65deeae23686"></a> 資源群組
在 Azure Resource Manager 中，您可以使用資源群組來管理 Azure 訂用帳戶中的所有應用程式資源。 利用整合式的方法，在資源群組中，所有資源都有相同的生命週期。 例如，所有資源會同時建立，並且同時刪除。 深入了解[資源群組](../../../azure-resource-manager/resource-group-overview.md#resource-groups)。

### <a name="3e85fbe0-84b1-4892-87af-d9b65ff91860"></a> Azure 資源群組上的 Azure 內部負載平衡器相依性

在 Azure 傳統部署模型中，Azure 內部負載平衡器 (Azure Load Balancer 服務) 和雲端服務之間具有相依性。 每個內部負載平衡器需要一個雲端服務。

在 Azure Resource Manager 中，每個 Azure 資源都需要放進 Azure 資源群組中，而這對 Azure Load Balancer 也有效。 不過，不需要讓每個 Azure Load Balancer 都有一個 Azure 資源群組，例如，一個 Azure 資源群組可以包含多個 Azure Load Balancer。 環境更簡單且更有彈性。 

### <a name="support-for-sap-multi-sid-scenarios"></a>支援 SAP 多 SID 案例

在 Azure Resource Manager 中，您可以在一個叢集中安裝多個 SAP 系統識別碼 (SID) ASCS/SCS 執行個體。 因為每個 Azure 內部負載平衡器的多個 IP 位址支援，可能有多重 SID 執行個體。

若要使用 Azure 傳統部署模型，請依照 [Azure 上的 SAP NetWeaver：在 Azure 上使用 Windows Server 容錯移轉叢集搭配 SIOS DataKeeper 將 SAP ASCS/SCS 執行個體組成叢集](http://go.microsoft.com/fwlink/?LinkId=613056)中所述的步驟操作。

> [!IMPORTANT]
> 強烈建議您針對 SAP 安裝使用 Azure Resource Manager 部署模型。 它提供傳統部署模型所沒有的許多好處。 進一步了解 Azure [部署模型][virtual-machines-azure-resource-manager-architecture-benefits-arm]。   
>
>

## <a name="8ecf3ba0-67c0-4495-9c14-feec1a2255b7"></a> Windows Server 容錯移轉叢集
Windows Server 容錯移轉叢集是 Windows 中高可用性 SAP ASCS/SCS 安裝和 DBMS 的基礎。

容錯移轉叢集是由 1+n 個獨立伺服器 (節點) 所組成的群組，這些伺服器會共同運作以提升應用程式和服務的可用性。 如果發生節點失敗，Windows Server 容錯移轉叢集會計算發生的失敗次數，同時維護狀況良好的叢集，以提供應用程式和服務。 您可以從不同的仲裁模式選擇以達成容錯移轉叢集。

### <a name="1a3c5408-b168-46d6-99f5-4219ad1b1ff2"></a> 仲裁模式
當您使用 Windows Server 容錯移轉叢集時，可以從四種仲裁模式中選擇：

* **節點多數**。 叢集的每個節點皆可投票。 只有取得多數票 (也就是票數過半) 時，叢集才能運作。 我們建議針對節點數目為奇數的叢集使用此選項。 例如，具有七個節點的叢集中有三個節點可能會失敗，而叢集仍會達成多數並繼續執行。  
* **節點與磁碟多數**。 每個節點和叢集儲存體中的指定磁碟 (磁碟見證) 處於可用且通訊中狀態時，都可投票。 只有取得多數票 (也就是票數過半) 時，叢集才能運作。 此模式適用於節點數目為偶數的叢集環境。 如果半數的節點和該磁碟在線上，叢集就會維持狀況良好的狀態。
* **節點與檔案共用多數**。 每個節點再加上一個由系統管理員所建立的指定檔案共用 (檔案共用見證) 無論節點和檔案共用是否處於可用且通訊中狀態，皆可投票。 只有取得多數票 (也就是票數過半) 時，叢集才能運作。 此模式適用於節點數目為偶數的叢集環境。 類似於「節點與磁碟多數」模式，但它使用見證檔案共享，而不是見證磁碟。 此模式很容易實作，但如果檔案共用本身不具有高可用性，它可能會變成單一失敗點。
* **無多數：僅磁碟**。 只要有一個節點可用且正與叢集儲存體中的特定磁碟進行通訊，叢集就會擁有仲裁。 只有也在與該磁碟進行通訊的節點能夠加入叢集。 建議您不要使用此模式。
 

## <a name="fdfee875-6e66-483a-a343-14bbaee33275"></a> Windows Server 容錯移轉叢集內部部署
圖 1 顯示兩個節點的叢集。 如果節點之間的網路連線失敗，而且兩個節點仍保持運作，仲裁磁碟或檔案共用會決定哪一個節點將繼續提供叢集的應用程式與服務。 能夠存取仲裁磁碟或檔案共用的節點就是可以確保服務可繼續的節點。

因為此範例使用雙節點叢集，所以我們使用「節點與檔案共用多數」仲裁模式。 「節點與磁碟多數」也是有效的選項。 在生產環境中，建議您使用仲裁磁碟。 您可以使用網路和儲存體系統技術，使其成為高可用性。

![圖 1：Azure 中 SAP ASCS/SCS 之 Windows Server 容錯移轉叢集組態的範例][sap-ha-guide-figure-1000]

_**圖 1：** Azure 中 SAP ASCS/SCS 之 Windows Server 容錯移轉叢集組態的範例_

### <a name="be21cf3e-fb01-402b-9955-54fbecf66592"></a> 共用儲存體
圖 1 也顯示兩個節點的共用儲存體叢集。 在內部部署共用儲存體叢集中，叢集中的所有節點會偵測共用儲存體。 鎖定機制可防止資料損毀。 所有節點也可以偵測到另一個節點是否發生失敗。 如果一個節點發生失敗，剩餘的節點就會取得儲存體資源的擁有權並確保服務的可用性。

> [!NOTE]
> 您不需要與某些 DBMS 應用程式 (例如 SQL Server) 共用提供高可用性的磁碟。 SQL Server Always On 會將 DBMS 資料及記錄檔，從一個叢集節點的本機磁碟複寫到另一個叢集節點的本機磁碟。 在此情況下，Windows 叢集組態不需要共用磁碟。
>
>

### <a name="ff7a9a06-2bc5-4b20-860a-46cdb44669cd"></a> 網路與名稱解析
用戶端電腦可透過 DNS 伺服器提供的虛擬 IP 位址和虛擬主機名稱連線叢集。 內部部署節點與 DNS 伺服器可以處理多個 IP 位址。

在一般設定中，您會使用兩個或更多個網路連線：

* 一個連接到儲存體的專用連線
* 一個用於活動訊號的叢集內部網路連線
* 一個供用戶端用來連線到叢集的公用網路

## <a name="2ddba413-a7f5-4e4e-9a51-87908879c10a"></a> Azure 中的 Windows Server 容錯移轉叢集
相較於裸機或私人雲端部署，「Azure 虛擬機器」需要額外的步驟來設定 Windows Server 容錯移轉叢集。 當您建置共用叢集磁碟時，需要為 SAP ASCS/SCS 執行個體設定數個 IP 位址和虛擬的主機名稱。

在本文中，我們會討論在 Azure 中建置 SAP 高可用性中心服務叢集時所需要的重要概念與其他步驟。 我們會為您示範如何設定協力廠商工具 SIOS DataKeeper，以及如何設定 Azure 內部負載平衡器。 您可以使用這些工具，在 Azure 中透過檔案共用見證建立 Windows 容錯移轉叢集。

![圖 2：Azure 中沒有共用磁碟的 Windows Server 容錯移轉叢集組態][sap-ha-guide-figure-1001]

_**圖 2：** Azure 中沒有共用磁碟的 Windows Server 容錯移轉叢集組態_

### <a name="1a464091-922b-48d7-9d08-7cecf757f341"></a> 使用 SIOS DataKeeper 在 Azure 中共用磁碟
您需要針對高可用性的 SAP ASCS/SCS 執行個體叢集共用儲存體。 自 2016 年 9 月起，Azure 即不再提供可用於建立共用儲存體叢集的共用儲存體。 您可以使用協力廠商軟體 SIOS DataKeeper Cluster Edition 以建立鏡像儲存體，以模擬叢集共用儲存體。 SIOS 解決方案提供即時同步的資料複寫。 可以為叢集建立共用磁碟資源的方式為：

1. 將額外的磁碟連接到 Windows 叢集組態中的每個虛擬機器 (VM)。
2. 在兩個虛擬機器節點上執行 SIOS DataKeeper Cluster Edition。
3. 設定 SIOS DataKeeper Cluster Edition，使它將來源虛擬機器的額外磁碟連接磁碟區內容鏡像至目標虛擬機器的額外磁碟連接磁碟區。 SIOS DataKeeper 會提取來源和目標本機磁碟區，並將它們當作一個共用磁碟來呈現給「Windows Server 容錯移轉叢集」。

取得 [SIOS DataKeeper](http://us.sios.com/products/datakeeper-cluster/)的詳細資訊。

![圖 3：Azure 中使用 SIOS DataKeeper 的 Windows Server 容錯移轉叢集組態][sap-ha-guide-figure-1002]

_**圖 3：** Azure 中使用 SIOS DataKeeper 的 Windows Server 容錯移轉叢集組態_

> [!NOTE]
> 您不需要與某些 DBMS 產品 (例如 SQL Server) 共用提供高可用性的磁碟。 SQL Server Always On 會將 DBMS 資料及記錄檔，從一個叢集節點的本機磁碟複寫到另一個叢集節點的本機磁碟。 在此情況下，Windows 叢集組態不需要共用磁碟。
>
>

### <a name="44641e18-a94e-431f-95ff-303ab65e0bcb"></a> Azure 中的名稱解析
Azure 雲端平台不提供設定虛擬 IP 位址的選項，例如浮動 IP 位址。 您需要一個替代解決方案來設定虛擬 IP，以便連線到雲端的叢集資源。
Azure 具有 Azure Load Balancer 服務中的內部負載平衡器。 使用內部負載平衡器，用戶端可透過叢集虛擬 IP 位址連線叢集。
您需要將內部負載平衡器部署在包含叢集節點的資源群組中。 接著，使用內部負載平衡器的探查連接埠來設定所有必要的連接埠轉送規則。
用戶端可以透過虛擬主機名稱來進行連線。 DNS 伺服器會解析叢集 IP 位址，而內部負載平衡器則會處理對作用中叢集節點的轉送。

## <a name="2e3fec50-241e-441b-8708-0b1864f66dfa"></a> Azure 基礎結構即服務 (IaaS) 中的 SAP NetWeaver 高可用性
若要達成 SAP 應用程式高可用性，例如 SAP 軟體元件，您需要保護下列元件：

* SAP 應用程式伺服器執行個體
* SAP ASCS/SCS 執行個體
* DBMS 伺服器

如需在高可用性案例中保護 SAP 元件的詳細資訊，請參閱 [SAP NetWeaverAzure 的虛擬機器規劃和實作][planning-guide-11]。

### <a name="93faa747-907e-440a-b00a-1ae0a89b1c0e"></a> 高可用性的 SAP 應用程式伺服器
對於 SAP 應用程式伺服器和對話方塊執行個體，您通常不需要特定的高可用性解決方案。 您可以透過備援來達成高可用性，而且您會在 Azure 虛擬機器之不同的執行個體中，設定多個對話方塊執行個體。 您應該至少要有兩個 SAP 應用程式執行個體安裝在 Azure 虛擬機器的兩個執行個體中。

![圖 4：高可用性的 SAP 應用程式伺服器][sap-ha-guide-figure-2000]

_**圖 4：** 高可用性的 SAP 應用程式伺服器_

所有裝載 SAP 應用程式伺服器執行個體的虛擬機器都必須放置在同一個 Azure 可用性設定組中。 Azure 可用性設定組可確保：

* 所有虛擬機器都是相同升級網域的一部分。 例如，升級網域可確保虛擬機器不會在計劃性維護停機期間同時更新。
* 所有虛擬機器都是相同容錯網域的一部分。 例如，容錯網域可確保部署虛擬機器，以便不會有任何單一失敗點影響所有虛擬機器的可用性。

深入了解如何[管理虛擬機器的可用性][virtual-machines-manage-availability]。

僅限非受控的磁碟：由於 Azure 儲存體帳戶是潛在的單一失敗點，因此您務必擁有至少兩個 Azure 儲存體帳戶，且至少要將兩個虛擬機器分散到其中。 在理想的設定中，執行 SAP 對話方塊執行個體的每一個虛擬機器磁碟會部署在不同的儲存體帳戶中。

### <a name="f559c285-ee68-4eec-add1-f60fe7b978db"></a> 高可用性的 SAP ASCS/SCS 執行個體
圖 5 是高可用性的 SAP ASCS/SCS 執行個體範例。

![圖 5：高可用性的 SAP ASCS/SCS 執行個體][sap-ha-guide-figure-2001]

_**圖 5：** 高可用性的 SAP ASCS/SCS 執行個體_

#### <a name="b5b1fd0b-1db4-4d49-9162-de07a0132a51"></a> 在 Azuer 中使用 Windows Server 容錯移轉叢集的 SAP ASCS/SCS 執行個體高可用性
相較於裸機或私人雲端部署，「Azure 虛擬機器」需要額外的步驟來設定 Windows Server 容錯移轉叢集。 為了建置 Windows 容錯移轉叢集，您需要一個共用叢集磁碟、數個 IP 位址、數個虛擬主機名稱，以及 Azure 內部負載平衡器，來將 SAP ASCS/SCS 執行個體組成叢集。 本文稍後會有詳細討論。

![圖 6：Azure 中使用 SIOS DataKeeper 之 SAP ASCS/SCS 的 Windows Server 容錯移轉叢集組態][sap-ha-guide-figure-1002]

_**圖 6：** Azure 中使用 SIOS DataKeeper 之 SAP ASCS/SCS 的 Windows Server 容錯移轉叢集組態_

### <a name="ddd878a0-9c2f-4b8e-8968-26ce60be1027"></a> 高可用性的 DBMS 執行個體
DBMS 也是 SAP 系統的單一連絡點。 您需要使用高可用性的解決方案來保護它。 圖 7 顯示在 Azure 中使用 Windows Server 容錯移轉叢集和 Azure 內部負載平衡器的 SQL Server Always On 高可用性解決方案的範例。 SQL Server Always On 會使用自己的 DBMS 複寫來複寫 DBMS 資料和記錄檔。 在此情況下，您不需要叢集共用磁碟，以簡化整個設定。

![圖 7：高可用性的 SAP DBMS，使用 SQL Server Always On 的範例][sap-ha-guide-figure-2003]

_**圖 7：** 高可用性的 SAP DBMS，使用 SQL Server Always On 的範例_

如需有關使用 Azure Resource Manager 部署模型在 Azure 中將 SQL Server 組成叢集的詳細資訊，請參閱下列文章：

* [用 Resource Manager 在 Azure 虛擬機器中設定 Always On 可用性群組][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]
* [在 Azure 中為 Always On 可用性群組設訂內部負載平衡器][virtual-machines-windows-portal-sql-alwayson-int-listener]

## <a name="045252ed-0277-4fc8-8f46-c5a29694a816"></a> 端對端高可用性部署案例

### <a name="deployment-scenario-using-architectural-template-1"></a>使用架構範本 1 的部署案例

圖 8 顯示**一個** SAP 系統在 Azure 中 SAP NetWeaver 高可用性架構的範例。 此案例如下所示設定︰

- 一個專用的叢集用於 SAP ASCS/SCS 執行個體。
- 一個專用的叢集用於 DBMS 執行個體。
- SAP 應用程式伺服器執行個體會部署在他們自己專用的 VM。

![圖 8：SAP 高可用性架構範本 1，包含用於 ASCS/SCS 和用於 DBMS 執行個體的專用叢集][sap-ha-guide-figure-2004]

_**圖 8：** SAP 高可用性架構範本 1，包含用於 ASCS/SCS 和用於 DBMS 執行個體的專用叢集_

### <a name="deployment-scenario-using-architectural-template-2"></a>使用架構範本 2 的部署案例

圖 9 顯示**一個** SAP 系統在 Azure 中 SAP NetWeaver 高可用性架構的範例。 此案例如下所示設定︰

- 一個專用的叢集用於 SAP ASCS/SCS 和 DBMS 執行個體。
- SAP 應用程式伺服器執行個體會部署在自己專用的 VM。

![圖 9：SAP 高可用性架構範本 2，包含用於 ASCS/SCS 的專用叢集和用於 DBMS 的專用叢集][sap-ha-guide-figure-2005]

_**圖 9：** SAP 高可用性架構範本 2，包含用於 ASCS/SCS 的專用叢集和用於 DBMS 的專用叢集_

### <a name="deployment-scenario-using-architectural-template-3"></a>使用架構範本 3 的部署案例

圖 10 顯示**兩個** SAP 系統與 &lt;SID1&gt; 和 &lt;SID2&gt;，在 Azure 中 SAP NetWeaver 高可用性架構的範例。 此案例如下所示設定︰

- 一個專用的叢集用於 SAP ASCS/SCS SID1 執行個體和 SAP ASCS/SCS SID2 執行個體 (一個叢集)。
- 一個專用的叢集用於 DBMS SID1，另一個專用的叢集用於 DBMS SID2 (兩個叢集)。
- SAP 系統 SID1 的 SAP 應用程式伺服器執行個體有自己專用的 VM。
- SAP 系統 SID2 的 SAP 應用程式伺服器執行個體有自己專用的 VM。

![圖 10：SAP 高可用性架構範本 3，包含用於不同 ASCS/SCS 執行個體的專用叢集][sap-ha-guide-figure-6003]

_**圖 10：** SAP 高可用性架構範本 3，包含用於不同 ASCS/SCS 執行個體的專用叢集_

## <a name="78092dbe-165b-454c-92f5-4972bdbef9bf"></a> 準備基礎架構

### <a name="prepare-the-infrastructure-for-architectural-template-1"></a>準備架構範本 1 的基礎結構
適用於 SAP 的 Azure Resource Manager 範本有助於簡化部署所需的資源。

Azure Resource Manager 中的三層式範本也支援高可用性案例，例如架構範本 1，有兩個叢集。 每個叢集是適用於 SAP ASCS/SCS 和 DBMS 的 SAP 單一失敗點。

以下是可取得我們在本文中說明之範例案例的 Azure Resource Manager 範本︰

* [Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image) \(英文\)  
* [使用受控磁碟的 Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md) \(英文\)  
* [自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image)
* [使用受控磁碟的自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md) \(英文\)

若要準備架構範本 1 的基礎結構：

- 在 Azure 入口網站中，在 [參數] 刀鋒視窗的 [SYSTEMAVAILABILITY] 方塊中，選取 [HA]。

  ![圖 11：設定 SAP 高可用性 Azure Resource Manager 參數][sap-ha-guide-figure-3000]

_**圖 11：** 設定 SAP 高可用性 Azure Resource Manager 參數_


  範本會建立：

  * **虛擬機器**：
    * SAP 應用程式伺服器虛擬機器︰<*SAPSystemSID*>-di-<*Number*>
    * ASCS/SCS 叢集虛擬機器︰<*SAPSystemSID*>-ascs-<*Number*>
    * DBMS 叢集：<*SAPSystemSID*>-db-<*Number*>

  * **所有虛擬機器的網路卡，包含關聯的 IP 位址**：
    * <*SAPSystemSID*>-nic-di-<*Number*>
    * <*SAPSystemSID*>-nic-ascs-<*Number*>
    * <*SAPSystemSID*>-nic-db-<*Number*>

  * **Azure 儲存體帳戶 (僅限非受控磁碟)**

  * 下列項目的**可用性群組**：
    * SAP 應用程式伺服器虛擬機器︰<*SAPSystemSID*>-avset-di
    * SAP ASCS/SCS 叢集虛擬機器︰<*SAPSystemSID*>-avset-ascs
    * DBMS 叢集虛擬機器︰<*SAPSystemSID*>-avset-db

  * **Azure 內部負載平衡器**：
    * 包含 ASCS/SCS 執行個體的所有連接埠與 IP 位址 <*SAPSystemSID*>-lb-ascs
    * 包含 SQL Server DBMS 的所有連接埠與 IP 位址 <*SAPSystemSID*>-lb-ascs

  * **網路安全性群組**：<*SAPSystemSID*>-nsg-ascs-0  
    * 包含向 <*SAPSystemSID*>-ascs-0 虛擬機器開放的外部遠端桌面通訊協定 (RDP) 連接埠

> [!NOTE]
> 網路卡的所有 IP 位址和 Azure 內部負載平衡器依預設為**動態**。 將其變更為**靜態** IP 位址。 本文稍後會描述做法。
>
>

### <a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a> 使用公司網路連線能力 (跨單位) 部署虛擬機器以用於生產環境中
針對生產環境 SAP 系統，使用 Azure 站對站 VPN 或 Azure ExpressRoute，部署具有 [公司網路連線能力 (跨單位)][planning-guide-2.2] 的 Azure 虛擬機器。

> [!NOTE]
> 您可以使用您的 Azure 虛擬網路執行個體。 虛擬網路和子網路已建立並備妥。
>
>

1.  在 Azure 入口網站中，在 [參數] 刀鋒視窗的 [NEWOREXISTINGSUBNET] 方塊中，選取 [存在]。
2.  在 [SUBNETID] 方塊中，新增已備妥的「Azure 網路 SubnetID」的完整字串，這是您打算用來部署 Azure 虛擬機器的位置。
3.  若要取得所有 Azure 網路子網路的清單，請執行此 PowerShell 命令：

  ```PowerShell
  (Get-AzureRmVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
  ```

  [ID] 欄位顯示 **SUBNETID**。
4. 若要取得所有 **SUBNETID** 值的清單，請執行此 PowerShell 命令：

  ```PowerShell
  (Get-AzureRmVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
  ```

  **SUBNETID** 看起來像這樣：

  ```
  /subscriptions/<SubscriptionId>/resourceGroups/<VPNName>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<SubnetName>
  ```

### <a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a> 用於測試和示範的 SAP 執行個體僅限部署雲端
您可以在僅限雲端部署模型中部署您的高可用性 SAP 系統。 這種部署主要對於示範和測試的使用案例相當有用。 但不適用於生產環境使用案例。

- 在 Azure 入口網站中，在 [參數] 刀鋒視窗的 [NEWOREXISTINGSUBNET] 方塊中，選取 [新增]。 將 [SUBNETID] 欄位保留空白。

  SAP Azure Resource Manager 範本將會自動建立 Azure 虛擬網路和子網路。

> [!NOTE]
> 您也需要在相同的 Azure 虛擬網路執行個體中，針對 Active Directory 和 DNS 部署至少一個專用的虛擬機器。 此範本不會建立這些虛擬機器。
>
>


### <a name="prepare-the-infrastructure-for-architectural-template-2"></a>準備架構範本 2 的基礎結構

您可以使用這個適用於 SAP 的 Azure Resource Manager 範本，以協助簡化部署必要的 SAP 架構範本 2 基礎結構資源。

您可以在這裡取得此部署案例的 Azure Resource Manager 範本：

* [Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged) \(英文\)  
* [使用受控磁碟的 Azure Marketplace 映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md) \(英文\)  
* [自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged)
* [使用受控磁碟的自訂映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md) \(英文\)


### <a name="prepare-the-infrastructure-for-architectural-template-3"></a>準備架構範本 3 的基礎結構

您可以準備基礎結構，並將 SAP 設定為**多 SID**。 例如，您可以將額外的 SAP ASCS/SCS 執行個體新增至現有的叢集組態以建立 SAP 多 SID 組態。 如需詳細資訊，請參閱[將額外的 SAP ASCS/SCS 執行個體設定為現有的叢集組態以在 Azure Resource Manager 中建立 SAP 多 SID 組態][sap-ha-multi-sid-guide]。

如果您想要建立新的多 SID 叢集，可以使用多 SID [GitHub 上的快速入門範本](https://github.com/Azure/azure-quickstart-templates)。
若要建立新的多 SID 叢集，您需要部署下列三個範本︰

* [ASCS/SCS 範本](#ASCS-SCS-template)
* [資料庫範本](#database-template)
* [應用程式伺服器範本](#application-servers-template)

下列各節有更多關於您需要在範本中提供之範本和參數的詳細資料。

#### <a name="ASCS-SCS-template">ASCS/SCS 範本</a>

ASCS/SCS 範本會部署兩部虛擬機器，可讓您用來建立裝載多個 ASCS/SCS 執行個體的 Windows Server 容錯移轉叢集。

若要設定 ASCS/SCS 多 SID 範本，請在 [ASCS/SCS 多 SID 範本][sap-templates-3-tier-multisid-xscs-marketplace-image]或[使用受控磁碟的 ASCS/SCS 多 SID 範本][sap-templates-3-tier-multisid-xscs-marketplace-image-md]中輸入下列參數的值︰

  - **資源前置詞**。  設定資源前置詞，其用來前置部署期間建立的所有資源。 因為資源不只屬於一個 SAP 系統，資源的前置詞不是一個 SAP 系統的 SID。  前置詞必須介於**三到六個字元**。
  - **堆疊類型**。 選取 SAP 系統的堆疊類型。 根據堆疊類型而定，Azure Load Balancer 每個 SAP 系統會有一個 (ABAP 或僅 Java) 或兩個 (ABAP + Java) 私人 IP 位址。
  -  **OS 類型**。 選取虛擬機器的作業系統。
  -  **SAP 系統計數**。 選取您想要安裝在此叢集中的 SAP 系統數目。
  -  **系統可用性**。 選取 **HA**。
  -  **管理員使用者名稱和管理員密碼**。 建立可用來登入電腦的新使用者。
  -  **新的或現有的子網路**。 設定應該建立新的虛擬網路和子網路，還是應該使用現有子網路。 如果您已經有連接到內部部署網路的虛擬網路，請選取**現有**虛擬網路。
  -  **子網路識別碼**。如果您想將 VM 部署至現有的 VNet (其中具有定義 VM 應指派的目的子網路)，請說明該特定子網路的 ID。 該 ID 通常看起來像這樣：/subscriptions/<*訂閱 ID*>/resourceGroups/<*資源群組名稱*>/providers/Microsoft.Network/virtualNetworks/<*虛擬網路名稱*>/subnets/<*子網路名稱*>

範本會部署一個 Azure Load Balancer 執行個體，可支援多個 SAP 系統。

- 針對執行個體號碼 00、10、20... 設定的 ASCS 執行個體
- 針對執行個體號碼 01、11、21... 設定的 ASCS 執行個體
- 針對執行個體號碼 02、12、22... 設定的 ASCS 加入佇列複寫伺服器 (ERS) (僅限 Linux) 執行個體
- 針對執行個體號碼 03、13、23... 設定的 SCS ERS (僅限 Linux) 執行個體

負載平衡器包含 1 個 (2 個適用於 Linux) VIP、1 個 VIP 適用於 ASCS/SCS 和1 個 VIP 適用於 ERS (僅 Linux)。

下列清單包含所有的負載平衡規則 (其中 x 是 SAP 系統的數目，例如 1、2、3...)︰
- 每個 SAP 系統的 Windows 特定連接埠︰445、5985
- ASCS 連接埠 (執行個體數目 x0)︰32x0、36x0、39x0、81x0、5x013、5x014、5x016
- SCS 連接埠 (執行個體數目 x1)︰32x1、33x1、39x1、81x1、5x113、5x114、5x116
- Linux 上的 ASCS ERS 連接埠 (執行個體數目 x2)︰33x2、5x213、5x214、5x216
- Linux 上的 SCS ERS 連接埠 (執行個體數目 x3)︰33x3、5x313、5x314、5x316

負載平衡器設定為使用下列的探查連接埠 (其中 x 是 SAP 系統的數目，例如 1、2、3...)︰
- ASCS/SCS 內部負載平衡器探查連接埠︰620x0
- ERS 內部負載平衡器探查連接埠 (僅 Linux)︰621x2

#### <a name="database-template"></a> 資料庫範本

資料庫範本會部署一個或兩個虛擬機器，可讓您用來安裝一個 SAP 系統的關聯式資料庫管理系統 (RDBMS)。 例如，如果您部署五個 SAP 系統的 ASCS/SCS 範本，您需要部署此範本五次。

若要設定資料庫多 SID 範本，請在[資料庫多 SID 範本][sap-templates-3-tier-multisid-db-marketplace-image]或[使用受控磁碟的資料庫多 SID 範本][sap-templates-3-tier-multisid-db-marketplace-image-md]中輸入下列參數的值︰

  -  **SAP 系統識別碼**。輸入您想要安裝的 SAP 系統之 SAP 系統識別碼。 識別碼將用於已部署資源的前置詞。
  -  **OS 類型**。 選取虛擬機器的作業系統。
  -  **Dbtype**。 選取您想要在叢集上安裝的資料庫類型。 如果您想要安裝 Microsoft SQL Server，選取 **SQL**。 如果您打算在虛擬機器上安裝 SAP HANA，選取 **HANA**。 請務必選取正確的作業系統類型︰針對 SQL 選取 **Windows**，針對 HANA 選取 Linux 散發套件。 連線到虛擬機器的 Azure Load Balancer 會設定為支援所選取的資料庫類型︰
    * **SQL**。 負載平衡器會負載平衡連接埠 1433。 請務必針對您的 SQL Server Always On 設定使用此連接埠。
    * **HANA**。 負載平衡器會負載平衡連接埠 35015 和 35017。 請務必以執行個體數目 **50** 安裝 SAP HANA。
    負載平衡器會使用探查連接埠 62550。
  -  **SAP 系統大小**。 設定新系統將提供的 SAP 數量。 如果您不確定系統將需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
  -  **系統可用性**。 選取 **HA**。
  -  **管理員使用者名稱和管理員密碼**。 建立可用來登入電腦的新使用者。
  -  **子網路識別碼**。輸入您在部署 ASCS/SCS 範本期間使用的子網路識別碼，或做為 ASCS/SCS 範本部署的一部分建立之子網路識別碼。

#### <a name="application-servers-template"></a> 應用程式伺服器範本

應用程式伺服器範本會部署兩個或多個虛擬機器，可用來做為一個 SAP 系統的 SAP 應用程式伺服器執行個體。 例如，如果您部署五個 SAP 系統的 ASCS/SCS 範本，您需要部署此範本五次。

若要設定應用程式伺服器多 SID 範本，請在[應用程式伺服器多 SID 範本][sap-templates-3-tier-multisid-apps-marketplace-image]或[使用受控磁碟的應用程式伺服器多 SID 範本][sap-templates-3-tier-multisid-apps-marketplace-image-md]中輸入下列參數的值︰

  -  **SAP 系統識別碼**。輸入您想要安裝的 SAP 系統之 SAP 系統識別碼。 識別碼將用於已部署資源的前置詞。
  -  **OS 類型**。 選取虛擬機器的作業系統。
  -  **SAP 系統大小**。 新系統將提供的 SAP 數量。 如果您不確定系統將需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
  -  **系統可用性**。 選取 **HA**。
  -  **管理員使用者名稱和管理員密碼**。 建立可用來登入電腦的新使用者。
  -  **子網路識別碼**。輸入您在部署 ASCS/SCS 範本期間使用的子網路識別碼，或做為 ASCS/SCS 範本部署的一部分建立之子網路識別碼。


### <a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Azure 虛擬網路
在我們的範例中，Azure 虛擬網路的位址空間是 10.0.0.0/16。 有一個名為 **Subnet**、網址範圍為 10.0.0.0/24 的子網路。 所有虛擬機器和內部負載平衡器會部署在此虛擬網路中。

> [!IMPORTANT]
> 請勿對客體作業系統內的網路設定進行任何變更。 包括 IP 位址、DNS 伺服器和子網路。 在 Azure 中設定所有網路設定。 動態主機設定通訊協定 (DHCP) 服務會散佈您的設定。
>
>

### <a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a> DNS IP 位址

若要設定所需的 DNS IP 位址，請執行下列步驟。

1.  在 Azure 入口網站的 [DNS 伺服器] 刀鋒視窗上，確定您的虛擬網路 [DNS 伺服器] 選項是設定為 [自訂 DNS]。
2.  根據您擁有的網路類型選取設定。 如需詳細資訊，請參閱下列資源：
    * [公司網路連線能力 (跨單位)][planning-guide-2.2]︰請新增內部部署 DNS 伺服器的 IP 位址。  
    您可以將內部部署 DNS 伺服器擴充至 Azure 中執行的虛擬機器。 在這種情況下，您可以加入執行 DNS 服務之 Azure 虛擬機器的 IP 位址。
    * [僅限雲端部署][planning-guide-2.1]：在做為 DNS 伺服器的相同虛擬網路執行個體中，部署其他虛擬機器。 新增您已設定執行 DNS 服務之 Azure 虛擬機器的 IP 位址。

    ![圖 12：設定 Azure 虛擬網路的 DNS 伺服器][sap-ha-guide-figure-3001]

    _**圖 12：** 設定 Azure 虛擬網路的 DNS 伺服器_

  > [!NOTE]
  > 如果您變更 DNS 伺服器的 IP 位址，您就必須重新啟動 Azure 虛擬機器，才能套用變更並傳播新的 DNS 伺服器。
  >
  >

在我們的範例中，是在下列 Windows 虛擬機器上安裝並設定 DNS 服務：

| 虛擬機器角色 | 虛擬機器主機名稱 | 網路卡名稱 | 靜態 IP 位址 |
| --- | --- | --- | --- |
| 第一部 DNS 伺服器 |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| 第二部 DNS 伺服器 |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

### <a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a> SAP ASCS/SCS 叢集執行個體及 DBMS 叢集執行個體的主機名稱和靜態 IP 位址

對於內部部署，您需要下列保留的主機名稱和 IP 位址：

| 虛擬主機名稱角色 | 虛擬主機名稱 | 虛擬靜態 IP 位址 |
| --- | --- | --- |
| SAP ASCS/SCS 第一個叢集虛擬主機名稱 (用於叢集管理) |pr1-ascs-vir |10.0.0.42 |
| SAP ASCS/SCS 執行個體虛擬主機名稱 |pr1-ascs-sap |10.0.0.43 |
| SAP DBMS 第二個叢集虛擬主機名稱 (叢集管理) |pr1-dbms-vir |10.0.0.32 |

當您建立叢集時，請建立虛擬主機名稱 **pr1-ascs-vir** 和 **pr1-dbms-vir**，以及管理叢集本身的相關 IP 位址。 如需如何執行這項操作的詳細資訊，請參閱[在叢集組態中收集叢集節點][sap-ha-guide-8.12.1]。

您可以在 DNS 伺服器上手動建立其他兩個虛擬主機名稱，**pr1-ascs-sap** 和 **pr1-dbms-sap**，以及相關的 IP 位址。 叢集 SAP ASCS/SCS 執行個體和叢集 DBMS 執行個體使用這些資源。 如需如何執行這項操作的詳細資訊，請參閱[建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱][sap-ha-guide-9.1.1]。

### <a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a> 設定 SAP 虛擬機器的靜態 IP 位址
部署要在叢集中使用的虛擬機器之後，您需要設定所有虛擬機器的靜態 IP 位址。 在 Azure 虛擬網路組態中執行此動作，而非在客體作業系統中執行此動作。

1.  在 Azure 入口網站中選取 [資源群組] > [網路卡] > [設定] > [IP 位址]。
2.  在 [IP 位址] 刀鋒視窗 [指派] 下，選取 [靜態]。 在 [IP 位址] 方塊中，輸入您要使用的 IP 位址。

  > [!NOTE]
  > 如果您變更網路卡的 IP 位址，您就必須重新啟動 Azure 虛擬機器，才能套用變更。  
  >
  >

  ![圖 13：為每個虛擬機器的網路卡設定靜態 IP 位址][sap-ha-guide-figure-3002]

  _**圖 13：** 為每個虛擬機器的網路卡設定靜態 IP 位址_

  為所有網路介面重複此步驟，亦即為所有虛擬機器，包括您要用於 Active Directory/DNS 服務的虛擬機器。

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

### <a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a> 為 Azure 內部負載平衡器設定靜態 IP 位址

SAP Azure Resource Manager 範本會建立用於 SAP ASCS/SCS 執行個體叢集和用於 DBMS 叢集的 Azure 內部負載平衡器。

> [!IMPORTANT]
> SAP ASCS/SCS 之虛擬主機名稱的 IP 位址與 SAP ASCS/SCS 內部負載平衡器：**pr1-lb-ascs** 的 IP 位址相同。
> DBMS 之虛擬名稱的 IP 位址與 DBMS 內部負載平衡器：**pr1-lb-dbms** 的 IP 位址相同。
>
>

若要為 Azure 內部負載平衡器設定靜態 IP 位址：

1.  初始部署會將內部負載平衡器的 IP 位址設定為 [動態]。 在 Azure 入口網站 [IP 位址] 刀鋒視窗 [指派] 下，選取 [靜態]。
2.  請將「內部負載平衡器」**pr1-lb-ascs** 的 IP 位址設定為 SAP ASCS/SCS 執行個體的虛擬主機名稱 IP 位址。
3.  請將「內部負載平衡器」**pr1-lb-dbms** 的 IP 位址設定為 DBMS 執行個體的虛擬主機名稱 IP 位址。

  ![圖 14：為 SAP ASCS/SCS 執行個體的內部負載平衡器設定靜態 IP 位址][sap-ha-guide-figure-3003]

  _**圖 14：** 為 SAP ASCS/SCS 執行個體的內部負載平衡器設定靜態 IP 位址_

在我們的範例中，有兩個 Azure 內部負載平衡器具備這些靜態 IP 位址：

| Azure 內部負載平衡器角色 | Azure 內部負載平衡器名稱 | 靜態 IP 位址 |
| --- | --- | --- |
| SAP ASCS/SCS 執行個體內部負載平衡器 |pr1-lb-ascs |10.0.0.43 |
| SAP DBMS 內部負載平衡器 |pr1-lb-dbms |10.0.0.33 |


### <a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a> Azure 內部負載平衡器的預設 ASCS/SCS 負載平衡規則

SAP Azure Resource Manager 範本會建立所需的連接埠：
* ABAP ASCS 執行個體預設的執行個體號碼為 **00**
* Java SCS 執行個體預設的執行個體號碼為 **01**

當您安裝 SAP ASCS/SCS 執行個體時，必須為 ABAP ASCS 執行個體使用預設的執行個體號碼 **00**，並為 Java SCS 執行個體使用預設的執行個體號碼 **01**。

接著，為 SAP NetWeaver 連接埠建立下列必要的內部負載平衡端點。

若要建立必要的內部負載平衡端點，首先，為 SAP NetWeaver ABAP ASCS 連接埠建立下列必要的內部負載平衡端點：

| 服務/負載平衡規則名稱 | 預設連接埠號碼 | (執行個體號碼為 00 的 ASCS 執行個體) (ERS 為 10) 的具體連接埠 |
| --- | --- | --- |
| 加入佇列伺服器 / *lbrule3200* |32<*InstanceNumber*> |3200 |
| ABAP 訊息伺服器 / *lbrule3600* |36<*InstanceNumber*> |3600 |
| 內部 ABAP 訊息 / *lbrule3900* |39<*InstanceNumber*> |3900 |
| 訊息伺服器 HTTP / *Lbrule8100* |81<*InstanceNumber*> |8100 |
| SAP 啟動服務 ASCS HTTP / *Lbrule50013* |5<*InstanceNumber*>13 |50013 |
| SAP 啟動服務 ASCS HTTPS / *Lbrule50014* |5<*InstanceNumber*>14 |50014 |
| 加入佇列複寫 / *Lbrule50016* |5<*InstanceNumber*>16 |50016 |
| SAP 啟動服務 ERS HTTP *Lbrule51013* |5<*InstanceNumber*>13 |51013 |
| SAP 啟動服務 ERS HTTP *Lbrule51014* |5<*InstanceNumber*>14 |51014 |
| Win RM *Lbrule5985* | |5985 |
| 檔案共用 *Lbrule445* | |445 |

_**表 1：** SAP NetWeaver ABAP ASCS 執行個體的連接埠號碼_

接著，為 SAP NetWeaver Java SCS 連接埠建立下列負載平衡端點：

| 服務/負載平衡規則名稱 | 預設連接埠號碼 | (執行個體號碼為 01 的 SCS 執行個體) (ERS 為 11) 的具體連接埠 |
| --- | --- | --- |
| 加入佇列伺服器 / *lbrule3201* |32<*InstanceNumber*> |3201 |
| 閘道伺服器 / *lbrule3301* |33<*InstanceNumber*> |3301 |
| Java 訊息伺服器 / *lbrule3900* |39<*InstanceNumber*> |3901 |
| 訊息伺服器 HTTP / *Lbrule8101* |81<*InstanceNumber*> |8101 |
| SAP 啟動服務 SCS HTTP / *Lbrule50113* |5<*InstanceNumber*>13 |50113 |
| SAP 啟動服務 SCS HTTPS / *Lbrule50114* |5<*InstanceNumber*>14 |50114 |
| 加入佇列複寫 / *Lbrule50116* |5<*InstanceNumber*>16 |50116 |
| SAP 啟動服務 ERS HTTP *Lbrule51113* |5<*InstanceNumber*>13 |51113 |
| SAP 啟動服務 ERS HTTP *Lbrule51114* |5<*InstanceNumber*>14 |51114 |
| Win RM *Lbrule5985* | |5985 |
| 檔案共用 *Lbrule445* | |445 |

_**表 2：** SAP NetWeaver Java SCS 執行個體的連接埠號碼_

![圖 15：Azure 內部負載平衡器的預設 ASCS/SCS 負載平衡規][sap-ha-guide-figure-3004]

_**圖 15：** Azure 內部負載平衡器的預設 ASCS/SCS 負載平衡規_

將負載平衡器 **pr1-lb-dbms** 的 IP 位址設定為 DBMS 執行個體之虛擬主機名稱的 IP 位址。

### <a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> 變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則

如果您想要將其他號碼用於 SAP ASCS 或 SCS 執行個體，您就必須從預設值變更其連接埠的名稱和值。

1.  在 Azure 入口網站中，選取 [<*SID*>-lb-ascs 負載平衡器] > [負載平衡規則]。
2.  針對屬於 SAP ASCS 或 SCS 執行個體的所有負載平衡規則，變更下列值：

  * 名稱
  * Port
  * 後端連接埠

  例如，如果您要將預設 ASCS 執行個體號碼從 00 變更為 31，您需要針對表 1 中所列的所有通訊埠進行變更。

  以下是連接埠 *lbrule3200*的更新範例。

  ![圖 16：變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則][sap-ha-guide-figure-3005]

  _**圖 16：** 變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則_

### <a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> 將 Windows 虛擬機器新增至網域

將靜態 IP 位址指派給虛擬機器之後，請將虛擬機器新增至網域。

![圖 17：將虛擬機器新增至網域][sap-ha-guide-figure-3006]

_**圖 17：** 將虛擬機器新增至網域_

### <a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> 在 SAP ASCS/SCS 執行個體的兩個叢集節點上都新增登錄項目

Azure Load Balancer 具有內部負載平衡器，會在連線閒置一段時間 (閒置逾時) 時關閉連線。 對話方塊執行個體中的 SAP 工作程序則會在需要傳送第一個加入佇列/清除佇列要求時，立即開啟對SAP 加入佇列程序的連線。 這些連線通常會持續維持已建立狀態，直到工作程序或加入佇列程序重新啟動為止。 不過，如果連線閒置一段時間，Azure 內部負載平衡器就會關閉連線。 這不是問題，因為如果連線不存在，SAP 工作程序會對加入佇列程序重新建立連線。 這些活動會記載於 SAP 程序的開發人員追蹤，但它們會對這些追蹤建立大量額外的內容。 因此，最好先變更兩個叢集節點上的 TCP/IP `KeepAliveTime` 和 `KeepAliveInterval`。 結合這些 TCP/IP 參數中的變更與 SAP 設定檔參數中的變更，稍後在本文中會加以描述。

若要在 SAP ASCS/SCS 執行個體的兩個叢集節點上新增登錄項目，首先，在 SAP ASCS/SCS 的兩個 Windows 叢集節點上新增這些 Windows 登錄項目︰

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 變數名稱 |`KeepAliveTime` |
| 變數類型 |REG_DWORD (十進位) |
| 值 |120000 |
| 文件連結 |[https://technet.microsoft.com/library/cc957549.aspx](https://technet.microsoft.com/library/cc957549.aspx) |

_**表 3：** 變更第一個 TCP/IP 參數_

然後，在 SAP ASCS/SCS 的兩個 Windows 叢集節點上都新增下列 Windows 登錄項目：

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 變數名稱 |`KeepAliveInterval` |
| 變數類型 |REG_DWORD (十進位) |
| 值 |120000 |
| 文件連結 |[https://technet.microsoft.com/library/cc957548.aspx](https://technet.microsoft.com/library/cc957548.aspx) |

_**表 4：** 變更第二個 TCP/IP 參數_

**若要套用變更，請重新啟動這兩個叢集節點**。

### <a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> 設定 SAP ASCS/SCS 執行個體的 Windows Server 容錯移轉叢集

設定 SAP ASCS/SCS 執行個體的 Windows Server 容錯移轉叢集包含下列工作︰

- 收集叢集組態中的叢集節點
- 設定叢集檔案共用見證

#### <a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a> 收集叢集組態中的叢集節點

1.  在新增角色及功能精靈中，將容錯移轉叢集新增至這兩個叢集節點。
2.  使用 [容錯移轉叢集管理員] 來設定容錯移轉叢集。 在容錯移轉叢集管理員中，選取 [建立叢集]，然後僅新增第一個叢集、節點 A 的名稱。還不要新增第二個節點；您將在稍後的步驟新增第二個節點。

  ![圖 18：加入第一個叢集節點的伺服器或虛擬機器名稱][sap-ha-guide-figure-3007]

  _**圖 18：** 加入第一個叢集節點的伺服器或虛擬機器名稱_

3.  輸入叢集的網路名稱 (虛擬主機名稱)。

  ![圖 19︰輸入叢集名稱][sap-ha-guide-figure-3008]

  _**圖 19︰** 輸入叢集名稱_

4.  建立叢集之後，會執行叢集驗證測試。

  ![圖 20：執行叢集驗證檢查][sap-ha-guide-figure-3009]

  _**圖 20：** 執行叢集驗證檢查_

  您可以忽略此時程序中有關磁碟的任何警告。 稍後您將新增檔案共用見證與 SIOS 共用磁碟。 在這個階段，您不必擔心有仲裁。

  ![圖 21：找不到仲裁磁碟][sap-ha-guide-figure-3010]

  _**圖 21：**_

  ![圖 22：核心叢集資源需要新的 IP 位址][sap-ha-guide-figure-3011]

  _**圖 22：** 核心叢集資源需要新的 IP 位址_

5.  變更核心叢集服務的 IP 位址。 直到您變更核心叢集服務的 IP 位址之前皆無法啟動叢集，因為伺服器的 IP 位址指向其中一個虛擬機器節點。 在核心叢集服務 IP 資源的**屬性**頁面上執行此動作。

  例如，我們需要為叢集虛擬主機名稱 **pr1-ascs-vir** 指派 IP 位址 (在我們的範例中為 **10.0.0.42**)。

  ![圖 23：在 [屬性] 對話方塊中，變更 IP 位址][sap-ha-guide-figure-3012]

  _**圖 23：** 在 [屬性]  對話方塊中，變更 IP 位址_

  ![圖 24：指派為叢集保留的 IP 位址][sap-ha-guide-figure-3013]

  _**圖 24：** 指派為叢集保留的 IP 位址_

6.  將叢集虛擬主機名稱上線。

  ![圖 25：叢集核心服務以正確的 IP 位址啟動並正常執行][sap-ha-guide-figure-3014]

  _**圖 25：** 叢集核心服務以正確的 IP 位址啟動並正常執行_

7.  新增第二個叢集節點。

  既然核心叢集服務已啟動並正常執行，您便可以新增第二個叢集節點。

  ![圖 26：新增第二個叢集節點][sap-ha-guide-figure-3015]

  _**圖 26：**_

8.  輸入第二個叢集節點主機的名稱。

  ![圖 27︰輸入第二個叢集節點主機名稱][sap-ha-guide-figure-3016]

  _**圖 27︰** 輸入第二個叢集節點主機名稱_

  > [!IMPORTANT]
  > 請確定**沒有**選取 [新增適合的儲存裝置到叢集] 核取方塊。  
  >
  >

  ![圖 28：請勿選取核取方塊][sap-ha-guide-figure-3017]

  _**圖 28：** 請**勿**選取核取方塊_

  您可以忽略有關仲裁和磁碟的警告。 您將在稍後設定仲裁和共用磁碟，如[為 SAP ASCS/SCS 叢集共用磁碟安裝 SIOS DataKeeper Cluster Edition][sap-ha-guide-8.12.3] 中所述。

  ![圖 29：忽略有關磁碟仲裁的警告][sap-ha-guide-figure-3018]

  _**圖 29：** 忽略有關磁碟仲裁的警告_


#### <a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a> 設定叢集檔案共用見證

設定叢集檔案共用見證包含下列工作︰

- 建立檔案共用
- 在容錯移轉叢集管理員中設定檔案共用見證仲裁

##### <a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a> 建立檔案共用

1.  選取檔案共用見證，而不是仲裁磁碟。 SIOS DataKeeper 支援此選項。

  在本文的範例中，檔案共用見證是在 Azure 中執行的 Active Directory/DNS 伺服器上。 檔案共用見證稱為 **domcontr-0**。 因為您已設定 Azure 的 VPN 連線 (透過網站間 VPN 或 Azure ExpressRoute)，所以您的 Active Directory/DNS 服務是內部部署，而且不適用於執行檔案共用見證。

  > [!NOTE]
  > 如果您的 Active Directory/DNS 服務僅在內部部署上執行，請勿在執行內部部署的 Active Directory/DNS Windows 作業系統上設定檔案共用見證。 在 Azure 和 Active Directory DNS 內部部署中執行之叢集節點之間的網路延遲可能太大，而且會造成連線問題。 請務必在執行位置靠近叢集節點的 Azure 虛擬機器上設定檔案共用見證。  
  >
  >

  仲裁磁碟機至少需要 1,024 MB 的可用空間。 我們建議使用 2,048 MB 的仲裁磁碟機可用空間。

2.  新增叢集名稱物件。

  ![圖 30：為叢集名稱物件指派共用上的權限][sap-ha-guide-figure-3019]

  _**圖 30：** 為叢集名稱物件指派共用上的權限_

  請確定權限能夠為叢集名稱物件 (在我們的範例中為 **pr1-ascs-vir$**) 變更共用中的資料。

3.  若要將叢集名稱物件新增至清單中，請選取 [新增]。 變更篩選，以檢查電腦物件，除了圖 31 所示的項目以外。

  ![圖 31：變更物件類型以包含電腦][sap-ha-guide-figure-3020]

  _**圖 31：** 變更物件類型以包含電腦_

  ![圖 32︰選取電腦核取方塊][sap-ha-guide-figure-3021]

  _**圖 32︰** 選取**電腦**核取方塊_

4.  輸入叢集名稱物件，如圖 31 所示。 由於已經建立記錄，因此您可以變更權限，如圖 30 所示。

5.  選取共用的 [安全性] 索引標籤，然後為叢集名稱物件設定更詳細的權限。

  ![圖 33：為叢集名稱物件設定檔案共用仲裁上的安全性屬性][sap-ha-guide-figure-3022]

  _**圖 33：** 為叢集名稱物件設定檔案共用仲裁上的安全性屬性_

##### <a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a> 在容錯移轉叢集管理員中設定檔案共用見證仲裁

1.  開啟設定仲裁設定精靈。

  ![圖 34：啟動設定叢集仲裁設定精靈][sap-ha-guide-figure-3023]

  _**圖 34：** 啟動設定叢集仲裁設定精靈_

2.  在 [選取仲裁設定] 頁面上，選取 [選取仲裁見證]。

  ![圖 35：您可以選擇的仲裁組態][sap-ha-guide-figure-3024]

  _**圖 35：** 您可以選擇的仲裁組態_

3.  在 [選取仲裁見證] 頁面上，選取 [設定檔案共用見證]。

  ![圖 36︰選取檔案共用見證][sap-ha-guide-figure-3025]

  _**圖 36︰** 選取檔案共用見證_

4.  輸入檔案共用的 UNC 路徑 (在我們的範例中為 \\domcontr-0\FSW)。 若要查看您可進行變更的清單，選取 [下一步]。

  ![圖 37：定義見證共用的檔案共用位置][sap-ha-guide-figure-3026]

  _**圖 37：** 定義見證共用的檔案共用位置_

5.  選取您要的變更，然後選取 [下一步]。 您需要成功重新設定叢集組態，如圖 38 中所示。  

  ![圖 38：確認您已重新設定叢集][sap-ha-guide-figure-3027]

  _**圖 38：** 確認您已重新設定叢集_

成功安裝 Windows 容錯移轉叢集之後，有些閾值需要變更，讓容錯移轉偵測適應 Azure 中的條件。 需要變更的參數記載於此部落格中： https://blogs.msdn.microsoft.com/clustering/2012/11/21/tuning-failover-cluster-network-thresholds/。 假設為 ASCS/SCS 建置 Windows 叢集組態的兩部 VM 位於相同子網路中，則必須將下列參數變更為這些值︰
- SameSubNetDelay = 2
- SameSubNetThreshold = 15

這些設定已經過客戶測試，一方面提供有足夠彈性的折衷辦法。 另一方面，這些設定會在 SAP 軟體或節點/VM 失敗的真實錯誤情況中，提供夠快的容錯移轉。 

### <a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> 為 SAP ASCS/SCS 叢集共用磁碟安裝 SIOS DataKeeper Cluster Edition

您現在在 Azure 中有運作中的 Windows Server 容錯移轉叢集組態。 但是，若要安裝 SAP ASCS/SCS 執行個體，您需要有一個共用磁碟資源。 您無法在 Azure 中建立所需的共用磁碟資源。 SIOS DataKeeper Cluster Edition 是一種可用來建立共用磁碟資源的協力廠商解決方案。

為 SAP ASCS/SCS 叢集共用磁碟安裝 SIOS DataKeeper Cluster Edition 包含下列工作︰

- 新增 .NET Framework 3.5
- 安裝 SIOS DataKeeper
- 設定 SIOS DataKeeper

#### <a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a> 新增 .NET Framework 3.5
Microsoft .NET Framework 3.5 不會自動啟用或安裝在 Windows Server 2012 R2。 因為 SIOS DataKeeper 需要 .NET Framework 位於您安裝 DataKeeper 的所有節點上，您必須在叢集中所有虛擬機器的客體作業系統上安裝 .NET Framework 3.5。

有兩種方式可以新增 .NET Framework 3.5：

- 在 Windows 中使用 [新增角色及功能精靈]，如圖 39 所示。

  ![圖 39：使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5][sap-ha-guide-figure-3028]

  _**圖 39：** 使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5_

  ![圖 40︰當您使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5 時的安裝進度列][sap-ha-guide-figure-3029]

  _**圖 40︰** 當您使用 [新增角色及功能精靈] 安裝 .NET Framework 3.5 時的安裝進度列_

- 使用命令列工具 dism.exe。 針對這類型的安裝，您需要存取 Windows 安裝媒體上的 SxS 目錄。 在提高權限的命令提示字元上，輸入：

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

#### <a name="dd41d5a2-8083-415b-9878-839652812102"></a> 安裝 SIOS DataKeeper

在叢集中的每個節點上安裝 SIOS DataKeeper Cluster Edition。 若要使用 SIOS DataKeeper 建立虛擬共用儲存體，請建立同步處理的鏡像，然後模擬叢集共用儲存體。

安裝 SIOS 軟體之前，請先建立網域使用者 **DataKeeperSvc**。

> [!NOTE]
> 將 **DataKeeperSvc** 使用者同時新增到兩個叢集節點上的 [本機管理員] 群組中。
>
>

若要安裝 SIOS DataKeeper：

1.  在兩個叢集節點上都安裝 SIOS 軟體。

  ![SIOS 安裝程式][sap-ha-guide-figure-3030]

  ![圖 41：第一個 SIOS DataKeeper 安裝頁面][sap-ha-guide-figure-3031]

  _**圖 41：** 第一個 SIOS DataKeeper 安裝頁面_

2.  在圖 42 所示的對話方塊中，選取 [是]。

  ![圖 42：DataKeeper 通知將停用某個服務][sap-ha-guide-figure-3032]

  _**圖 42：** DataKeeper 通知將停用某個服務_

3.  在圖 43 顯示的對話方塊中，建議您選取 [網域或伺服器帳戶]。

  ![圖 43：SIOS DataKeeper 的使用者選項][sap-ha-guide-figure-3033]

  _**圖 43：** SIOS DataKeeper 的使用者選項_

4.  輸入您為 SIOS DataKeeper 建立的網域帳戶使用者名稱和密碼。

  ![圖 44：為 SIOS DataKeeper 安裝輸入網域使用者名稱和密碼][sap-ha-guide-figure-3034]

  _**圖 44：** 為 SIOS DataKeeper 安裝輸入網域使用者名稱和密碼_

5.  為您的 SIOS DataKeeper 執行個體安裝授權金鑰，如圖 45 所示。

  ![圖 45：輸入您的 SIOS DataKeeper 授權金鑰][sap-ha-guide-figure-3035]

  _**圖 45：** 輸入您的 SIOS DataKeeper 授權金鑰_

6.  出現提示時，重新啟動虛擬機器。

#### <a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a> 設定 SIOS DataKeeper

在兩個節點上都安裝 SIOS DataKeeper 之後，您必須開始設定組態。 組態的目標是要在連接至每個虛擬機器的額外磁碟之間進行同步資料複寫。

1.  啟動 DataKeeper 管理與組態工具，然後選取 [連線伺服器]。 (在圖 46 中，此選項是用紅色圈起來的部分。)

  ![圖 46：SIOS DataKeeper 管理和組態工具][sap-ha-guide-figure-3036]

  _**圖 46：** SIOS DataKeeper 管理和組態工具_

2.  輸入管理和組態工具應連接之第一個節點的名稱或 TCP/IP 位址，然後在第二個步驟中，插入第二個節點。

  ![圖 47：插入管理和組態工具應連接之第一個節點的名稱或 TCP/IP 位址，然後在第二個步驟中，插入第二個節點][sap-ha-guide-figure-3037]

  _**圖 47：** 插入管理和組態工具應連接之第一個節點的名稱或 TCP/IP 位址，然後在第二個步驟中，插入第二個節點_

3.  在兩個節點之間建立複寫作業。

  ![圖 48：建立複寫作業][sap-ha-guide-figure-3038]

  _**圖 48：** 建立複寫作業_

  精靈會引導您完成建立複寫作業的程序。
4.  定義名稱、TCP/IP 位址和來源節點的磁碟區。

  ![圖 49：定義複寫作業的名稱][sap-ha-guide-figure-3039]

  _**圖 49：** 定義複寫作業的名稱_

  ![圖 50：定義節點 (應該是目前的來源節點) 的基底資料][sap-ha-guide-figure-3040]

  _**圖 50：** 定義節點 (應該是目前的來源節點) 的基底資料_

5.  定義名稱、TCP/IP 位址和目標節點的磁碟區。

  ![圖 51：定義節點 (應該是目前的目標節點) 的基底資料][sap-ha-guide-figure-3041]

  _**圖 51：** 定義節點 (應該是目前的目標節點) 的基底資料_

6.  定義壓縮演算法。 在我們的範例中，建議壓縮複寫資料流。 尤其是在重新同步處理的情況下，壓縮複寫資料流可大幅縮短重新同步處理時間。 請注意，壓縮會使用虛擬機器的 CPU 和 RAM 資源。 隨著壓縮率增加，使用的 CPU 資源數量也跟著增加。 您也可以稍後調整此設定。

7.  另一個您需要檢查的設定是複寫是以非同步方式還是同步方式進行。 *當您保護 SAP ASCS/SCS 組態時，您必須使用同步複寫*。  

  ![圖 52︰定義複寫詳細資料][sap-ha-guide-figure-3042]

  _**圖 52︰** 定義複寫詳細資料_

8.  定義是否應該向 Windows Server 容錯移轉叢集的叢集組態指出複寫作業所複寫的磁碟區是共用磁碟。 就 SAP ASCS/SCS 組態而言，選取 [是] ，如此 Windows 叢集才會將複寫的磁碟區視為可用來作為叢集磁碟區的共用磁碟。

  ![圖 53：選取 [是] 以將複寫的磁碟區設定為叢集磁碟區][sap-ha-guide-figure-3043]

  _**圖 53：** 選取 [是] 以將複寫的磁碟區設定為叢集磁碟區_

  建立磁碟區之後，DataKeeper 管理和組態工具會顯示複寫作業是使用中。

  ![圖 54：SAP ASCS/SCS 共用磁碟的 DataKeeper 同步鏡像處於作用中狀態][sap-ha-guide-figure-3044]

  _**圖 54：** SAP ASCS/SCS 共用磁碟的 DataKeeper 同步鏡像處於作用中狀態_

  「容錯移轉叢集管理員」現在會顯示磁碟為 DataKeeper 磁碟，如圖 55 所示。

  ![圖 55：「容錯移轉叢集管理員」顯示 DataKeeper 複寫的磁碟][sap-ha-guide-figure-3045]

  _**圖 55：**「容錯移轉叢集管理員」顯示 DataKeeper 複寫的磁碟_

## <a name="a06f0b49-8a7a-42bf-8b0d-c12026c5746b"></a> 安裝 SAP NetWeaver 系統

我們將不會說明 DBMS 設定，因為設定會視您使用的 DBMS 系統而異。 不過，我們會假設 DBMS 在高可用性方面的疑慮已藉由不同 DBMS 廠商為 Azure 提供的功能支援而獲得解決。 例如，適用於 SQL Server 的 Always On 或資料庫鏡像，以及適用於 Oracle 資料庫的 Oracle Data Guard。 在本文使用的案例中，我們並未對 DBMS 加入更多保護。

當不同的 DBMS 服務與 Azure 中這種叢集 SAP ASCS/SCS 組態互動時，沒有任何特殊的考量。

> [!NOTE]
> SAP NetWeaver ABAP 系統、Java 系統及 ABAP+Java 系統的安裝程序幾乎完全相同。 最顯著的差異在於 SAP ABAP 系統有一個 ASCS 執行個體。 SAP Java 系統有一個 SCS 執行個體。 SAP ABAP+Java 系統有一個 ASCS 執行個體和一個 SCS 執行個體在相同 Microsoft 容錯移轉叢集群組中執行。 我們會明確說明每個 SAP NetWeaver 安裝堆疊的所有安裝差異。 您可以假設所有其他部分都相同。  
>
>

### <a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a> 使用高可用性 ASCS/SCS 執行個體安裝 SAP

> [!IMPORTANT]
> 請不要將分頁檔放在 DataKeeper 鏡像磁碟區上。 DataKeeper 不支援鏡像磁碟區。 您可以將分頁檔留在 Azure 虛擬機器的暫存磁碟機 D 中，此為預設值。 如果還不在此磁碟機中，請將 Windows 分頁檔移至 Azure 虛擬機器的磁碟機 D:。
>
>

使用高可用性 ASCS/SCS 執行個體安裝 SAP 包含下列工作︰

- 建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱
- 安裝 SAP 的第一個叢集節點
- 修改 ASCS/SCS 執行個體的 SAP 設定檔
- 新增探查連接埠
- 開啟 Windows 防火牆探查連接埠

#### <a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a> 建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱

1.  在「Windows DNS 管理員」中，為 ASCS/SCS 執行個體的虛擬主機名稱建立 DNS 項目。

  > [!IMPORTANT]
  > 您指派給 ASCS/SCS 執行個體之虛擬主機名稱的 IP 位址，必須與指派給 Azure Load Balancer (**<*SID*>-lb-ascs**) 的 IP 位址相同。  
  >
  >

  虛擬 SAP ASCS/SCS 主機名稱 (**pr1-ascs-sap**) 的 IP 位址與 Azure Load Balancer (**pr1-lb-ascs**) 的 IP 位址相同。

  ![圖 56：定義 SAP ASCS/SCS 叢集虛擬名稱和 TCP/IP 位址的 DNS 項目][sap-ha-guide-figure-3046]

  _**圖 56：** 定義 SAP ASCS/SCS 叢集虛擬名稱和 TCP/IP 位址的 DNS 項目_

2.  若要定義指派給虛擬主機名稱的 IP 位址，選取 [DNS 管理員] > [網域]

  ![圖 57：SAP ASCS/SCS 叢集組態的新虛擬名稱和 TCP/IP 位址][sap-ha-guide-figure-3047]

  _**圖 57：** SAP ASCS/SCS 叢集組態的新虛擬名稱和 TCP/IP 位址_

#### <a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> 安裝 SAP 的第一個叢集節點

1.  在叢集節點 A 上執行第一個叢集節點選項。例如，在 **pr1-ascs-0** 主機上。
2.  如果要為 Azure 內部負載平衡器保留預設連接埠，請選取：

  * **ABAP 系統**：**ASCS** 執行個體號碼 **00**
  * **Java 系統**：**SCS** 執行個體號碼 **01**
  * **ABAP+Java 系統**：**ASCS** 執行個體號碼 **00** 和 **SCS** 執行個體號碼 **01**

  如果要針對 ABAP ASCS 執行個體使用 00 以外的執行個體號碼，以及針對 Java SCS 執行個體使用 01 以外的執行個體號碼，您必須先變更 Azure 內部負載平衡器預設的負載平衡規則，如[變更 Azure 內部負載平衡器的 ASCS/SCS 預設負載平衡規則所述][sap-ha-guide-8.9]。

接下來幾個步驟並未在標準 SAP 安裝文件中說明。

> [!NOTE]
> SAP 安裝文件說明如何安裝第一個 ASCS/SCS 叢集節點。
>
>

#### <a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> 修改 ASCS/SCS 執行個體的 SAP 設定檔

您需要新增新的設定檔參數。 此設定檔參數可避免 SAP 工作程序與加入佇列伺服器之間的連線在閒置時間太長時關閉。 我們在本文中的[在 SAP ASCS/SCS 執行個體的兩個叢集節點上都新增登錄項目][sap-ha-guide-8.11]已提到問題案例。 在該部分，我們也介紹了對一些基本 TCP/IP 連線參數所做的兩項變更。 在第二個步驟中，您必須設定讓加入佇列伺服器傳送 `keep_alive` 訊號，如此連線才不會達到 Azure 內部負載平衡器的閒置臨界值。

若要修改 ASCS/SCS 執行個體的 SAP 設定檔：

1.  將此設定檔參數新增至 SAP ASCS/SCS 執行個體設定檔：

  ```
  enque/encni/set_so_keepalive = true
  ```
  在我們的範例中，路徑為：

  `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`

  例如，新增到 SAP SCS 執行個體設定檔和對應的路徑：

  `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`

2.  若要套用變更，請重新啟動 SAP ASCS /SCS 執行個體。

#### <a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> 新增探查連接埠

使用內部負載平衡器探查功能，讓整個叢集組態使用 Azure Load Balancer。 Azure 內部負載平衡器通常會在參與的虛擬機器之間，平均分配內送的工作負載。 不過，這對某些叢集組態不會產生作用，因為只有一個執行個體是主動的。 另一個執行個體是被動的，而且不接受任何工作負載。 Azure 內部負載平衡器僅將工作指派給主動的執行個體時，探查功能才可提供協助。 使用探查功能，內部負載平衡器便能夠偵測哪些執行個體是主動的，然後只以該執行個體作為工作負載的目標。

若要新增探查連接埠︰

1.  執行 PowerShell 命令檢查目前 **ProbePort** 設定。 在叢集組態中的其中一個虛擬機器中執行該檢查。

  ```PowerShell
  $SAPSID = "PR1"     # SAP <SID>

  $SAPNetworkIPClusterName = "SAP $SAPSID IP"
  Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter
  ```

2.  定義探查連接埠。 預設探查連接埠號碼為 **0**。 在我們的範例中，我們會使用探查連接埠 **62000**。

  ![圖 58：叢集組態探查連接埠預設為 0][sap-ha-guide-figure-3048]

  _**圖 58：** 預設叢集組態探查連接埠為 0_

  SAP Azure Resource Manager 範本中已定義連接埠號碼。 您可以在 PowerShell 中指派連接埠號碼。

  若要針對 **SAP <*SID*> IP** 叢集資源設定新的 ProbePort 值，執行下列 PowerShell 指令碼。 請更新您環境的 PowerShell 變數。 指令碼執行之後，系統會提示您重新啟動 SAP 叢集群組，以啟動變更。

  ```PowerShell
  $SAPSID = "PR1"      # SAP <SID>
  $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

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

  讓 **SAP <*SID*>** 叢集角色上線之後，請確認已將 **ProbePort** 設定為新值。

  ```PowerShell
  $SAPSID = "PR1"     # SAP <SID>

  $SAPNetworkIPClusterName = "SAP $SAPSID IP"
  Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter

  ```

  ![圖 59：設定新值之後，探查叢集連接埠][sap-ha-guide-figure-3049]

  _**圖 59：** 設定新值之後，探查叢集連接埠_

#### <a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a> 開啟 Windows 防火牆探查連接埠

您需要開啟這兩個叢集節點上的 Windows 防火牆探查連接埠。 使用下列指令碼開啟 Windows 防火牆探查連接埠。 請更新您環境的 PowerShell 變數。

  ```PowerShell
  $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

  New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

**ProbePort** 已設為 **62000**。 現在您可以從其他主機存取檔案共用 **\\\ascsha-clsap\sapmnt**，例如從 **ascsha-dbas**。

### <a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> 安裝資料庫執行個體

若要安裝資料庫執行個體，請依照 SAP 安裝文件中所述的程序。

### <a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> 安裝第二個叢集節點

若要安裝第二個叢集，請依照 SAP 安裝指南中的步驟。

### <a name="094bc895-31d4-4471-91cc-1513b64e406a"></a> 變更 SAP ERS Windows 服務執行個體的啟動類型

將兩個叢集節點上 SAP ERS Windows 服務的啟動類型都變更為 [自動 (延遲啟動)]。

![圖 60：將 SAP ERS 執行個體的服務類型變更為延遲的自動類型][sap-ha-guide-figure-3050]

_**圖 60：** 將 SAP ERS 執行個體的服務類型變更為延遲的自動類型_

### <a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> 安裝 SAP 主要應用程式伺服器

在您指派來裝載 PAS 的虛擬機器上安裝主要應用程式伺服器 (PAS) 執行個體 <*SID*>-di-0。 Azure 或 DataKeeper 特定的設定沒有相依性。

### <a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a> 安裝 SAP 其他應用程式伺服器

在您指派來裝載 SAP 應用程式伺服器執行個體的所有虛擬機器上安裝 SAP 其他應用程式伺服器 (AAS)。 例如，在 <SID>-di-1 到 <SID>-di-&lt;n&gt;。

> [!NOTE]
> 這樣就完成高可用性的 SAP NetWeaver 系統的安裝。 接下來，繼續進行容錯移轉測試。
>


## <a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> 測試 SAP ASCS/SCS 執行個體容錯移轉和 SIOS 複寫
使用「容錯移轉叢集管理員」和 SIOS DataKeeper 管理和組態工具，可輕鬆地測試及監視 SAP ASCS/SCS 執行個體容錯移轉和 SIOS 磁碟複寫。

### <a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a> SAP ASCS/SCS 執行個體在叢集節點 A 上執行

**SAP PR1**叢集群組在叢集節點 A 上執行，例如，在 **pr1-ascs-0** 上。 將屬於 **SAP PR1** 叢集群組並由 ASCS/SCS 執行個體使用的共用磁碟 S，指派給叢集節點 A。

![圖 61：容錯移轉叢集管理員︰SAP <SID> 叢集群組在叢集節點 A 上執行][sap-ha-guide-figure-5000]

_**圖 61：** 容錯移轉叢集管理員︰SAP <SID> 叢集群組在叢集節點 A 上執行_

在 SIOS DataKeeper 管理和組態工具中，您可以看到共用磁碟資料以同步方式從叢集節點 A 上的來源磁碟區 S 複寫到叢集節點 B 上的目標磁碟區 S。例如，從 **pr1-ascs-0 [10.0.0.40]** 複寫到 **pr1-ascs-1 [10.0.0.41]**。

![圖 62：在 SIOS DataKeeper 中，將本機磁碟區從叢集節點 A 的磁碟區複寫到叢集節點 B][sap-ha-guide-figure-5001]

_**圖 62：** 在 SIOS DataKeeper 中，將本機磁碟區從叢集節點 A 的磁碟區複寫到叢集節點 B_

### <a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a> 從節點 A 到節點 B 進行容錯移轉

1.  使用下列其中一個選項以起始將 SAP <SID> 叢集群組從叢集節點 A 到叢集節點 B 的容錯移轉：
  - 使用容錯移轉叢集管理員  
  - 使用容錯移轉叢集 PowerShell

  ```PowerShell
  $SAPSID = "PR1"     # SAP <SID>

  $SAPClusterGroup = "SAP $SAPSID"
  Move-ClusterGroup -Name $SAPClusterGroup

  ```
2.  在 Windows 客體作業系統內重新啟動叢集節點 A (這會起始將 SAP <SID> 叢集群組從節點 A 容錯移轉到節點 B 的自動容錯移轉)。  
3.  從 Azure 入口網站重新啟動叢集節點 A (這會起始將 SAP <SID> 叢集群組從節點 A 到節點 B 的自動容錯移轉)。  
4.  使用 Azure PowerShell 重新啟動叢集節點 A (這會起始將 SAP <SID> 叢集群組從節點 A 到節點 B 的自動容錯移轉)。

  容錯移轉之後，SAP <SID> 叢集群組會在叢集節點 B 上執行。例如，在 **pr1-ascs-1** 上執行。

  ![圖 63：在「容錯移轉叢集管理員」中，SAP <SID> 叢集群組在叢集節點 B 上執行][sap-ha-guide-figure-5002]

  _**圖 63**：在「容錯移轉叢集管理員」中，SAP <*SID*> 叢集群組在叢集節點 B 上執行_

  共用磁碟現在已掛接在叢集節點 B 上。SIOS DataKeeper 正在將資料從叢集節點 B 上的來源磁碟區 S 複寫到叢集節點 A 上的目標磁碟區 S。例如，從 **pr1-ascs-1 [10.0.0.41]** 複寫到 **pr1-ascs-0 [10.0.0.40]**。

  ![圖 64：SIOS DataKeeper 將本機磁碟區從叢集節點 B 複寫到叢集節點 A][sap-ha-guide-figure-5003]

  _**圖 64：** SIOS DataKeeper 將本機磁碟區從叢集節點 B 複寫到叢集節點 A_
