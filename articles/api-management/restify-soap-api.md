---
title: 使用 Azure 入口網站匯入 SOAP API 並轉換為 REST | Microsoft Docs
description: 了解如何使用 API 管理匯入 SOAP API 並轉換為 REST。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 11/22/2017
ms.author: apimpm
ms.openlocfilehash: 940756917c8f377e7d134818409e6287a4031e15
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38239803"
---
# <a name="import-a-soap-api-and-convert-to-rest"></a>匯入 SOAP API 並轉換為 REST

本文會示範如何匯入 SOAP API 並轉換成 REST。 本文也會示範如何測試 APIM API。

在本文中，您將了解：

> [!div class="checklist"]
> * 匯入 SOAP API 並轉換為 REST
> * 在 Azure 入口網站中測試 API
> * 在開發人員入口網站中測試 API

## <a name="prerequisites"></a>先決條件

完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-api"> </a>匯入和發佈後端 API

1. 選取 [API 管理] 下方的 [API]。
2. 從 [加入新的 API] 清單選取 [WSDL]。

    ![SOAP API](./media/restify-soap-api/wsdl-api.png)
3. 在 **WSDL 規格**中，輸入 SOAP API 的所在 URL。
4. 按一下 [SOAP 至 REST] 選項按鈕。 按一下此選項時，APIM 會嘗試在 XML 和 JSON 之間進行自動轉換。 在此案例中，取用者應呼叫 API 作為 Restful API，其會傳回 JSON。 APIM 會將每個要求轉換為 SOAP 呼叫。

    ![SOAP 至 REST](./media/restify-soap-api/soap-to-rest.png)

5. 按 tab 鍵。

    下列欄位會填入來自 SOAP API 的資訊：顯示名稱、名稱、描述。
6. 新增 API URL 尾碼。 此尾碼是用來在這個 APIM 執行個體中識別此特定 API 的名稱。 它在這個 APIM 執行個體中必須是唯一的。
9. 透過將 API 關聯至某個產品來發佈 API。 在本案例中，我們使用「無限制」的產品。  如果您想要發佈 API 以供開發人員使用，請將它新增至產品。 您可以在 API 建立期間執行此動作，或稍後設定它。

    產品是一或多個 API 的關聯。 您可以包括數個 API，並透過開發人員入口網站將它們提供給開發人員。 開發人員必須先訂閱產品，才能取得 API 的存取權。 當他們訂閱時，就能取得適用於該產品中任何 API 的中訂用帳戶金鑰。 如果您建立了 APIM 執行個體，您就已經是系統管理員，因此根據預設，您已訂閱每一個產品。

    依預設，每個 API 管理執行個體會隨附兩個範例產品：

    * **入門**
    * **無限制**   
10. 選取 [建立] 。

## <a name="test-the-new-apim-api-in-the-azure-portal"></a>在 Azure 入口網站中測試新的 APIM API

您可以從 Azure 入口網站直接呼叫作業，以便檢視和測試 API 的作業。  

1. 選取您在上一個步驟中建立的 API。
2. 按 [測試] 索引標籤。
3. 選取某個作業。

    頁面會顯示查詢參數的欄位和標頭的欄位。 其中一個標頭是 "Ocp-Apim-Subscription-Key"，它適用於與此 API 相關聯之產品的訂用帳戶金鑰。 如果您建立了 APIM 執行個體，您就已經是系統管理員，因此會自動填入此金鑰。 
1. 按 [傳送]。

    後端會回應 **200 確定**與部分資料。

## <a name="call-operation"> </a>從開發人員入口網站呼叫作業

您也可以在**開發人員入口網站**呼叫作業來測試 API。 

1. 選取您在「匯入和發佈後端 API」步驟中建立的 API。
2. 按 [開發人員入口網站]。

    「開發人員入口網站」隨即開啟。
3. 選取您所建立的 **API**。
4. 按一下您想要測試的作業。
5. 按 [試試看]。
6. 按 [傳送]。
    
    叫用作業之後，開發人員入口網站會顯示 [回應狀態]、[回應標頭]，以及任何的 [回應內容]。

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [轉換及保護已發佈的 API](transform-api.md)