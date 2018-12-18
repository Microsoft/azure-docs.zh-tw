---
title: 監視 Azure Cosmos DB 要求及儲存體 | Microsoft Docs
description: 了解如何監視 Azure Cosmos DB 帳戶的效能計量 (如要求和伺服器錯誤) 和使用量計量 (如儲存體耗用量)。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 09/19/2017
ms.author: sngun
ms.openlocfilehash: 1f4bf4ba660e83be9879c56d04b27c932d563e17
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49354054"
---
# <a name="monitor-azure-cosmos-db"></a>監視 Azure Cosmos DB
您可以在 [Azure 入口網站](https://portal.azure.com/)中監視 Azure Cosmos DB 帳戶。 對於每個 Azure Cosmos DB 帳戶，會提供一套完整的度量，供監視輸送量、儲存體、可用性、延遲和一致性。

您可以在 [帳戶] 頁面、新的 [度量] 頁面，或 Azure 監視器中檢閱度量。

## <a name="view-performance-metrics-on-the-metrics-page"></a>在度量頁面上檢視效能度量
1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [所有服務]，捲動至 [資料庫]，按一下 [Azure Cosmos DB]，然後按一下您想要檢視其效能計量的 Azure Cosmos DB 帳戶名稱。
2. 當新頁面載入時，於 [資源] 功能表的 [監視] 下，按一下 [度量]。
3. 當 [度量] 頁面開啟時，請從 [集合] 下拉式清單選取要檢閱的集合。

   Azure 入口網站會顯示可用集合度量的套件。 請注意，輸送量、儲存體、可用性、延遲和一致性度量會提供個別索引標籤上。 若要取得所提供度量的其他詳細資料，請按一下每個度量窗格右上角的雙箭頭。

   ![顯示度量套件的 [監視] 功能濾鏡的螢幕擷取畫面](./media/monitor-accounts/metrics-suite.png)

## <a name="view-performance-metrics-by-using-azure-monitoring"></a>使用 Azure 監視器檢視效能度量
1. 在 [Azure 入口網站](https://portal.azure.com/)，按一下左側列中的 [監視器]。
2. 在資源功能表中，按一下 [度量]。
3. 在 [監視 - 計量] 視窗的 [資源群組] 下拉式功能表中，選取與您要監視的 Azure Cosmos DB 帳戶相關聯的資源群組。 
4. 在 [資源] 下拉式功能表中，選取要監視的資料庫帳戶。
5. 在 [可用的度量] 清單中，選取要顯示的度量。 使用 CTRL 按鈕同時選取多項。 

## <a name="view-performance-metrics-on-the-account-page"></a>在帳戶頁面上檢視效能度量
1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [所有服務]，捲動至 [資料庫]，按一下 [Azure Cosmos DB]，然後按一下您想要檢視其效能計量的 Azure Cosmos DB 帳戶名稱。
2. [監視]  功能濾鏡預設會顯示以下圖格：
   
   * 當日的要求總數。
   * 已使用儲存體。
   
   ![[監視] 功能濾鏡的螢幕擷取畫面，可顯示要求和儲存體使用量](./media/monitor-accounts/documentdb-total-requests-and-usage.png)
3. 按一下 [要求] 磚右上方的雙箭號上會開啟詳細的 [度量] 頁面。
4. [度量] 頁面會顯示要求總數的相關詳細資料。 

## <a name="set-up-alerts-in-the-portal"></a>在入口網站中設定警示
1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [所有服務]，按一下 [Azure Cosmos DB]，然後按一下您想要設定其效能計量警示的 Azure Cosmos DB 帳戶名稱。
2. 在資源功能表中，按一下 [警示規則] 以開啟 [警示規則] 頁面。  
   ![已選取 [警示規則] 組件的螢幕擷取畫面](./media/monitor-accounts/madocdb10.5.png)
3. 在 [警示規則] 頁面中，按一下 [新增警示]。  
   ![包含反白顯示 [新增警示] 按鈕的 [警示規則] 頁面螢幕擷取畫面](./media/monitor-accounts/madocdb11.png)
4. 在 [新增警示規則] 頁面中，指定：
   
   * 您設定之警示規則的名稱。
   * 新警示規則的描述。
   * 警示規則的度量。
   * 決定警示何時啟動的條件、臨界值和期間。 例如，過去 15 分鐘伺服器錯誤計算大於 5。
   * 警示引發時是否傳送電子郵件給服務管理員和共同管理員。
   * 警示通知的其他電子郵件地址。  
     ![[新增警示規則] 頁面的螢幕擷取畫面](./media/monitor-accounts/madocdb12.png)

## <a name="monitor-azure-cosmos-db-programmatically"></a>以程式設計方式監視 Azure Cosmos DB
可在入口網站中取得的帳戶層級計量 (例如，帳戶儲存體使用量和要求總數) 無法透過 SQL API 取得。 不過，您可以使用 SQL API 來擷取集合層級的使用量資料。 若要擷取集合層級的資料，請執行下列動作：

* 若要使用 REST API，請 [在集合上執行 GET](https://msdn.microsoft.com/library/mt489073.aspx)。 集合的配額和使用量資訊會在回應的 x-ms-resource-quota 和 x-ms-resource-usage 標頭中傳回。
* 若要使用 .NET SDK，請使用 [DocumentClient.ReadDocumentCollectionAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.readdocumentcollectionasync.aspx) 方法，此方法會傳回包含 **CollectionSizeUsage**、**DatabaseUsage**、**DocumentUsage** 等幾個使用量屬性的 [ResourceResponse](https://msdn.microsoft.com/library/dn799209.aspx)。

若要存取其他度量，請使用 [Azure 監視器 SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights)。 您可以呼叫下列程式碼來擷取可用的度量定義：

    https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroup}/providers/Microsoft.DocumentDb/databaseAccounts/{DocumentDBAccountName}/metricDefinitions?api-version=2015-04-08

擷取個別度量的查詢會使用下列格式：

    https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroup}/providers/Microsoft.DocumentDb/databaseAccounts/{DocumentDBAccountName}/metrics?api-version=2015-04-08&$filter=%28name.value%20eq%20%27Total%20Requests%27%29%20and%20timeGrain%20eq%20duration%27PT5M%27%20and%20startTime%20eq%202016-06-03T03%3A26%3A00.0000000Z%20and%20endTime%20eq%202016-06-10T03%3A26%3A00.0000000Z

如需詳細資訊，請參閱 [Retrieving Resource Metrics via the Azure Monitor REST API (透過 Azure 監視器 API 擷取資源度量)](https://blogs.msdn.microsoft.com/cloud_solution_architect/2016/02/23/retrieving-resource-metrics-via-the-azure-insights-api/)。 請注意，"Azure Insights" 已重新命名為「Azure 監視器」。  此部落格項目參考的是舊名稱。

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure Cosmos DB 容量規劃，請參閱 [Azure Cosmos DB 容量規劃工具計算機 (英文)](https://www.documentdb.com/capacityplanner)。

