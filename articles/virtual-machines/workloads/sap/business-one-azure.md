---
title: Azure 虛擬機器上的 SAP Business One | Microsoft Docs
description: Azure 上的 SAP Business One。
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: juergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/15/2018
ms.author: msjuergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e8bd5ddab4553807f59b7afdf32fbfc1703e3d75
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949520"
---
# <a name="sap-business-one-on-azure-virtual-machines"></a>Azure 虛擬機器上的 SAP Business One
本文提供在 Azure 虛擬機器上部署 SAP Business One 的指引。 本文無法取代 SAP Business One 的安裝文件。 本文應涵蓋在 Azure 基礎結構上執行 Business One 應用程式的基本規劃與部署指引。

Business One 支援兩種不同的資料庫：
- SQL Server - 請參閱 [SAP 附註 #928839 - Microsoft SQL Server 的版本規劃](https://launchpad.support.sap.com/#/notes/928839) \(英文\)
- SAP HANA - 如需 SAP HANA 確切的 SAP Business One 支援對照表，請參閱 [SAP產品可用性對照表](https://support.sap.com/pam) \(英文\)

關於 SQL Server 的基本部署考量如 [SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms-guide)中所述。 SAP HANA 的考量則會在本文中提及。

## <a name="prerequisites"></a>必要條件
若要使用本指南，您需要下列 Azure 元件的基本知識：

- [Windows 上的 Azure 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-manage-vm)
- [Linux 上的 Azure 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-manage-vm)
- [使用 PowerShell 來進行 Azure 網路作業和虛擬網路管理](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-virtual-network)
- [使用 CLI 來進行 Azure 網路作業和虛擬網路管理](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-virtual-network)
- [使用 Azure CLI 來管理 Azure 磁碟](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-manage-disks)

即使您只對 Business One 感興趣，[SAP NetWeaver 的 Azure 虛擬機器規劃和實作](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide)一文也提供了不錯的資訊。

文中假設您在部署 SAP Business One 時：

- 熟悉在 VM 之類的指定基礎結構上安裝 SAP HANA
- 熟悉在 Azure VM 之類的基礎結構上安裝 SAP Business One 應用程式
- 熟悉操作 SAP Business One 與選擇的 DBMS 系統
- 熟悉在 Azure 中部署基礎結構

本文並未涵蓋上述所有範圍。

除了 Azure 文件，您還應注意參考 Business One 的主要 SAP 附註，或 SAP Business One 的核心附註：

- [528296 - SAP Business One 版本與相關產品的一般概觀附註](https://launchpad.support.sap.com/#/notes/528296) \(英文\)
- [2216195 - SAP HANA 專屬版 SAP Business One 9.2 的版本更新附註](https://launchpad.support.sap.com/#/notes/2216195) \(英文\)
- [2483583 - SAP Business One 9.3 的核心附註](https://launchpad.support.sap.com/#/notes/2483583) \(英文\)
- [2483615 - SAP Business One 9.3 的版本更新附註](https://launchpad.support.sap.com/#/notes/2483615) \(英文\)
- [2483595 - Collective Note for SAP Business One 9.3 一般問題的共同附註](https://launchpad.support.sap.com/#/notes/2483595) \(英文\)
- [2027458 - SAP HANA 專屬版 SAP Business One 的 SAP HANA 相關主題共同諮詢附註](https://launchpad.support.sap.com/#/notes/2027458) \(英文\)


## <a name="business-one-architecture"></a>Business One 架構
Business One 是兩層式的應用程式：

- 用戶端層含有「複雜型」用戶端
- 資料庫層包含租用戶的資料庫結構描述

[SAP Business One 系統管理員指南](https://help.sap.com/http.svc/rc/879bd9289df34a47af838e67d74ea302/9.3/en-US/AdministratorGuide_SQL.pdf) \(英文\) 更清楚地說明了在用戶端中執行的元件，以及在伺服器端中執行的元件。 

由於用戶端層與 DBMS 層之間的重要互動會嚴重延遲，所以在 Azure 中部署時，這兩層都必須位於 Azure 中。 之後，針對 Business One 用戶端元件，使用者通常會以 RDS 連至一或多個執行 RDS 服務的 VM。

### <a name="sizing-vms-for-sap-business-one"></a>調整 SAP Business One 的 VM 大小

關於調整用戶端 VM 的大小，SAP 在 [SAP Business One 硬體需求指南](https://help.sap.com/http.svc/rc/011000358700000244612011e/9.3/en-US/B1_Hardware_Requirements_Guide.pdf) \(英文\) 文件中記載了各式資源需求。 對於 Azure，您必須著重於計算文件的第 2.4 章中所述的需求。

因為 Azure 虛擬機器負責裝載 Business One 用戶端元件與 DBMS 主機，因此只允許支援 SAP NetWeaver 的 VM。 若要尋找支援 SAP NetWeaver 的 Azure VM 清單，請閱讀 [SAP 附註 #1928533](https://launchpad.support.sap.com/#/notes/1928533) \(英文\)。

由於執行 SAP HANA 作為 Business One 的 DBMS 後端，因此只有在 [HANA 認證 IaaS 平台清單](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure%23SAP%20Business%20One) \(英文\) 中列為適用於 Business on HANA 的 VM，才支援用於 HANA。 將 SAP HANA 作為 DBMS 系統才有的這項較嚴格限制，不會影響 Business One 用戶端元件。

### <a name="operating-system-releases-to-use-for-sap-business-one"></a>要用於 SAP Business One 的作業系統版本

基本上，最好一律使用最新的作業系統版本。 特別是 Linux 環境中，會在 Suse 和 Red Hat 較新的不同次要版本中引進新的 Azure 功能。 在 Windows 端，強烈建議使用 Windows Server 2016。


## <a name="deploying-infrastructure-in-azure-for-sap-business-one"></a>為 SAP Business One 在 Azure 中部署基礎結構
接下來的數章中，會說明部署 SAP 相關的基礎結構部分。

### <a name="azure-network-infrastructure"></a>Azure 網路基礎結構
在 Azure 中部署時所需的網路基礎結構，取決於您是否為自己而部署單一 Business One 系統。 或者，您是主機服務提供者，為客戶的數十個 Business One 系統提供主機服務。 您連線至 Azure 的方式，對於設計也會有些許的改變。 在許多可能的設計當中，其中一種設計便是您可以使用 VPN 連線至 Azure，以及透過 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-plan-design) 或 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 將 Active Directory 擴充至 Azure。

![Business One 的簡易網路設定](./media/business-one-azure/simple-network-with-VPN.PNG)

此處顯示的簡易設定引進了數個安全性執行個體，以便控制和限制路由傳送。 一開始的配置如下： 

- 客戶內部部署端上的路由器/防火牆。
- 下一個執行個體是 [Azure 網路安全性群組](https://docs.microsoft.com/azure/virtual-network/security-overview)，可用來引進在其中執行 SAP Business One 設定的 Azure VNet 路由與安全性規則。
- 為了避免 Business One 用戶端使用者也能看到執行 Business One 伺服器的資料庫執行伺服器，您應該將裝載 Business One 用戶端和 Business One 伺服器的 VM 分別放在 VNet 內的兩個不同子網路中。
- 您可再次使用指派給兩個不同子網路的 Azure NSG，以限制 Business One 伺服器的存取權。

更複雜的 Azure 網路設定版本是以 Azure [記載的中樞與輪輻架構最佳做法](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)為基礎。 中樞與輪輻的架構模式會將第一個簡易設定變更為如下所示：


![Business One 的中樞與輪輻設定](./media/business-one-azure/hub-spoke-network-with-VPN.PNG)

若使用者是透過網際網路連線來連至 Azure，而非透過私人連線，則 Azure 中的網路設計應符合 [Azure 和網際網路之間的 DMZ](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/secure-vnet-dmz) 所述 Azure 參考架構中記載的原則。

### <a name="business-one-database-server"></a>Business One 資料庫伺服器
SQL Server 與 SAP HANA 均為可用的資料庫類型。 雖與 DBMS 無關，但您應閱讀[針對 SAP 工作負載部署 Azure 虛擬機器 DBMS 的考量](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general) \(英文\) 一文，以概略了解 Azure VM 中的 DBMS 部署，以及相關的網路作業與儲存體主題。

雖然已在特定和一般資料庫文件中強調，但您應熟悉下列主題：

- [管理 Azure 中 Windows 虛擬機器的可用性](https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability)和[管理 Linux 虛擬機器的可用性](https://docs.microsoft.com/azure/virtual-machines/linux/manage-availability)
- [虛擬機器的 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/)

這些文件應可協助您選擇儲存體類型與高可用性設定。

基本上，您應該：

- 使用 [Azure 進階儲存體](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage)，而不是 [Azure 標準儲存體](https://docs.microsoft.com/azure/virtual-machines/windows/standard-storage)
- 使用 Azure 受控磁碟，而不是非受控磁碟
- 確定磁碟設定中設定了充足的 IOPS 與 I/O 輸送量
- 合併 /hana/data 與 /hana/log 磁碟區，以便有高成本效益的儲存體設定


#### <a name="sql-server-as-dbms"></a>SQL Server 作為 DBMS
如需將 SQL Server 部署為 Business One 的 DBMS，請依照[適用於 SAP NetWeaver 的 SQL Server Azure 虛擬機器 DBMS 部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_sqlserver) \(英文\) 一文進行。 

SQL Server 的 DBMS 端粗略估算如下：

| 使用者人數 | vCPU | 記憶體 | 範例 VM 類型 |
| --- | --- | --- | --- |
| 最多 20 人 | 4 | 16 GB | D4s_v3、E4s_v3 |
| 最多 40 人 | 8 | 32 GB | D8s_v3、E8s_v3 |
| 最多 80 人 | 16 | 64 GB | D16s_v3、E16s_v3 |
| 最多 150 人 | 32 | 128 GB | D32s_v3、E32s_v3 |

由以上所列的大小，應可大致了解要從哪裡開始著手。 不論您需要較少或較多的資源，都可在 Azure 上輕鬆進行調整。 只要重新啟動 VM，即可在 VM 類型之間進行變更。


#### <a name="sap-hana-as-dbms"></a>SAP HANA 作為 DBMS
使用 SAP HANA 作為 DBMS 時，您應在下列各節遵循 [Azure 上的 SAP Hana 作業指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)一文中的考量。

對於 SAP HANA 作為 Azure 中 Business One 的資料庫，您應閱讀 [Azure 虛擬機器的 SAP Hana 高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-overview)一文和該文件所指向的文件，以取得有關其高可用性和災害復原設定。

對於 SAP HANA 備份與還原策略，您應閱讀 [Azure 虛擬機器上 SAP HANA 的備份指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-guide)一文和該文件所指向的文件。

 
### <a name="business-one-client-server"></a>Business One 用戶端伺服器
對於這些元件而言，儲存體考量並非其主要的考量。 不過，您仍然需要可靠的平台。 因此，即使是基底 VHD，此 VM 也應使用 Azure 進階儲存體。 使用 [SAP Business One 硬體需求指南](https://help.sap.com/http.svc/rc/011000358700000244612011e/9.3/en-US/B1_Hardware_Requirements_Guide.pdf) \(英文\) 中所提供的資料來調整 VM 大小。 對於 Azure，您必須著重於計算文件的第 2.4 章中所述的需求。 計算需求時，您必須對照下列文件，以找出理想的 VM：

- [Azure 中的 Windows 虛擬機器大小](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)
- [SAP 附註 #1928533](https://launchpad.support.sap.com/#/notes/1928533) \(英文\)

針對 CPU 與記憶體，比較所需的數量與 Microsoft 記載的數量。 在選擇 VM 時，也請記得考量網路輸送量。








