---
title: 以群組或集合批次處理訊息 - Azure Logic Apps | Microsoft Docs
description: 在 Azure Logic Apps 中以批次方式傳送和接收訊息
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, jonfan, LADocs
ms.topic: article
ms.date: 08/19/2018
ms.openlocfilehash: bd31de8f60fff5630141f708714083fe76220d11
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47410148"
---
# <a name="send-receive-and-batch-process-messages-in-azure-logic-apps"></a>在 Azure Logic Apps 中傳送、接收及批次處理訊息

若要以群組方式一起傳送和處理訊息，您可以建立批次解決方案，以將訊息收集到*批次*中，直到符合您針對發行和處理分批訊息所指定的準則為止。 批次處理可降低您的邏輯應用程式處理訊息的頻率。 本文說明如何藉由在相同 Azure 訂用帳戶、Azure 區域內建立兩個邏輯應用程式，並遵循這個特定順序，來建置批次處理解決方案： 

* [「批次接收者」](#batch-receiver)邏輯應用程式，會接受訊息並收集到批次中，直到符合指定準則而可釋出和處理這些訊息。

  請務必先建立批次接收者，這樣一來，您之後才可以在建立批次傳送者時選取批次目的地。

* 一或多個[「批次傳送者」](#batch-sender)邏輯應用程式，可將訊息傳送給先前建立的批次接收者。 

   您也可以指定唯一索引鍵，例如客戶編號，以便將目標批次「分割」或劃分為以該索引鍵為基礎的邏輯子集。 如此一來，接收者應用程式即可使用相同的索引鍵來收集所有項目並同時處理它們。

請務必讓批次接收者和批次傳送者共用相同的 Azure 訂用帳戶和 Azure 區域。 如果未共用，您就無法在建立批次傳送者時選取批次接收者，原因是兩者並無法看到彼此。

## <a name="prerequisites"></a>必要條件

若要遵循此範例，您需要這些項目：

* Azure 訂用帳戶。 如果您沒有訂用帳戶，您可以[開始使用免費 Azure 帳戶](https://azure.microsoft.com/free/)。 或者，請[註冊預付型方案訂用帳戶](https://azure.microsoft.com/pricing/purchase-options/)。

* 電子郵件帳戶與任何 [Azure Logic Apps 所支援的電子郵件提供者](../connectors/apis-list.md)

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識 

* 若要使用 Visual Studio 而非 Azure 入口網站，請務必[設定 Visual Studio 以便使用 Logic Apps](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)。

<a name="batch-receiver"></a>

## <a name="create-batch-receiver"></a>建立批次接收者

若要能夠將訊息傳送到批次中，該批次必須先成為這些訊息的傳送目的地。 因此，您必須先建立「批次接收者」邏輯應用程式 (其會從**批次**觸發程序來開始)。 這樣一來，當您建立「批次傳送者」邏輯應用程式時，就可以選取批次接收者邏輯應用程式。 批次接收者會繼續收集訊息，直到符合指定準則而可釋出和處理這些訊息。 雖然批次接收者不需要知道批次傳送者的任何訊息，但批次傳送者必須知道訊息的傳送目的地。 

1. 在 [Azure 入口網站](https://portal.azure.com)或 Visual Studio 中，使用下列名稱來建立一個邏輯應用程式："BatchReceiver" 

2. 在 Logic Apps 設計工具中，新增**次**觸發程序，它會啟動您的邏輯應用程式工作流程。 在搜尋方塊中，輸入 "batch" 作為篩選條件。 選取此觸發程序：**批次訊息**

   ![新增 [批次訊息] 觸發程序](./media/logic-apps-batch-process-send-receive-messages/add-batch-receiver-trigger.png)

3. 設定批次接收者屬性： 

   | 屬性 | 說明 | 
   |----------|-------------|
   | **批次模式** | - **內嵌**：用來定義批次觸發程序內的發行準則 <br>- **整合帳戶**：透過[整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)，可定義多個發行準則組態。 透過整合帳戶，您可以在同一個地方維護上述所有組態，而不是在個別的邏輯應用程式中進行維護。 | 
   | **批次名稱** | 您的批次名稱 (在此範例中為 "TestBatch")，只適用於 [內嵌] 批次模式 |  
   | **發行準則** | 僅適用於 [內嵌] 批次模式，可選取在處理每個批次之前要先符合的準則： <p>- **以訊息計數為基礎**：要在批次中收集的訊息數目，例如 10 則訊息 <br>- **以容量大小為基礎**：批次大小上限 (以位元組為單位)，例如 100 MB <br>- **以排程為基礎**：批次發行之間的間隔和頻率，例如 10 分鐘。 最小週期是 60 秒或 1 分鐘。 小數的分鐘值會無條件進位至 1 分鐘。 若要指定開始日期和時間，請選擇 [顯示進階選項]。 <br>- **全選**：使用所有指定的準則。 | 
   ||| 
   
   此範例會選取所有準則：

   ![提供批次觸發程序詳細資料](./media/logic-apps-batch-process-send-receive-messages/batch-receiver-criteria.png)

4. 現在新增可處理每個批次的一或多個動作。 

   在此範例中，新增可在批次觸發程序引發時傳送電子郵件的動作。 
   當批次有 10 則訊息、達到 10 MB，或在經過 10 分鐘之後，觸發程序會執行並傳送一封電子郵件。

   1. 在批次觸發程序下方，選擇 [新增步驟]。

   2. 在搜尋方塊中，輸入「傳送電子郵件」作為篩選條件。
   根據您的電子郵件提供者，選取電子郵件連接器。
      
      例如，如果您有私人帳戶，例如 @outlook.com 或 @hotmail.com，請選取 Outlook.com 連接器。 
      如果您有 Gmail 帳戶，請選取 Gmail 連接器。 
      此範例使用 Office 365 Outlook。 

   3. 選取此動作：**傳送電子郵件 - <電子郵件提供者>**

      例如︰

      ![為您的電子郵件提供者選取「傳送電子郵件」動作](./media/logic-apps-batch-process-send-receive-messages/batch-receiver-send-email-action.png)

5. 如果出現提示，請登入您的電子郵件帳戶。 

6. 設定所新增動作的屬性。

   * 在 [收件者] 方塊中，輸入收件者的電子郵件地址。 
   為了測試用途，您可以使用自己的電子郵件地址。

   * 在 [主旨] 方塊中，當動態內容清單出現時，請選取 [分割名稱] 欄位。

     ![從動態內容清單中，選取 [分割名稱]](./media/logic-apps-batch-process-send-receive-messages/send-email-action-details.png)

     在稍後的區段中，您可以指定唯一的分割索引鍵，將目標批次分成邏輯子集，以便可以傳送訊息至該處。 
     每個集合都有一個由批次傳送者邏輯應用程式產生的唯一數字。 
     這項功能可讓您使用具有多個子集的單一批次，並用您提供的名稱定義每個子集。

     > [!IMPORTANT]
     > 資料分割有 5,000 則訊息或 80 MB 的限制。 如果符合任何一個條件，Logic Apps 可以發行批次，即使不符合您定義的發行條件。

   * 在 [內文] 方塊中，當動態內容清單出現時，請選取 [訊息識別碼] 欄位。 

     Logic Apps 設計工具會自動在傳送電子郵件動作前後新增 "For each" 迴圈，因為該動作會將上一個動作的輸出視為集合，而非批次。 

     ![對於 [內文]，選取 [訊息識別碼]](./media/logic-apps-batch-process-send-receive-messages/send-email-action-details-for-each.png)

7.  儲存您的邏輯應用程式。 您現在已建立批次接收者。

    ![儲存您的邏輯應用程式](./media/logic-apps-batch-process-send-receive-messages/save-batch-receiver-logic-app.png)

8. 如果您使用 Visual Studio，請務必[將批次接收者邏輯應用程式部署至 Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure)。 否則，當您建立批次傳送者時，將無法選取批次接收者。

<a name="batch-sender"></a>

## <a name="create-batch-sender"></a>建立批次傳送者

現在，請建立一或多個會將訊息傳送給接收者邏輯應用程式的批次傳送者邏輯應用程式。 在每個批次傳送者中，您可以指定批次接收者和批次名稱、訊息內容，和任何其他設定。 您可以選擇性地提供唯一的分割索引鍵，將批次分成邏輯子集來收集具有該索引鍵的訊息。 

* 請確定您已經[建立批次接收者](#batch-receiver)，因此當您建立批次傳送者時，可以選取現有批次接收者作為目的地批次。 雖然批次接收者不需要知道批次傳送者的任何訊息，但批次傳送者必須知道訊息的傳送目的地。 

* 請務必讓批次接收者和批次傳送者共用相同的 Azure 區域和 Azure 訂用帳戶。 如果未共用，您就無法在建立批次傳送者時選取批次接收者，原因是兩者並無法看到彼此。

1. 建立具有名稱 "BatchSender" 的另一個邏輯應用程式

   1. 在搜尋方塊中，輸入 "recurrence" 作為篩選條件。 
   選取此觸發程序：**定期 - 排程**

      ![新增「定期 - 排程」觸發程序](./media/logic-apps-batch-process-send-receive-messages/add-schedule-trigger-batch-sender.png)

   2. 設定頻率和間隔，每分鐘執行傳送者邏輯應用程式。

      ![設定重複觸發程序的頻率和間隔](./media/logic-apps-batch-process-send-receive-messages/recurrence-trigger-batch-sender-details.png)

2. 新增動作來將訊息傳送至批次。

   1. 在定期觸發程序下方，選擇 [新增步驟]。

   2. 在搜尋方塊中，輸入 "batch" 作為篩選條件。 
   選取 [動作] 清單，然後選取此動作：**選擇包含批次觸發程序的 Logic Apps 工作流程 - 將訊息傳送給批次**

      ![選取 [選擇包含批次觸發程序的 Logic Apps 工作流程]](./media/logic-apps-batch-process-send-receive-messages/send-messages-batch-action.png)

   3. 選取您先前建立的批次接收者邏輯應用程式。

      ![選取「批次接收者」邏輯應用程式](./media/logic-apps-batch-process-send-receive-messages/batch-sender-select-batch-receiver.png)

      > [!NOTE]
      > 此清單也會顯示有批次觸發程序的任何其他邏輯應用程式。 
      > 
      > 如果您使用 Visual Studio，而且您未看到任何可選取的批次接收者，請確認您已將批次接收者部署至 Azure。 如果您尚未這麼做，請了解如何[將批次接收者邏輯應用程式部署至 Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure)。 

   4. 選取此動作：**Batch_messages - <*your-batch-receiver*>**

      ![選取此動作："Batch_messages - <your-logic-app>"](./media/logic-apps-batch-process-send-receive-messages/batch-sender-select-batch.png)

3. 設定批次傳送者的屬性：

   | 屬性 | 說明 | 
   |----------|-------------| 
   | **批次名稱** | 接收者邏輯應用程式所定義的批次名稱，在本例中為 "TestBatch" <p>**重要**：批次名稱會在執行階段驗證，而且必須符合接收者邏輯應用程式所指定的名稱。 變更批次名稱會導致批次傳送者失敗。 | 
   | **訊息內容** | 您要傳送的訊息內容 | 
   ||| 

   針對此範例，請新增此運算式，它會將目前日期和時間插入到您傳送給批次的訊息內容：

   1. 按一下 [訊息內容] 方塊內部。 

   2. 當動態內容清單出現時，選擇 [運算式]。 

   3. 輸入運算式 `utcnow()`，然後選擇 [確定]。 

      ![在 [訊息內容] 中，選擇 [運算式]，輸入 "utcnow()"，然後選擇 [確定]。](./media/logic-apps-batch-process-send-receive-messages/batch-sender-details.png)

4. 現在設定批次的資料分割。 在 "BatchReceiver" 動作中，選擇 [顯示進階選項] 並設定下列屬性：

   | 屬性 | 說明 | 
   |----------|-------------| 
   | **分割名稱** | 選擇性的唯一分割索引鍵，可用於將目標批次分割誠邏輯子集，以及根據該索引鍵收集訊息 | 
   | **訊息識別碼** | 選擇性的訊息識別碼，空白時是一個產生的 全域唯一識別碼 (GUID) | 
   ||| 

   針對此範例，在 [分割名稱] 中，請新增可產生介於 1 到 5 隨機數字的運算式。 讓 [訊息識別碼] 方塊保持空白。
   
   1. 在 [分割名稱] 方塊內按一下，動態內容清單隨即顯示。 

   2. 在動態內容清單中，選擇 [運算式]。
   
   3. 輸入運算式 `rand(1,6)`，然後選擇 [確定]。

      ![設定目標批次的資料分割](./media/logic-apps-batch-process-send-receive-messages/batch-sender-partition-advanced-options.png)

      這個 **rand** 函式會產生一到五之間的數字。 
      因此您會將此批次分割成五個有編號的資料分割，此運算式會以動態方式設定。

5. 儲存您的邏輯應用程式。 傳送者邏輯應用程式現在看起來應該類似這個範例：

   ![儲存您的傳送者邏輯應用程式](./media/logic-apps-batch-process-send-receive-messages/batch-sender-finished.png)

## <a name="test-your-logic-apps"></a>儲存邏輯應用程式

若要測試批次處理解決方案，請讓邏輯應用程式執行幾分鐘的時間。 很快，您會開始收到以五封為單位的電子郵件群組，全都具有相同的資料分割索引鍵。

批次傳送者邏輯應用程式會每分鐘執行、產生介於 1 到 5 的隨機數字，並使用這個產生的數字為訊息傳送目標批次的分割索引鍵。 每次批次有五個項目具有相同的分割索引鍵時，批次接收者邏輯應用程式便會引發，並針對每個訊息傳送郵件。

> [!IMPORTANT]
> 完成測試時，請確定您停用 BatchSender 邏輯應用程式，以停止傳送訊息，並避免造成收件匣過多郵件。

## <a name="next-steps"></a>後續步驟

* [批次處理並傳送 EDI 訊息](../logic-apps/logic-apps-scenario-edi-send-batch-messages.md)
* [使用 JSON 建置於邏輯應用程式定義上](../logic-apps/logic-apps-author-definitions.md)
* [在 Visual Studio 中使用 Azure Logic Apps 和 Functions 建置邏輯應用程式](../logic-apps/logic-apps-serverless-get-started-vs.md)
* [適用於邏輯應用程式的例外狀況處理與記錄錯誤](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)
