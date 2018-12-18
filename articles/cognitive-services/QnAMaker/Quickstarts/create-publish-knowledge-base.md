---
title: 快速入門：建立 KB - QnA Maker
titleSuffix: Azure Cognitive Services
description: 您可以從自己的內容 (例如常見問題集或產品手冊) 建立 QnA Maker 知識庫 (KB)。 此範例中的 QnA Maker KB 是從簡單的常見問題集網頁建立而來，以回答 BitLocker 金鑰復原問題。
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: quickstart
ms.date: 09/12/2018
ms.author: diberry
ms.openlocfilehash: 99f9259c9fa73a9ba4842b2684cde8a44f73145a
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389661"
---
# <a name="quickstart-create-train-and-publish-your-knowledge-base"></a>快速入門：建立、訓練及發佈知識庫

您可以從自己的內容 (例如常見問題集或產品手冊) 建立 QnA Maker 知識庫 (KB)。 此範例中的 QnA Maker KB 是從簡單的常見問題集網頁建立而來，以回答 BitLocker 金鑰復原問題。

## <a name="prerequisite"></a>必要條件

> [!div class="checklist"]
> * 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-a-qna-maker-knowledge-base"></a>建立 QnA Maker 知識庫

1. 利用您的 Azure 認證登入 QnAMaker.ai。

2. 在 QnA Maker 網站上，選取 [建立知識庫]。

   ![建立新的 KB](../media/qna-maker-create-kb.png)

3. 在 [建立] 頁面的步驟 1 中，選取 [建立 QnA 服務]。 系統會將您導向 [Azure 入口網站](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker)，以在您的訂用帳戶中設定 QnA Maker 服務。 如果 Azure 入口網站逾時，請在網站上選取 [再試一次]。 連線之後，您的 Azure 儀表板隨即出現。

4. 您在 Azure 中成功建立新的 QnA Maker 服務之後，請回到 qnamaker.ai/create。 從步驟 2 的下拉式清單中選取您的 QnA 服務。 如果您已建立新的 QnA 服務，請務必重新整理頁面。

   ![選取 QnA 服務 KB](../media/qnamaker-quickstart-kb/qnaservice-selection.png)

5. 在步驟 3 中，將您的 KB 命名為**我的範例 QnA KB**。

6. 若要將內容新增至您的 KB，請選取三種類型的資料來源。 在步驟 4 的 [填入您的 KB] 之下，在 [URL] 方塊中新增 [BitLocker 修復常見問題集](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview-and-requirements-faq) URL。

   ![選取 QnA 服務 KB](../media/qnamaker-quickstart-kb/add-datasources.png)

7. 在步驟 5 中，選取 [建立您的 KB]。

8. 建立 KB 時，會出現一個快顯視窗。 擷取程序需要幾分鐘的時間來讀取 HTML 頁面並找出問題和回答。

9. 已成功建立 KB 之後，[知識庫] 頁面隨即開啟。 您可以在此頁面上編輯 KB 的內容。

10. 在右上方選取 [新增 QnA 配對]，以在 KB 的 [編輯] 區段中新增資料列。 在 [問題] 之下，輸入 **Hi**。 在 [回答] 之下，輸入**Hello.Ask me bitlocker questions.**

   ![加入 QnA 組](../media/qnamaker-quickstart-kb/add-qna-pair.png)

11. 在右上方，選取 [儲存並定型] 來儲存您的編輯內容並將 QnA Maker 模型定型。 除非儲存編輯內容，否則不會保留。

   ![儲存並定型](../media/qnamaker-quickstart-kb/add-qna-pair2.png)

12. 在右上方，選取 [測試] 來測試您所做的變更生效。 在方塊中輸入 **hi there**，然後選取 Enter。 您應會看到您所建立的回答成為回應。

13. 選取 [檢查]，更詳細地檢查回應。 測試視窗用來測試您對 KB 所做的變更，然後加以發佈。

   ![測試面板](../media/qnamaker-quickstart-kb/inspect-panel.png)

14. 再次選取 [測試]，以關閉 [測試] 快顯視窗。

15. 在 [編輯] 旁邊的功能表中，選取 [發佈]。 若要接著確認，請選取頁面上的 [發佈]。

16. QnA Maker 服務現在已成功發佈。 您可以在您的應用程式或 Bot 程式碼中使用此端點。

   ![發佈](../media/qnamaker-quickstart-kb/publish-sucess.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [建立知識庫](../How-To/create-knowledge-base.md)
