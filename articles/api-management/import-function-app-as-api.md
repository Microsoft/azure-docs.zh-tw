---
title: 在 Azure API 管理中匯入 Azure 函式應用程式作為 API | Microsoft Docs
description: 此教學課程說明如何將 Azure 函式應用程式匯入至 Azure API 管理作為 API。
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
ms.date: 08/28/2018
ms.author: apimpm
ms.openlocfilehash: b5a38b9f16224c8b2b128b464106fe83d018b308
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46129755"
---
# <a name="import-an-azure-function-app-as-an-api-in-azure-api-management"></a>在 Azure API 管理中匯入 Azure 函式應用程式作為 API

Azure API 管理支援將 Azure 函式應用程式匯入為新的 API，或將其附加至現有的 API。 此程序會在 Azure 函式應用程式中自動產生主機金鑰，而此金鑰接著會指派給 Azure API 管理中的具名值。

此文章將逐步說明如何在 Azure API 管理中匯入 Azure 函式應用程式作為 API。 此外也會說明其測試程序。

您將了解如何：

> [!div class="checklist"]
> * 匯入 Azure 函式應用程式作為 API
> * 將 Azure 函式應用程式附加至 API
> * 檢視新的 Azure 函式應用程式主機金鑰與 Azure API 管理的具名值
> * 在 Azure 入口網站中測試 API
> * 在開發人員入口網站中測試 API

## <a name="prerequisites"></a>先決條件

* 完成[建立 Azure API 管理執行個體](get-started-create-service-instance.md)快速入門。
* 確定您的訂用帳戶中有 Azure Functions 應用程式。 如需詳細資訊，請參閱[建立 Azure 函式應用程式](../azure-functions/functions-create-first-azure-function.md#create-a-function-app)。 其中必須包含具有 HTTP 觸發程序的函式，且授權層級設定必須設為 [匿名] 或 [函式]。

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="add-new-api-from-azure-function-app"></a>匯入 Azure 函式應用程式作為新的 API

請依照下列步驟從 Azure 函式應用程式建立新的 API。

1. 在您的 **Azure API 管理**服務執行個體中，從左側的功能表中選取 [API]。

2. 在 [新增 API] 清單中，選取 [函式應用程式]。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-01.png)

3. 按一下 [瀏覽] 以選取要匯入的函式。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-02.png)

4. 按一下 [函式應用程式] 區段，從可用的函式應用程式清單中選擇。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-03.png)

5. 找出您要從中匯入函式的函式應用程式並按一下，然後按 [選取]。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-04.png)

6. 選取您要匯入的函式，然後按一下 [選取]。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-05.png)

    > [!NOTE]
    > 您只能匯入設有 HTTP 觸發程序、且授權層級設定設為 [匿名] 或 [函式] 的函式。

7. 切換至 [完整] 檢視，然後將 [產品] 指派給您的新 API。 視需要編輯其他預先填入的欄位。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-06.png)

8. 按一下頁面底部的 [新增] 。

## <a name="append-azure-function-app-to-api"></a>將 Azure 函式應用程式附加至現有的 API

請依照下列步驟，將 Azure 函式應用程式附加至現有的 API。

1. 在您的 **Azure API 管理**服務執行個體中，從左側的功能表中選取 [API]。

2. 選擇要在其中匯入 Azure 函式應用程式的 API。 按一下 **...**，然後從內容功能表中選取 [匯入]。

    ![從函式應用程式附加](./media/import-function-app-as-api/append-01.png)

3. 按一下 [函式應用程式] 圖格。

    ![從函式應用程式附加](./media/import-function-app-as-api/append-02.png)

4. 在快顯視窗中，按一下 [瀏覽]。

    ![從函式應用程式附加](./media/import-function-app-as-api/append-03.png)

5. 按一下 [函式應用程式] 區段，從可用的函式應用程式清單中選擇。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-03.png)

6. 找出您要從中匯入函式的函式應用程式並按一下，然後按 [選取]。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-04.png)

7. 選取您要匯入的函式，然後按一下 [選取]。

    ![從函式應用程式新增](./media/import-function-app-as-api/add-05.png)

8. 按一下 [匯入] 。

    ![從函式應用程式附加](./media/import-function-app-as-api/append-04.png)

## <a name="function-app-import-keys"></a>產生的 Azure 函式應用程式主機金鑰

匯入 Azure 函式應用程式後會自動產生：
* 位於函式應用程式內、名為 apim-{*您的 Azure API 管理服務執行個體名稱*} 的主機金鑰、
* 位於 Azure API 管理執行個體內、名稱為 {*您的 Azure 函式應用程式執行個體名稱*}-key 的具名值，其中包含建立的主機金鑰。

> [!WARNING]
> 移除或變更 Azure 函式應用程式主機金鑰的值或 Azure API 管理具名值，將會中斷服務之間的通訊。 這些值不會自動同步。
>
> 如果需要輪替主機金鑰，請確定 Azure API 管理中的具名值也須一併修改。

### <a name="access-azure-function-app-host-key"></a>存取 Azure 函式應用程式主機金鑰

1. 瀏覽至 Azure 函式應用程式執行個體。

2. 從概觀中選取 [函式應用程式設定]。

    ![從函式應用程式新增](./media/import-function-app-as-api/keys-02-a.png)

3. 金鑰位於 [主機金鑰] 區段中。

    ![從函式應用程式新增](./media/import-function-app-as-api/keys-02-b.png)

### <a name="access-the-named-value-in-azure-api-management"></a>存取 Azure API 管理中的具名值

瀏覽至您的 Azure API 管理執行個體，然後從左側的功能表中選取 [具名值]。 Azure 函式應用程式金鑰儲存於此處。

![從函式應用程式新增](./media/import-function-app-as-api/keys-01.png)

## <a name="test-in-azure-portal"></a>在 Azure 入口網站中測試新的 API 管理 API

您可以直接從 Azure 入口網站呼叫作業。 使用 Azure 入口網站可方便您檢視和測試 API 的作業。  

1. 選取您在先前的小節中建立的 API。

2. 選取 [測試] 索引標籤。

3. 選取作業。

    頁面會顯示查詢參數的欄位和標頭的欄位。 其中一個標頭是 **Ocp-Apim-Subscription-Key**，它適用於此 API 相關產品的訂用帳戶金鑰。 如果您建立了 API 管理執行個體，您就已經是系統管理員，因此會自動填入此金鑰。 

4. 選取 [傳送]。

    後端會回應 **200 確定**與部分資料。

## <a name="test-in-developer-portal"></a>從開發人員入口網站呼叫作業

您也可以從開發人員入口網站呼叫作業，以測試 API。 

1. 選取您在[匯入和發佈後端 API](#create-api) 中建立的 API。

2. 選取 [開發人員入口網站]。

    開發人員入口網站隨即開啟。

3. 選取您所建立的 **API**。

4. 選取您要測試的作業。

5. 選取 [試試看]。

6. 選取 [傳送]。
    
    叫用作業之後，開發人員入口網站會顯示 [回應狀態]、[回應標頭]，以及任何的 [回應內容]。

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [轉換及保護已發佈的 API](transform-api.md)