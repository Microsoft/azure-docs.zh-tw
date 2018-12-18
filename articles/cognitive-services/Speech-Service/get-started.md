---
title: 免費試用語音服務
description: 探索如何免費試用語音服務。
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: erhopf
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 09/24/2018
ms.author: erhopf
ms.openlocfilehash: 7ca423a4953dbe05af119d6b6eb12976c722e374
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393264"
---
# <a name="try-the-speech-service-for-free"></a>免費試用語音服務

開始使用語音服務相當簡單且價格合理。 30 天免費試用版可讓您探索服務可以執行哪些作業，以及判斷它是否符合您的應用程式需求。

如果您需要更多時間，請登入 Microsoft Azure 帳戶，其隨附 $200 美元的服務額度，您可以將此額度應用於長達 30 天的付費語音服務訂用帳戶。

最後，語音服務會提供適用於開發應用程式的免費少量定價層。 即使在服務額度到期後，您仍可保留此免費訂用帳戶。

## <a name="free-trial"></a>免費試用

30 天免費試用可讓您在有限的時間內存取標準定價層。

若要註冊 30 天免費試用：

1. 請移至[試用認知服務](https://azure.microsoft.com/try/cognitive-services/)。

1. 選取 [語音識別 API] 索引標籤。

   ![語音服務索引標籤](media/index/try-speech-api-free-trial1.png)
   
1. 在 [語音服務] 底下，選取 [取得 API 金鑰] 按鈕。

   ![API 金鑰](media/index/try-speech-api-free-trial2.png)

1. 同意條款，並從下拉式功能表中選取您的地區設定。

   ![同意條款](media/index/try-speech-api-free-trial3.png)

1. 使用 Microsoft、Facebook、LinkedIn 或 GitHub 帳戶登入。

    您可以在 [Microsoft 帳戶入口網站](https://account.microsoft.com/account)註冊免費的 Microsoft 帳戶。 若要開始使用，請按一下 [使用 Microsoft 登入]，然後在系統要求您登入時，按一下 [建立帳戶]。 依照步驟來建立及驗證新的 Microsoft 帳戶。

登入「試用認知服務」之後，您的免費試用便開始啟用。 顯示的網頁會列出您目前有試用訂用帳戶的所有 Azure 認知服務。 **語音服務**旁邊會列出兩個訂用帳戶金鑰。 您可以在應用程式中使用任一金鑰。

> [!NOTE]
> 所有免費試用訂用帳戶都位於美國西部區域。 當您提出要求時，務必使用 `westus` 端點。

## <a name="new-azure-account"></a>新的 Azure 帳戶

新的 Azure 帳戶會收到可用 30 天的 $200 美元服務額度。 您可以使用此額度來進一步探索語音服務，或開始進行應用程式開發。

若要註冊新的 Azure 帳戶，請前往 [Azure 登入頁面](https://azure.microsoft.com/free/ai/)，按一下 [開始免費使用] 並使用您的 Microsoft 帳戶建立新 Azure 帳戶。

您可以在 [Microsoft 帳戶入口網站](https://account.microsoft.com/account)註冊免費的 Microsoft 帳戶。 若要開始使用，請按一下 [使用 Microsoft 登入]，然後在系統要求您登入時，按一下 [建立帳戶]。 依照步驟來建立及驗證新的 Microsoft 帳戶。

建立 Azure 帳戶後，請遵循下一節中的步驟，以開始訂閱語音服務。

## <a name="create-a-speech-resource-in-azure"></a>在 Azure 中建立語音資源

若要將語音服務資源 (免費或付費層) 新增至您的 Azure 帳戶：

1. 使用您的 Microsoft 帳戶，登入 [Azure 入口網站](https://portal.azure.com/)。

1. 選取入口網站左上方的 [建立資源]。

    ![建立資源](media/index/try-speech-api-create-speech1.png)

1. 在 [新增] 視窗中，搜尋 **speech**。

1. 在搜尋結果中，選取 [Speech]。

    ![選取 Speech](media/index/try-speech-api-create-speech2.png)

1. 在 [Speech] 底下選取 [建立] 按鈕。

    ![選取 [建立] 按鈕](media/index/try-speech-api-create-speech3.png)

1. 在 [建立] 底下輸入：

    * 新資源的名稱。 此名稱可協助您區分相同服務的多個訂用帳戶。
    * 選擇與新資源相關聯的 Azure 訂用帳戶來決定費用的計費方式。
    * 選擇將使用此資源的區域。 目前，語音服務適用於東亞、北歐和美國西部區域。
    * 選擇免費或付費的定價層。 如需每一層的定價和使用量配額完整資訊，請按一下 [檢視完整定價詳細資料]。
    * 為此語音訂用帳戶建立新的資源群組，或將該訂用帳戶指派給現有的資源群組。 資源群組可協助組織各種 Azure 訂用帳戶。
    * 為了方便未來存取您的訂用帳戶，請選取 [釘選到儀表板] 核取方塊。
    * 選取 [建立]。

    ![選取 [建立] 按鈕](media/index/try-speech-api-create-speech4.png)

    建立和部署新的語音資源可能需要一些時間。 選取 [快速入門] 可查看新資源的相關資訊。

    ![快速入門面板](media/index/try-speech-api-create-speech5.png)

1. 在 [快速入門] 的步驟 1 底下按一下 [金鑰] 連結，以顯示您的訂用帳戶金鑰。 每個訂用帳戶都有兩個金鑰，您可以在應用程式中使用任一個金鑰。 選取每個金鑰旁邊的按鈕，將它複製到剪貼簿，以便貼到您的程式碼中。

> [!NOTE]
> 您可以在一或多個區域中建立無限數量的標準層訂用帳戶。 不過，您可以建立只有一個免費層的訂用帳戶。 免費層上的模型部署若持續 7 天未使用，將會自動解除委任。

## <a name="switch-to-a-new-subscription"></a>切換至新的訂用帳戶

若要從一個訂用帳戶切換至另一個，例如免費試用版到期時或發佈您的應用程式時，請將程式碼中的區域和訂用帳戶金鑰取代為新 Azure 資源的區域和訂用帳戶金鑰。

> [!NOTE]
> 免費試用版金鑰會建立在美國西部 (`westus`) 區域。 透過 Azure 儀表板建立的訂用帳戶可能會位在其他某些區域中 (如果您選擇的話)。

* 如果您的應用程式使用[語音 SDK](speech-sdk.md)，您會在建立語音設定時提供區域程式碼，例如 `westus`。
* 如果您的應用程式使用其中一個語音服務 [REST API](rest-apis.md)，則區域會是您提出要求時所用端點 URI 的一部份。

為某區域建立的金鑰，就只能在該區域中使用。 若嘗試在其他區域使用，將會導致驗證錯誤。

## <a name="next-steps"></a>後續步驟

完成個我們的任何一個 10 分鐘快速入門，或查看我們的 SDK 範例：

> [!div class="nextstepaction"]
> [快速入門：以 C# 辨識語音](quickstart-csharp-dotnet-windows.md)
> [語音 SDK 範例](speech-sdk.md#get-the-samples)
