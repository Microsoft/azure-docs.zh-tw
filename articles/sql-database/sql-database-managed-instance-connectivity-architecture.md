---
title: Azure SQL Database 受控執行個體連線架構 | Microsoft Docs
description: 此文章提供 Azure SQL Database 受控執行個體通訊概觀並說明連線架構，以及不同的元件如何運作以將流量導向到受控執行個體。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: bonova, carlrab
manager: craigg
ms.date: 08/16/2018
ms.openlocfilehash: 312425d3ea02d15a992b9a694f09cb2be73b6221
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161585"
---
# <a name="azure-sql-database-managed-instance-connectivity-architecture"></a>Azure SQL Database 受控執行個體連線架構 

此文章提供 Azure SQL Database 受控執行個體通訊概觀並說明連線架構，以及不同的元件如何運作以將流量導向到受控執行個體。  

## <a name="communication-overview"></a>通訊概觀 

下圖顯示連線到受控執行個體的實體，以及受控執行個體必須存取以正確運作的資源。 

![連線架構實體](./media/managed-instance-connectivity-architecture/connectivityarch001.png)

圖表底部描述的通訊代表連線到受控執行個體做為資料來源的客戶應用程式與工具。  

受控執行個體是平台即服務 (PaaS) 供應項目，Microsoft 使用自動化代理程式 (管理、部署與維護) 來管理此服務。 因為受控執行個體完全是 Microsoft 的責任，客戶無法透過 RDP 來存取受控執行個體虛擬叢集機器。 

由使用者或應用程式起始的某些 SQL Server 作業可能需要受控執行個體才能與該平台互動。 其中一個案例是建立受控執行個體資料庫，這是透過入口網站、PowerShell 或 Azure CLI 公開的資源。 

受控執行個體依賴其他 Azure 服務以正常運作 (例如依賴 Azure 儲存體來進行備份、依賴 Azure 服務匯流排來進行遙測、依賴 Azure AD 來進行驗證、依賴 Azure Key Vault 來進行 TDE 等) 並據以起始連到這些服務的連線。 

上述所有通訊都經過加密並使用憑證簽署。 為確認通訊方受信任，受控執行個體會持續連絡「憑證授權單位」來驗證這些憑證。 若憑證已被撤銷或受控執行個體無法驗證憑證，它會關閉連線以保護資料。 

## <a name="high-level-connectivity-architecture"></a>高層級連線架構 

在較高的層級中，受控執行個體是服務元件集合，在專用隔離虛擬機器集合上裝載，這些虛擬機器在客戶虛擬網路子網路內從虛擬叢集執行。 

可以在單一虛擬叢集中裝載多個受控執行個體。 當客戶變更子網路中佈建的執行個體數目時，叢集會視需要自動擴充或縮減。 

客戶應用程式可以連線到受控執行個體、查詢及更新資料庫，但它們必須在虛擬網路或對等虛擬網路或 VPN / Express Route 連線網路內使用具有私人 IP 位址的端點來執行。  

![連線架構圖](./media/managed-instance-connectivity-architecture/connectivityarch002.png)

Microsoft 管理與部署服務是在虛擬網路外部執行，因此受控執行個體與 Microsoft 服務之間的連線會經過具有私人 IP 位址的端點。 由於網路位址轉譯 (NAT)，當受控執行個體建立連出連線時，在接收端它看起來就像是來自此公開 IP。 

管理流量會流經客戶虛擬網路。 這表示虛擬網路基礎結構的元素會影響且甚至可能會傷害管理流量，導致執行個體進入失敗狀態並變得無法使用。 

> [!IMPORTANT]
> 為要改善客戶體驗與服務可用性，Microsoft 在會影響受控執行個體運作的 Azure 虛擬網路基礎結構元素上套用「網路意圖原則」。 這是一個平台機制，能以背景方式向使用者溝通網路需求，主要目標是防止網路設定錯誤並確保正常的受控執行個體作業。 刪除受控執行個體時，會一併移除「網路意圖原則」。 

## <a name="virtual-cluster-connectivity-architecture"></a>虛擬叢集連線架構 

讓我們深入了解受控執行個體連線架構。 下圖顯示虛擬叢集的概念性配置。 

![連線架構圖 - 虛擬叢集](./media/managed-instance-connectivity-architecture/connectivityarch003.png)

客戶使用具有 <mi_name>.<clusterid>.database.windows.net 格式的主機名稱連線到受控執行個體。 雖然此主機名稱已在公開 DNS 區域中註冊而且可由公眾解析，但它會解析為私人 IP 位址。 

這個私人 IP 位址屬於受控執行個體內部負載平衡器 (ILB)，此負載平衡器會將流量導向到受控執行個體閘道 (GW)。 多個受控執行個體可能會刻意在相同的叢集中執行，GW 使用受控執行個體主機名稱來將流量導向到正確的 SQL Engine 服務。 

管理與部署服務會使用對應到外部負載平衡器的公開端點來連線到受控執行個體。 只有在已預先定義並專門由受控執行個體管理元件所使用之連接埠集合上收到的流量才會路由傳送到那些節點。 管理元件與管理平面之間的所有通訊都會互相進行憑證驗證。 

## <a name="next-steps"></a>後續步驟 

- 如需概觀，請參閱 [受控執行個體是什麼](sql-database-managed-instance.md)。 
- 如需 VNet 設定的詳細資訊，請參閱 [受控執行個體 VNet 設定](sql-database-managed-instance-vnet-configuration.md)。 
- 如需快速入門，請參閱＜如何建立受控執行個體＞： 
  - 從 [Azure 入口網站](sql-database-managed-instance-get-started.md) 
  - 使用 [PowerShell](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2018/06/27/quick-start-script-create-azure-sql-managed-instance-using-powershell/) 
  - 使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/101-sqlmi-new-vnet/) 
  - 使用 [Azure Resource Manager 範本 (包含 SSMS)](https://portal.azure.com/) 

