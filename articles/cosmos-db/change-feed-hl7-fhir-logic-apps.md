---
title: 變更 HL7 FHIR 資源的摘要 - Azure Cosmos DB | Microsoft Docs
description: 了解如何使用 Azure Logic Apps、Azure Cosmos DB 和服務匯流排，來設定 HL7 FHIR 病患之醫療保健記錄的變更通知。
keywords: hl7 fhir
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 02/08/2017
ms.author: sngun
ms.openlocfilehash: d40ab5d6bb29878c633a2645810d6256ac661071
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39213695"
---
# <a name="notifying-patients-of-hl7-fhir-health-care-record-changes-using-logic-apps-and-azure-cosmos-db"></a>使用 Logic Apps 與 Azure Cosmos DB 對 HL7 FHIR 病患的醫療保健記錄變更發出通知

某家醫療保健組織最近與 Howard Edidin 這位 Azure MVP 連絡，因為他們想要在其病患入口網站中新增功能。 他們需要在病患的健康記錄有所更新時對病患發出通知，並讓他們能夠訂閱這些更新。 

本文會逐步引導您了解使用 Azure Cosmos DB、Logic Apps 和服務匯流排，為此醫療保健組織所建立的變更摘要通知方案。 

## <a name="project-requirements"></a>專案需求
- 提供者以 XML 格式傳送 HL7 綜合臨床文件架構 (Consolidated-Clinical Document Architecture, C-CDA) 文件。 C-CDA 文件幾乎包含所有類型的臨床文件，包括家族史和免疫記錄之類的臨床文件，以及系統管理、工作流程和財務方面的文件。 
- C-CDA 文件會轉換成 JSON 格式的 [HL7 FHIR 資源](http://hl7.org/fhir/2017Jan/resourcelist.html)。
- 修改過的 FHIR 資源文件會以 JSON 格式透過電子郵件傳送。

## <a name="solution-workflow"></a>方案工作流程 

概括而言，此專案需要下列工作流程步驟︰ 
1. 將 C-CDA 文件轉換成 FHIR 資源。
2. 針對修改過的 FHIR 資源執行週期性觸發輪詢。 
2. 呼叫自訂應用程式 FhirNotificationApi 來連線到 Azure Cosmos DB，並查詢是否有新的或修改過的文件。
3. 將回應儲存到服務匯流排佇列。
4. 輪詢服務匯流排佇列中的新訊息。
5. 傳送電子郵件通知給病患。

## <a name="solution-architecture"></a>方案架構
此方案需要三個 Logic Apps 才能符合上述需求，並完成方案工作流程。 這三個邏輯應用程式分別是︰
1. **HL7-FHIR-Mapping 應用程式**︰接收 HL7 C-CDA 文件、將其轉換為 FHIR 資源，然後儲存至 Azure Cosmos DB。
2. **EHR 應用程式**︰查詢 Azure Cosmos DB FHIR 存放庫，並將回應儲存到服務匯流排佇列。 此邏輯應用程式使用 [API 應用程式](#api-app)來擷取新的和變更的文件。
3. **處理通知應用程式**︰傳送內文中有 FHIR 資源文件的電子郵件通知。

![此 HL7 FHIR 醫療保健方案所使用的三個 Logic Apps](./media/change-feed-hl7-fhir-logic-apps/health-care-solution-hl7-fhir.png)



### <a name="azure-services-used-in-the-solution"></a>此方案所使用的 Azure 服務

#### <a name="azure-cosmos-db-sql-api"></a>Azure Cosmos DB SQL API
Azure Cosmos DB 是 FHIR 資源的存放庫，如下圖所示。

![此 HL7 FHIR 醫療保健教學課程所使用的 Azure Cosmos DB 帳戶](./media/change-feed-hl7-fhir-logic-apps/account.png)

#### <a name="logic-apps"></a>Logic Apps
Logic Apps 會處理工作流程程序。 下列螢幕擷取畫面顯示為此方案所建立的邏輯應用程式。 


1. **HL7-FHIR-Mapping 應用程式**︰接收 HL7 C-CDA 文件，然後使用適用於 Logic Apps 的企業整合套件將其轉換為 FHIR 資源。 企業整合套件會處理 C-CDA 與 FHIR 資源的對應。

    ![用來接收 HL7 FHIR 醫療保健記錄的邏輯應用程式](./media/change-feed-hl7-fhir-logic-apps/hl7-fhir-logic-apps-json-transform.png)


2. **EHR 應用程式**︰查詢 Azure Cosmos DB FHIR 存放庫，並將回應儲存到服務匯流排佇列。 GetNewOrModifiedFHIRDocuments 應用程式的程式碼在下面。

    ![用來查詢 Azure Cosmos DB 的邏輯應用程式](./media/change-feed-hl7-fhir-logic-apps/hl7-fhir-logic-apps-api-app.png)

3. **處理通知應用程式**︰傳送內文中有 FHIR 資源文件的電子郵件通知。

    ![會傳送內文中有 HL7 FHIR 資源之電子郵件給病患的邏輯應用程式](./media/change-feed-hl7-fhir-logic-apps/hl7-fhir-logic-apps-send-email.png)

#### <a name="service-bus"></a>服務匯流排
下圖顯示病患佇列。 電子郵件主旨會使用 Tag 屬性值。

![此 HL7 FHIR 教學課程所使用的服務匯流排佇列](./media/change-feed-hl7-fhir-logic-apps/hl7-fhir-service-bus-queue.png)

<a id="api-app"></a>

#### <a name="api-app"></a>API 應用程式
API 應用程式會連線到 Azure Cosmos DB，並依資源類型查詢新的或修改過的 FHIR 文件。 此應用程式有一個控制器 **FhirNotificationApi** 與一項作業 **GetNewOrModifiedFhirDocuments**，請參閱 [API 應用程式來源](#api-app-source)。

我們會使用來自 Azure Cosmos DB SQL .NET API 的 [`CreateDocumentChangeFeedQuery`](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.createdocumentchangefeedquery.aspx) \(英文\) 類別。 如需詳細資訊，請參閱[變更摘要文章](change-feed.md)。 

##### <a name="getnewormodifiedfhirdocuments-operation"></a>GetNewOrModifiedFhirDocuments 作業

**輸入**
- DatabaseId
- CollectionId
- HL7 FHIR 資源類型名稱
- 布林值︰從頭開始
- Int：傳回的文件數目

**輸出**
- 成功︰狀態碼︰200，回應︰文件清單 (JSON 陣列)
- 失敗︰狀態碼︰404，回應：「找不到 'resource name' 資源類型的文件」

<a id="api-app-source"></a>

**API 應用程式的來源**

```csharp

    using System.Collections.Generic;
    using System.Linq;
    using System.Net;
    using System.Net.Http;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Microsoft.Azure.Documents;
    using Microsoft.Azure.Documents.Client;
    using Swashbuckle.Swagger.Annotations;
    using TRex.Metadata;
    
    namespace FhirNotificationApi.Controllers
    {
        /// <summary>
        ///     FHIR Resource Type Controller
        /// </summary>
        /// <seealso cref="System.Web.Http.ApiController" />
        public class FhirResourceTypeController : ApiController
        {
            /// <summary>
            ///     Gets the new or modified FHIR documents from Last Run Date 
            ///     or create date of the collection
            /// </summary>
            /// <param name="databaseId"></param>
            /// <param name="collectionId"></param>
            /// <param name="resourceType"></param>
            /// <param name="startfromBeginning"></param>
            /// <param name="maximumItemCount">-1 returns all (default)</param>
            /// <returns></returns>
            [Metadata("Get New or Modified FHIR Documents",
                "Query for new or modifed FHIR Documents By Resource Type " +
                "from Last Run Date or Begining of Collection creation"
            )]
            [SwaggerResponse(HttpStatusCode.OK, type: typeof(Task<dynamic>))]
            [SwaggerResponse(HttpStatusCode.NotFound, "No New or Modifed Documents found")]
            [SwaggerOperation("GetNewOrModifiedFHIRDocuments")]
            public async Task<dynamic> GetNewOrModifiedFhirDocuments(
                [Metadata("Database Id", "Database Id")] string databaseId,
                [Metadata("Collection Id", "Collection Id")] string collectionId,
                [Metadata("Resource Type", "FHIR resource type name")] string resourceType,
                [Metadata("Start from Beginning ", "Change Feed Option")] bool startfromBeginning,
                [Metadata("Maximum Item Count", "Number of documents returned. '-1 returns all' (default)")] int maximumItemCount = -1
            )
            {
                var collectionLink = UriFactory.CreateDocumentCollectionUri(databaseId, collectionId);
    
                var context = new DocumentDbContext();  
    
                var docs = new List<dynamic>();
    
                var partitionKeyRanges = new List<PartitionKeyRange>();
                FeedResponse<PartitionKeyRange> pkRangesResponse;
    
                do
                {
                    pkRangesResponse = await context.Client.ReadPartitionKeyRangeFeedAsync(collectionLink);
                    partitionKeyRanges.AddRange(pkRangesResponse);
                } while (pkRangesResponse.ResponseContinuation != null);
    
                foreach (var pkRange in partitionKeyRanges)
                {
                    var changeFeedOptions = new ChangeFeedOptions
                    {
                        StartFromBeginning = startfromBeginning,
                        RequestContinuation = null,
                        MaxItemCount = maximumItemCount,
                        PartitionKeyRangeId = pkRange.Id
                    };
    
                    using (var query = context.Client.CreateDocumentChangeFeedQuery(collectionLink, changeFeedOptions))
                    {
                        do
                        {
                            if (query != null)
                            {
                                var results = await query.ExecuteNextAsync<dynamic>().ConfigureAwait(false);
                                if (results.Count > 0)
                                    docs.AddRange(results.Where(doc => doc.resourceType == resourceType));
                            }
                            else
                            {
                                throw new HttpResponseException(new HttpResponseMessage(HttpStatusCode.NotFound));
                            }
                        } while (query.HasMoreResults);
                    }
                }
                if (docs.Count > 0)
                    return docs;
                var msg = new StringContent("No documents found for " + resourceType + " Resource");
                var response = new HttpResponseMessage
                {
                    StatusCode = HttpStatusCode.NotFound,
                    Content = msg
                };
                return response;
            }
        }
    }
    
```

### <a name="testing-the-fhirnotificationapi"></a>測試 FhirNotificationApi 

以下影像示範如何使用 Swagger 來測試 [FhirNotificationApi](#api-app-source)。

![用來測試 API 應用程式的 Swagger 檔案](./media/change-feed-hl7-fhir-logic-apps/hl7-fhir-testing-app.png)


### <a name="azure-portal-dashboard"></a>Azure 入口網站儀表板

下圖顯示此方案在 Azure 入口網站中執行的所有 Azure 服務。

![顯示此 HL7 FHIR 教學課程所使用之所有服務的 Azure 入口網站](./media/change-feed-hl7-fhir-logic-apps/hl7-fhir-portal.png)


## <a name="summary"></a>總結

- 您已了解 Azure Cosmos DB 原生支援對新的或修改過的文件發出通知，以及其使用方式有多麼簡單。 
- 利用 Logic Apps，您可以建立工作流程，而不需要撰寫任何程式碼。
- 使用 Azure 服務匯流排佇列來處理 HL7 FHIR 文件的散發。

## <a name="next-steps"></a>後續步驟
如需 Azure Cosmos DB 的詳細資訊，請參閱 [Azure Cosmos DB 首頁](https://azure.microsoft.com/services/cosmos-db/)。 如需 Logic Apps 的詳細資訊，請參閱 [Logic Apps](https://azure.microsoft.com/services/logic-apps/)。


