---
title: 在 Azure 中使用檔案共用於 Windows 容錯移轉叢集上進行 SAP ASCS/SCS 執行個體叢集處理 | Microsoft Docs
description: 了解如何在 Azure 中使用檔案共用於 Windows 容錯移轉叢集上進行 SAP ASCS/SCS 執行個體叢集處理。
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
ms.openlocfilehash: 646c30be171a5aaaa17e40eae3cef6952b2b2747
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34657055"
---
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[kb4025334]:https://support.microsoft.com/help/4025334/windows-10-update-kb4025334

[dv2-series]:https://docs.microsoft.com/azure/virtual-machines/windows/sizes-general#dv2-series
[ds-series]:https://docs.microsoft.com/azure/virtual-machines/windows/sizes-general#ds-series

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
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-volumes-s2d-choosing-filesystem]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-filesystem
[choosing-the-size-of-volumes-s2d]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-size-of-volumes
[deploy-sofs-s2d-in-azure]:https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-storage-spaces-direct-deployment
[s2d-in-win-2016]:https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview
[deep-dive-volumes-in-s2d]:https://blogs.technet.microsoft.com/filecab/2016/08/29/deep-dive-volumes-in-spaces-direct/

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

[1869038]:https://launchpad.support.sap.com/#/notes/1869038 

# <a name="cluster-an-sap-ascsscs-instance-on-a-windows-failover-cluster-by-using-a-file-share-in-azure"></a>在 Azure 中使用檔案共用於 Windows 容錯移轉叢集上進行 SAP ASCS/SCS 執行個體叢集處理

> ![Windows][Logo_Windows] Windows
>

Windows Server 容錯移轉叢集是 Windows 中高可用性 SAP ASCS/SCS 安裝和 DBMS 的基礎。

容錯移轉叢集是由 1+n 個獨立伺服器 (節點) 所組成的群組，這些伺服器會共同運作以提升應用程式和服務的可用性。 如果發生節點失敗，Windows Server 容錯移轉叢集會計算發生的失敗次數，以及仍然維持狀況良好的叢集，以提供應用程式和服務。 您可以從不同的仲裁模式選擇以達成容錯移轉叢集。

## <a name="prerequisites"></a>先決條件
在開始本文所述的工作之前，請檢閱此文章：

* [SAP NetWeaver 的 Azure 虛擬機器高可用性架構和案例][sap-high-availability-architecture-scenarios]

> [!IMPORTANT]
> 針對 SAP NetWeaver 7.40 (和更新版本)，包含 SAP 核心 7.49 (和更新版本)，支援使用檔案共用進行 SAP ASCS/SCS 執行個體叢集處理。
>


## <a name="windows-server-failover-clustering-in-azure"></a>Azure 中的 Windows Server 容錯移轉叢集

相較於裸機或私人雲端部署，Azure 虛擬機器需要額外的步驟來設定 Windows Server 容錯移轉叢集。 當您建置叢集時，需要為 SAP ASCS/SCS 執行個體設定數個 IP 位址和虛擬主機名稱。

### <a name="name-resolution-in-azure-and-the-cluster-virtual-host-name"></a>Azure 和叢集虛擬主機名稱中的名稱解析

Azure 雲端平台不提供設定虛擬 IP 位址的選項，例如浮動 IP 位址。 您需要一個替代解決方案來設定虛擬 IP，以便連線到雲端的叢集資源。 

Azure Load Balancer 服務可為 Azure 提供「內部負載平衡器」。 使用內部負載平衡器，用戶端可透過叢集虛擬 IP 位址連線叢集。 

在包含叢集節點的資源群組中部署內部負載平衡器。 接著，使用內部負載平衡器的探查連接埠來設定所有必要的連接埠轉送規則。 用戶端可以透過虛擬主機名稱來進行連線。 DNS 伺服器會解析叢集 IP 位址。 內部負載平衡器則會處理對作用中叢集節點的連接埠轉送。

![圖 1：Azure 中沒有共用磁碟的 Windows Server 容錯移轉叢集設定][sap-ha-guide-figure-1001]

_**圖 1**：Azure 中不含共用磁碟的 Windows Server 容錯移轉叢集設定_

## <a name="sap-ascsscs-ha-with-file-share"></a>含檔案共用的 SAP ASCS/SCS HA

SAP 已針對在 Windows 容錯移轉叢集上進行 SAP ASCS/SCS 執行個體叢集處理，為叢集共用磁碟開發新的方法和替代方案。 您可以使用 SMB 檔案共用 (而不是使用叢集共用磁碟) 部署 SAP 全域主機檔案。

> [!NOTE]
> SMB 檔案共用是使用叢集共用磁碟進行 SAP ASCS/SCS 執行個體叢集處理的替代方案。  
>

此架構在以下幾方面是明確的：

* SAP 中央服務 (包含自己的檔案結構、訊息及加入佇列處理序) 與 SAP 全域主機檔案是分開的。
* SAP 中央服務會在 SAP ASCS/SCS 執行個體之下執行。
* SAP ASCS/SCS 執行個體已叢集處理，並且可使用 \<ASCS/SCS 虛擬主機名稱\> 虛擬主機名稱來存取。
* SAP 全域檔案會置於 SMB 檔案共用，並可使用 \<SAP 全域主機\> 主機名稱: \\\\&lt;SAP 全域主機&gt;\sapmnt\\&lt;SID&gt;\SYS\... 加以存取。
* SAP ASCS/SCS 執行個體是安裝於這兩個叢集節點上的本機磁碟。
* \<ASCS/SCS 虛擬主機名稱\> 網路名稱與 &lt;SAP 全域主機&gt; 不同。

![圖 2：含 SMB 檔案共用的 SAP ASCS/SCS HA 架構][sap-ha-guide-figure-8004]

_**圖 2：** 含 SMB 檔案共用的新 SAP ASCS/SCS HA 架構_

SMB 檔案共用的必要條件：

* SMB 3.0 (或更新版本) 通訊協定。
* 能夠為 Active Directory 使用者群組和 `computer$` 電腦物件設定 Active Directory 存取控制清單 (ACL)。
* 檔案共用必須啟用 HA：
    * 用來儲存檔案的磁碟不可是單一失敗點。
    * 伺服器或 VM 的停機時間不會在檔案共用上造成停機時間。

SAP \<SID\> 叢集角色不包含叢集共用磁碟或一般檔案共用叢集資源。


![圖 3：可用於檔案共用的 SAP \<SID\> 叢集角色資源][sap-ha-guide-figure-8005]

_**圖 3：** 可用於檔案共用的 SAP &lt;SID&gt; 叢集角色資源_


## <a name="scale-out-file-shares-with-storage-spaces-direct-in-azure-as-an-sapmnt-file-share"></a>在 Azure 中將包含儲存空間直接存取的向外延展檔案共用作為 SAPMNT 檔案共用

您可以使用向外延展檔案共用裝載並保護 SAP 全域主機檔案。 向外延展檔案共用也會提供高可用性的 SAPMNT 檔案共用服務。

![圖 4：用來保護 SAP 全域主機檔案的向外延展檔案共用][sap-ha-guide-figure-8006]

_**圖 4：** 用來保護 SAP 全域主機檔案的向外延展檔案共用_

> [!IMPORTANT]
> 在 Microsoft Azure 雲端和內部部署環境中完全支援向外延展檔案共用。
>

向外延展檔案共用提供高可用及可水平調整的 SAPMNT 檔案共用。

儲存空間直接存取會當作向外延展檔案共用的共用磁碟使用。 您可以使用儲存空間直接存取搭配使用伺服器與本機儲存體來建立高度可用且可調整的儲存體。 用於向外延展檔案共用 (例如用於 SAP 全域主機檔案) 的共用儲存體不是單一失敗點。

> [!IMPORTANT]
>如果您不打算設定災害復原，建議在 Azure 中使用向外延展檔案共用作為高可用性檔案共用的解決方案。
>

### <a name="sap-prerequisites-for-scale-out-file-shares-in-azure"></a>Azure 中向外延展檔案共用的 SAP 必要條件

若要使用向外延展檔案共用，您的系統必須符合下列需求：

* 至少有兩個叢集節點用於向外延展檔案共用。
* 每個節點至少必須有兩個本機磁碟。
* 基於效能考量，您必須使用「鏡像復原」：
    * 雙向鏡像適用於包含兩個叢集節點的向外延展檔案共用。
    * 三向鏡像適用於包含三個 (或多個) 叢集節點的向外延展檔案共用。
* 建議搭配三向鏡像使用包含三個 (或多個) 叢集節點的向外延展檔案共用。
    相較於包含兩個叢集節點的向外延展檔案共用設定和雙向鏡像，此設定提供更大的延展性以及更強的儲存體復原功能。
* 您必須使用 Azure 進階磁碟。
* 建議您使用 Azure 受控磁碟。
* 建議您使用復原檔案系統 (ReFS) 格式化磁碟區。
    * 如需詳細資訊，請參閱 [SAP 附註 1869038 - ReFs 檔案系統的 SAP 支援][1869038]以及＜在儲存空間直接存取中規劃磁碟區＞一文的[選擇檔案系統][planning-volumes-s2d-choosing-filesystem]章節。
    * 請確定您有安裝 [Microsoft KB4025334 累積更新][kb4025334]。
* 您可以使用 DS 系列或 DSv2 系列 Azure VM 大小。
* 若要讓 VM 之間具備良好的網路效能，供儲存空間直接存取磁碟同步之用，請使用至少具有「高」網路頻寬的 VM 類型。
    如需詳細資訊，請參閱 [DSv2 系列][dv2-series]和 [DS 系列][ds-series]規格。
* 建議您在儲存體集區中保留一些未配置的容量。 在儲存體集區中保留一些未配置的容量，可在磁碟機故障時，讓磁碟區空間「就地」修復。 這可改善資料安全性和效能。  如需詳細資訊，請參閱[選擇磁碟區大小][choosing-the-size-of-volumes-s2d]。
* 向外延展檔案共用 Azure VM 必須在其自己的 Azure 可用性設定組中部署。
* 您不需要為向外延展檔案共用網路名稱 (例如 \<SAP 全域主機\>) 設定 Azure 內部負載平衡器。 這是針對 SAP ASCS/SCS 執行個體的 \<ASCS/SCS 虛擬主機名稱\> 或針對 DBMS 進行。 向外延展檔案共用會在所有叢集節點之間向外延展負載。 \<SAP 全域主機\> 會對所有叢集節點使用本機 IP 位址。


> [!IMPORTANT]
> 您無法為指向 \<SAP 全域主機\> 的 SAPMNT 檔案共用重新命名。 SAP 僅支援共用名稱 "sapmnt"。

> 如需詳細資訊，請參閱 [SAP 附註 2492395 - 是否可變更共用名稱 sapmnt？][2492395]

### <a name="configure-sap-ascsscs-instances-and-a-scale-out-file-share-in-two-clusters"></a>在兩個叢集中設定 SAP ASCS/SCS 執行個體和向外延展檔案共用

您可以使用 SAP ASCS/SCS 本身的 SAP \<SID\> 叢集角色，在一個叢集中部署其執行個體。 在此情況下，您要使用其他叢集角色，在另一個叢集上設定向外延展檔案共用。

> [!IMPORTANT]
>在此案例中，SAP ASCS/SCS 執行個體是設定為使用 UNC 路徑 \\\\&lt;SAP 全域主機&gt;\sapmnt\\&lt;SID&gt;\SYS\. 存取 SAP 全域主機
>

![圖 5：在兩個叢集中部署的 SAP ASCS/SCS 執行個體和向外延展檔案共用][sap-ha-guide-figure-8007]

_**圖 5：** 在兩個叢集中部署的 SAP ASCS/SCS 執行個體和向外延展檔案共用_

> [!IMPORTANT]
> 在 Azure 雲端中，用於 SAP 和向外延展檔案共用的每個叢集都必須部署在自己的 Azure 可用性設定組中。 這可確保叢集 VM 在基礎 Azure 基礎結構中，以分散方式放置。
>

## <a name="generic-file-share-with-sios-datakeeper-as-cluster-shared-disks"></a>將 SIOS DataKeeper 當作叢集共用磁碟的一般檔案共用


> [!IMPORTANT]
> 建議您將向外延展檔案共用解決方案用於高可用性檔案共用。
>
> 如果您打算也為高可用性檔案共用設定災害復原，則必須將一般檔案共用與 SISO DataKeeper 用於叢集共用磁碟。
>

一般檔案共用是獲得高可用性檔案共用的另一個選項。

在此情況下，您可以使用第三方的 SIOS 解決方案作為叢集共用磁碟。

## <a name="next-steps"></a>後續步驟

* [使用 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和檔案共用，為 SAP HA 準備 Azure 基礎結構][sap-high-availability-infrastructure-wsfc-file-share]
* [在 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和檔案共用上安裝 SAP NetWeaver HA][sap-high-availability-installation-wsfc-shared-disk]
* [在 Azure 中針對 UDP 儲存體部署兩個節點的儲存空間直接存取向外延展檔案伺服器][deploy-sofs-s2d-in-azure]
* [Windows Server 2016 中的儲存空間直接存取][s2d-in-win-2016]
* [深入探討：儲存空間直接存取中的磁碟區][deep-dive-volumes-in-s2d]
