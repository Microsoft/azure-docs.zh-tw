---
title: 使用 Azure 入口網站手動新增 API | Microsoft Docs
description: 本教學課程示範如何使用 API 管理 (APIM) 來手動新增 API。
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/27/2018
ms.author: apimpm
ms.openlocfilehash: 35b4777c7de4db1f8514b24e7b1e4d11775d0ca0
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43247897"
---
# <a name="add-an-api-manually"></a>手動新增 API

本文中的步驟會示範如何使用 Azure 入口網站，來將 API 手動新增至 API 管理 (APIM) 執行個體。 常見的情況是，您會想在模擬 API 時建立空白的 API 並手動進行定義。 如需有關模擬 API 的詳細資訊，請參閱[模擬 API 回應](mock-api-responses.md)。

如果您想要匯入現有的 API，請參閱[相關主題](#related-topics)一節。

在本文中，我們會建立空白的 API，並指定 [httpbin.org](http://httpbin.org) (公用測試服務) 作為後端 API。

## <a name="prerequisites"></a>必要條件

完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-an-api"></a>建立 API

1. 選取 [API 管理] 下方的 [API]。
2. 從左側功能表中選取 [+ 新增 API]。
3. 從清單選取 [空白 API]。

    ![空白 API](media/add-api-manually/blank-api.png)
4. 輸入 API 的設定。

    ![設定](media/add-api-manually/settings.png)

    |**名稱**|**值**|**說明**|
    |---|---|---|
    |**顯示名稱**|「*空白 API*」 |此名稱會顯示於開發人員入口網站中。|
    |**Web 服務 URL** (選擇性)| 「 http://httpbin.org」| 如果您想要模擬 API，您可以不輸入任何項目。 <br/>在此案例中，我們會輸入 [http://httpbin.org](http://httpbin.org)。這是公用的測試服務。 <br/>如果您想要匯入會自動對應至後端的 API，請參閱[相關主題](#related-topics)一節中的其中一個主題。|
    |**URL 配置**|"*HTTPS*"|在此案例中，即使後端有不安全的 HTTP 存取，我們還是會指定安全的 HTTPS APIM 存取給後端。 <br/>這種狀況 (HTTPS 至 HTTP) 稱為 HTTPS 終止。 如果您的 API 存在於虛擬網路中 (而您知道存取是安全的，即使不使用 HTTPS)，您就可以這麼做。 <br/>您可以使用「HTTPS 終止」來減少一些 CPU 循環。|
    |**URL 尾碼**|"*hbin*"| 此尾碼是用來在這個 APIM 執行個體中識別此特定 API 的名稱。 它在這個 APIM 執行個體中必須是唯一的。|
    |**產品**|"*無限制*" |透過將 API 關聯至某個產品來發佈 API。 如果您想要發佈 API 以供開發人員使用，請將它新增至產品。 您可以在 API 建立期間執行此動作，或稍後設定它。<br/><br/>產品是一或多個 API 的關聯。 您可以包括數個 API，並透過開發人員入口網站將它們提供給開發人員。 <br/>開發人員必須先訂閱產品，才能取得 API 的存取權。 當他們訂閱時，就能取得適用於該產品中任何 API 的中訂用帳戶金鑰。 如果您建立了 APIM 執行個體，您就已經是系統管理員，因此根據預設，您已訂閱每一個產品。<br/><br/> 根據預設，每個「API 管理」執行個體都隨附兩個範例產品：**入門**與**無限制**。| 
5. 選取 [建立] 。

此時，APIM 中沒有對應至後端 API 中任何作業的作業。 如果您呼叫透過後端公開的作業，而不是呼叫透過 APIM 公開的作業，您會收到 **404**。

>[!NOTE] 
> 依預設，當您新增 API 時 (即使 API 已連線到某些後端服務)，APIM 不會公開任何作業，直到您將作業加入白名單。 若要將您的後端服務作業加入白名單，請建立對應至後端作業的 APIM 作業。

## <a name="add-and-test-an-operation"></a>新增和測試作業

本節會示範如何新增 "/get" 作業，以便將其對應至後端 "http://httpbin.org/get" 作業。

### <a name="add-an-operation"></a>新增作業

1. 選取您在上一個步驟中建立的 API。
2. 按一下 [+ 新增作業]。
3. 在 **URL** 中選取 **GET**，並在資源中輸入 "*/get*"。
4. 輸入 "*FetchData* 作為**顯示名稱**。
5. 選取 [ **儲存**]。

### <a name="test-an-operation"></a>測試作業

在 Azure 入口網站中測試作業。 或者，您可以在**開發人員入口網站**中測試作業。

1. 選取 [測試] 索引標籤。
2. 選取 **FetchData**。
3. 按 [傳送]。

此時會出現 "http://httpbin.org/get" 作業產生的回應。 如果您想要轉換您的作業，請參閱[轉換及保護您的 API](transform-api.md)。

## <a name="add-and-test-a-parameterized-operation"></a>新增和測試參數化作業

本節會示範如何新增採用參數的作業。 在此案例中，我們將作業對應至 "http://httpbin.org/status/200"。

### <a name="add-the-operation"></a>新增作業

1. 選取您在上一個步驟中建立的 API。
2. 按一下 [+ 新增作業]。
3. 在 **URL** 中選取 **GET**，並在資源中輸入"*/status/{code}*"。 您可以選擇性地提供與此參數相關聯的一些資訊。 例如，在 [類型] 中輸入「數字」，在 [值] 中輸入「200」(預設)。
4. 輸入 "GetStatus" 作為**顯示名稱**。
5. 選取 [ **儲存**]。

### <a name="test-the-operation"></a>測試作業 

在 Azure 入口網站中測試作業。  或者，您可以在**開發人員入口網站**中測試作業。

1. 選取 [測試] 索引標籤。
2. 選取 **GetStatus**。 依預設，此字碼值會設定為 "200"。 您可以加以變更以測試其他值。 例如，輸入 "418"。
3. 按 [傳送]。

    此時會出現 "http://httpbin.org/status/200" 作業產生的回應。 如果您想要轉換您的作業，請參閱[轉換及保護您的 API](transform-api.md)。

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [轉換及保護已發佈的 API](transform-api.md)
