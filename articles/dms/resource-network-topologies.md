---
title: 使用 Azure 資料庫移轉服務進行 Azure SQL Database 受控執行個體移轉的網路拓撲 | Microsoft Docs
description: 了解資料庫移轉服務的來源和目標設定。
services: database-migration
author: pochiraju
ms.author: rajpo
manager: ''
ms.reviewer: ''
ms.service: database-migration
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 10/10/2018
ms.openlocfilehash: 39bcea36f3599530413aa9fc4dbb308ee2fb1681
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49066848"
---
# <a name="network-topologies-for-azure-sql-db-managed-instance-migrations-using-the-azure-database-migration-service"></a>使用 Azure 資料庫移轉服務進行 Azure SQL DB 受控執行個體移轉的網路拓樸
本文會討論 Azure 資料庫移轉服務進行內部部署 SQL Server 到 Azure SQL Database 受控執行個體移轉時，用於提供全面移轉體驗的各種網路拓撲。

## <a name="azure-sql-database-managed-instance-configured-for-hybrid-workloads"></a>針對混合式工作負載設定 Azure SQL Database 受控執行個體 
如果您的 Azure SQL Database 受控執行個體是連線到內部部署網路，請使用此拓撲。 這種方法提供最簡單的網路路由，並在移轉期間產生最大的資料輸送量。

![混合式工作負載的網路拓撲](media\resource-network-topologies\hybrid-workloads.png)

**需求**
- 在此案例中，Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務執行個體是建立在相同 Azure VNET 中，但是會使用不同的子網路。  
- 此案例中使用的 VNET 會使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 來連線到內部部署網路。

## <a name="azure-sql-database-managed-instance-isolated-from-the-on-premises-network"></a>Azure SQL Database 受控執行個體與內部部署網路分開
如果您的環境需符合一或多個下列情況案例，請使用此網路拓撲：
- Azure SQL Database 受控執行個體與內部部署的連線功能獨立分開，但您的 Azure 資料庫移轉服務執行個體是連線到內部部署網路。
- 如果角色型存取控制 (RBAC) 原則已就緒，而您必須限制使用者存取裝載 Azure SQL Database 受控執行個體的相同訂用帳戶。
- Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務使用的 VNET 位於不同訂用帳戶。

![受控執行個體與內部部署網路分開的網路拓撲](media\resource-network-topologies\mi-isolated-workload.png)

**需求**
- 此案例中 Azure 資料庫移轉服務使用的 VNET 也必須使用 https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 來連線到內部部署網路。
- 在 Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務使用的 VNET 之間設定 [VNET 網路對等互連](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)。


## <a name="cloud-to-cloud-migrations-shared-vnet"></a>雲端至雲端的移轉：共用的 VNET

如果來源 SQL Server 裝載於 Azure VM，而且與 Azure SQL Database 受控執行個體及 Azure 資料庫移轉服務共用相同的 VNET，則請使用此拓撲。

![使用共用的 VNET 進行雲端至雲端移轉的網路拓撲](media\resource-network-topologies\cloud-to-cloud.png)

**需求**
- 沒有其他需求。

## <a name="cloud-to-cloud-migrations-isolated-vnet"></a>雲端至雲端的移轉：隔離的 VNET

如果您的環境需符合一或多個下列情況案例，請使用此網路拓撲：
- Azure SQL Database 受控執行個體乃是佈建在隔離的 VNET 中。
- 如果角色型存取控制 (RBAC) 原則已就緒，而您必須限制使用者存取裝載 Azure SQL Database 受控執行個體的相同訂用帳戶。
- Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務使用的 VNET 位於不同訂用帳戶。

![使用隔離的 VNET 進行雲端至雲端移轉的網路拓撲](media\resource-network-topologies\cloud-to-cloud-isolated.png)

**需求**
- 在 Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務使用的 VNET 之間設定 [VNET 網路對等互連](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)。


## <a name="see-also"></a>另請參閱
- [將 SQL Server 遷移至 Azure SQL Database 受控執行個體](https://docs.microsoft.com/azure/dms/tutorial-sql-server-to-managed-instance)
- [使用 Azure 資料庫移轉服務的必要條件概觀](https://docs.microsoft.com/azure/dms/pre-reqs)
- [使用 Azure 入口網站建立虛擬網路](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

## <a name="next-steps"></a>後續步驟
如需 Azure 資料庫移轉服務和公開預覽期間區域可用性的概觀，請參閱[什麼是 Azure 資料庫移轉服務預覽](dms-overview.md)一文。 