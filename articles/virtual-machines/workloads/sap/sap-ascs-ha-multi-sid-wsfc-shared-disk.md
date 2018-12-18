---
title: 在 Azure 上搭配 Windows Server 容錯移轉叢集和共用磁碟的 SAP ASCS/SCS 執行個體多重 SID 高可用性
description: 在 Azure 上搭配 Windows Server 容錯移轉叢集和共用磁碟之 SAP ASCS/SCS 執行個體的多重 SID 高可用性
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
ms.openlocfilehash: ee5dc346def58ea7362a763d088145eb0d04a608
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34656725"
---
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1869038]:https://launchpad.support.sap.com/#/notes/1869038
[2287140]:https://launchpad.support.sap.com/#/notes/2287140
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[sap-installation-guides]:http://service.sap.com/instguides

[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md
[networking-limits-azure-resource-manager]:../../../azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md


[sap-net-weaver-ports]:https://help.sap.com/viewer/ports
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-net-weaver-ports-ascs-scs-ports]:sap-high-availability-infrastructure-wsfc-shared-disk.md#0f3ee255-b31e-4b8a-a95a-d9ed6200468b

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

[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md

[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk-install-ascs]:sap-high-availability-installation-wsfc-shared-disk.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-high-availability-installation-wsfc-shared-disk-modify-ascs-profile]:sap-high-availability-installation-wsfc-shared-disk.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556
[sap-high-availability-installation-wsfc-shared-disk-add-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#10822f4f-32e7-4871-b63a-9b86c76ce761
[sap-high-availability-installation-wsfc-shared-disk-win-firewall-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#4498c707-86c0-4cde-9c69-058a7ab8c3ac
[sap-high-availability-installation-wsfc-shared-disk-change-ers-service-startup-type]:sap-high-availability-installation-wsfc-shared-disk.md#094bc895-31d4-4471-91cc-1513b64e406a
[sap-high-availability-installation-wsfc-shared-disk-test-ascs-failover-and-sios-repl]:sap-high-availability-installation-wsfc-shared-disk.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9


[sap-high-availability-installation-wsfc-file-share]:sap-high-availability-installation-wsfc-file-share.md
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868

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

# <a name="sap-ascsscs-instance-multi-sid-high-availability-with-windows-server-failover-clustering-and-shared-disk-on-azure"></a>在 Azure 上搭配 Windows Server 容錯移轉叢集和共用磁碟的 SAP ASCS/SCS 執行個體多重 SID 高可用性

> ![Windows][Logo_Windows] Windows
>

Microsoft 在 2016 年 9 月發行的功能，可讓您使用 [Azure 內部負載平衡器][load-balancer-multivip-overview]管理多個虛擬 IP 位址。 這項功能已存在 Azure 外部負載平衡器。 

如果您有 SAP 部署，則必須使用內部負載平衡器，建立 SAP Central Services 執行個體的 Windows 叢集組態。

本文將著重於如何將單一 ASCS/SCS 安裝移至 SAP 多重 SID 設定，方法是使用共用磁碟，將其他 SAP ASCS/SCS 叢集執行個體安裝至現有 Windows Server 容錯移轉叢集 (WSFC) 叢集。 完成此程序之後，您將已設定 SAP 多重 SID 叢集。

> [!NOTE]
> 這項功能僅適用於 Azure Resource Manager 部署模型。
>
>每個 Azure 內部負載平衡器的私人前端 IP 都有數量限制。
>
>一個 WSFC 叢集中 SAP ASCS/SCS 執行個體數目上限等於每個 Azure 內部負載平衡器的私人前端 IP 數目上限。
>

如需負載平衡器限制的詳細資訊，請參閱[網路限制：Azure Resource Manager][networking-limits-azure-resource-manager] 中的「每個負載平衡器的私人前端 IP」一節。

## <a name="prerequisites"></a>先決條件

您已經使用**檔案共用**來設定要用於一個 SAP ASCS/SCS 執行個體的 WSFC 叢集，如下圖所示。

![高可用性的 SAP ASCS/SCS 執行個體][sap-ha-guide-figure-6001]

> [!IMPORTANT]
> 安裝程式必須符合下列條件︰
> * SAP ASCS / SCS 執行個體必須共用相同的 WSFC 叢集。
> * 每個資料庫管理系統 (DBMS) SID 必須有自己專用的 WSFC 叢集。
> * SAP 應用程式伺服器屬於必須擁有自己專用 VM 的一個 SAP 系統 SID。

## <a name="sap-ascsscs-multi-sid-architecture-with-shared-disk"></a>搭配共用磁碟的 SAP ASCS/SCS 多重 SID 架構

目標是在相同 WSAFC 叢集中安裝多個 SAP ABAP ASCS 或 SAP Java SCS 叢集執行個體，如這裡所詳述：

![Azure 中多個 SAP ASCS/SCS 叢集執行個體][sap-ha-guide-figure-6002]

如需負載平衡器限制的詳細資訊，請參閱[網路限制：Azure Resource Manager][networking-limits-azure-resource-manager] 中的「每個負載平衡器的私人前端 IP」一節。

具有兩個高度可用 SAP 系統的完整配置畫面如下所示：

![具有兩個 SAP 系統 SID 的 SAP 高可用性多 SID 設定][sap-ha-guide-figure-6003]

## <a name="25e358f8-92e5-4e8d-a1e5-df7580a39cb0"></a>準備 SAP 多重 SID 案例的基礎結構

若要準備您的基礎結構，您可以安裝額外的 SAP ASCS/SCS 執行個體，並使用下列參數︰

| 參數名稱 | 值 |
| --- | --- |
| SAP ASCS/SCS SID |pr1-lb-ascs |
| SAP DBMS 內部負載平衡器 | PR5 |
| SAP 虛擬主機名稱 | pr5-sap-cl |
| SAP ASCS/SCS 虛擬主機 IP 位址 (其他 Azure Load Balancer IP 位址) | 10.0.0.50 |
| SAP ASCS/SCS 執行個體號碼 | 50 |
| 其他 SAP ASCS/SCS 執行個體的 ILB 探查連接埠 | 62350 |

> [!NOTE]
> 對於 SAP ASCS/SCS 叢集執行個體，每個 IP 位址需要唯一的探查連接埠。 例如，如果 Azure 內部負載平衡器上有一個 IP 位址使用探查連接埠 62300，該負載平衡器上的任何其他 IP 位址就不能使用探查連接埠 62300。
>
>針對本文目的，因為已保留探查連接埠 62300，我們會使用探查連接埠 62350。

您可以在具有兩個節點的現有 WSFC 叢集中安裝額外 SAP ASCS/SCS 執行個體︰

| 虛擬機器角色 | 虛擬機器主機名稱 | 靜態 IP 位址 |
| --- | --- | --- |
| ASCS/SCS 執行個體的第一個叢集節點 |pr1-ascs-0 |10.0.0.10 |
| ASCS/SCS 執行個體的第二個叢集節點 |pr1-ascs-1 |10.0.0.9 |

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance-on-the-dns-server"></a>在 DNS 伺服器上建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱

您可以使用下列參數為 ASCS/SCS 執行個體的虛擬主機名稱建立 DNS 項目：

| 新的 SAP ASCS/SCS 虛擬主機名稱 | 相關聯的 IP 位址 |
| --- | --- | --- |
|pr5-sap-cl |10.0.0.50 |

新的主機名稱和 IP 位址會顯示在 DNS 管理員中，如下列螢幕擷取畫面所示︰

![DNS 管理員清單反白顯示已定義之新的 SAP ASCS/SCS 叢集虛擬名稱和 TCP/IP 位址的 DNS 項目][sap-ha-guide-figure-6004]

> [!NOTE]
> 您指派給 ASCS/SCS 執行個體之虛擬主機名稱的新 IP 位址必須與指派給 SAP Azure Load Balancer 的新 IP 位址相同。
>
>在我們的案例中，IP 位址是 10.0.0.50。

### <a name="add-an-ip-address-to-an-existing-azure-internal-load-balancer-by-using-powershell"></a>使用 PowerShell 將 IP 位址新增至現有的 Azure 內部負載平衡器

若要在相同的 WSFC 叢集中建立多個 SAP ASCS/SCS 執行個體，請使用 PowerShell 將 IP 位址新增至現有的 Azure 內部負載平衡器。 每個 IP 位址都需要有自己的負載平衡規則、探查連接埠、前端 IP 集區和後端集區。

下列指令碼會將新的 IP 位址新增至現有的負載平衡器。 請更新您環境的 PowerShell 變數。 指令碼會為所有 SAP ASCS/SCS 連接埠建立所有必要的負載平衡規則。

```powershell

# Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>
Clear-Host
$ResourceGroupName = "SAP-MULTI-SID-Landscape"      # Existing resource group name
$VNetName = "pr2-vnet"                        # Existing virtual network name
$SubnetName = "Subnet"                        # Existing subnet name
$ILBName = "pr2-lb-ascs"                      # Existing ILB name                      
$ILBIP = "10.0.0.50"                          # New IP address
$VMNames = "pr2-ascs-0","pr2-ascs-1"          # Existing cluster virtual machine names
$SAPInstanceNumber = 50                       # SAP ASCS/SCS instance number: must be a unique value for each cluster
[int]$ProbePort = "623$SAPInstanceNumber"     # Probe port: must be a unique value for each IP and load balancer

$ILB = Get-AzureRmLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName

$count = $ILB.FrontendIpConfigurations.Count + 1
$FrontEndConfigurationName ="lbFrontendASCS$count"
$LBProbeName = "lbProbeASCS$count"

# Get the Azure virtual network and subnet
$VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName
$Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

# Add a second front-end and probe configuration
Write-Host "Adding new front end IP Pool '$FrontEndConfigurationName' ..." -ForegroundColor Green
$ILB | Add-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id
$ILB | Add-AzureRmLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 10  | Set-AzureRmLoadBalancer

# Get a new updated configuration
$ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

# Get an updated LP FrontendIpConfig
$FEConfig = Get-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

# Add a back-end configuration into an existing ILB
$BackEndConfigurationName  = "backendPoolASCS$count"
Write-Host "Adding new backend Pool '$BackEndConfigurationName' ..." -ForegroundColor Green
$BEConfig = Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB | Set-AzureRmLoadBalancer

# Get an updated config
$ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

# Assign VM NICs to the back-end pool
$BEPool = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
foreach($VMName in $VMNames){
        $VM = Get-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VMName
        $NICName = ($VM.NetworkInterfaceIDs[0].Split('/') | select -last 1)        
        $NIC = Get-AzureRmNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName                
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools += $BEPool
        Write-Host "Assigning network card '$NICName' of the '$VMName' VM to the backend pool '$BackEndConfigurationName' ..." -ForegroundColor Green
        Set-AzureRmNetworkInterface -NetworkInterface $NIC
        #start-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VM.Name
}


# Create load-balancing rules
$Ports = "445","32$SAPInstanceNumber","33$SAPInstanceNumber","36$SAPInstanceNumber","39$SAPInstanceNumber","5985","81$SAPInstanceNumber","5$SAPInstanceNumber`13","5$SAPInstanceNumber`14","5$SAPInstanceNumber`16"
$ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
$FEConfig = get-AzureRMLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$BEConfig = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

Write-Host "Creating load balancing rules for the ports: '$Ports' ... " -ForegroundColor Green

foreach ($Port in $Ports) {

        $LBConfigrulename = "lbrule$Port" + "_$count"
        Write-Host "Creating load balancing rule '$LBConfigrulename' for the port '$Port' ..." -ForegroundColor Green

        $ILB | Add-AzureRmLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $HealthProbe -Protocol tcp -FrontendPort  $Port -BackendPort $Port -IdleTimeoutInMinutes 30 -LoadDistribution Default -EnableFloatingIP   
}

$ILB | Set-AzureRmLoadBalancer

Write-Host "Successfully added new IP '$ILBIP' to the internal load balancer '$ILBName'!" -ForegroundColor Green

```
在執行指令碼之後，結果會顯示在 Azure 入口網站中，如下列螢幕擷取畫面所示︰

![Azure 入口網站中新的前端 IP 集區][sap-ha-guide-figure-6005]

### <a name="add-disks-to-cluster-machines-and-configure-the-sios-cluster-share-disk"></a>將磁碟新增至叢集機器，並設定 SIOS 叢集共用磁碟

對於每個額外 SAP ASCS/SCS 執行個體，您必須新增叢集共用磁碟。 針對 Windows Server 2012 R2，目前使用的 WSFC 叢集共用磁碟是 SIOS DataKeeper 軟體解決方案。

執行下列動作：
1. 將額外磁碟或大小相同的磁碟 (您需要等量的磁碟) 新增至每個叢集節點中，並將其格式化。
2. 使用 SIOS DataKeeper 設定儲存體複寫。

此程序假設您已在 WSFC 叢集機器上安裝了 SIOS DataKeeper。 如果已經安裝，您現在必須在電腦之間設定複寫。 [為 SAP ASCS/SCS 叢集共用磁碟安裝 SIOS DataKeeper Cluster Edition][sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]會詳細描述此程序。  

![新的 SAP ASCS/SCS 共用磁碟的 DataKeeper 同步鏡像][sap-ha-guide-figure-6006]

### <a name="deploy-vms-for-sap-application-servers-and-the-dbms-cluster"></a>針對 SAP 應用程式伺服器和 DBMS 叢集部署 VM

若要完成第二個 SAP 系統的基礎結構準備，執行下列作業︰

1. 為 SAP 應用程式伺服器部署專用的 VM，並將其放在各自專用的可用性群組中。
2. 為 DBMS 叢集部署專用的 VM，並將其放在各自專用的可用性群組中。

## <a name="install-an-sap-netweaver-multi-sid-system"></a>安裝 SAP NetWeaver 多重 SID 系統

如需安裝第二個 SAP SID2 系統的完整程序說明，請參閱 [SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和共用磁碟上的 SAP NetWeaver HA 安裝][sap-high-availability-installation-wsfc-shared-disk]。

高階程序如下所示︰

1. [使用高可用性 ASCS/SCS 執行個體安裝 SAP][sap-high-availability-installation-wsfc-shared-disk-install-ascs]。  
 在此步驟中，您要在現有 WSFC 叢集節點 1 上使用高可用性 ASCS/SCS 執行個體安裝 SAP。

2. [修改 ASCS/SCS 執行個體的 SAP 設定檔][sap-high-availability-installation-wsfc-shared-disk-modify-ascs-profile]。

3. [設定探查連接埠][sap-high-availability-installation-wsfc-shared-disk-add-probe-port]。  
 在此步驟中，您要使用 PowerShell 設定 SAP 叢集資源 SAP SID2 IP 探查連接埠。 在其中一個 SAP ASCS/SCS 叢集節點上執行此組態。

4. 安裝資料庫執行個體。  
 若要安裝第二個叢集，請依照 SAP 安裝指南中的步驟。

5. 安裝第二個叢集節點。  
 在此步驟中，您要在現有 WSFC 叢集節點 2 上使用高可用性 ASCS/SCS 執行個體安裝 SAP。 若要安裝第二個叢集，請依照 SAP 安裝指南中的步驟。

6. 開啟 SAP ASCS/SCS 執行個體和 ProbePort 的 Windows 防火牆連接埠。  
    在用於 SAP ASCS/SCS 執行個體的兩個叢集節點上，您要開啟 SAP ASCS/SCS 所使用的所有 Windows 防火牆連接埠。 這些 SAP ASCS/SCS 執行個體連接埠會列示在 [SAP ASCS / SCS 連接埠][sap-net-weaver-ports-ascs-scs-ports]一章中。

    如需所有其他 SAP 連接埠的清單，請參閱[所有 SAP 產品的 TCP/IP 連接埠][sap-net-weaver-ports]。  

    此外，開啟 Azure 內部負載平衡器探查連接埠，在我們的案例中為 62350。 這如[本文][sap-high-availability-installation-wsfc-shared-disk-win-firewall-probe-port]所述。

7. [變更 SAP Evaluated Receipts Settlement (ERS) Windows 服務執行個體的啟動類型][sap-high-availability-installation-wsfc-shared-disk-change-ers-service-startup-type]。

8. 在新的專用 VM 上安裝 SAP 主要應用程式伺服器，如 SAP 安裝指南所述。  

9. 在新的專用 VM 上安裝 SAP 其他應用程式伺服器，如 SAP 安裝指南所述。

10. [測試 SAP ASCS/SCS 執行個體容錯移轉和 SIOS 複寫][sap-high-availability-installation-wsfc-shared-disk-test-ascs-failover-and-sios-repl]。

## <a name="next-steps"></a>後續步驟

- [網路限制：Azure Resource Manager][networking-limits-azure-resource-manager]
- [Azure Load Balancer 的多個 VIP][load-balancer-multivip-overview]
