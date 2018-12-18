---
title: 部署的資源容量 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 為您的 QnA Maker 部署選擇容量的指南
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 582ace641cadbc7ad3a622def07f70ed51ccac53
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389797"
---
# <a name="choosing-capacity-for-your-qna-maker-deployment"></a>選擇 QnA Maker 部署的容量

QnA Maker 服務會採用對三項 Azure 資源的相依性：
1.  App Service (用於執行階段)
2.  Azure 搜尋服務 (用來儲存 QnA)
3.  App Insights (選擇性，用來儲存交談記錄和遙測資料)

在建立 QnA Maker 服務之前，您應決定您適合使用上述服務的哪幾層。 

一般而言，您有三個參數必須考量：
1. **您所需的服務輸送量**：根據您的需求為您的應用程式服務選取適當的[應用程式方案](https://azure.microsoft.com/pricing/details/app-service/plans/)。 您可以[相應增加](https://docs.microsoft.com/azure/app-service/web-sites-scale)或相應減少應用程式。 這應該也會影響您的 Azure 搜尋服務 SKU 選取，請參閱[這裡](https://docs.microsoft.com/azure/search/search-sku-tier)以取得更多詳細資訊。

2. **知識庫的大小和數量**：針對您的案例選擇適當的 [Azure 搜尋服務 SKU](https://azure.microsoft.com/pricing/details/search/)。 您可以在特定層中發佈 N-1 個知識庫，其中，N 是該層中允許的索引數上限。 同時也請檢查每個層允許的大小上限和文件數目。

3. **作為來源的文件數目**：QnA Maker 管理服務的免費 SKU 會將您可以透過入口網站和 API 管理的文件數目限制為 3 (每份文件的大小為 1 MB)。 標準 SKU 則不會限制您可管理的文件數目。 如需詳細資訊，請參閱[這裡](https://aka.ms/qnamaker-pricing)。

下表為您提供某些高階指導方針。

|                        | QnA Maker 管理 | App Service 方案 | Azure 搜尋服務 | 限制                      |
| ---------------------- | -------------------- | ----------- | ------------ | -------------------------------- |
| 測試        | 免費 SKU             | 免費層   | 免費層    | 最多發佈 2 個 KB，大小為 50 MB  |
| 開發/測試環境   | 標準 SKU         | 共用      | 基本        | 最多發佈 14 KB，大小為 2 GB    |
| 生產環境 | 標準 SKU         | 基本       | 標準     | 最多發佈 49 個 KB，大小為 25 GB |

如要升級 QnA Maker 堆疊，請參閱[升級 QnA Maker 服務](../How-To/upgrade-qnamaker-service.md)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [升級 QnA Maker 服務](../How-To/upgrade-qnamaker-service.md)
