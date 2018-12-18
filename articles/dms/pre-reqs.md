---
title: 使用 Azure 資料庫移轉服務的必要條件概觀 | Microsoft Docs
description: 深入了解使用 Azure 資料庫移轉服務來執行資料庫移轉的必要條件概觀。
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
ms.openlocfilehash: 3dc449724e405f83ce976b9f8b01a89c25d693fe
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36221245"
---
# <a name="overview-of-prerequisites-for-using-the-azure-database-migration-service"></a>使用 Azure 資料庫移轉服務的必要條件概觀
您必須注意數個必要條件，以確保在執行資料庫移轉時，Azure 資料庫移轉服務能順利執行。 某些必要條件會套用到服務所支援的所有案例 (來源目標配對)，而有些必要條件則是特定案例獨有的。

下列各節列出與使用 Azure 的資料庫移轉服務相關聯的必要條件。

## <a name="prerequisites-common-across-migration-scenarios"></a>在移轉案例之間通用的必要條件
Azure 資料庫移轉服務必要條件在所有支援的移轉案例中都是通用的，包含以下需求：
- 使用 Azure Resource Manager 部署模型來建立 Azure 資料庫移轉服務的 VNET，其使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 提供站對站連線能力給您的內部部署來源伺服器。
- 確定您的 Azure 虛擬網路 (VNET) 網路安全性群組規則不會封鎖下列通訊埠 443、53、9354、445、12000。 如需 Azure VNET NSG 流量篩選的詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)。
- 使用來源資料庫前面的防火牆應用裝置時，您可能必須新增防火牆規則，才能讓 Azure 資料庫移轉服務存取來源資料庫，以進行移轉。
- 設定[用於 Database Engine 存取的 Windows 防火牆](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
- 啟用 TCP/IP 通訊協定，在 SQL Server Express 安裝期間預設會停用，方法是遵循[啟用或停用伺服器網路通訊協定](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure)一文中的指示。

## <a name="prerequisites-for-migrating-sql-server-to-azure-sql-database"></a>將 SQL Server 移轉至 Azure SQL Database 的先決條件 
除了所有移轉案例中都通用的 Azure 資料庫移轉服務先決條件之外，還有一些是適用於個別案例的必要條件。

當您使用 Azure 資料庫移轉服務來執行將 SQL Server 移轉到 Azure SQL Database 時，除了所有移轉案例中都通用的必要條件之外，請務必提出下列其他先決條件：

- 您可以依照[在 Azure 入口網站中建立 Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal) 一文中的詳細資料來建立 Azure SQL Database 執行個體。
- 下載及安裝[資料移轉小幫手](https://www.microsoft.com/download/details.aspx?id=53595) v3.3 或更新版本。
- 開啟您的 Windows 防火牆以允許 Azure 資料庫移轉服務存取來源 SQL Server，其預設會通過 TCP 連接埠 1433。
- 如果您使用動態連接埠執行多個具名 SQL Server 執行個體，也許會想要啟用 SQL Browser 服務並允許通過防火牆存取 UDP 連接埠 1434，讓 Azure 資料庫移轉服務連線來源伺服器上的具名執行個體。
- 針對 Azure SQL Database 伺服器建立伺服器層級的[防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允許 Azure 資料庫移轉服務存取目標資料庫。 提供用於 Azure 資料庫移轉服務之 VNET 的子網路範圍。
- 確定用來連線至來源 SQL Server 執行個體的認證具有 [CONTROL SERVER](https://docs.microsoft.com/sql/t-sql/statements/grant-server-permissions-transact-sql) 權限。
- 確定用來連線至目標 Azure SQL DB 執行個體的認證，在目標 Azure SQL 資料庫上具有 CONTROL DATABASE 權限。

   > [!NOTE]
   > 如需使用 Azure 資料庫移轉服務執行從 SQL Server 移轉到 Azure SQL Database 所需的必要條件完整清單，請參閱本教學課程[將 SQL Server 移轉到 Azure SQL Database](https://docs.microsoft.com/azure/dms/tutorial-sql-server-to-azure-sql)。
   > 

## <a name="prerequisites-for-migrating-sql-server-to-azure-sql-database-managed-instance"></a>將 SQL Server 移轉至 Azure SQL Database 受控執行個體的先決條件
- 依照[在 Azure 入口網站中建立 Azure SQL Database 受控執行個體](https://aka.ms/sqldbmi) \(英文\) 一文中的詳細資料，建立 Azure SQL Database 受控執行個體的執行個體。
- 開啟您的防火牆，以針對 Azure 資料庫移轉服務 IP 位址或子網路範圍，允許連接埠 445 上的 SMB 流量。
- 開啟您的 Windows 防火牆以允許 Azure 資料庫移轉服務存取來源 SQL Server，其預設會通過 TCP 連接埠 1433。
- 如果您使用動態連接埠執行多個具名 SQL Server 執行個體，也許會想要啟用 SQL Browser 服務並允許通過防火牆存取 UDP 連接埠 1434，讓 Azure 資料庫移轉服務連線來源伺服器上的具名執行個體。
- 確定用來連線來源 SQL Server 和目標受控執行個體的登入是 sysadmin 伺服器角色的成員。
- 建立 Azure 資料庫移轉服務可用來備份來源資料庫的網路共用。
- 確認執行來源 SQL Server 執行個體的服務帳戶在您所建立的網路共用上具有寫入權限，而且來源伺服器的電腦帳戶對相同的共用具備讀取/寫入存取權。
- 記下在您先前建立的網路共用上具有完整控制權限的 Windows 使用者 (和密碼)。 Azure 資料庫移轉服務會模擬該使用者認證，以便將備份檔案上傳至 Azure 儲存體容器以進行還原作業。
- 利用[使用儲存體總管來管理 Azure Blob 儲存體資源](https://docs.microsoft.com/azure/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container)一文中的步驟，來建立 Blob 容器並擷取其 SAS URI。 建立 SAS URI 時，請務必選取原則視窗上的所有權限 (讀取、寫入、刪除、列示)。

   > [!NOTE]
   > 如需使用 Azure 資料庫移轉服務執行從 SQL Server 移轉到 Azure SQL Database 受控執行個體所需的先決條件完整清單，請參閱[將 SQL Server 移轉到 Azure SQL Database 受控執行個體](https://aka.ms/migratetomiusingdms) \(英文\) 教學課程。

## <a name="next-steps"></a>後續步驟
如需 Azure 資料庫移轉服務和區域可用性的概觀，請參閱[什麼是 Azure 資料庫移轉服務](dms-overview.md)一文。 