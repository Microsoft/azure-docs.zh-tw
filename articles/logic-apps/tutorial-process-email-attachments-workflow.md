---
title: 建置處理電子郵件和附件的工作流程 - Azure Logic Apps | Microsoft Docs
description: 本教學課程說明如何建立自動化工作流程，以讓您使用 Azure Logic Apps、Azure 儲存體和 Azure Functions 來處理電子郵件和附件
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: tutorial
ms.custom: mvc
ms.date: 07/20/2018
ms.reviewer: klam, LADocs
ms.openlocfilehash: 133cc9d8fa52bb655e9baaad53ee157fdc7524f7
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39429409"
---
# <a name="process-emails-and-attachments-with-azure-logic-apps"></a>使用 Azure Logic Apps 來處理電子郵件和附件

Azure Logic Apps 可協助您自動執行工作流程，並整合 Azure 服務、Microsoft 服務和其他軟體即服務 (SaaS) 應用程式與內部部署系統的資料。 此教學課程說明如何建置[邏輯應用程式](../logic-apps/logic-apps-overview.md)，用以處理內送電子郵件和任何附件。 此邏輯應用程式可處理該內容、將內容儲存至 Azure 儲存體，以及傳送檢閱該內容的通知。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定 [Azure 儲存體](../storage/common/storage-introduction.md)和儲存體總管，用以檢查已儲存的電子郵件和附件。
> * 建立從電子郵件中移除 HTML 的 [Azure 函式](../azure-functions/functions-overview.md)。 此教學課程包含可供您用於此函式的程式碼。
> * 建立空白邏輯應用程式。
> * 新增用來監視電子郵件附件的觸發程序。
> * 新增會檢查電子郵件是否有附件的條件。
> * 新增會在電子郵件有附件時呼叫 Azure 函式的動作。
> * 新增會為電子郵件和附件建立儲存體 Blob 的動作。
> * 新增傳送電子郵件通知的動作。

當您完成時，邏輯應用程式大致如下列工作流程所示︰

![完成的邏輯應用程式概觀](./media/tutorial-process-email-attachments-workflow/overview.png)

如果您沒有 Azure 訂用帳戶，請在開始前，先<a href="https://azure.microsoft.com/free/" target="_blank">註冊一個免費的 Azure 帳戶</a>。 

## <a name="prerequisites"></a>必要條件

* Logic Apps 支援的任何電子郵件提供者 (例如 Office 365 Outlook、Outlook.com 或 Gmail) 所提供的電子郵件帳戶。 對於其他提供者，請[檢閱這裡的連接器清單](https://docs.microsoft.com/connectors/)。

  此邏輯應用程式會使用 Office 365 Outlook 帳戶。 
  如果您使用不同的電子郵件帳戶，整體步驟將維持不變，但您的 UI 外觀可能會略有不同。

* 下載並安裝<a href="https://storageexplorer.com/" target="_blank">免費 Microsoft Azure 儲存體總管</a>。 此工具可協助您確認儲存體容器已正確設定。

## <a name="sign-in-to-azure-portal"></a>登入 Azure 入口網站

使用您的 Azure 帳戶認證登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。

## <a name="set-up-storage-to-save-attachments"></a>設定用來儲存附件的儲存體

您可以將內送電子郵件和附件儲存為 [Azure 儲存體容器](../storage/common/storage-introduction.md)中的 Blob。 

1. 您必須先透過下列設定[建立儲存體帳戶](../storage/common/storage-create-storage-account.md#create-a-storage-account)，才能建立儲存體容器：

   | 設定 | 值 | 說明 | 
   |---------|-------|-------------| 
   | **名稱** | attachmentstorageacct | 儲存體帳戶的名稱 | 
   | **部署模型** | 資源管理員 | 管理資源部署的[部署模型](../azure-resource-manager/resource-manager-deployment-model.md) | 
   | **帳戶類型** | 一般用途 | [儲存體帳戶類型](../storage/common/storage-introduction.md#types-of-storage-accounts) | 
   | **位置** | 美國西部 | 用來儲存您儲存體帳戶相關資訊的區域 | 
   | **複寫** | 本機備援儲存體 (LRS) | 此設定會指定如何複製、儲存、管理及同步處理您的資料。 請參閱[複寫](../storage/common/storage-introduction.md#replication)。 | 
   | **效能** | 標準 | 此設定會指定支援的資料類型和用來儲存資料的媒體。 請參閱[儲存體帳戶類型](../storage/common/storage-introduction.md#types-of-storage-accounts)。 | 
   | **需要安全傳輸** | 已停用 | 此設定會指定連線要求所需的安全性。 請參閱[需要安全傳輸](../storage/common/storage-require-secure-transfer.md)。 | 
   | **訂用帳戶** | <your-Azure-subscription-name> | Azure 訂用帳戶的名稱 | 
   | **資源群組** | LA-Tutorial-RG | 用來組織及管理相關資源的 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱。 <p>**注意：** 資源群組會存在於某個特定區域內。 雖然此教學課程中提及的項目可能並非適用於所有區域，但請盡可能使用相同的區域。 | 
   | **設定虛擬網路** | 已停用 | 在此教學課程中，請保留 [停用] 設定。 | 
   |||| 

   若要建立儲存體帳戶，您也可以使用 [Azure PowerShell](../storage/common/storage-quickstart-create-storage-account-powershell.md) 或 [Azure CLI](../storage/common/storage-quickstart-create-storage-account-cli.md)。

2. 在 Azure 部署您的儲存體帳戶之後，請取得儲存體帳戶的存取金鑰：

   1. 在儲存體帳戶功能表的 [設定] 下，選取 [存取金鑰]。 

   2. 複製您的儲存體帳戶名稱和 **key1**，然後將這些值儲存到其他安全之處。

      ![複製並儲存儲存體帳戶名稱和金鑰](./media/tutorial-process-email-attachments-workflow/copy-save-storage-name-key.png)

   若要取得儲存體帳戶的存取金鑰，您也可以使用 [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.storage/get-azurermstorageaccountkey) 或 [Azure CLI](https://docs.microsoft.com/cli/azure/storage/account/keys?view=azure-cli-latest.md#az-storage-account-keys-list)。 

3. 建立電子郵件附件的 Blob 儲存體容器。
   
   1. 在您的儲存體帳戶功能表上，選取 [概觀]。 
   在 [服務] 下，選取 [Blob]。

      ![新增 Blob 儲存體容器](./media/tutorial-process-email-attachments-workflow/create-storage-container.png)

   2. 在 [容器] 頁面開啟後，在工具列上選取 [容器]。 

   3. 在 [新增容器] 下方，輸入「附件」作為容器名稱。 
   在 [公用存取層級] 下選取 [容器 (容器和 Blob 匿名讀取權限)]，然後選擇 [確定]。

      完成作業後，您可以在 Azure 入口網站中的儲存體帳戶中找到您的儲存體容器，位置如下：

      ![已完成的儲存體容器](./media/tutorial-process-email-attachments-workflow/created-storage-container.png)

   若要建立儲存體容器，您也可以使用 [Azure PowerShell](https://docs.microsoft.com/powershell/module/azure.storage/new-azurestoragecontainer) 或 [Azure CLI](https://docs.microsoft.com/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-create)。 

接著，將儲存體總管連線至您的儲存體帳戶。

## <a name="set-up-storage-explorer"></a>設定儲存體總管

現在，將儲存體總管連線至儲存體帳戶，以確認邏輯應用程式可將附件正確地儲存為儲存體容器中的 Blob。

1. 開啟 [Microsoft Azure 儲存體總管]。 

   儲存體總管會提示您連線至儲存體帳戶。 

2. 在 [連線到 Azure 儲存體] 窗格中，選取 [使用儲存體帳戶名稱和金鑰]，然後選擇 [下一步]。 

   ![儲存體總管 - 連線至儲存體帳戶](./media/tutorial-process-email-attachments-workflow/storage-explorer-choose-storage-account.png)

   > [!TIP]
   > 若未出現提示，請在儲存體總管工具列上選擇 [新增帳戶]。

3. 在 [帳戶名稱]  下方，提供您的儲存體帳戶名稱。 在 [帳戶金鑰] 下方，提供您先前儲存的存取金鑰。 選擇 [下一步]。

4. 確認您的連線資訊，然後選擇 [連線]。 

   儲存體總管會建立連線，並在 [總管] 視窗中的 [(本機與已連結的資源)] > [儲存體帳戶] 下顯示您的儲存體帳戶。 

5. 若要尋找您的 Blob 儲存體容器，請在 [儲存體帳戶] 下方展開您的儲存體帳戶 (在本文中為 **attachmentstorageacct**)，然後展開**附件**容器所在的 **Blob Containers**，例如： 

   ![儲存體總管 - 尋找儲存體容器](./media/tutorial-process-email-attachments-workflow/storage-explorer-check-contianer.png)

接著，建立從內送電子郵件中移除 HTML 的 [Azure 函式](../azure-functions/functions-overview.md)。

## <a name="create-function-to-clean-html"></a>建立會清除 HTML 的函式

現在，使用這些步驟所提供的程式碼片段，建立會從每個內送電子郵件中移除 HTML 的 Azure 函式。 這樣一來，電子郵件內容將更為簡潔而易於處理。 然後，您就可以從邏輯應用程式呼叫此函式。

1. 您必須先透過下列設定[建立函式應用程式](../azure-functions/functions-create-function-app-portal.md)，才能建立函式：

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **應用程式名稱** | CleanTextFunctionApp | 函式應用程式的全域獨特描述性名稱 | 
   | **訂用帳戶** | <your-Azure-subscription-name> | 您先前使用的相同 Azure 訂用帳戶 | 
   | **資源群組** | LA-Tutorial-RG | 您先前使用的相同 Azure 資源群組 | 
   | **主控方案** | 取用方案 | 此設定會決定如何配置和調整執行函式應用程式所需的資源，例如運算能力。 請參閱[主控方案比較](../azure-functions/functions-scale.md)。 | 
   | **位置** | 美國西部 | 您先前使用的相同區域 | 
   | **儲存體** | cleantextfunctionstorageacct | 為您的函式應用程式建立儲存體帳戶。 請一律使用小寫字母和數字。 <p>**注意：** 此儲存體帳戶包含您的函式應用程式，且不同於您先前為電子郵件附件建立的儲存體帳戶。 | 
   | **Application Insights** | 關閉 | 可開啟 [Application Insights](../application-insights/app-insights-overview.md) 的應用程式監視功能，但在此教學課程中，請選擇 [關閉] 設定。 | 
   |||| 

   如果函式應用程式未在部署之後自動開啟，請在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>中尋找您的應用程式。 在主要 Azure 功能表上選取 [函式應用程式]，然後選取您的函式應用程式。 

   ![選取函數應用程式](./media/tutorial-process-email-attachments-workflow/select-function-app.png)

   如果 [函式應用程式] 未出現在 Azure 功能表上，請移至 [所有服務]。 在搜尋方塊中尋找 [函式應用程式]，並加以選取。 如需詳細資訊，請參閱[建立函式](../azure-functions/functions-create-first-azure-function.md)。

   否則，Azure 會自動開啟您的函式應用程式，如下所示：

   ![已建立的函式應用程式](./media/tutorial-process-email-attachments-workflow/function-app-created.png)

   若要建立函式應用程式，您也可以使用 [Azure CLI](../azure-functions/functions-create-first-azure-function-azure-cli.md)，或是 [PowerShell 和 Resource Manager 範本](../azure-resource-manager/resource-group-template-deploy.md)。

2. 在 [函式應用程式] 下展開 **CleanTextFunctionApp**，然後選取 [函式]。 在函式工具列上，選取 [+ 新增函式]。

   ![建立新的函式](./media/tutorial-process-email-attachments-workflow/function-app-new-function.png)

3. 在 [選擇下列範本或移至快速入門] 下方開啟 [案例] 清單，然後選取 [Core]。 在 [HTTP 觸發程序] 範本中，選取 [C#]。

   ![選取函式範本](./media/tutorial-process-email-attachments-workflow/function-select-httptrigger-csharp-function-template.png)

   > [!NOTE]
   > 此範例會提供 C# 範例程式碼，讓您能夠依照範例操作而無須熟悉 C#。

4. 在 [新增函式] 窗格中的 [名稱] 下方，輸入 ```RemoveHTMLFunction```。 讓 [授權等級] 設定為 [函式]，然後選擇 [建立]。

   ![函式命名](./media/tutorial-process-email-attachments-workflow/function-provide-name.png)

5. 編輯器開啟之後，請將範本程式碼取代為此範例程式碼，以移除 HTML 並將結果傳回至呼叫端：

   ``` CSharp
   using System.Net;
   using System.Text.RegularExpressions;

   public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
   {
      log.Info($"HttpWebhook triggered");

      // Parse query parameter
      string emailBodyContent = await req.Content.ReadAsStringAsync();

      // Replace HTML with other characters
      string updatedBody = Regex.Replace(emailBodyContent, "<.*?>", string.Empty);
      updatedBody = updatedBody.Replace("\\r\\n", " ");
      updatedBody = updatedBody.Replace(@"&nbsp;", " ");

      // Return cleaned text
      return req.CreateResponse(HttpStatusCode.OK, new { updatedBody });
   }
   ```

6. 完成之後，請選擇 [儲存]。 若要測試您的函式，請在編輯器右邊緣的箭號 (**<**) 圖示下方選擇 [測試]。 

   ![開啟 [測試] 窗格](./media/tutorial-process-email-attachments-workflow/function-choose-test.png)

7. 在 [測試] 窗格的 [要求本文] 下輸入這一行，然後選擇 [執行]。

   ```json
   {"name": "<p><p>Testing my function</br></p></p>"}
   ```

   ![測試您的函式](./media/tutorial-process-email-attachments-workflow/function-run-test.png)

   [輸出] 視窗會顯示函式的結果：

   ```json
   {"updatedBody":"{\"name\": \"Testing my function\"}"}
   ```

確認您的函式運作正常後，請建立邏輯應用程式。 雖然本教學課程說明的是如何建立從電子郵件中移除 HTML 的函式，但 Logic Apps 也提供 **HTML 轉換為文字**的連接器。

## <a name="create-your-logic-app"></a>建立邏輯應用程式

1. 在主要 Azure 功能表中，選取 [建立資源] > 
[整合] > [邏輯應用程式]。

   ![建立邏輯應用程式](./media/tutorial-process-email-attachments-workflow/create-logic-app.png)

2. 在 [建立邏輯應用程式] 下，提供有關於邏輯應用程式的這項資訊，如下所示和描述。 完成時，請選擇 [釘選到儀表板] > [建立]。

   ![提供邏輯應用程式資訊](./media/tutorial-process-email-attachments-workflow/create-logic-app-settings.png)

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **名稱** | LA-ProcessAttachment | 邏輯應用程式的名稱 | 
   | **訂用帳戶** | <your-Azure-subscription-name> | 您先前使用的相同 Azure 訂用帳戶 | 
   | **資源群組** | LA-Tutorial-RG | 您先前使用的相同 Azure 資源群組 |
   | **位置** | 美國西部 | 您先前使用的相同區域 | 
   | **Log Analytics** | 關閉 | 在本教學課程中，請選擇 [關閉] 設定。 | 
   |||| 

3. 在 Azure 部署您的應用程式之後，Logic Apps 設計工具隨即開啟，並顯示含有簡介影片和常用邏輯應用程式模式範本的頁面。 在 [範本] 底下，選擇 [空白邏輯應用程式]。

   ![選擇空白邏輯應用程式範本](./media/tutorial-process-email-attachments-workflow/choose-logic-app-template.png)

接下來，請新增[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)，用以接聽含有附件的內送電子郵件。 每個邏輯應用程式都必須使用觸發程序啟動，而該觸發程序會在特定事件發生或新資料符合特定條件時引發。 如需詳細資訊，請參閱[建立您的第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="monitor-incoming-email"></a>監視內送電子郵件

1. 在設計工具的搜尋方塊中，輸入「新的電子郵件送達時」作為篩選條件。 請為您的電子郵件提供者選取此觸發程序：**新的電子郵件送達時 - <*your-email-provider*>**

   例如︰

   ![為電子郵件提供者選取此觸發程序：「新的電子郵件送達時」](./media/tutorial-process-email-attachments-workflow/add-trigger-when-email-arrives.png)

   * 對於 Azure 工作或學校帳戶，選取 Office 365 Outlook。 
   * 對於個人 Microsoft 帳戶，選取 Outlook.com。 

2. 如果系統要求提供認證，請登入您的電子郵件帳戶，讓 Logic Apps 能夠連線至您的電子郵件帳戶。

3. 現在，請提供觸發程序用來篩選新電子郵件的準則。

   1. 指定檢查電子郵件的資料夾、間隔和頻率。

      ![指定檢查郵件的資料夾、間隔和頻率](./media/tutorial-process-email-attachments-workflow/set-up-email-trigger.png)

      | 設定 | 值 | 說明 | 
      | ------- | ----- | ----------- | 
      | **資料夾** | Inbox | 要檢查的電子郵件資料夾 | 
      | **間隔** | 1 | 在檢查之間所要等待的間隔數目 | 
      | **頻率** | 分鐘 | 在檢查之間每個間隔的時間單位 | 
      |  |  |  | 
  
   2. 選擇 [顯示進階選項]，並指定下列設定：

      | 設定 | 值 | 說明 | 
      | ------- | ----- | ----------- | 
      | **具有附件** | 是 | 僅取得含附件的電子郵件。 <p>**注意：** 觸發程序並不會從您的帳戶移除任何電子郵件，而只會檢查新的訊息及處理符合主旨篩選條件的電子郵件。 | 
      | **包含附件** | 是 | 取得附件並作為工作流程的輸入，而非只是檢查是否有附件。 | 
      | **主旨篩選** | ```Business Analyst 2 #423501``` | 要在電子郵件主旨中尋找的文字 | 
      |  |  |  | 

4. 若要立即隱藏觸發程序的詳細資料，請按一下觸發程序的標題列內部。

   ![摺疊圖形以隱藏詳細資料](./media/tutorial-process-email-attachments-workflow/collapse-trigger-shape.png)

5. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。

   邏輯應用程式目前作用中，但不會執行檢查電子郵件以外的任何其他作業。 
   接下來請新增條件，以指定繼續執行工作流程的準則。

## <a name="check-for-attachments"></a>檢查是否有附件

現在，請新增只會選取含附件之電子郵件的條件。

1. 在觸發程序下方，選擇 [新增步驟] > [新增條件]。

   ![[新增步驟]、[新增條件]](./media/tutorial-process-email-attachments-workflow/add-condition-under-trigger.png)

2. 以更適當的描述為條件重新命名。

   1. 在條件的標題列上，選擇 **省略符號** (**...**) 按鈕 > [重新命名]。

      ![重新命名條件](./media/tutorial-process-email-attachments-workflow/condition-rename.png)

   2. 以下列描述為條件重新命名：```If email has attachments and key subject phrase```

3. 建立會檢查電子郵件是否有附件的條件。 

   1. 在第一個資料列的 [和] 下方，按一下左側方塊內部。 
   從顯示的動態內容清單中，選取 [有附件] 屬性。

      ![建置條件](./media/tutorial-process-email-attachments-workflow/build-condition.png)

   2. 在中間的方塊中，將運算子保留為 [等於]。

   3. 在右側的方塊中輸入 **True** 值，以與觸發程序中的 [有附件] 屬性值進行比較。

      ![建置條件](./media/tutorial-process-email-attachments-workflow/finished-condition.png)

      如果這兩個值相等，表示電子郵件至少有一個附件，因此符合條件，工作流程會繼續執行。

   在您可以使用程式碼編輯器視窗來檢視的基礎邏輯應用程式定義中，此條件會如下列範例所示：

   ```json
   "Condition": {
      "actions": { <actions-to-run-when-condition-passes> },
      "expression": {
         "and": [ {
            "equals": [
               "@triggerBody()?['HasAttachment']",
                 "True"
            ]
         } ]
      },
      "runAfter": {},
      "type": "If"
   }
   ```

4. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。

### <a name="test-your-condition"></a>測試您的條件

現在，請測試條件是否正常運作：

1. 如果您的邏輯應用程式尚未執行，請在設計工具工具列上選擇 [執行]。

   此步驟可讓您手動啟動邏輯應用程式，而不必等待指定的間隔經過。 
   不過，在測試電子郵件送達收件匣之前，並不會執行任何動作。 

2. 將符合此準則的電子郵件傳送給自己：

   * 您的電子郵件主旨具有您在觸發程序的 [主旨篩選條件] 中指定的文字：```Business Analyst 2 #423501```

   * 您的電子郵件有一個附件。 
   現在，請建立一個空的文字檔，並將該檔案附加到您的電子郵件。

   當電子郵件送達時，邏輯應用程式會檢查是否有附件和指定的主旨文字。
   如果符合條件，觸發程序即會引發，並使 Logic Apps 引擎建立邏輯應用程式執行個體及啟動工作流程。 

3. 若要確認觸發程序已引發且邏輯應用程式已成功執行，請在邏輯應用程式功能表上選擇 [概觀]。

   ![檢查觸發程序和執行歷程記錄](./media/tutorial-process-email-attachments-workflow/checkpoint-run-history.png)

   如果邏輯應用程式未觸發，或雖然成功觸發但未執行，請參閱[對邏輯應用程式進行疑難排解](../logic-apps/logic-apps-diagnosing-failures.md)。

接著，請定義要為 [若為 true] 分支採取的動作。 若要儲存電子郵件及任何附件，請從電子郵件內文中移除任何 HTML，然後在儲存體容器中為電子郵件與附件建立 Blob。

> [!NOTE]
> 電子郵件沒有附件時，邏輯應用程式不需要為 [若為 false] 分支執行任何動作。 如果您在完成此教學課程後想要做額外練習，您可以新增任何要為 [若為 false] 分支執行的適當動作。

## <a name="call-removehtmlfunction"></a>呼叫 RemoveHTMLFunction

此步驟會將您先前建立的 Azure 函式新增至邏輯應用程式，並將電子郵件觸發程序中的電子郵件內文內容傳至您的函式。

1. 在邏輯應用程式功能表上，選取 [邏輯應用程式設計工具]。 在 [若為 true] 分支中，選擇 [新增動作]。

   ![在 [若為 true] 內，新增動作](./media/tutorial-process-email-attachments-workflow/if-true-add-action.png)

2. 在搜尋方塊中尋找「azure 函式」，並選取下列動作：**選擇 Azure 函式 – Azure Functions**

   ![選取「選擇 Azure 函式」的動作](./media/tutorial-process-email-attachments-workflow/add-action-azure-function.png)

3. 選取您先前建立的函式應用程式：**CleanTextFunctionApp**

   ![選取您的 Azure 函式應用程式](./media/tutorial-process-email-attachments-workflow/add-action-select-azure-function-app.png)

4. 現在，選取您的函式：**RemoveHTMLFunction**

   ![選取您的 Azure 函式](./media/tutorial-process-email-attachments-workflow/add-action-select-azure-function.png)

5. 以下列說明為函式圖形重新命名：```Call RemoveHTMLFunction to clean email body```

6. 現在，指定您的函式要處理的輸入。 

   1. 在 [要求本文] 下方，輸入此文字並加上尾端空格： 
   
      ```{ "emailBody": ``` 

      當您在後續步驟中處理此輸入時，在您的輸入正確地格式化為 JSON 之前，都會出現 JSON 無效的相關錯誤。
      在您先前測試此函式時，為此函式指定的輸入使用的是 JavaScript 物件標記法 (JSON)。 
      因此，要求本文也必須使用相同的格式。

      此外，當游標位於 [要求本文] 方塊內時，也會出現動態內容清單，讓您可從先前的動作選取可用的屬性值。 
      
   2. 在動態內容清單中的 [新的電子郵件送達時] 下方，選取 [內文] 屬性。 在此屬性後面，請記得加上右大括號：```}```

      ![指定要傳至函式的要求本文](./media/tutorial-process-email-attachments-workflow/add-email-body-for-function-processing.png)

   完成上述步驟後，函式的輸入會如下列範例所示︰

   ![已完成而要傳至函式的要求本文](./media/tutorial-process-email-attachments-workflow/add-email-body-for-function-processing-2.png)

7. 儲存您的邏輯應用程式。

接著，新增會在您的儲存體容器中建立 Blob 的動作，以便您儲存電子郵件內文。

## <a name="create-blob-for-email-body"></a>為電子郵件內文建立 Blob

1. 在 [若為 true] 區塊中，從您的 Azure 函式下方選擇 [新增動作]。 

2. 在搜尋方塊中，輸入「建立 Blob」作為篩選條件，並選取下列動作：**建立 Blob - Azure Blob 儲存體**

   ![新增為電子郵件內文建立 Blob 的動作](./media/tutorial-process-email-attachments-workflow/create-blob-action-for-email-body.png)

3. 透過以下說明的設定建立儲存體帳戶的連線。 完成之後，請選擇 [建立]。

   ![建立儲存體帳戶的連線](./media/tutorial-process-email-attachments-workflow/create-storage-account-connection-first.png)

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | 連線名稱 | AttachmentStorageConnection | 連線的描述性名稱 | 
   | **儲存體帳戶** | attachmentstorageacct | 您先前建立用來儲存附件之儲存體帳戶的名稱 | 
   |||| 

4. 以下列說明為 [建立 Blob] 動作重新命名：```Create blob for email body```

5. 在 [建立 Blob] 動作中提供此資訊並選取這些欄位，以建立 Blob，說明如下：

   ![提供電子郵件內文的 Blob 資訊](./media/tutorial-process-email-attachments-workflow/create-blob-for-email-body.png)

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **資料夾路徑** | /attachments | 您先前建立之容器的路徑和名稱。 在此範例中，請按一下資料夾圖示，然後選取 "/attachments" 容器。 | 
   | **Blob 名稱** | **寄件者**欄位 | 此範例中，請使用寄件者的名稱作為 Blob 的名稱。 按一下此方塊內部讓動態內容清單顯示，然後在 [新的電子郵件送達時] 動作下方選取 [寄件者] 欄位。 | 
   | **Blob 內容** | **內容**欄位 | 在此範例中，請使用無 HTML 電子郵件內文作為 Blob 內容。 按一下此方塊內部讓動態內容清單顯示，然後在 [呼叫 RemoveHTMLFunction 以清除電子郵件內文] 動作下方選取 [內文]。 |
   |||| 

   完成上述步驟後，動作會如下列範例所示︰

   ![已完成的「建立 Blob」動作](./media/tutorial-process-email-attachments-workflow/create-blob-for-email-body-done.png)

6. 儲存您的邏輯應用程式。 

### <a name="check-attachment-handling"></a>檢查附件處理情形

現在，請測試邏輯應用程式是否依照您指定的方式處理電子郵件：

1. 如果您的邏輯應用程式尚未執行，請在設計工具工具列上選擇 [執行]。

2. 將符合此準則的電子郵件傳送給自己：

   * 您的電子郵件主旨具有您在觸發程序的 [主旨篩選條件] 中指定的文字：```Business Analyst 2 #423501```

   * 您的電子郵件至少有一個附件。 
   現在，請建立一個空的文字檔，並將該檔案附加到您的電子郵件。

   * 您的電子郵件在內文中有一些測試內容，例如： 

     ```
     Testing my logic app
     ```

   如果邏輯應用程式未觸發，或雖然成功觸發但未執行，請參閱[對邏輯應用程式進行疑難排解](../logic-apps/logic-apps-diagnosing-failures.md)。

3. 確認邏輯應用程式已將電子郵件儲存至正確的儲存體容器。 

   1. 在儲存體總管中，依序展開 **[(Local and Attached)]\(本機與已連結的資源\)** > 
    **[儲存體帳戶]** > **[attachmentstorageacct (外部)]** > 
    **[Blob 容器]** > **[附件]** 。

   2. 檢查**附件**容器中是否有電子郵件。 

      此時，只有電子郵件會出現在容器中，因為邏輯應用程式尚未處理附件。

      ![檢查儲存體總管中是否有已儲存的電子郵件](./media/tutorial-process-email-attachments-workflow/storage-explorer-saved-email.png)

   3. 完成作業後，請刪除儲存體總管中的電子郵件。

4. (選擇性) 若要測試 [若為 false] 分支 (至此尚未執行任何動作)，您可以傳送不符合準則的電子郵件。

接下來，請新增迴圈以處理所有電子郵件附件。

## <a name="process-attachments"></a>處理附件

若要處理電子郵件中的每個附件，請在邏輯應用程式的工作流程中新增 **For each** 迴圈。

1. 在 [為電子郵件內文建立 Blob] 圖形下方，選取 [更多] > [新增 For each]。

   ![新增 "for each" 迴圈](./media/tutorial-process-email-attachments-workflow/add-for-each-loop.png)

2. 以下列說明為迴圈重新命名：```For each email attachment```

3. 現在，指定要由迴圈處理的資料。 按一下 [選取先前步驟中的輸出] 方塊內部，讓動態內容清單開啟，然後選取 [附件]。 

   ![取得「附件」](./media/tutorial-process-email-attachments-workflow/select-attachments.png)

   [附件] 欄位會傳入一個陣列，其中包含電子郵件所含的所有附件。 
   **For each** 迴圈會對隨陣列傳入的每個項目重複執行動作。

4. 儲存您的邏輯應用程式。

接著，請新增會在您的**附件**儲存體容器中將每個附件儲存為 Blob 的動作。

## <a name="create-blob-for-each-attachment"></a>為附件建立 Blob

1. 在 [針對每個電子郵件附件] 迴圈中選擇 [新增動作]，以便您對每個找到的附件指定執行工作。

   ![將動作新增至迴圈](./media/tutorial-process-email-attachments-workflow/for-each-add-action.png)

2. 在搜尋方塊中，輸入「建立 Blob」作為篩選條件，然後選取下列動作：**建立 Blob - Azure Blob 儲存體**

   ![新增建立 Blob 的動作](./media/tutorial-process-email-attachments-workflow/create-blob-action-for-attachments.png)

3. 以下列說明為**建立 Blob 2** 動作重新命名：```Create blob for each email attachment```

4. 在 [為每個電子郵件附件建立 Blob] 動作中提供此資訊，並為您要建立的每個 Blob 選取屬性，說明如下：

   ![提供 Blob 資訊](./media/tutorial-process-email-attachments-workflow/create-blob-per-attachment.png)

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **資料夾路徑** | /attachments | 您先前建立之容器的路徑和名稱。 在此範例中，請按一下資料夾圖示，然後選取 "/attachments" 容器。 | 
   | **Blob 名稱** | **名稱**欄位 | 此範例中，請使用附件的名稱作為 Blob 的名稱。 按一下此方塊內部讓動態內容清單顯示，然後在 [新的電子郵件送達時] 動作下方選取 [名稱] 欄位。 | 
   | **Blob 內容** | **內容**欄位 | 在此範例中，請使用 [內容] 欄位作為 Blob 內容。 按一下此方塊內部讓動態內容清單顯示，然後在 [新的電子郵件送達時] 動作下方選取 [內容]。 |
   |||| 

   完成上述步驟後，動作會如下列範例所示︰

   ![已完成的「建立 Blob」動作](./media/tutorial-process-email-attachments-workflow/create-blob-per-attachment-done.png)

5. 儲存您的邏輯應用程式。 

### <a name="check-attachment-handling"></a>檢查附件處理情形

接著，請測試邏輯應用程式是否依照您指定的方式處理附件：

1. 如果您的邏輯應用程式尚未執行，請在設計工具工具列上選擇 [執行]。

2. 將符合此準則的電子郵件傳送給自己：

   * 您的電子郵件主旨具有您在觸發程序的 [主旨篩選條件] 中指定的文字：```Business Analyst 2 #423501```

   * 您的電子郵件至少有兩個附件。 
   現在，請建立兩個空的文字檔，並將這些檔案附加到您的電子郵件。

   如果邏輯應用程式未觸發，或雖然成功觸發但未執行，請參閱[對邏輯應用程式進行疑難排解](../logic-apps/logic-apps-diagnosing-failures.md)。

3. 確認邏輯應用程式已將電子郵件和附件儲存至正確的儲存體容器。 

   1. 在儲存體總管中，依序展開 **[(Local and Attached)]\(本機與已連結的資源\)** > 
    **[儲存體帳戶]** > **[attachmentstorageacct (外部)]** > 
    **[Blob 容器]** > **[附件]** 。

   2. 檢查**附件**容器中是否有電子郵件和附件。

      ![檢查是否有已儲存的電子郵件和附件](./media/tutorial-process-email-attachments-workflow/storage-explorer-saved-attachments.png)

   3. 完成作業後，請刪除儲存體總管中的電子郵件和附件。

接下來，請新增讓邏輯應用程式傳送電子郵件以檢閱附件的動作。

## <a name="send-email-notifications"></a>傳送電子郵件通知

1. 在 [若為 true] 分支中的 [針對每個電子郵件附件] 迴圈下，選擇 [新增動作]。 

   ![在 "for each" 迴圈下新增動作](./media/tutorial-process-email-attachments-workflow/add-action-send-email.png)

2. 在搜尋方塊中輸入「傳送電子郵件」作為篩選條件，然後為您的電子郵件提供者選取 [傳送電子郵件] 動作。 

   若要篩選特定服務的動作清單，您可以先選取連接器。

   ![為您的電子郵件提供者選取「傳送電子郵件」動作](./media/tutorial-process-email-attachments-workflow/add-action-select-send-email.png)

   * 對於 Azure 工作或學校帳戶，選取 Office 365 Outlook。 
   * 對於個人 Microsoft 帳戶，選取 Outlook.com。 

3. 如果系統要求您提供認證，請登入您的電子郵件帳戶，讓 Logic Apps 能夠建立您電子郵件帳戶的連線。

4. 以下列說明為 [傳送電子郵件] 動作重新命名：```Send email for review```

5. 提供此動作的資訊，然後選取要包含在電子郵件中的欄位，說明如下。 若要在編輯方塊中新增空白的行，請按 Shift + Enter。  

   ![傳送電子郵件通知](./media/tutorial-process-email-attachments-workflow/send-email-notification.png)

   如果您在動態內容清單中找不到預期的欄位，請選取 [新的電子郵件送達時] 旁的 [更多資訊]。 

   | 設定 | 值 | 注意 | 
   | ------- | ----- | ----- | 
   | **內文** | ```Please review new applicant:``` <p>```Applicant name: ``` **寄件者** <p>```Application file location: ``` **路徑** <p>```Application email content: ``` **內文** | 電子郵件內文內容。 按一下此方塊內部，輸入範例文字，然後從動態內容清單中選取下列欄位： <p>- [新的電子郵件送達時] 下的 [寄件者] 欄位 </br>- [為電子郵件內文建立 Blob] 下的 [路徑] 欄位 </br>- [呼叫 RemoveHTMLFunction 以清除電子郵件內文] 下的 [內文] 欄位 | 
   | **主旨**  | ```ASAP - Review applicant for position: ``` **主旨** | 您要包含的電子郵件主旨。 按一下此方塊內部，輸入範例文字，然後選取 [新的電子郵件送達時] 下方的 [主旨] 欄位。 | 
   | **To** | <*recipient-email-address*> | 為了測試用途，您可以使用自己的電子郵件地址。 | 
   |||| 

   > [!NOTE] 
   > 如果您選取其中包含陣列的欄位 (例如 [內容] 欄位，這是包含附件的陣列)，設計工具會自動為參考該欄位的動作新增 "For each" 迴圈。 如此一來，應用程式邏輯可以在每個陣列項目上執行該動作。 若要移除迴圈，請移除陣列的欄位、將參考動作移至迴圈外、選擇迴圈標題列上的省略符號 (**...**)，然後選擇 [刪除]。
     
6. 儲存您的邏輯應用程式。 

現在，請測試您的邏輯應用程式，此時它看起來會如同下列範例：

![完成的邏輯應用程式](./media/tutorial-process-email-attachments-workflow/complete.png)

## <a name="run-your-logic-app"></a>執行邏輯應用程式

1. 將符合此準則的電子郵件傳送給自己：

   * 您的電子郵件主旨具有您在觸發程序的 [主旨篩選條件] 中指定的文字：```Business Analyst 2 #423501```

   * 您的電子郵件有一或多個附件。 
   您可以重複使用先前測試使用的空文字檔。 
   為了讓案例更為真實，請附加履歷表檔案。

   * 電子郵件內文包含以下文字，您可以複製並貼上：

     ```
     Name: Jamal Hartnett   
     
     Street address: 12345 Anywhere Road   
     
     City: Any Town   
     
     State or Country: Any State   
     
     Postal code: 00000   
     
     Email address: jamhartnett@outlook.com   
     
     Phone number: 000-000-0000   
     
     Position: Business Analyst 2 #423501   

     Technical skills: Dynamics CRM, MySQL, Microsoft SQL Server, JavaScript, Perl, Power BI, Tableau, Microsoft Office: Excel, Visio, Word, PowerPoint, SharePoint, and Outlook   

     Professional skills: Data, process, workflow, statistics, risk analysis, modeling; technical writing, expert communicator and presenter, logical and analytical thinker, team builder, mediator, negotiator, self-starter, self-managing  
     
     Certifications: Six Sigma Green Belt, Lean Project Management   
     
     Language skills: English, Mandarin, Spanish   
     
     Education: Master of Business Administration   
     ```

2. 執行邏輯應用程式。 如果成功，邏輯應用程式會傳送電子郵件給您，如下列範例所示：

   ![邏輯應用程式傳送的電子郵件通知](./media/tutorial-process-email-attachments-workflow/email-notification.png)

   若未收到任何電子郵件，請檢查電子郵件的垃圾郵件資料夾。 
   您的垃圾電子郵件篩選器可能會重新導向這類郵件。 
   或者，如果您不確定邏輯應用程式是否正確執行，請參閱[針對邏輯應用程式進行疑難排解](../logic-apps/logic-apps-diagnosing-failures.md)。

恭喜，您現在已建立並執行可在不同的 Azure 服務間自動執行工作並呼叫自訂程式碼的邏輯應用程式。

## <a name="clean-up-resources"></a>清除資源

如果不再需要，請刪除包含邏輯應用程式的資源群組和相關資源。 在主要 Azure 功能表上，移至 [資源群組]，然後選取邏輯應用程式的資源群組。 選擇 [刪除資源群組]。 輸入資源群組名稱作為確認，然後選擇 [刪除]。

![刪除邏輯應用程式資源群組](./media/tutorial-process-email-attachments-workflow/delete-resource-group.png)

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您藉由 Azure 服務 (例如 Azure 儲存體和 Azure Functions) 的整合，建立了可處理及儲存電子郵件附件的邏輯應用程式。 現在，請深入了解可用來建置邏輯應用程式的其他連接器。

> [!div class="nextstepaction"]
> [深入了解 Logic Apps 的連接器](../connectors/apis-list.md)
