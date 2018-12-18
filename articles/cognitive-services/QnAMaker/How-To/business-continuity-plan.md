---
title: 商務持續性方案 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 商務持續性方案的主要目標是建立彈性的知識庫端點，藉以確保聊天機器人或應用程式使用知識庫時不會停機。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 41e7425a2e2e6dd8dc8416538cf77e5b8f273284
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041933"
---
# <a name="create-a-business-continuity-plan-for-your-qna-maker-service"></a>建立 QnA Maker 服務的商務持續性方案

商務持續性方案的主要目標是建立彈性的知識庫端點，藉以確保聊天機器人或應用程式使用知識庫時不會停機。

![QnA Maker bcp 方案](../media/qnamaker-how-to-bcp-plan/qnamaker-bcp-plan.png)

以上呈現的高階構想如下所示：

1. 在 [Azure 配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)中設定兩個平行的 [QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)。

2. 使主要和次要的 Azure 搜尋索引保持同步。使用[這裡](https://github.com/pchoudhari/QnAMakerBackupRestore)的 github 範例了解如何備份還原 Azure 索引。

3. 使用[連續匯出](https://docs.microsoft.com/azure/application-insights/app-insights-export-telemetry)備份 Application Insights。

4. 設定主要和次要堆疊後，使用[流量管理員](https://docs.microsoft.com/azure/traffic-manager/)設定兩個端點，並設定路由方法。

5. 您必須建立流量管理員端點的 SSL 憑證。 在您的應用程式服務中[繫結 SSL 憑證](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)。

6. 最後，在聊天機器人或應用程式中使用流量管理員端點。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [選擇 QnA Maker 部署的容量](../Tutorials/choosing-capacity-qnamaker-deployment.md)
