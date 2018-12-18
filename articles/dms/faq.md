---
title: 使用 Azure 資料庫移轉服務的相關常見問題集 | Microsoft Docs
description: 了解使用 Azure 資料庫移轉服務來執行資料庫移轉的相關常見問題集。
services: database-migration
author: HJToland3
ms.author: jtoland
manager: ''
ms.reviewer: ''
ms.service: database-migration
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 06/19/2018
ms.openlocfilehash: 2fd5049b8b65620087e3c1ec42b6a5dcb0e0741a
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36214098"
---
# <a name="faq-about-using-the-azure-database-migration-service"></a>使用 Azure 資料庫移轉服務的相關常見問題集
本文列出使用 Azure 資料庫移轉服務的相關常見問題集以及相關解答。

### <a name="q-what-is-azure-database-migration-service"></a>問： 何謂 Azure 資料庫移轉服務？
Azure 資料庫移轉服務是一個完全受控的服務，能夠從多個資料庫來源無縫移轉到 Azure 資料平台，將停機時間降到最低。 服務目前已正式推出，後續開發重點放在下列工作：
- 可靠性和效能。
- 反覆新增來源目標組。
- 持續投資無衝突的移轉。

### <a name="q-what-source-target-pairs-does-the-azure-database-migration-service-currently-support"></a>問： Azure 資料庫移轉服務目前支援哪些來源目標組？
這項服務目前支援從 SQL Server 移轉到 Azure SQL Database；而針對此支援案例，您現在可以移至 Azure 入口網站來開始使用 Azure 資料庫移轉服務。 從 Oracle 移轉至 Azure SQL Database 等其他來源目標組，則可透過受限制的私人預覽來使用。 若要獲得參與這些案例之受限私人預覽的機會，請在[這裡](https://aka.ms/dms-preview/)註冊。

### <a name="q-how-does-the-azure-database-migration-service-compare-to-other-microsoft-database-migration-tools-such-as-the-database-migration-assistant-dma-or-sql-server-migration-assistant-ssma"></a>問： Azure 資料庫移轉服務和其他 Microsoft 資料庫移轉工具 (例如，資料庫移轉小幫手 (DMA) 或 SQL Server 移轉小幫手 (SSMA)) 相較之下，孰優孰劣？
若要將資料庫大規模移轉至 Microsoft Azure，建議您使用 Azure 資料庫移轉服務這個方法。 如需 Azure 資料庫移轉服務和其他 Microsoft 資料庫移轉工具相較之下孰優孰劣的詳細資訊，以及在各種案例中使用這項服務的建議，請參閱[區分 Microsoft 的資料庫移轉工具和服務](https://blogs.msdn.microsoft.com/datamigration/2017/10/13/differentiating-microsofts-database-migration-tools-and-services/)部落格文章。

### <a name="q-how-does-the-azure-database-migration-service-compare-to-the-azure-migrate-offering"></a>問： Azure 資料庫移轉服務和 Azure Migrate 供應項目相較之下，孰優孰劣？
Azure 移轉服務可協助將內部部署虛擬機器移轉至 Azure IaaS。 此服務會評估移轉適用性，以及如何根據效能來調整大小，並估計在 Azure 中執行內部部署機器的成本。 Azure Migrate 可用來將內部部署的 VM 型工作負載隨即移轉到 Azure IaaS VM。 不過，與 Azure 資料庫移轉服務不同的是，Azure Migrate 並非特製化的資料庫移轉服務供應項目，因此不適用於 Azure PaaS 關聯式資料庫平台 (例如，Azure SQL Database、SQL Azure 或 Azure SQL Database 受控執行個體)。

### <a name="q-what-versions-of-sql-server-does-the-azure-database-migration-service-support-as-a-source"></a>問： Azure 資料庫移轉服務支援使用哪些版本的 SQL Server 作為來源？
從 SQL Server 移轉時，Azure 資料庫移轉服務支援 SQL Server 2005 到 SQL Server 2017。

### <a name="q-what-is-a-summary-of-the-steps-required-to-use-the-azure-database-migration-service-to-perform-a-database-migration"></a>問： 使用 Azure 資料庫移轉服務來執行資料庫移轉所需進行之步驟大致為何？
在典型的簡單資料庫移轉期間，您必須：
1.  建立目標資料庫。
2.  使用[資料庫移轉小幫手](https://www.microsoft.com/en-us/download/details.aspx?id=53595)移轉資料庫結構描述。
3.  建立 Azure 資料庫移轉服務的執行個體。
4.  建立移轉專案，以指定用於移轉的來源資料庫、目標資料庫和資料表。
5.  起始完整載入。
6.  挑選後續驗證。
7.  將生產環境手動轉換至新的雲端型資料庫。 

### <a name="q-what-are-the-prerequisites-for-using-the-azure-database-migration-service"></a>問： 使用 Azure 資料庫移轉服務的必要條件為何？
您必須注意數個必要條件，以確保在執行資料庫移轉時，Azure 資料庫移轉服務能順利執行。 某些必要條件會套用到服務所支援的所有案例 (來源目標組)，而有些必要條件則是特定案例獨有的。
Azure 資料庫移轉服務必要條件在所有支援的移轉案例中都是通用的，包含以下需求：
- 使用 Azure Resource Manager 部署模型來建立 Azure 資料庫移轉服務的 VNET，其使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 提供站對站連線能力給您的內部部署來源伺服器。
- 確定您的 Azure 虛擬網路 (VNET) 網路安全性群組規則不會封鎖下列通訊埠 443、53、9354、445、12000。 如需 Azure VNET NSG 流量篩選的詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)。
- 使用來源資料庫前面的防火牆應用裝置時，您可能必須新增防火牆規則，才能讓 Azure 資料庫移轉服務存取來源資料庫，以進行移轉。
 
如需使用 Azure 資料庫移轉服務完成特定移轉案例所需的所有必要條件清單，請參閱 Azure 資料庫移轉服務[文件](https://docs.microsoft.com/azure/dms/dms-overview) (網址為 docs.microsoft.com) 中的相關教學課程。

### <a name="q-how-do-i-find-the-ip-address-for-the-azure-database-migration-service-so-that-i-can-create-an-allow-list-for-the-firewall-rules-used-to-access-my-source-database-for-migration"></a>問： 要如何找到 Azure 資料庫移轉服務的 IP 位址，以便建立用來存取移轉來源資料庫之防火牆規則的允許清單？
請新增防火牆規則，允許 Azure 資料庫移轉服務存取移轉來源資料庫。 服務的 IP 位址是動態的，但如果您使用的是 Express Route，則會由公司網路私下指派此位址。 若要識別適當的 IP 位址，最簡單的方式是查看和所佈建 Azure 資料庫移轉服務資源相同的資源群組，以尋找相關聯的網路介面。 網路介面資源的名稱通常會以 NIC 前置詞作為開頭，後面接著獨特字元和數字序列，例如 NIC-jj6tnztnmarpsskr82rbndyp。 藉由選取此網路介面資源，您即可在 Azure 入口網站的 [資源概觀] 頁面上，看到必須包含在允許清單中的 IP 位址。

此外，您可能還需要在允許清單上包含 SQL Server 所接聽的連接埠來源。 根據預設，來源是連接埠 1433，但您也可以將來源 SQL Server 設定為接聽其他連接埠。 在此情況下，您也必須在允許清單上包含這些連接埠。 您可以使用動態管理檢視查詢來判斷 SQL Server 所接聽的連接埠：

```sql
    SELECT DISTINCT 
        local_tcp_port 
    FROM sys.dm_exec_connections 
    WHERE local_tcp_port IS NOT NULL
```
您也可以藉由查詢 SQL Server 錯誤記錄來判斷 SQL Server 所接聽的連接埠：

```sql
    USE master
    GO
    xp_readerrorlog 0, 1, N'Server is listening on' 
    GO
```

### <a name="q-are-there-any-recommendations-for-optimizing-the-performance-of-the-azure-database-migration-service"></a>問： 是否有任何將 Azure 資料庫移轉服務效能最佳化的建議？
若要提升使用服務來移轉資料庫的速度，您可以執行幾項工作：
- 在建立服務執行個體時，請使用多重 CPU 一般用途定價層，以讓服務充分利用多個 vCPU 來進行平行處理，提升資料轉送速度。
- 在資料移轉作業期間，暫時將 Azure SQL Database 目標執行個體相應增加為進階層 SKU，讓使用較低層級 SKU 時可能會影響資料轉送活動的 Azure SQL Database 節流減到最少。

### <a name="q-how-do-i-set-up-an-azure-virtual-network"></a>問： 如何設定 Azure 虛擬網路？
有多個 Microsoft 教學課程可逐步引導您完成 Azure VNET 的設定程序，至於官方文件，則會在 [Azure 虛擬網路](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)一文中出現。

### <a name="q-why-is-my-azure-database-migration-service-unavailable-or-stopped"></a>問： 為什麼我的 Azure 資料庫移轉服務無法使用或已停止？
如果使用者明確地停止 Azure 資料庫移轉服務 (DMS) 或服務未使用達 24 小時，服務將會處於停止或自動暫停的狀態。 在每個情況下，服務都將無法使用並處於停止的狀態。  若要繼續使用中的移轉，請重新啟動服務。

### <a name="q-where-can-i-leave-feedback-about-the-azure-database-migration-service"></a>問： 我可以在哪裡留下有關 Azure 資料庫移轉服務的意見反應？
我們很樂於聽取您的意見。 請透過 User Voice ([這裡](https://feedback.azure.com/forums/906100-azure-database-migration-service)) 傳送有關於 Azure 資料庫移轉服務的意見反應和想法。

## <a name="next-steps"></a>後續步驟
如需 Azure 資料庫移轉服務和區域可用性的概觀，請參閱[什麼是 Azure 資料庫移轉服務](dms-overview.md)一文。 
