---
title: 在 Azure 入口網站中建立認知服務 API 帳戶 | Microsoft Docs
description: 如何在 Azure 入口網站中建立 Microsoft 認知服務 API 帳戶。
services: cognitive-services
documentationcenter: ''
author: garyericson
manager: cgronlun
editor: ''
ms.assetid: b6176bb2-3bb6-4ebf-84d1-3598ee6e01c6
ms.service: cognitive-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/01/2018
ms.author: garye
ms.reviewer: gibattag
ms.openlocfilehash: ed5f19b23375ecb83e19274c7405e9a1208a7985
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39036147"
---
# <a name="create-a-cognitive-services-apis-account-in-the-azure-portal"></a>在 Azure 入口網站中建立認知服務 API 帳戶

若要使用 Microsoft 辨識服務 API，您必須先在 Azure 入口網站中建立帳戶。

1. 登入 [Azure 入口網站](http://portal.azure.com)。

2. 按一下 [+ 建立資源]。

3. 在 Azure Marketplace 下方，選取 [AI + 認知服務]，並探索可用 API 的清單。 按一下 [查看全部]，以查看認知服務 API 的完整清單。 按一下您選擇的 API 以繼續。

    ![選取認知服務 API](media/cognitive-services-apis-create-account/select-cognitive-services-apis.png)

4. 在 [建立] 頁面上，提供下列資訊：

   - **帳戶名稱：** 帳戶的名稱。 我們建議使用描述性的名稱，例如 *AFaceAPIAccount*。

   - **訂用帳戶：** 選取其中一個您至少具備參與者權限的可用 Azure 訂用帳戶。

   - **API 類型：** 選擇您想要使用的認知服務 API。 如需各種可用認知服務 API 的詳細資訊，請瀏覽[認知服務](https://azure.microsoft.com/services/cognitive-services/)網站。

   - **定價層：** 認知服務帳戶的費用取決於實際使用方式與您選擇的選項。 如需每個 API 定價的詳細資訊，請參閱[定價頁面](https://azure.microsoft.com/pricing/details/cognitive-services/)。

   - **資源群組：** 資源群組是共用相同生命週期、權限及原則的資源集合。 若要深入了解資源群組，請參閱[透過入口網站管理 Azure 資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-portal)。

   - **資源群組位置：** 這只有在選取的 API 為全域 (未繫結至特定位置) 的情況下才為必要。 不過，如果 API 為全域且未繫結至某個位置，您必須為資源群組指定與認知服務 API 帳戶相關聯之中繼資料所在的位置。 這個位置對於您帳戶的執行階段可用性沒有任何影響。 若要深入了解資源群組，請參閱[透過入口網站管理 Azure 資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-portal)。

   - **明確認可線上服務條款：** 若要建立帳戶，訂用帳戶擁有者或參與者 (如 [Azure 角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/overview)所定義) 需要明確地認可適用於[線上服務條款](https://www.microsoft.com/en-us/Licensing/product-licensing/products.aspx)中認知服務的條款。 

     訂用帳戶擁有者可以藉由遵循[使用 Azure 入口網站來指派和管理資源原則](../azure-policy/assign-policy-definition.md)一文，然後指派「不允許的資源類型」原則定義並指定 **Microsoft.CognitiveServices/accounts** 作為目標資源類型，透過 [Azure 原則](../azure-policy/azure-policy-introduction.md)針對特定的資源群組或訂用帳戶停用認知服務帳戶的建立。

     如果已停用帳戶建立，即會在帳戶建立期間顯示下列錯誤：

     ![帳戶建立錯誤](media/cognitive-services-apis-create-account/error-message.png)

5. 若要將帳戶釘選到 Azure 入口網站儀表板，請按一下 [釘選到儀表板]。

6. 按一下 [建立]  來建立帳戶。

成功部署認知服務帳戶之後，請按一下儀表板中的磚以檢視帳戶資訊。

您可以使用 [概觀] 區段中的 [端點 URL]，以及 [金鑰] 區段中的金鑰，來開始在應用程式中進行 API 呼叫。

![顯示帳戶資訊](media/cognitive-services-apis-create-account/display-account.png)

![顯示帳戶金鑰](media/cognitive-services-apis-create-account/account-keys.png)

### <a name="next-steps"></a>後續步驟

如需所有可用 Microsoft 認知服務的相關詳細資訊，請參閱[認知服務](https://azure.microsoft.com/services/cognitive-services/)。

如需使用部分範例認知服務 API 的快速入門指南，請參閱：

 - [電腦視覺 C# 快速入門](computer-vision/quickstarts/csharp.md)
 - [文字分析與 Python](text-analytics/quickstarts/python.md)
 - [使用 JavaScript 的臉部 API](face/quickstarts/javascript.md)
