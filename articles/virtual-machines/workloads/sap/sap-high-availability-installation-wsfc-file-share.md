---
title: 在 Azure 之 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和檔案共用上安裝 SAP NetWeaver 高可用性 | Microsoft Docs
description: 在 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和檔案共用上安裝 SAP NetWeaver 高可用性
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 71296618-673b-4093-ab17-b7a80df6e9ac
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2fa9cebe092f726b2df3dec99cee1bb97ccc92dc
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34658650"
---
# <a name="install-sap-netweaver-high-availability-on-a-windows-failover-cluster-and-file-share-for-sap-ascsscs-instances-on-azure"></a>在 Azure 之 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和檔案共用上安裝 SAP NetWeaver 高可用性

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1596496]:https://launchpad.support.sap.com/#/notes/1596496

[sap-installation-guides]:http://service.sap.com/instguides

[sap-powershell-scrips]:https://github.com/Azure-Samples/sap-powershell

[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md

[s2d-in-win-2016]:https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview
[sofs-overview]:https://technet.microsoft.com/library/hh831349(v=ws.11).aspx
[new-in-win-2016-storage]:https://docs.microsoft.com/windows-server/storage/whats-new-in-storage

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:http://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[sap-blog-new-sap-cluster-resource-dll]:https://blogs.sap.com/2017/08/28/new-sap-cluster-resource-dll-is-available/
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk-create-ascs-virt-host]:sap-high-availability-installation-wsfc-shared-disk.md#a97ad604-9094-44fe-a364-f89cb39bf097
[sap-high-availability-installation-wsfc-shared-disk-add-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#10822f4f-32e7-4871-b63a-9b86c76ce761

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md

[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-shared-disk-vpn]:sap-high-availability-infrastructure-wsfc-shared-disk.md#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-high-availability-infrastructure-wsfc-shared-disk-change-def-ilb]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-wsfc]:sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-high-availability-infrastructure-wsfc-shared-disk-add-dot-net]:sap-high-availability-infrastructure-wsfc-shared-disk.md#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios-both-nodes]:sap-high-availability-infrastructure-wsfc-shared-disk.md#dd41d5a2-8083-415b-9878-839652812102
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006

[sap-official-ha-file-share-document]:https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html

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

本文說明如何在 Azure 上透過 Windows 容錯移轉叢集 (WSFC) 和向外延展檔案共用作為叢集 SAP ASCS/SCS 執行個體的選項，以安裝並設定高可用的 SAP 系統。

## <a name="prerequisites"></a>先決條件

在開始安裝之前，請檢閱下列文章：

* [架構指南：使用檔案共用在 Windows 容錯移轉叢集上進行 SAP ASCS/SCS 執行個體叢集處理][sap-high-availability-guide-wsfc-file-share]

* [使用 SAP ASCS/SCS 執行個體的 Windows 容錯移轉叢集和檔案共用，為 SAP 高可用性準備 Azure 基礎結構][sap-high-availability-infrastructure-wsfc-file-share]

需要來自 SAP 的下列可執行檔和 DLL：
* SAP Software Provisioning Manager (SWPM) 安裝工具版本 SPS21 或更新版本。
* 請下載最新的 NTCLUST.SAR 封存與新 SAP 叢集資源 DLL。 新的 SAP 叢集 DLL 在 Windows Server 容錯移轉叢集上透過檔案共用支援 SAP ASCS/SCS 高可用性。

  如需有關新 SAP 叢集資源 DLL 的詳細資訊，請參閱此部落格：[有可用的新 SAP 叢集資源 DLL！][sap-blog-new-sap-cluster-resource-dll]。

設定會視使用的 DBMS 而異，因此我們不會說明資料庫管理系統 (DBMS) 設定。 不過，我們會假設 DBMS 在高可用性方面的疑慮已藉由不同 DBMS 廠商為 Azure 提供的功能支援而獲得解決。 此類功能包括適用於 SQL Server 的 Always On 或資料庫鏡像，以及適用於 Oracle 資料庫的 Oracle Data Guard。 在本文使用的案例中，我們並未對 DBMS 加入更多保護。

當不同的 DBMS 服務與 Azure 中這種叢集 SAP ASCS/SCS 組態互動時，沒有任何特殊的考量。

> [!NOTE]
> SAP NetWeaver ABAP 系統、Java 系統及 ABAP+Java 系統的安裝程序幾乎完全相同。 最顯著的差異在於 SAP ABAP 系統有一個 ASCS 執行個體。 SAP Java 系統有一個 SCS 執行個體。 SAP ABAP+Java 系統有一個 ASCS 執行個體和一個 SCS 執行個體在相同 Microsoft 容錯移轉叢集群組中執行。 我們會明確說明每個 SAP NetWeaver 安裝堆疊的所有安裝差異。 您可以假設所有其他部分都相同。  
>
>

## <a name="install-an-ascsscs-instance-on-an-ascsscs-cluster"></a>在 ASCS/SCS 叢集上安裝 ASCS/SCS 執行個體

> [!IMPORTANT]
>
> 目前，SAP SWPM 安裝工具不支援具有檔案共用設定的高可用性設定。 因此，SAP 系統的安裝必須經過一些手動作業，例如需安裝和叢集 SAP ASCS/SCS 執行個體並個別設定 SAP 全域主機。  
>
> DBMS 執行個體和 SAP 應用程式伺服器的安裝 (和叢集) 的安裝步驟並未更改。
>

### <a name="install-an-ascsscs-instance-on-your-local-drive"></a>在本機磁碟機上安裝 ASCS/SCS 執行個體

在 ASCS/SCS 叢集的兩個節點上都安裝 SAP ASCS/SCS 執行個體。 將其安裝於本機磁碟機。 在我們的範例中，本機磁碟機為 C:\\，但是您可以選擇任何其他本機磁碟機。  

若要安裝執行個體，在 SAP SWPM 安裝工具中，移至：

**\<產品>** > **\<DBMS>** > **安裝** > **應用程式伺服器 ABAP** (或 **Java**) > **分散式系統** > **ASCS/SCS 執行個體**

> [!IMPORTANT]
> 目前，SAP SWPM 安裝工具不支援檔案共用案例。 您無法使用下列安裝路徑：
>
> **\<產品>** > **\<DBMS>** > **安裝** > **應用程式伺服器 ABAP** (或 **Java**) > **高可用性系統** > …
>

### <a name="remove-sapmnt-and-create-an-saploc-file-share"></a>移除 SAPMNT 並建立 SAPLOC 檔案共用

SWMP 已在 C:\\usr\\sap 資料夾上建立 SAPMNT 本機共用。

移除兩個 ASCS/SCS 叢集節點上的 SAPMNT 檔案共用。

執行下列 PowerShell 指令碼：

```PowerShell
Remove-SmbShare sapmnt -ScopeName * -Force
 ```

如果 SAPLOC 共用不存在，請在兩個 ASCS/SCS 叢集節點上建立。

執行下列 PowerShell 指令碼：

```PowerShell
#Create SAPLOC share and set security
$SAPSID = "PR1"
$DomainName = "SAPCLUSTER"
$SAPSIDGlobalAdminGroupName = "$DomainName\SAP_" + $SAPSID + "_GlobalAdmin"
$HostName = $env:computername
$SAPLocalAdminGroupName = "$HostName\SAP_LocalAdmin"
$SAPDisk = "C:"
$SAPusrSapPath = "$SAPDisk\usr\sap"

New-SmbShare -Name saploc -Path c:\usr\sap -FullAccess "BUILTIN\Administrators", $SAPSIDGlobalAdminGroupName , $SAPLocalAdminGroupName  
 ```

## <a name="prepare-an-sap-global-host-on-the-sofs-cluster"></a>在 SOFS 叢集上準備 SAP 全域主機

在 SOFS 叢集上建立下列磁碟區和檔案共用：

* SOFS 叢集共用磁碟區 (CSV) 上的 SAP GLOBALHOST 檔案 C:\ClusterStorage\Volume1\usr\sap\\<SID>\SYS\ 結構

* SAPMNT 檔案共用

* 透過下列項目的完全控制，設定 SAPMNT 檔案共用和資料夾的安全性：
    * \<DOMAIN>\SAP_\<SID>_GlobalAdmin 使用者群組
    * SAP ASCS/SCS 叢集節點電腦物件 \<DOMAIN>\ClusterNode1$ and \<DOMAIN>\ClusterNode2$

若要建立具有鏡像復原的 CSV 磁碟區，請在其中一個 SOFS 叢集節點上執行下列 PowerShell Cmdlet：


```PowerShell
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName SAPPR1 -FileSystem CSVFS_ReFS -Size 5GB -ResiliencySettingName Mirror
```
若要建立 SAPMNT 並設定資料夾和共用安全性，請在其中一個 SOFS 叢集節點上執行下列 PowerShell 指令碼：

```PowerShell
# Create SAPMNT on file share
$SAPSID = "PR1"
$DomainName = "SAPCLUSTER"
$SAPSIDGlobalAdminGroupName = "$DomainName\SAP_" + $SAPSID + "_GlobalAdmin"

# SAP ASCS/SCS cluster nodes
$ASCSClusterNode1 = "ascs-1"
$ASCSClusterNode2 = "ascs-2"

# Define SAP ASCS/SCS cluster node computer objects
$ASCSClusterObjectNode1 = "$DomainName\$ASCSClusterNode1$"
$ASCSClusterObjectNode2 = "$DomainName\$ASCSClusterNode2$"

# Create usr\sap\.. folders on CSV
$SAPGlobalFolder = "C:\ClusterStorage\Volume1\usr\sap\$SAPSID\SYS"
New-Item -Path $SAPGlobalFOlder -ItemType Directory

$UsrSAPFolder = "C:\ClusterStorage\Volume1\usr\sap\"

# Create a SAPMNT file share and set share security
New-SmbShare -Name sapmnt -Path $UsrSAPFolder -FullAccess "BUILTIN\Administrators", $SAPSIDGlobalAdminGroupName, $ASCSClusterObjectNode1, $ASCSClusterObjectNode2 -ContinuouslyAvailable $false -CachingMode None -Verbose

# Get SAPMNT file share security settings
Get-SmbShareAccess sapmnt

# Set file and folder security
$Acl = Get-Acl $UsrSAPFolder

# Add a file security object of SAP_<sid>_GlobalAdmin group
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($SAPSIDGlobalAdminGroupName,"FullControl", 'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add  a security object of the clusternode1$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSClusterObjectNode1,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add a security object of the clusternode2$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSClusterObjectNode2,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Set security
Set-Acl $UsrSAPFolder $Acl -Verbose
 ```
## <a name="stop-ascsscs-instances-and-sap-services"></a>停止 ASCS/SCS 執行個體和 SAP 服務

執行以下步驟：
1. 在兩個 ASCS/SCS 叢集節點上停止 SAP ASCS/SCS 執行個體。
2. 在兩個叢集節點上停止 SAP ASCS/SCS Windows 服務 **SAP\<SID>_\<InstanceNumber>**。

## <a name="move-the-sys-folder-to-the-sofs-cluster"></a>將 \SYS\... 資料夾移至 SOFS 叢集

執行以下步驟：
1. 將 SYS 資料夾 (例如，C:\usr\sap\\<SID>\SYS) 從其中一個 ASCS/SCS 叢集節點複製到 SOFS 叢集 (例如，複製到 C:\ClusterStorage\Volume1\usr\sap\\<SID>\SYS)。
2. 從兩個 ASCS/SCS 叢集節點中刪除 C:\usr\sap\\<SID>\SYS 資料夾。

## <a name="update-the-cluster-security-setting-on-the-sap-ascsscs-cluster"></a>更新 SAP ASCS/SCS 叢集上的叢集安全性設定

在其中一個 SAP ASCS/SCS 叢集節點上執行下列 PowerShell 指令碼：

```PowerShell
# Grant <DOMAIN>\SAP_<SID>_GlobalAdmin group access to the cluster

$SAPSID = "PR1"
$DomainName = "SAPCLUSTER"
$SAPSIDGlobalAdminGroupName = "$DomainName\SAP_" + $SAPSID + "_GlobalAdmin"

# Set full access for the <DOMAIN>\SAP_<SID>_GlobalAdmin group
Grant-ClusterAccess -User $SAPSIDGlobalAdminGroupName -Full

#Check security settings
Get-ClusterAccess
```

## <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a>建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱

建立 SAP ASCS/SCS 叢集網路名稱 (例如，**pr1-ascs [10.0.6.7]**)，如[建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱][sap-high-availability-installation-wsfc-shared-disk-create-ascs-virt-host]中所述。

## <a name="update-the-default-and-sap-ascsscs-instance-profile"></a>更新預設和 SAP ASCS/SCS 執行個體設定檔

若要使用新的 SAP ASCS/SCS 虛擬主機名稱和 SAP 全域主機名稱，您必須更新預設和 SAP ASCS/SCS 執行個體設定檔 \<SID>_ASCS/SCS\<Nr>_<Host>。


| 舊值 |  |
| --- | --- |
| SAP ASCS/SCS 主機名稱 = SAP 全域主機 | ascs-1 |
| SAP ASCS/SCS 執行個體設定檔名稱 | PR1_ASCS00_ascs-1 |

| 新值 |  |
| --- | --- |
| SAP ASCS/SCS 主機名稱 | **pr1-ascs** |
| SAP 全域主機 | **sapglobal** |
| SAP ASCS/SCS 執行個體設定檔名稱 | PR1\_ASCS00\_**pr1-ascs** |

### <a name="update-sap-default-profile"></a>更新 SAP 預設設定檔


| 參數名稱 | 參數值 |
| --- | --- |
| SAPGLOBALHOST | **sapglobal** |
| rdisp/mshost | **pr1-ascs** |
| enque/serverhost | **pr1-ascs** |

### <a name="update-the-sap-ascsscs-instance-profile"></a>更新 SAP ASCS/SCS 執行個體設定檔

| 參數名稱 | 參數值 |
| --- | --- |
| SAPGLOBALHOST | **sapglobal** |
| DIR_PROFILE | \\\sapglobal\sapmnt\PR1\SYS\profile |
| _PF | $(DIR_PROFILE)\PR1\_ASCS00_ pr1-ascs |
| Restart_Program_02 = local$(_MS) pf=$(_PF) | **Start**_Program_02 = local$(_MS) pf=$(_PF) |
| SAPLOCALHOST | **pr1-ascs** |
| Restart_Program_03 = local$(_EN) pf=$(_PF) | **Start**_Program_03 = local$(_EN) pf=$(_PF) |
| gw/netstat_once | **0** |
| enque/encni/set_so_keepalive  | **true** |
| service/ha_check_node | **1** |

> [!IMPORTANT]
>您可以使用 **Update-SAPASCSSCSProfile** PowerShell Cmdlet 來自動化設定檔更新。
>
>PowerShell Cmdlet 同時支援 SAP ABAP ASCS 和 SAP Java SCS 執行個體。
>

將 [**SAPScripts.psm1**][sap-powershell-scrips] 複製到本機磁碟機 C:\tmp，並執行下列 PowerShell Cmdlet：

```PowerShell
Import-Module C:\tmp\SAPScripts.psm1

Update-SAPASCSSCSProfile -PathToAscsScsInstanceProfile \\sapglobal\sapmnt\PR1\SYS\profile\PR1_ASCS00_ascs-1 -NewASCSHostName pr1-ascs -NewSAPGlobalHostName sapglobal -Verbose  
```

![圖 1：SAPScripts.psm1 輸出][sap-ha-guide-figure-8012]

_**圖 1**：SAPScripts.psm1 輸出_

## <a name="update-the-sidadm-user-environment-variable"></a>更新 \<sid>adm 使用者環境變數

1. 請在兩個 ASCS/SCS 叢集節點上更新 \<sid>adm 使用者環境的新 GLOBALHOST UNC 路徑。
2. 以 \<sid>adm 使用者的身分登入，然後啟動 Regedit.exe 工具。
3. 移至 **HKEY_CURRENT_USER** > **Environment**，然後將變數更新為新值：

| 變數 | 值 |
| --- | --- |
| RSEC_SSFS_DATAPATH | \\\\**sapglobal**\sapmnt\PR1\SYS\global\security\rsecssfs\data |
| RSEC_SSFS_KEYPATH | \\\\**sapglobal**\sapmnt\PR1\SYS\global\security\rsecssfs\key |
| SAPEXE | \\\\**sapglobal**\sapmnt\PR1\SYS\exe\uc\NTAMD64 |
| SAPLOCALHOST  | **pr1-ascs** |


## <a name="install-a-new-saprcdll-file"></a>安裝新的 saprc.dll 檔案

1. 安裝支援檔案共用案例的新版 SAP 叢集資源。

2. 從 SAP Service Marketplace 下載最新的 NTCLUST.SAR 套件。

3. 在其中一個 ASCS/SCS 叢集節點上解壓縮 NTCLUS.SAR，然後從命令提示字元中執行下列命令，以安裝新的 saprc.dll 檔案：

```
.\NTCLUST\insaprct.exe -yes -install
```

系統將在兩個 ASCS/SCS 叢集節點上安裝新的 saprc.dll 檔案。

如需詳細資訊，請參閱 [SAP 附註 1596496 - 如何更新叢集資源監視器的 SAP 資源類型 DLL][1596496]。

## <a name="create-a-sap-sid-cluster-group-network-name-and-ip"></a>建立 SAP <SID> 叢集群組、網路名稱和 IP

若要建立 SAP \<SID > 叢集群組、ASCS/SCS 網路名稱和對應的 IP 位址，請執行下列 PowerShell Cmdlet：

```PowerShell
# Create SAP Cluster Group
$SAPSID = "PR1"
$SAPClusterGroupName = "SAP $SAPSID"
$SAPIPClusterResourceName = "SAP $SAPSID IP"
$SAPASCSNetworkName = "pr1-ascs"
$SAPASCSIPAddress = "10.0.6.7"
$SAPASCSSubnetMask = "255.255.255.0"

# Create an SAP ASCS instance virtual IP cluster resource
Add-ClusterGroup -Name $SAPClusterGroupName -Verbose

#Create an SAP ASCS virtual IP address
$SAPIPClusterResource = Add-ClusterResource -Name $SAPIPClusterResourceName -ResourceType "IP Address" -Group $SAPClusterGroupName -Verbose

# Set a static IP address
$param1 = New-Object Microsoft.FailoverClusters.PowerShell.ClusterParameter $SAPIPClusterResource,Address,$SAPASCSIPAddress
$param2 = New-Object Microsoft.FailoverClusters.PowerShell.ClusterParameter $SAPIPClusterResource,SubnetMask,$SAPASCSSubnetMask
$params = $param1,$param2
$params | Set-ClusterParameter

# Create a corresponding network name
$SAPNetworkNameClusterResourceName = $SAPASCSNetworkName
Add-ClusterResource -Name $SAPNetworkNameClusterResourceName -ResourceType "Network Name" -Group $SAPClusterGroupName -Verbose

# Set a network DNS name
$SAPNetworkNameClusterResource = Get-ClusterResource $SAPNetworkNameClusterResourceName
$SAPNetworkNameClusterResource | Set-ClusterParameter -Name Name -Value $SAPASCSNetworkName

#Check the updated values
$SAPNetworkNameClusterResource | Get-ClusterParameter

#Set resource dependencies
Set-ClusterResourceDependency -Resource $SAPNetworkNameClusterResourceName -Dependency "[$SAPIPClusterResourceName]" -Verbose

#Start an SAP <SID> cluster group
Start-ClusterGroup -Name $SAPClusterGroupName -Verbose
```

## <a name="register-the-sap-start-service-on-both-nodes"></a>在兩個節點上註冊 SAP 啟動服務

重新註冊 SAP ASCS/SCS 啟動服務，以指向新的設定檔和設定檔路徑。

您必須在兩個 ASCS/SCS 叢集節點上執行此重新註冊。

在提高權限的命令提示字元中執行下列命令：

```
C:\usr\sap\PR1\ASCS00\exe\sapstartsrv.exe -r -p \\sapglobal\sapmnt\PR1\SYS\profile\PR1_ASCS00_pr1-ascs -s PR1 -n 00 -U SAPCLUSTER\SAPServicePR1 -P mypasswd12 -e SAPCLUSTER\pr1adm
```

![圖 2：重新安裝 SAP 服務][sap-ha-guide-figure-8013]

_**圖 2：** 重新安裝 SAP 服務_

請確定參數都正確無誤，然後選取 [手動] 作為 [啟動類型]。

## <a name="stop-the-ascsscs-service"></a>停止 ASCS/SCS 服務

在兩個 ASCS/SCS 叢集節點上停止 SAP ASCS/SCS 服務 SAP\<SID>_\<InstanceNumber>。

## <a name="create-a-new-sap-service-and-sap-instance-resources"></a>建立新的 SAP 服務和 SAP 執行個體資源

若要完成建立 SAP SAP\<SID> 叢集群組的資源，請建立下列資源：

* SAP \<SID> \<InstanceNumber> 服務
* SAP \<SID> \<InstanceNumber> 執行個體

執行下列 PowerShell cmdlet：

```PowerShell
$SAPSID = "PR1"
$SAPInstanceNumber = "00"
$SAPNetworkNameClusterResourceName = "pr1-ascs"

$SAPServiceName = "SAP$SAPSID"+ "_" + $SAPInstanceNumber

$SAPClusterGroupName = "SAP $SAPSID"
$SAPServiceClusterResourceName = "SAP $SAPSID $SAPInstanceNumber Service"

$SAPASCSServiceClusterResource = Add-ClusterResource -Name $SAPServiceClusterResourceName -Group $SAPClusterGroupName -ResourceType "SAP Service" -SeparateMonitor -Verbose
$SAPASCSServiceClusterResource  | Set-ClusterParameter  -Name ServiceName -Value $SAPServiceName

#Set resource dependencies
Set-ClusterResourceDependency -Resource $SAPASCSServiceClusterResource  -Dependency "[$SAPNetworkNameClusterResourceName]" -Verbose

$SAPInstanceClusterResourceName = "SAP $SAPSID $SAPInstanceNumber Instance"

# Create SAP instance cluster resource
$SAPASCSServiceClusterResource = Add-ClusterResource -Name $SAPInstanceClusterResourceName -Group $SAPClusterGroupName -ResourceType "SAP Resource" -SeparateMonitor -Verbose

#Set SAP instance cluster resource parameters
$SAPASCSServiceClusterResource  | Set-ClusterParameter  -Name SAPSystemName -Value $SAPSID -Verbose
$SAPASCSServiceClusterResource  | Set-ClusterParameter  -Name SAPSystem -Value $SAPInstanceNumber -Verbose

#Set resource dependencies
Set-ClusterResourceDependency -Resource $SAPASCSServiceClusterResource  -Dependency "[$SAPServiceClusterResourceName]" -Verbose
```

## <a name="add-a-probe-port"></a>新增探查連接埠

使用 PowerShell 設定 SAP 叢集資源 SAP-SID-IP 探查連接埠。 如[本文][sap-high-availability-installation-wsfc-shared-disk-add-probe-port]所述，請在其中一個 SAP ASCS/SCS 叢集節點上執行此設定。

## <a name="install-an-ers-instance-on-both-cluster-nodes"></a>在兩個叢集節點上安裝 ERS 執行個體

在 ASCS/SCS 叢集的兩個節點上安裝加入佇列複寫伺服器 (ERS) 執行個體。 在 SWPM 功能表上，遵循此安裝路徑：

**\<產品>** > **\<DBMS>** > **安裝** > **其他 SAP 系統執行個體** > **加入佇列複寫伺服器執行個體**

## <a name="install-a-dbms-instance-and-sap-application-servers"></a>安裝 DBMS 執行個體和 SAP 應用程式伺服器

透過安裝下列項目，完成 SAP 系統安裝：
* DBMS 執行個體。
* 主要 SAP 應用程式伺服器。
* 其他 SAP 應用程式伺服器。

## <a name="next-steps"></a>後續步驟

* [在沒有共用磁碟的容錯移轉叢集上安裝 ASCS/SCS 執行個體 - 高可用性檔案共用的官方 SAP 指導方針][sap-official-ha-file-share-document]

* [Windows Server 2016 中的儲存空間直接存取][s2d-in-win-2016]

* [用於應用程式資料的向外延展檔案伺服器概觀][sofs-overview]

* [Windows Server 2016 中儲存空間的新功能][new-in-win-2016-storage]
