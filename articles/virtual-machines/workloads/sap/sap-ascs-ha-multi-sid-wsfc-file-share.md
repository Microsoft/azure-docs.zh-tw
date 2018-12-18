---
title: 在 Azure 上搭配 Windows Server 容錯移轉叢集和檔案共用之 SAP ASCS/SCS 執行個體的多重 SID 高可用性 | Microsoft Docs
description: 在 Azure 上搭配 Windows Server 容錯移轉叢集和檔案共用之 SAP ASCS/SCS 執行個體的多重 SID 高可用性
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: cbf18abe-41cb-44f7-bdec-966f32c89325
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1e21357eeb795a26874cddb90b4d3a6303b83ac0
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189628"
---
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1869038]:https://launchpad.support.sap.com/#/notes/1869038
[2287140]:https://launchpad.support.sap.com/#/notes/2287140
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[sap-official-ha-file-share-document]:https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html
[s2d-in-win-2016]:https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview
[sofs-overview]:https://technet.microsoft.com/library/hh831349(v=ws.11).aspx
[new-in-win-2016-storage]:https://docs.microsoft.com/windows-server/storage/whats-new-in-storage

[sap-installation-guides]:http://service.sap.com/instguides
[sap-installation-guides-file-share]:https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html
[networking-limits-azure-resource-manager]:../../../azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md
[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md
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

[sap-ascs-ha-multi-sid-wsfc-shared-disk]:sap-ascs-ha-multi-sid-wsfc-shared-disk.md
[sap-ascs-ha-multi-sid-wsfc-shared-disk-infrast-prepare]:sap-ascs-ha-multi-sid-wsfc-shared-disk.md#25e358f8-92e5-4e8d-a1e5-df7580a39cb0
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

[sap-ha-guide-figure-6001]:media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png

[sap-ha-guide-figure-8001]:./media/virtual-machines-shared-sap-high-availability-guide/8001.png
[sap-ha-guide-figure-8002]:./media/virtual-machines-shared-sap-high-availability-guide/8002.png
[sap-ha-guide-figure-8003]:./media/virtual-machines-shared-sap-high-availability-guide/8003.png
[sap-ha-guide-figure-8004]:./media/virtual-machines-shared-sap-high-availability-guide/8004.png
[sap-ha-guide-figure-8005]:./media/virtual-machines-shared-sap-high-availability-guide/8005.png
[sap-ha-guide-figure-8006]:./media/virtual-machines-shared-sap-high-availability-guide/8006.png
[sap-ha-guide-figure-8007]:./media/virtual-machines-shared-sap-high-availability-guide/8007.png
[sap-ha-guide-figure-8008]:./media/virtual-machines-shared-sap-high-availability-guide/8008.png
[sap-ha-guide-figure-8009]:./media/virtual-machines-shared-sap-high-availability-guide/8009.png
[sap-ha-guide-figure-8010]:./media/virtual-machines-shared-sap-high-availability-guide/8010.png
[sap-ha-guide-figure-8011]:./media/virtual-machines-shared-sap-high-availability-guide/8011.png
[sap-ha-guide-figure-8012]:./media/virtual-machines-shared-sap-high-availability-guide/8012.png
[sap-ha-guide-figure-8013]:./media/virtual-machines-shared-sap-high-availability-guide/8013.png
[sap-ha-guide-figure-8014]:./media/virtual-machines-shared-sap-high-availability-guide/8014.png
[sap-ha-guide-figure-8015]:./media/virtual-machines-shared-sap-high-availability-guide/8015.png
[sap-ha-guide-figure-8016]:./media/virtual-machines-shared-sap-high-availability-guide/8016.png
[sap-ha-guide-figure-8017]:./media/virtual-machines-shared-sap-high-availability-guide/8017.png
[sap-ha-guide-figure-8018]:./media/virtual-machines-shared-sap-high-availability-guide/8018.png
[sap-ha-guide-figure-8019]:./media/virtual-machines-shared-sap-high-availability-guide/8019.png
[sap-ha-guide-figure-8020]:./media/virtual-machines-shared-sap-high-availability-guide/8020.png
[sap-ha-guide-figure-8021]:./media/virtual-machines-shared-sap-high-availability-guide/8021.png
[sap-ha-guide-figure-8022]:./media/virtual-machines-shared-sap-high-availability-guide/8022.png
[sap-ha-guide-figure-8023]:./media/virtual-machines-shared-sap-high-availability-guide/8023.png
[sap-ha-guide-figure-8024]:./media/virtual-machines-shared-sap-high-availability-guide/8024.png
[sap-ha-guide-figure-8025]:./media/virtual-machines-shared-sap-high-availability-guide/8025.png


[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/resource-group-overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md

# <a name="sap-ascsscs-instance-multi-sid-high-availability-with-windows-server-failover-clustering-and-file-share-on-azure"></a>在 Azure 上搭配 Windows Server 容錯移轉叢集和檔案共用的 SAP ASCS/SCS 執行個體多重 SID 高可用性

> ![Windows][Logo_Windows] Windows
>

Microsoft 在 2016 年 9 月發行的功能，可讓您使用 [Azure 內部負載平衡器][load-balancer-multivip-overview]管理多個虛擬 IP 位址。 這項功能已存在 Azure 外部負載平衡器。

如果您有 SAP 部署，可以使用內部負載平衡器，建立 SAP Central Services (ASCS/SCS) 執行個體的 Windows 叢集組態。

本文將著重於如何將單一 ASCS/SCS 安裝移至 SAP 多 SID 組態，方法是使用**檔案共用**，將其他 SAP ASCS/SCS 叢集執行個體安裝至現有 Windows Server 容錯移轉叢集 (WSFC) 叢集。 完成此程序之後，您將已設定 SAP 多重 SID 叢集。

> [!NOTE]
>
> 這項功能僅適用於 Azure Resource Manager 部署模型。
>
>每個 Azure 內部負載平衡器的私人前端 IP 都有數量限制。
>
>一個 WSFC 叢集中 SAP ASCS/SCS 執行個體數目上限等於每個 Azure 內部負載平衡器的私人前端 IP 數目上限。
>

如需負載平衡器限制的詳細資訊，請參閱[網路限制：Azure Resource Manager][networking-limits-azure-resource-manager] 中的「每個負載平衡器的私人前端 IP」一節。

## <a name="prerequisites"></a>必要條件

您已經使用**檔案共用**來設定要用於一個 SAP ASCS/SCS 執行個體的 WSFC 叢集，如下圖所示。

![圖 1：在兩個叢集中部署 SAP ASCS/SCS 執行個體和 SOFS 部署][sap-ha-guide-figure-8007]

_**圖 1：** 在兩個叢集中部署 SAP ASCS/SCS 執行個體和 SOFS 部署_

> [!IMPORTANT]
> 安裝程式必須符合下列條件︰
> * SAP ASCS / SCS 執行個體必須共用相同的 WSFC 叢集。
> * 屬於不同 SAP SID 的不同 SAP 全域主機檔案共用，必須共用相同的 SOFS 叢集。
> * 每個資料庫管理系統 (DBMS) SID 必須有自己專用的 WSFC 叢集。
> * SAP 應用程式伺服器屬於必須擁有自己專用 VM 的一個 SAP 系統 SID。

## <a name="sap-ascsscs-multi-sid-architecture-with-file-share"></a>使用檔案共用的 SAP ASCS/SCS 多重 SID 架構

目標是在相同 WSFC 叢集中安裝多個 SAP 進階商業應用程式程式設計 (ASCS) 或 SAP Java (SCS) 叢集執行個體，如這裡所詳述： 

![圖 2：兩個叢集中的 SAP 多重 SID 設定][sap-ha-guide-figure-8008]

_**圖 2：** 兩個叢集中的 SAP 多重 SID 設定_

安裝其他 **SAP \<SID2>** 系統等同於安裝一個 <SID> 系統。 ASCS/SCS 叢集以及檔案共用 SOFS 叢集上需要另兩個準備步驟。

## <a name="prepare-the-infrastructure-for-an-sap-multi-sid-scenario"></a>準備 SAP 多重 SID 案例的基礎結構

### <a name="prepare-the-infrastructure-on-the-domain-controller"></a>準備網域控制站上的基礎結構

建立網域群組 **\<Domain>\SAP_\<SID2>_GlobalAdmin**，例如，使用 \<SID2> = PR2。 網域群組名稱是 \<Domain>\SAP_PR2_GlobalAdmin。

### <a name="prepare-the-infrastructure-on-the-ascsscs-cluster"></a>準備 ASCS/SCS 叢集上的基礎結構

您必須針對第二個 SAP \<SID>，準備現有 SAP ASCS/SCS 叢集上的基礎結構：

* 在 DNS 伺服器上建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱。
* 使用 PowerShell 將 IP 位址新增至現有的 Azure 內部負載平衡器。

[SAP 多重 SID 案例的基礎結構準備][sap-ascs-ha-multi-sid-wsfc-shared-disk-infrast-prepare]中說明這些步驟。


### <a name="prepare-the-infrastructure-on-an-sofs-cluster-by-using-the-existing-sap-global-host"></a>使用現有的 SAP 全域主機準備 SOFS 叢集上的基礎結構

您可以重複使用現有的 \<SAPGlobalHost> 和第一個 SAP <SID1> 系統的 Volume1。

![圖 3：多重 SID SOFS 與 SAP 全域主機名稱相同][sap-ha-guide-figure-8014]

_**圖 3：** 多重 SID SOFS 與 SAP 全域主機名稱相同_

> [!IMPORTANT]
>對於第二個 **SAP \<SID2>** 系統，使用的是相同的 Volume1 及相同的 **\<SAPGlobalHost>** 網路名稱。
>因為您已將 **SAPMNT** 設定為各種不同 SAP 系統的共用名稱，因此若要重複使用 **\<SAPGlobalHost>** 網路名稱，您必須使用相同的 **Volume1**。
>
><SID2> 全域主機的檔案路徑為 C:\ClusterStorage\\**Volume1**\usr\sap\<SID2>\SYS\.
>

針對 \<SID2> 系統，您必須準備 SAP 全域主機 ..\SYS\.. SOFS 叢集上的資料夾。

若要為 \<SID2> 執行個體準備 SAP 全域主機，請執行下列 PowerShell 指令碼：


```PowerShell
##################
# SAP multi-SID
##################

$SAPSID2 = "PR2"
$DomainName2 = "SAPCLUSTER"
$SAPSIDGlobalAdminGroupName2 = "$DomainName2\SAP_" + $SAPSID2 + "_GlobalAdmin"

# SAP ASCS/SCS cluster nodes
$ASCSCluster2Node1 = "ja1-ascs-0"
$ASCSCluster2Node2 = "ja1-ascs-1"

# Define the SAP ASCS/SCS cluster node computer objects
$ASCSCluster2ObjectNode1 = "$DomainName2\$ASCSCluster2Node1$"
$ASCSCluster2ObjectNode2 = "$DomainName2\$ASCSCluster2Node2$"

# Create usr\sap\.. folders on CSV
$SAPGlobalFolder2 = "C:\ClusterStorage\Volume1\usr\sap\$SAPSID2\SYS"
New-Item -Path $SAPGlobalFolder2 -ItemType Directory

# Add permissions for the SAP SID2 system
Grant-SmbShareAccess -Name sapmnt -AccountName $SAPSIDGlobalAdminGroupName2, $ASCSCluster2ObjectNode1, $ASCSCluster2ObjectNode2 -AccessRight Full -Force


$UsrSAPFolder = "C:\ClusterStorage\Volume1\usr\sap\"

# Set file and folder security
$Acl = Get-Acl $UsrSAPFolder

# Add the security object of the SAP_<sid>_GlobalAdmin group
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($SAPSIDGlobalAdminGroupName2,"FullControl", 'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode1$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSCluster2ObjectNode1,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode2$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSCluster2ObjectNode2,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Set security
Set-Acl $UsrSAPFolder $Acl -Verbose
```

### <a name="prepare-the-infrastructure-on-the-sofs-cluster-by-using-a-different-sap-global-host"></a>使用不同的 SAP 全域主機準備 SOFS 叢集上的基礎結構

您可以設定第二個 SOFS (例如，針對第二個 **\<SID2>**，使用 **\<SAPGlobalHost2>** 和不同的 **Volume2** 來設定第二個 SOFS 叢集)。

![圖 4：多重 SID SOFS 與 SAP GLOBAL 主機名稱 2 相同][sap-ha-guide-figure-8015]

_**圖 4：** 多重 SID SOFS 與 SAP GLOBAL 主機名稱 2 相同_

若要使用 \<SAPGlobalHost2> 建立第二個 SOFS 角色，請執行這個 PowerShell 指令碼：

```PowerShell
# Create SOFS with SAP Global Host Name 2
$SAPGlobalHostName = "sapglobal2"
Add-ClusterScaleOutFileServerRole -Name $SAPGlobalHostName
```

建立第二個 **Volume2**。 執行這個 PowerShell 指令碼：

```PowerShell
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName SAPPR2 -FileSystem CSVFS_ReFS -Size 5GB -ResiliencySettingName Mirror
```

![圖 5：多重 SID SOFS 與 SAP GLOBAL 主機名稱 2 相同][sap-ha-guide-figure-8016]

_**圖 5：**「容錯移轉叢集管理員」中的第二個 Volume2_

針對第二個 \<SID2>，建立 SAP GLOBAL 資料夾並設定檔案安全性。

執行這個 PowerShell 指令碼：

```PowerShell
# Create a folder for <SID2> on a second Volume2 and set file security
$SAPSID = "PR2"
$DomainName = "SAPCLUSTER"
$SAPSIDGlobalAdminGroupName = "$DomainName\SAP_" + $SAPSID + "_GlobalAdmin"

# SAP ASCS/SCS cluster nodes
$ASCSClusterNode1 = "ascs-1"
$ASCSClusterNode2 = "ascs-2"

# Define SAP ASCS/SCS cluster node computer objects
$ASCSClusterObjectNode1 = "$DomainName\$ASCSClusterNode1$"
$ASCSClusterObjectNode2 = "$DomainName\$ASCSClusterNode2$"

# Create usr\sap\.. folders on CSV
$SAPGlobalFolder = "C:\ClusterStorage\Volume2\usr\sap\$SAPSID\SYS"
New-Item -Path $SAPGlobalFOlder -ItemType Directory

$UsrSAPFolder = "C:\ClusterStorage\Volume2\usr\sap\"

# Set file and folder security
$Acl = Get-Acl $UsrSAPFolder

# Add the file security object of the SAP_<sid>_GlobalAdmin group
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($SAPSIDGlobalAdminGroupName,"FullControl", 'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode1$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSClusterObjectNode1,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode2$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSClusterObjectNode2,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Set security
Set-Acl $UsrSAPFolder $Acl -Verbose
```

若要針對第二個 SAP \<SID2>，在包含 *\<SAPGlobalHost2>* 主機名稱的 Volume2 上建立 SAPMNT 檔案共用，請在「容錯移轉叢集管理員」中啟動 [新增檔案共用] 精靈。

在 **saoglobal2** SOFS 叢集群組上按一下滑鼠右鍵，然後選取 [新增檔案共用]。

![圖 6：啟動 [新增檔案共用] 精靈][sap-ha-guide-figure-8017]

_**圖 6：** 啟動 [新增檔案共用] 精靈_

<br>
![圖 7：「選取 SMB 共用 - 快速」][sap-ha-guide-figure-8018]

_**圖 7：** 選取 [SMB 共用 - 快速]_

<br>
![圖 8：選取 [sapglobalhost2] 並在 Volume2 上指定路徑][sap-ha-guide-figure-8019]

_**圖 8：** 選取 [sapglobalhost2] 並在 Volume2 上指定路徑_

<br>
![圖 9：將檔案共用名稱設定為 [sapmnt]][sap-ha-guide-figure-8020]

_**圖 9：** 將檔案共用名稱設定為「sapmnt」_

<br>
![圖 10：停用所有設定][sap-ha-guide-figure-8021]

_**圖 10：** 停用所有設定_

<br>
請針對下列項目將 [ 完全控制] 權限指派給檔案和 sapmnt 共用：
* **SAP_\<SID>_GlobalAdmin** 網域使用者群組
* ASCS/SCS 叢集節點 **ascs-1$** 和 **ascs-2$** 的電腦物件

![圖 11：將完全控制權限指派給使用者群組和電腦帳戶][sap-ha-guide-figure-8022]

_**圖 11：** 將「完全控制」指派給使用者群組和電腦帳戶_

<br>
![圖 12：選取 [建立]][sap-ha-guide-figure-8023]

_**圖 12：** 選取 [建立]_

<br>
![圖 13：已建立繫結至 sapglobal2 主機和 Volume2 的第二個 sapmnt][sap-ha-guide-figure-8024]

_**圖 13：** 已建立繫結至 sapglobal2 主機和 Volume2 的第二個 sapmnt_

<br>
## <a name="install-sap-netweaver-multi-sid"></a>安裝 SAP NetWeaver 多重 SID

### <a name="install-sap-sid2-ascsscs-and-ers-instances"></a>安裝 SAP \<SID2> ASCS/SCS 和 ERS 執行個體

遵循先前針對一個 SAP \<SID> 所說明的相同安裝和設定步驟。

### <a name="install-dbms-and-sap-application-servers"></a>安裝 DBMS 和 SAP 應用程式伺服器
安裝的 DBMS 和 SAP 應用程式伺服器，如先前所述。

## <a name="next-steps"></a>後續步驟

* [在沒有共用磁碟的容錯移轉叢集上安裝 ASCS/SCS 執行個體][sap-official-ha-file-share-document]：HA 檔案共用的官方 SAP 指導方針

* [Windows Server 2016 中的儲存空間直接存取][s2d-in-win-2016]

* [用於應用程式資料的向外延展檔案伺服器概觀][sofs-overview]

* [Windows Server 2016 中儲存空間的新功能][new-in-win-2016-storage]
