---
title: 使用 REST API 將 Azure 資源的自訂計量傳送至 Azure 監視器計量的存放區
description: 使用 REST API 將 Azure 資源的自訂計量傳送至 Azure 監視器計量的存放區
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: howto
ms.date: 09/24/2018
ms.author: ancav
ms.component: metrics
ms.openlocfilehash: c01440437eae3cb076627ab9f2221e33b833f472
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977226"
---
# <a name="send-custom-metrics-for-an-azure-resource-to-the-azure-monitor-metric-store-using-a-rest-api"></a>使用 REST API 將 Azure 資源的自訂計量傳送至 Azure 監視器計量的存放區

本文將說明如何透過 REST API 將 Azure 資源的自訂計量傳送至 Azure 監視器計量存放區。  當計量位於 Azure 監視器之後，您可以執行使用標準計量執行的所有作業，例如圖表、警示、將它們路由傳送至其他外部工具等。  

>[!NOTE] 
>REST API 僅允許傳送 Azure 資源的自訂計量。 若要傳送不同環境或內部部署的資源的計量，您可以使用 [Application Insights](../application-insights/app-insights-api-custom-events-metrics.md)。    


## <a name="create-and-authorize-a-service-principal-to-emit-metrics"></a>建立及授權服務主體以發出計量 

使用[建立服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)中找到的指示，在您的 Azure Active Directory 租用戶中建立服務主體。 

進行此流程時請注意下列事項： 

- 您可以針對登入 URL 填入任何 URL。  
- 為此應用程式建立新用戶端密碼  
- 儲存金鑰和用戶端識別碼以便在稍後步驟中使用。  

將您希望發出計量的資源的權限授予作為步驟 1「監視計量發行者」一部分所建立的應用程式。 如果您計劃使用應用程式針對許多資源發出自訂計量，您可以在資源群組或訂用帳戶層級上授與這些權限。 

## <a name="get-an-authorization-token"></a>取得授權權杖
開啟命令提示字元並執行下列命令
```shell
curl -X POST https://login.microsoftonline.com/<yourtenantid>/oauth2/token -F "grant_type=client_credentials" -F "client_id=<insert clientId from earlier step> " -F "client_secret=<insert client secret from earlier step>" -F "resource=https://monitoring.azure.com/"
```
儲存回應中的存取權杖

![存取權杖](./media/metrics-store-custom-rest-api/accesstoken.png)

## <a name="emit-the-metric-via-the-rest-api"></a>透過 REST API 發出計量 

1. 將下列 JSON 貼到檔案中，並在您的本機電腦上將它儲存為 custommetric.json。 更新 JSON 檔案中的時間參數。 
    
    ```json
    { 
        "time": "2018-09-13T16:34:20”, 
        "data": { 
            "baseData": { 
                "metric": "QueueDepth", 
                "namespace": "QueueProcessing", 
                "dimNames": [ 
                  "QueueName", 
                  "MessageType" 
                ], 
                "series": [ 
                  { 
                    "dimValues": [ 
                      "ImagesToProcess", 
                      "JPEG" 
                    ], 
                    "min": 3, 
                    "max": 20, 
                    "sum": 28, 
                    "count": 3 
                  } 
                ] 
            } 
        } 
    } 
    ``` 

1. 在命令提示字元視窗中，發佈計量資料 
    - Azure 區域 – 必須符合您要發出其計量的資源的部署資源。 
    - ResourceID – 您要針對其追蹤計量的 Azure 資源的資源識別碼。  
    - 存取權杖 – 貼上稍早取得的權杖

    ```Shell 
    curl -X POST curl -X POST https://<azureRegion>.monitoring.azure.com/<resourceId> /metrics -H "Content-Type: application/json" -H "Authorization: Bearer <AccessToken>" -d @custommetric.json 
    ```
1. 變更 JSON 檔案中的時間戳記和值。 
1. 重複前兩個步驟執行數次，讓您擁有數分鐘的資料。

## <a name="troubleshooting"></a>疑難排解 
如果您在流程的某些部分收到錯誤，請考量下列事項

1. 您不能發出訂用帳戶或資源群組的計量作為您的 Azure 資源。 
1. 您不能將計量放入超過 20 分鐘的存放區。 計量存放區已針對警示與即時圖表而最佳化。 
2. 維度名稱的數目應該符合值，反之亦然。 檢查值。 
2. 您所發出的計量，其區域不支援自訂計量。 請參閱[支援的自訂計量 (預覽) 區域](metrics-custom-overview.md#supported-regions) 



## <a name="view-your-metrics"></a>檢視您的計量 

1. 登入 Azure 入口網站 

1. 在左側功能表中，按一下 [監視] 

1. 在 [監視] 頁面上，按一下 [計量]。 

   ![存取權杖](./media/metrics-store-custom-rest-api/metrics.png) 

1. 將彙總期間變更為 [過去 30 分鐘]。  

1. 在 [資源] 下拉式清單中，選取您發出其計量的資源。  

1. 在 [命名空間] 下拉式清單中，選取 [QueueProcessing] 

1. 在 [計量] 下拉式清單中，選取 [QueueDepth]。  

 
## <a name="next-steps"></a>後續步驟
- 深入了解[自訂計量](metrics-custom-overview.md)。