---
title: 連線到 OneDrive - Azure Logic Apps | Microsoft Docs
description: 使用 OneDrive REST API 和 Azure Logic Apps上傳及管理檔案
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 10/18/2016
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 50bd9ecdd665cf72c146c63ae25efa6773934a3e
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35295375"
---
# <a name="get-started-with-the-onedrive-connector"></a>開始使用 OneDrive 連接器
連線到 OneDrive 來管理您的檔案，包括上傳檔案、取得、刪除檔案等等。 

使用 OneDrive，您會︰ 

* 將檔案儲存在 OneDrive 中或更新 OneDrive 中的現有檔案以建置您的工作流程。 
* 使用觸發程序，在 OneDrive 內有檔案建立或更新時，啟動工作流程。
* 使用動作來建立檔案、刪除檔案等等。 例如，當收到有附件的新 Office 365 電子郵件時 (觸發程序)，在 OneDrive 建立新的檔案 (動作)。

本文說明如何在邏輯應用程式中使用 OneDrive 連接器，並且也列出觸發程序和動作。

若要深入瞭解 Logic Apps，請參閱[什麼是邏輯應用程式](../logic-apps/logic-apps-overview.md)以及[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="connect-to-onedrive"></a>連線到 OneDrive
您必須先建立與服務的「連線」，才能透過邏輯應用程式存取任何服務。 連線可讓邏輯應用程式與另一個服務連線。 例如，若要連線到 OneDrive，您必須先有 OneDrive「連線」。 若要建立連線，請輸入平常用來存取所要連線之服務的認證。 因此，在 OneDrive 中，請在 OneDrive 帳戶輸入認證以建立連線。

### <a name="create-the-connection"></a>建立連線
> [!INCLUDE [Steps to create a connection to OneDrive](../../includes/connectors-create-api-onedrive.md)]
> 
> 

## <a name="use-a-trigger"></a>使用觸發程序
觸發程序是可用來啟動邏輯應用程式中所定義之工作流程的事件。 觸發程序會以您想要的間隔和頻率「輪詢」服務。 [深入了解觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)。

1. 在邏輯應用程式中，輸入「onedrive」以取得觸發程序的清單︰  
   
    ![](./media/connectors-create-api-onedrive/onedrive-1.png)
2. 選取 [當檔案遭到修改時]。 如果連線已存在，則選取 [顯示選擇器] 按鈕以選取資料夾。
   
    ![](./media/connectors-create-api-onedrive/sample-folder.png)
   
    如果系統提示您登入，則輸入登入詳細資料來建立連線。 本文中的[建立連線](connectors-create-api-onedrive.md#create-the-connection)一節會列出步驟。 
   
   > [!NOTE]
   > 在此範例中，當所選資料夾中的檔案更新時，邏輯應用程式便會執行。 若要查看此觸發程序的結果，請新增另一個動作，以傳送電子郵件給您。 例如，新增 Office 365 Outlook「傳送電子郵件」動作，以在檔案更新時傳送電子郵件給您。 

3. 選取 [編輯] 按鈕，然後設定 [頻率] 和 [間隔] 值。 例如，如果您希望觸發程序每隔 15 分鐘輪詢一次，則將 [頻率] 設定為 [分鐘] 並將 [間隔] 設定為 [15]。 
   
    ![](./media/connectors-create-api-onedrive/trigger-properties.png)
4. **儲存**您的變更 (工具列的左上角)。 邏輯應用程式將會儲存，而且可能會自動啟用。

## <a name="use-an-action"></a>使用動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。 [深入了解動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)。

1. 選取加號。 您會看到幾個選擇︰[新增動作]、[新增條件] 或其中一個 [其他] 選項。
   
    ![](./media/connectors-create-api-onedrive/add-action.png)
2. 選擇 [新增動作] 。
3. 在文字方塊中，輸入「onedrive」以取得所有可用動作的清單。
   
    ![](./media/connectors-create-api-onedrive/onedrive-actions.png) 
4. 在我們的範例中，選擇 [OneDrive - 建立檔案]。 如果連線已存在，則選取 [資料夾路徑] 以供放置檔案、輸入 [檔案名稱]，然後選擇想要的 [檔案內容]︰  
   
    ![](./media/connectors-create-api-onedrive/sample-action.png)
   
    如果系統提示您輸入連線資訊，請輸入詳細資料以建立連線。 本文的[建立連線](connectors-create-api-onedrive.md#create-the-connection)一節會說明這些屬性。 
   
   > [!NOTE]
   > 在此範例中，我們會在 OneDrive 資料夾建立新檔案。 您可以使用另一個觸發程序的輸出以建立 OneDrive 檔案。 例如，新增 Office 365 Outlook「新的電子郵件送達時」觸發程序。 然後新增 OneDrive「建立檔案」動作，使用 ForEach 內的 [附件] 和 [內容類型] 欄位在 OneDrive 中建立新檔案。 
   > 
   > ![](./media/connectors-create-api-onedrive/foreach-action.png)

5. **儲存**您的變更 (工具列的左上角)。 邏輯應用程式將會儲存，而且可能會自動啟用。


## <a name="connector-specific-details"></a>連接器特定的詳細資料

檢視 Swagger 中定義的任何觸發程序和動作，另請參閱[連接器詳細資料](/connectors/onedriveconnector/)的所有限制。

## <a name="more-connectors"></a>其他連接器
返回 [API 清單](apis-list.md)。