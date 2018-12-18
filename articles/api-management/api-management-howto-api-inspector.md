---
title: 在 Azure API 管理中使用要求追蹤對 API 進行偵錯 | Microsoft Docs
description: 依照此教學課程的步驟，了解如何在 Azure API 管理中檢查要求處理步驟。
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
ms.openlocfilehash: cf9c56fa2ba75dc5b5ad4af59d111a0124f1a9df
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39057322"
---
# <a name="debug-your-apis-using-request-tracing"></a>使用要求追蹤對 API 進行偵錯

本教學課程會說明如何檢查要求處理，協助您對 API 進行偵錯和疑難排解。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 追蹤呼叫

![API 偵測器](media/api-management-howto-api-inspector/api-inspector001.PNG)

## <a name="prerequisites"></a>必要條件

+ 完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。
+ 以及完成下列教學課程：[匯入和發佈您的第一個 API](import-and-publish.md)。

## <a name="trace-a-call"></a>追蹤呼叫

1. 選取 [API]。
2. 按一下 API 清單中的 [Demo Conference API]。
3. 選取 [GetSpeakers] 作業。
4. 切換至 [測試]  索引標籤。
5. 務必包含名為 **Ocp-Apim-Trace** 且值為 **true** 的 HTTP 標頭。

    ![API 追蹤標頭](media/api-management-howto-api-inspector/api-management-tracing-header.png)

    > [!NOTE]
    > 如果 Ocp-Apim-Subscription-Key 未自動填入，您可以移至開發人員入口網站並且在設定檔頁面公開金鑰，以便擷取該金鑰。

6. 按一下 [傳送] 以進行 API 呼叫。 
7. 請等待呼叫完成。 
8. 移至 **API 主控台**中的 [追蹤] 索引標籤。 您可以按一下下列任一連結，跳至詳細的追蹤資訊：**輸入**、**後端**、**輸出**。

    在 [輸入] 區段中，您會看到 API 管理從呼叫者那邊接收的原始要求，以及所有套用到要求的原則，包括我們在步驟 2 中新增的速率限制與設定標頭原則。

    在 [後端] 區段中，您會看到 API 管理傳送至 API 後端的要求及其接收的回應。
    
    在 [輸出] 區段中，您會看到回應在傳回至呼叫者之前套用的所有原則。
 
    > [!TIP]
    > 每個步驟也會顯示 API 管理收到要求後耗用的時間。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 追蹤呼叫

前進到下一個教學課程：

> [!div class="nextstepaction"]
> [使用修訂](api-management-get-started-revise-api.md)
