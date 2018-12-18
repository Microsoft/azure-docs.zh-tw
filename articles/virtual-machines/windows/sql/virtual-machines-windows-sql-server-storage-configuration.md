---
title: SQL Server VM 的儲存體組態 | Microsoft Docs
description: 本主題說明 Azure 如何在佈建期間針對 SQL Server VM 設定儲存體 (Resource Manager 部署模型)。 它也會說明如何針對現有的 SQL Server VM 設定儲存體。
services: virtual-machines-windows
documentationcenter: na
author: ninarn
manager: craigg
tags: azure-resource-manager
ms.assetid: 169fc765-3269-48fa-83f1-9fe3e4e40947
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 12/05/2017
ms.author: ninarn
ms.openlocfilehash: 21c8b955d48da03559097db93b2cb66029a203ec
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29399078"
---
# <a name="storage-configuration-for-sql-server-vms"></a>SQL Server VM 的儲存體組態
當您在 Azure 中設定 SQL Server 虛擬機器映像時，入口網站有協助自動進行儲存體設定。 這包括將儲存體附加至 VM、讓該儲存體可供 SQL Server 存取，並加以設定以針對特定的效能需求最佳化。

本主題說明 Azure 如何在佈建期間針對 SQL Server VM 及針對現有的 VM 設定儲存體。 此設定是以執行 SQL Server 之 Azure VM 的 [效能最佳作法](virtual-machines-windows-sql-performance.md) 為基礎。

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>先決條件
若要使用自動儲存體組態設定，您的虛擬機器需具備下列特性︰

* 使用 [SQL Server 資源庫映像](virtual-machines-windows-sql-server-iaas-overview.md#payasyougo)佈建。
* 使用 [Resource Manager 部署模型](../../../azure-resource-manager/resource-manager-deployment-model.md)。
* 使用 [進階儲存體](../premium-storage.md)。

## <a name="new-vms"></a>新的 VM
下列各節說明如何為新的 SQL Server 虛擬機器設定儲存體。

### <a name="azure-portal"></a>Azure 入口網站
使用 SQL Server 資源庫映像佈建 Azure VM 時，您可以選擇自動為新的 VM 設定儲存體。 您可指定儲存體大小、效能限制，以及工作負載類型。 下列螢幕擷取畫面顯示在 SQL VM 佈建期間使用的儲存體設定刀鋒視窗。

![佈建期間的 SQL Server VM 儲存體設定](./media/virtual-machines-windows-sql-storage-configuration/sql-vm-storage-configuration-provisioning.png)

根據您的選擇，Azure 會在建立 VM 後執行下列儲存體設定工作︰

* 建立進階儲存體資料磁碟並將其連接到虛擬機器。
* 設定 SQL Server 可存取的資料磁碟。
* 根據指定的大小和效能 (IOPS 和輸送量) 需求，設定存放集區中的資料磁碟。
* 建立存放集區與虛擬機器上新磁碟機的關聯。
* 根據您指定的工作負載類型 (資料倉儲、交易式處理或一般)，最佳化這個新的磁碟機。

如需 Azure 如何進行儲存體設定的詳細資訊，請參閱 [儲存體設定章節](#storage-configuration)。 如需如何在 Azure 入口網站中建立 SQL Server VM 的完整逐步解說，請參閱 [佈建教學課程](virtual-machines-windows-portal-sql-server-provision.md)。

### <a name="resource-manage-templates"></a>Resource Manager 範本
如果您使用下列 Resource Manager 範本，則預設會連接兩個進階資料磁碟，但不會設定存放集區。 不過，您可以自訂這些範本，以變更連接到虛擬機器的進階資料磁碟數目。

* [使用自動備份建立 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-autobackup)
* [使用自動修補建立 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-autopatching)
* [使用自 AKV 整合建立 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-keyvault)

## <a name="existing-vms"></a>現有的 VM
對於現有的 SQL Server VM，您可以在 Azure 入口網站中修改某些儲存體設定。 選取您的 VM，移至 [設定] 區域，然後選取 [SQL Server 組態]。 [SQL Server 組態] 刀鋒視窗會顯示您 VM 目前的儲存體使用情況。 下圖顯示您的 VM 上存在的所有磁碟機。 對於每個磁碟機，儲存空間會顯示於四個區段中︰

* SQL 資料
* SQL 記錄檔
* 其他 (非 SQL 儲存體)
* 可用

![設定現有 SQL Server VM 的儲存體](./media/virtual-machines-windows-sql-storage-configuration/sql-vm-storage-configuration-existing.png)

若要設定此儲存體以加入新的磁碟機或擴充現有的磁碟機，請按一下圖表上方的 [編輯] 連結。

視您之前是否用過這項功能而定，您看到的設定選項會有所不同。 第一次使用時，您可以指定新磁碟機的儲存體需求。 如果您先前使用這項功能來建立磁碟機，您可以選擇擴充該磁碟機的儲存體。

### <a name="use-for-the-first-time"></a>第一次使用
如果這是您第一次使用這項功能，您可以指定新磁碟機的儲存體大小和效能限制。 這種經驗類似於您在佈建時所會看到的情形。 主要差別在於您不得指定工作負載類型。 這項限制可防止中斷虛擬機器上任何現有的 SQL Server 設定。

![設定 SQL Server 儲存體滑桿](./media/virtual-machines-windows-sql-storage-configuration/sql-vm-storage-usage-sliders.png)

Azure 會根據您的規格建立新的磁碟機。 在此案例中，Azure 會執行下列儲存體設定工作︰

* 建立進階儲存體資料磁碟並將其連接到虛擬機器。
* 設定 SQL Server 可存取的資料磁碟。
* 根據指定的大小和效能 (IOPS 和輸送量) 需求，設定存放集區中的資料磁碟。
* 建立存放集區與虛擬機器上新磁碟機的關聯。

如需 Azure 如何進行儲存體設定的詳細資訊，請參閱 [儲存體設定章節](#storage-configuration)。

### <a name="add-a-new-drive"></a>加入新的磁碟機
如果您已在 SQL Server VM 上設定儲存體，展開儲存體會顯示兩個新選項。 第一個選項是加入新的磁碟機，以提升您的 VM 的效能層級。

![將新的磁碟機新增至 SQL VM](./media/virtual-machines-windows-sql-storage-configuration/sql-vm-storage-configuration-add-new-drive.png)

不過，新增磁碟機之後，您必須執行一些額外的手動設定，才能達成效能提升。

### <a name="extend-the-drive"></a>擴充磁碟機
擴充儲存體的另一個選項是擴充現有的磁碟機。 此選項會增加您的磁碟機的可用儲存體，但不會提升效能。 使用存放集區，您無法在建立存放集區後改變資料行數目。 資料行數目會決定可以等量分散於資料磁碟的平行寫入數目。 因此，任何加入的資料磁碟均無法提升效能。 它們只能為寫入的資料提供更多的儲存空間。 這項限制也表示，在擴充磁碟機時，資料行數目會決定您可以新增的資料磁碟數目下限。 因此，如果您建立的存放集區包含四個資料磁碟，則資料行數目也是四個。 每當您擴充儲存體時，您就必須新增至少四個資料磁碟。

![延伸 SQL VM 的磁碟機](./media/virtual-machines-windows-sql-storage-configuration/sql-vm-storage-extend-a-drive.png)

## <a name="storage-configuration"></a>儲存體組態
本節提供參考，佈建的 SQL VM 或 Azure 入口網站中的組態期間會自動執行 Azure 的儲存體組態變更。

* 如果您為 VM 選取了小於兩個 TB 的儲存體，則 Azure 不會建立存放集區。
* 如果您為 VM 選取了至少兩個 TB 的儲存體，則 Azure 會設定存放集區。 本主題的下一節提供存放集區設定的詳細資料。
* 自動儲存體設定一律使用 [儲存體](../premium-storage.md) P30 資料磁碟。 因此，您選取的 TB 數目與連接到您 VM 的資料磁碟數目會有 1:1 的對應。

如需價格資訊，請參閱 [磁碟儲存體](https://azure.microsoft.com/pricing/details/storage) 索引標籤上的 **儲存體價格** 頁面。

### <a name="creation-of-the-storage-pool"></a>建立存放集區
Azure 會使用下列設定在 SQL Server VM 上建立存放集區。

| 設定 | 值 |
| --- | --- |
| 等量大小 |256 KB (資料倉儲)；64 KB (交易式) |
| 磁碟大小 |每個磁碟 1 TB |
| 快取 |讀取 |
| 配置大小 |64 KB NTFS 配置單位大小 |
| 立即檔案初始化 |已啟用 |
| 在記憶體中鎖定頁面 |已啟用 |
| 復原 |簡單復原 (無恢復功能) |
| 資料行數目 |資料磁碟數量<sup>1</sup> |
| TempDB 位置 |儲存在資料磁碟上<sup>2</sup> |

<sup>1</sup> 建立存放集區之後，您就無法改變存放集區中的資料行數目。

<sup>2</sup> 這項設定只適用於您使用儲存體組態功能所建立的第一個磁碟機。

## <a name="workload-optimization-settings"></a>工作負載最佳化設定
下表描述三個可用的工作負載類型選項以及其對應的最佳化︰

| 工作負載類型 | 說明 | 最佳化 |
| --- | --- | --- |
| **一般** |支援大多數工作負載的預設設定 |None |
| **交易式處理** |可將儲存體最佳化來處理傳統資料庫 OLTP 工作負載。 |追蹤旗標 1117<br/>追蹤旗標 1118 |
| **資料倉儲** |可將儲存體最佳化來處理分析和報告工作負載。 |追蹤旗標 610<br/>追蹤旗標 1117 |

> [!NOTE]
> 當您在儲存體設定步驟中選取 SQL 虛擬機器來進行佈建時，您只能指定此工作負載類型。
>
>

## <a name="next-steps"></a>後續步驟
如需有關在 Azure VM 中執行 SQL Server 的其他主題，請參閱 [Azure 虛擬機器上的 SQL Server](virtual-machines-windows-sql-server-iaas-overview.md)。
