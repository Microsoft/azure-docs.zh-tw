---
title: 設定 QnA Maker 服務 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 您必須先在 Azure 中設定 QnA Maker 服務，才能建立任何 QnA Maker 知識庫。 任何具備授權而可在訂用帳戶中建立新資源的人，皆可建立 QnA Maker 服務。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 103da0c65bcf9b9bd24130082f844b9bd12a9a02
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389712"
---
# <a name="create-a-qna-maker-service"></a>建立 QnA Maker 服務

您必須先在 Azure 中設定 QnA Maker 服務，才能建立任何 QnA Maker 知識庫。 任何具備授權而可在訂用帳戶中建立新資源的人，皆可建立 QnA Maker 服務。

此安裝程式會部署一些 Azure 資源。 這些資源會一同管理知識庫內容，並透過端點提供問答功能。

1. 登入 [Azure 入口網站](<https://portal.azure.com>)。

2.  按一下 [加入新資源]，並且在搜尋中鍵入「qna maker」，然後選取 QnA Maker 資源

    ![建立新的 QnA Maker 服務](../media/qnamaker-how-to-setup-service/create-new-resource.png)

3.  閱讀條款與條件之後，按一下 [建立]。

    ![建立新的 QnA Maker 服務](../media/qnamaker-how-to-setup-service/create-new-resource-button.png)

4. 在 **QnA Maker** 中，選取適當的層級和地區。

    ![建立新的 QnA Maker 服務](../media/qnamaker-how-to-setup-service/enter-qnamaker-info.png)

    * 在 [名稱] 中填入識別此 QnA Maker 服務的唯一名稱。 此名稱也會識別將與知識庫產生關聯的 QnA Maker 端點。
    * 選擇將部署 QnA Maker 資源的**訂用帳戶**。
    * 選取 QnA Maker 管理服務 (入口網站和管理 API) 的 [管理定價層]。 關於 SKU 價格的詳細資料，請參閱[這裡](https://aka.ms/qnamaker-pricing)。
    * 建立新的**資源群組** (建議選項)，或使用部署此 QnA Maker 資源的現有資源群組。
    * 選擇 Azure 搜尋服務的 [定價層搜尋]。 如果您看到免費層選項呈現灰色，表示您已經有免費的 Azure 搜尋層部署於您的訂用帳戶中。 在此情況下，您必須先開始使用基本 Azure 搜尋服務層。 關於 Azure 搜尋服務定價的詳細資料，請參閱[這裡](https://azure.microsoft.com/pricing/details/search/)。
    * 選擇您要部署 Azure 搜尋服務資料的**搜尋位置**。 必須儲存客戶資料的限制會通知您對於 Azure 搜尋服務選擇的位置。
    * 在 [應用程式名稱] 中，設定應用程式服務的名稱。
    * 應用程式服務預設為標準 (S1) 層。 您可以在建立之後變更方案。 如需應用程式服務定價的詳細資訊，請參閱[這裡](https://azure.microsoft.com/pricing/details/app-service/)。
    * 選擇將部署應用程式服務的**網站位置**。

        > [!NOTE]
        > 搜尋位置可能與網站位置不同。

    * 選擇您是否要啟用 **Application Insights**。 如果啟用 **Application Insights**，QnA Maker 會收集流量、交談記錄和錯誤的遙測資料。
    * 選擇將部署 Application Insights 資源的**App insights 位置**。

5. 所有的欄位均完成驗證後，您可以按一下 [建立] 開始在您的訂用帳戶中部署這些服務。 這需要幾分鐘才能完成。

6.  完成部署後，您會看到在您的訂用帳戶中已建立下列資源。

    ![建立新的 QnA Maker 服務](../media/qnamaker-how-to-setup-service/resources-created.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [建立和發佈知識庫](../Quickstarts/create-publish-knowledge-base.md)