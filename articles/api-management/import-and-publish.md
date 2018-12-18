---
title: 在 Azure API 管理中匯入和發佈您的第一個 API | Microsoft Docs
description: 了解如何使用 API 管理來匯入和發佈您的第一個 API。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom: mvc
ms.topic: tutorial
ms.date: 06/15/2018
ms.author: apimpm
ms.openlocfilehash: 538977b9057a5699d61d6c2cc44209367e3550e2
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38722842"
---
# <a name="import-and-publish-your-first-api"></a>匯入和發佈您的第一個 API 

本教學課程示範如何匯入位於 http://conferenceapi.azurewebsites.net?format=json 的「OpenAPI 規格」後端 API。 這個後端 API 是由 Microsoft 所提供且裝載於 Azure 上。 

一旦將此後端 API 匯入至 API 管理 (APIM)，APIM API 就會變成適用於此後端 API 的外觀。 當您匯入此後端 API 時，來源 API 和 APIM API 會完全相同。 APIM 可讓您根據需求自訂外觀，而不需碰觸到後端 API。 如需詳細資訊，請參閱[轉換及保護您的 API](transform-api.md)。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 匯入第一個 API
> * 在 Azure 入口網站中測試 API
> * 在開發人員入口網站中測試 API

![新增 API](./media/api-management-get-started/created-api.png)

## <a name="prerequisites"></a>先決條件

完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-api"> </a>匯入和發佈後端 API

本節示範如何匯入和發佈 OpenAPI 規格後端 API。
 
1. 選取 [API 管理] 下方的 [API]。
2. 從清單中選取 [OpenAPI 規格]。

    ![建立 API](./media/api-management-get-started/create-api.png)

    您可以在建立期間或稍後前往 [設定] 索引標籤來設定 API 值。欄位旁的紅色星號表示此為必要欄位。

    請使用下表中的值建立您的第一個 API。

    |設定|值|說明|
    |---|---|---|
    |**OpenAPI 規格**|http://conferenceapi.azurewebsites.net?format=json|參考實作 API 的服務。 API 管理則將要求轉送至此位址。|
    |**顯示名稱**|*Demo Conference API*|如果您在輸入服務 URL 之後按下 Tab，APIM 將根據 json 中的內容填寫此欄位。 <br/>此名稱會顯示於開發人員入口網站中。|
    |**名稱**|*demo-conference-api*|為 API 提供唯一的名稱。 <br/>如果您在輸入服務 URL 之後按下 Tab，APIM 將根據 json 中的內容填寫此欄位。|
    |**說明**|提供 API 的選擇性描述。|如果您在輸入服務 URL 之後按下 Tab，APIM 將根據 json 中的內容填寫此欄位。|
    |**URL 配置**|*HTTPS*|決定可使用哪些通訊協定來存取 API。 |
    |**API URL 尾碼**|*conference*|這個尾碼會附加到 API 管理服務的基底 URL。 API 管理依尾碼來區分 API，因此，特定發行者的每一個 API 必須有唯一的尾碼。|
    |**產品**|*無限制*|產品是一或多個 API 的關聯。 您可以將數個 API 納入產品中，並透過開發人員入口網站將它們提供給開發人員。 <br/>透過將 API 關聯至某個產品來發佈 API (在此範例中為 [無限制])。 若要將這個新的 API 新增至產品，請輸入產品名稱 (您也可以稍後從 [設定] 頁面進行此作業)。 可以重複此步驟多次來將 API 加入多個產品。<br/>為了取得 API 的存取權，開發人員必須先訂閱產品。 當他們訂閱時，就能取得適用於該產品中任何 API 的中訂用帳戶金鑰。 <br/> 如果您建立了 APIM 執行個體，表示您已經是系統管理員，因此已訂閱每一個產品。<br/> 根據預設，每個「API 管理」執行個體都隨附兩個範例產品：**入門**與**無限制**。 |
    |要為此 API 設定版本嗎?||如需版本設定的詳細資訊，請參閱[發佈多個 API 版本](api-management-get-started-publish-versions.md)。|
    
    >[!NOTE]
    > 若要發佈 API，您必須將它與產品產生關連。 您可以從 [設定] 頁面執行此作業。
    
3. 選取 [建立] 。

## <a name="test-the-new-apim-api-in-the-azure-portal"></a>在 Azure 入口網站中測試新的 APIM API

您可以從 Azure 入口網站直接呼叫作業，以便檢視和測試 API 的作業。  
1. 選取您在上一個步驟中建立的 API (從 [API] 索引標籤)。
2. 按 [測試] 索引標籤。![測試 API](./media/api-management-get-started/test-api.png)
3. 按一下 [GetSpeakers]。
    此頁面會顯示查詢參數的欄位 (在此案例中沒有此欄位) 和標頭。 其中一個標頭是 "Ocp-Apim-Subscription-Key"，它適用於與此 API 相關聯之產品的訂用帳戶金鑰。 系統會自動填入金鑰。
4. 按 [傳送]。

    後端會回應 **200 確定**與部分資料。

## <a name="call-operation"> </a>從開發人員入口網站呼叫作業

您也可以從**開發人員入口網站**呼叫作業來測試 API。

1. 瀏覽至**開發人員入口網站**。
![開發人員入口網站](./media/api-management-get-started/developer-portal.png)

2. 選取 [APIS]，按一下 [Demo Conference API]，然後按一下 [GetSpeakers]。
    
    此頁面會顯示查詢參數的欄位 (在此案例中沒有此欄位) 和標頭。 其中一個標頭是 "Ocp-Apim-Subscription-Key"，它適用於與此 API 相關聯之產品的訂用帳戶金鑰。 如果您建立了 APIM 執行個體，您就已經是系統管理員，因此會自動填入此金鑰。
3. 按 [試試看]。
4. 按 [傳送]。
    
    叫用作業之後，開發人員入口網站就會顯示回應。  

## <a name="next-steps"> </a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 匯入第一個 API
> * 在 Azure 入口網站中測試 API
> * 在開發人員入口網站中測試 API

前進到下一個教學課程：

> [!div class="nextstepaction"]
> [建立和發佈產品](api-management-howto-add-products.md)
