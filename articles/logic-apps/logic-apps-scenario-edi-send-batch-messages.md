---
title: 以群組或集合批次處理 EDI 訊息 - Azure Logic Apps | Microsoft Docs
description: 傳送 EDI 訊息以便在邏輯應用程式中批次處理
services: logic-apps
ms.service: logic-apps
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, LADocs
ms.topic: article
ms.date: 08/19/2018
ms.openlocfilehash: 7e058b7cebb9c2cdc3fb8b97bf99554b2f26dd8c
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43121570"
---
# <a name="send-edi-messages-in-batches-to-trading-partners-with-azure-logic-apps"></a>使用 Azure Logic Apps 對合作對象批次傳送 EDI 訊息

在企業對企業 (B2B) 案例中，合作夥伴通常是以群組或批次方式交換訊息。 當您使用 Logic Apps 建置批次處理解決方案時，您可以傳送訊息給合作對象，並以批次方式一起處理這些訊息。 本文說明如何使用 X12 作為範例，藉由建立「批次傳送者」邏輯應用程式和「批次接收者」邏輯應用程式，來批次處理 EDI 訊息。 

批次處理 X12 訊息的作用，和批次處理其他訊息相同；您可以使用批次觸發程序將訊息收集到批次中，並使用批次動作將訊息傳送至該批次。 此外，X12 批次處理會先包含 X12 編碼步驟，再讓訊息移至合作對象或其他目的地。 若要深入了解批次觸發程序和動作，請參閱[批次處理訊息](../logic-apps/logic-apps-batch-process-send-receive-messages.md)。

在本文中，您會藉由在相同 Azure 訂用帳戶、Azure 區域內建立兩個邏輯應用程式，並遵循這個特定順序，來建置批次處理解決方案：

* [「批次接收者」](#receiver)邏輯應用程式，會接受訊息並收集到批次中，直到符合指定準則而可釋出和處理這些訊息。 在此案例中，批次接收者也會使用指定的 X12 協議或合作夥伴身分識別，將批次中的訊息編碼。

  請務必先建立批次接收者，這樣一來，您之後才可以在建立批次傳送者時選取批次目的地。

* [「批次傳送者」](#sender)邏輯應用程式，會將訊息傳送給先前建立的批次接收者。 

請務必讓批次接收者和批次傳送者共用相同的 Azure 訂用帳戶和 Azure 區域。 如果未共用，您就無法在建立批次傳送者時選取批次接收者，原因是兩者並無法看到彼此。

## <a name="prerequisites"></a>必要條件

若要遵循此範例，您需要這些項目：

* Azure 訂用帳戶。 如果您沒有訂用帳戶，您可以[開始使用免費 Azure 帳戶](https://azure.microsoft.com/free/)。 或者，請[註冊預付型方案訂用帳戶](https://azure.microsoft.com/pricing/purchase-options/)。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 現有[整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)，已與 Azure 訂用帳戶相關聯，並連結到邏輯應用程式

* 整合帳戶中至少要有兩個現有[合作夥伴](../logic-apps/logic-apps-enterprise-integration-partners.md)。 每個合作夥伴在合作夥伴的屬性中都必須使用 X12 (標準運輸公司法典) 辨識符號作為企業身分識別。

* 整合帳戶中現有的 [X12 協議](../logic-apps/logic-apps-enterprise-integration-x12.md)

* 若要使用 Visual Studio 而非 Azure 入口網站，請務必[設定 Visual Studio 以便使用 Logic Apps](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)。

<a name="receiver"></a>

## <a name="create-x12-batch-receiver"></a>建立 X12 批次接收者

若要能夠將訊息傳送到批次中，該批次必須先成為這些訊息的傳送目的地。 因此，您必須先建立「批次接收者」邏輯應用程式 (其會從**批次**觸發程序來開始)。 這樣一來，當您建立「批次傳送者」邏輯應用程式時，就可以選取批次接收者邏輯應用程式。 批次接收者會繼續收集訊息，直到符合指定準則而可釋出和處理這些訊息。 雖然批次接收者不需要知道批次傳送者的任何訊息，但批次傳送者必須知道訊息的傳送目的地。 

針對此批次接收者，您需要指定批次模式、名稱、釋出準則、X12 協議，以及其他設定。 

1. 在 [Azure 入口網站](https://portal.azure.com)或 Visual Studio 中，使用下列名稱來建立一個邏輯應用程式："BatchX12Messages"

2. [將邏輯應用程式連結至整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account)。

3. 在 Logic Apps 設計工具中，新增**次**觸發程序，它會啟動您的邏輯應用程式工作流程。 在搜尋方塊中，輸入 "batch" 作為篩選條件。 選取此觸發程序：**批次訊息**

   ![新增批次觸發程序](./media/logic-apps-scenario-EDI-send-batch-messages/add-batch-receiver-trigger.png)

4. 設定批次接收者屬性： 

   | 屬性 | 值 | 注意 | 
   |----------|-------|-------|
   | **批次模式** | 內嵌 |  |  
   | **批次名稱** | TestBatch | 僅適用於**內嵌**批次模式 | 
   | **釋出準則** | 依據訊息計數、依據排程 | 僅適用於**內嵌**批次模式 | 
   | **訊息計數** | 10 | 僅適用於**依據訊息計數**釋出準則 | 
   | **間隔** | 10 | 僅適用於**依據排程**釋出準則 | 
   | **頻率** | 分鐘 | 僅適用於**依據排程**釋出準則 | 
   ||| 

   ![提供批次觸發程序詳細資料](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-release-criteria.png)

   > [!NOTE]
   > 此範例不會設定批次的資料分割，因此每個批次會使用相同的資料分割索引鍵。 若要深入了解資料分割，請參閱[批次處理訊息](../logic-apps/logic-apps-batch-process-send-receive-messages.md#batch-sender)。

5. 現在，新增動作來將每個批次編碼： 

   1. 在批次觸發程序下方，選擇 [新增步驟]。

   2. 在搜尋方塊中，輸入 "X12 batch" 作為篩選條件，然後選取這個動作 (任何版本)：**批次編碼 <版本> - X12** 

      ![選取「X12 批次編碼」動作](./media/logic-apps-scenario-EDI-send-batch-messages/add-batch-encode-action.png)

   3. 如果您先前未連線至整合帳戶，請立即建立該連線。 為連線提供名稱，選取您想要的整合帳戶，然後選擇 [建立]。

      ![建立批次編碼器與整合帳戶之間的連線](./media/logic-apps-scenario-EDI-send-batch-messages/batch-encoder-connect-integration-account.png)

   4. 為批次編碼器動作設定這些屬性：

      | 屬性 | 說明 |
      |----------|-------------|
      | **X12 協議的名稱** | 開啟清單，然後選取現有的協議。 <p>如果清單是空的，請務必[將邏輯應用程式連結至整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account) (具有所需協議)。 | 
      | **BatchName** | 按一下此方塊內部，然後在動態內容清單出現後，選取 [批次名稱] 權杖。 | 
      | **PartitionName** | 按一下此方塊內部，然後在動態內容清單出現後，選取 [資料分割名稱] 權杖。 | 
      | **項目** | 關閉 [項目詳細資料] 方塊，然後按一下此方塊內部。 在動態內容清單出現後，選取 [批次處理的項目] 權杖。 | 
      ||| 

      ![「批次編碼」動作詳細資料](./media/logic-apps-scenario-EDI-send-batch-messages/batch-encode-action-details.png)

      在 [項目] 方塊中：

      ![「批次編碼」動作項目](./media/logic-apps-scenario-EDI-send-batch-messages/batch-encode-action-items.png)

6. 儲存您的邏輯應用程式。 

7. 如果您使用 Visual Studio，請務必[將批次接收者邏輯應用程式部署至 Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure)。 否則，當您建立批次傳送者時，將無法選取批次接收者。

### <a name="test-your-logic-app"></a>測試應用程式邏輯

為了確定批次接收者可正常運作，您可以新增測試用的 HTTP 動作，並將批次處理的訊息傳送至[要求 Bin 服務](https://requestbin.fullcontact.com/)。 

1. 在 [X12 編碼] 動作下，選擇 [新增步驟]。 

2. 在搜尋方塊中輸入 "http" 作為篩選條件。 選取此動作：**HTTP - HTTP**
    
   ![選取 HTTP 動作](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-add-http-action.png)

3. 設定 HTTP 動作的屬性：

   | 屬性 | 說明 | 
   |----------|-------------|
   | **方法** | 從這個清單中選取 [POST]。 | 
   | **Uri** | 為要求 Bin 產生 URI，然後在這個方塊中輸入該 URI。 | 
   | **內文** | 按一下此方塊內部，然後在動態內容清單開啟後，選取 [主體] 權杖，其會出現在 [依合約名稱的批次編碼] 區段中。 <p>如果您沒有看到 [主體] 權杖，請選取 [依合約名稱的批次編碼] 旁邊的 [查看更多]。 | 
   ||| 

   ![提供 HTTP 動作詳細資料](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-add-http-action-details.png)

4. 儲存您的邏輯應用程式。 

   批次接收者邏輯應用程式類似下列範例︰ 

   ![儲存批次接收者邏輯應用程式](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-finished.png)

<a name="sender"></a>

## <a name="create-x12-batch-sender"></a>建立 X12 批次傳送者

現在，請建立一或多個會將訊息傳送給接收者邏輯應用程式的邏輯應用程式。 在每個批次傳送者中，您可以指定批次接收者邏輯應用程式和批次名稱、訊息內容，和任何其他設定。 您可以選擇性地提供唯一的資料分割索引鍵，將批次分成子集來收集具有該索引鍵的訊息。 

* 請確定您已經[建立批次接收者](#receiver)，因此當您建立批次傳送者時，可以選取現有批次接收者作為目的地批次。 雖然批次接收者不需要知道批次傳送者的任何訊息，但批次傳送者必須知道訊息的傳送目的地。 

* 請務必讓批次接收者和批次傳送者共用相同的 Azure 區域和 Azure 訂用帳戶。 如果未共用，您就無法在建立批次傳送者時選取批次接收者，原因是兩者並無法看到彼此。

1. 使用下列名稱來建立另一個邏輯應用程式："SendX12MessagesToBatch" 

2. 在搜尋方塊中，輸入「當 http 要求」作為篩選條件。 選取此觸發程序：[收到 HTTP 要求時] 
   
   ![新增「要求」觸發程序](./media/logic-apps-scenario-EDI-send-batch-messages/add-request-trigger-sender.png)

3. 新增動作來將訊息傳送至批次。

   1. 在 [HTTP 要求] 動作下，選擇 [新增步驟]。

   2. 在搜尋方塊中，輸入 "batch" 作為篩選條件。 
   選取 [動作] 清單，然後選取此動作：**選擇包含批次觸發程序的 Logic Apps 工作流程 - 將訊息傳送給批次**

      ![選取 [選擇包含批次觸發程序的 Logic Apps 工作流程]](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-select-batch-trigger.png)

   3. 現在，選取您先前建立的 "BatchX12Messages" 邏輯應用程式。

      ![選取「批次接收者」邏輯應用程式](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-select-batch-receiver.png)

   4. 選取此動作：**Batch_messages - <*your-batch-receiver*>**

      ![選取 [Batch_messages] 動作](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-select-batch-messages-action.png)

4. 設定批次傳送者的屬性。

   | 屬性 | 說明 | 
   |----------|-------------| 
   | **批次名稱** | 接收者邏輯應用程式所定義的批次名稱，在本例中為 "TestBatch" <p>**重要**：批次名稱會在執行階段驗證，而且必須符合接收者邏輯應用程式所指定的名稱。 變更批次名稱會導致批次傳送者失敗。 | 
   | **訊息內容** | 您想要傳送的訊息內容，在本例中為**主體**權杖 | 
   ||| 
   
   ![設定批次屬性](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-set-batch-properties.png)

5. 儲存您的邏輯應用程式。 

   批次傳送者邏輯應用程式類似下列範例︰

   ![儲存批次傳送者邏輯應用程式](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-finished.png)

## <a name="test-your-logic-apps"></a>儲存邏輯應用程式

若要測試批次處理解決方案，請從 [Postman](https://www.getpostman.com/postman) 或類似的工具，將 X12 訊息張貼到批次傳送者邏輯應用程式。 不久之後，您就會開始在要求 Bin 中收到 X12 訊息 (每隔 10 分鐘或每 10 個一批)，這些訊息全都具有相同的資料分割索引鍵。

## <a name="next-steps"></a>後續步驟

* [以批次方式處理訊息](../logic-apps/logic-apps-batch-process-send-receive-messages.md) 
