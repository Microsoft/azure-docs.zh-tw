---
title: 將 Azure API 管理服務部署到多個 Azure 區域 | Microsoft Docs
description: 瞭解如何將 Azure API 管理服務執行個體部署到多個 Azure 區域。
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: apimpm
ms.openlocfilehash: 2ec8d53b0d8da3a7d643362abf58d3a5d4b42e74
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42145547"
---
# <a name="how-to-deploy-an-azure-api-management-service-instance-to-multiple-azure-regions"></a>如何將 Azure API 管理服務執行個體部署到多個 Azure 區域

Azure API 管理支援多區域部署，可讓 API 發行者跨所需的任意數量 Azure 區域發佈單一 Azure API 管理服務。 這有助於降低地理上分散的 API 取用者感受到的要求延遲，並且可以改善某個區域離線時服務的可用性。

新的 Azure API 管理服務一開始只會在單一 Azure 區域 (主要區域) 中包含一個[單位][unit]。 您可以透過 Azure 入口網站輕鬆新增其他區域。 「API 管理」閘道伺服器會部署到每個區域，而呼叫流量則會路由傳送到距離最近的閘道。 如果區域離線，流量會自動重新導向到下一個最近的閘道。

> [!IMPORTANT]
> 多重區域部署僅供**[進階][Premium]** 層使用。

> [!NOTE]
> Azure API 管理只可跨區域複寫 API 閘道元件。 服務管理元件只裝載在主要區域中。 如果主要區域發生中斷狀況，則不可能將組態變更套用至 Azure API 管理服務執行個體 (包括設定或原則更新)。

## <a name="add-region"> </a>部署 API 管理服務執行個體到新區域

> [!NOTE]
> 如果您尚未建立 API 管理服務執行個體，請參閱[建立 API 管理服務執行個體][Create an API Management service instance]。

在 Azure 入口網站中，瀏覽至「API 管理」服務執行個體的 [級別與價格] 頁面。 

![調整索引標籤][api-management-scale-service]

若要部署新區域，請從工具列中按一下 [+加入區域]。

![加入區域][api-management-add-region]

從下拉式清單中選取位置，然後使用滑桿設定單位數量。

![指定單位][api-management-select-location-units]

按一下 [加入] 以將您的選項放入 [位置] 表格中。 

重複此程序，直到設定好所有位置為止，然後從工具列中按一下 [儲存] 來開始部署程序。

## <a name="remove-region"> </a>從區域中刪除 API 管理服務執行個體

在 Azure 入口網站中，瀏覽至「API 管理」服務執行個體的 [級別與價格] 頁面。 

![調整索引標籤][api-management-scale-service]

針對您想要移除的位置，使用表格最右邊的 [...] 按鈕來開啟操作功能表。 選取 [刪除] 選項。

確認刪除，然後按一下 [儲存] 來套用變更。

## <a name="route-backend"> </a>將 API 呼叫路由至區域後端服務

Azure API 管理只有一個後端服務 URL。 即使不同區域中有多個 Azure API 管理執行個體，API 閘道仍會將要求轉送到只部署在單一區域中的相同後端服務。 在此案例中，效能的提升只在於將回應快取到要求專屬區域中的 Azure API 管理內，但在連絡全球各地的後端時，可能仍會造成高延遲。

若要完全利用您系統的地理分佈，您應該將後端服務部署在 Azure API 管理執行個體所在的區域中。 然後，藉由使用原則和 `@(context.Deployment.Region)` 屬性，您可以將流量路由至您後端的本機執行個體。

1. 瀏覽至您的 Azure API 管理執行個體，然後按一下左側功能表中的 [API]。
2. 選取想要的 API。
3. 從 [輸入處理] 中的箭號下拉式清單中，按一下 [程式碼編輯器]。

    ![API 程式碼編輯器](./media/api-management-howto-deploy-multi-region/api-management-api-code-editor.png)

4. 使用 `set-backend` 結合條件式 `choose` 原則，在檔案的 `<inbound> </inbound>` 區段中建構適當的路由原則。

    例如，下列 XML 檔案將用於美國西部和東亞區域：

    ```xml
    <policies>
        <inbound>
            <base />
            <choose>
                <when condition="@("West US".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                    <set-backend-service base-url="http://contoso-us.com/" />
                </when>
                <when condition="@("East Asia".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                    <set-backend-service base-url="http://contoso-asia.com/" />
                </when>
                <otherwise>
                    <set-backend-service base-url="http://contoso-other.com/" />
                </otherwise>
            </choose>
        </inbound>
        <backend>
            <base />
        </backend>
        <outbound>
            <base />
        </outbound>
        <on-error>
            <base />
        </on-error>
    </policies>
    ```

[api-management-management-console]: ./media/api-management-howto-deploy-multi-region/api-management-management-console.png

[api-management-scale-service]: ./media/api-management-howto-deploy-multi-region/api-management-scale-service.png
[api-management-add-region]: ./media/api-management-howto-deploy-multi-region/api-management-add-region.png
[api-management-select-location-units]: ./media/api-management-howto-deploy-multi-region/api-management-select-location-units.png
[api-management-remove-region]: ./media/api-management-howto-deploy-multi-region/api-management-remove-region.png

[Create an API Management service instance]: get-started-create-service-instance.md
[Get started with Azure API Management]: get-started-create-service-instance.md

[Deploy an API Management service instance to a new region]: #add-region
[Delete an API Management service instance from a region]: #remove-region

[unit]: http://azure.microsoft.com/pricing/details/api-management/
[Premium]: http://azure.microsoft.com/pricing/details/api-management/
