---
title: 連線到 Office 365 Outlook - Azure Logic Apps | Microsoft Docs
description: 使用 Office 365 REST API 和 Azure Logic Apps 管理電子郵件、連絡人和行事曆
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
ms.openlocfilehash: 0355f121a09e1ba89f98a8af5037eb1371db2242
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39215633"
---
# <a name="get-started-with-the-office-365-outlook-connector"></a>開始使用 Office 365 Outlook 連接器
Office 365 Outlook 連接器能夠與 Office 365 中的 Outlook 互動。 使用此連接器來建立、編輯和更新連絡人和行事曆項目，也可取得、傳送及回覆電子郵件。

使用 Office 365 Outlook，您可以：

* 使用 Office 365 中的電子郵件和行事曆功能來建置您的工作流程。 
* 使用觸發程序，在有新的電子郵件時、行事曆項目更新時等情況啟動您的工作流程。
* 使用傳送電子郵件、建立新的行事曆事件等等的動作。 例如，當 Salesforce 中有新的物件時，傳送電子郵件給您的 Office 365 Outlook (動作)。 

本文說明如何在邏輯應用程式中使用 Office 365 Outlook 連接器，並且也列出觸發程序和動作。

> [!NOTE]
> 這個版本的文章適用於 Logic Apps 公開上市版本 (GA)。
> 
> 

若要深入瞭解 Logic Apps，請參閱[什麼是邏輯應用程式](../logic-apps/logic-apps-overview.md)以及[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="connect-to-office-365"></a>連接至 Office 365
您必須先建立與服務的「連線」，才能透過邏輯應用程式存取任何服務。 連線可讓邏輯應用程式與另一個服務連線。 例如，若要連線到 Office 365 Outlook，您必須先有 Office 365「連線」。 若要建立連線，請輸入平常用來存取所要連線之服務的認證。 因此，在 Office 365 Outlook 中，請在 Office 365 帳戶輸入認證以建立連線。

## <a name="create-the-connection"></a>建立連線
> [!INCLUDE [Steps to create a connection to Office 365](../../includes/connectors-create-api-office365-outlook.md)]
> 
> 

## <a name="use-a-trigger"></a>使用觸發程序
觸發程序是可用來啟動邏輯應用程式中所定義之工作流程的事件。 觸發程序會以您想要的間隔和頻率「輪詢」服務。 [深入了解觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)。

1. 在邏輯應用程式中，輸入 "office 365" 以取得觸發程序的清單︰  
   
    ![](./media/connectors-create-api-office365-outlook/office365-trigger.png)
2. 選取 [Office 365 Outlook - 即將來臨的事件快要開始時]。 如果連線已存在，則選取下拉式清單中的行事曆。
   
    ![](./media/connectors-create-api-office365-outlook/sample-calendar.png)
   
    如果系統提示您登入，則輸入登入詳細資料來建立連線。 本主題中的[建立連線](connectors-create-api-office365-outlook.md#create-the-connection)一節會列出步驟。 
   
   > [!NOTE]
   > 在此範例中，邏輯應用程式會在行事曆事件更新時執行。 若要查看此觸發程序的結果，請新增另一個動作，以傳送簡訊給您。 例如，加入 Twilio「傳送訊息」動作，以便在行事曆事件於 15 分鐘內開始時傳送簡訊給您。 
   > 
   > 
3. 選取 [編輯] 按鈕，然後設定 [頻率] 和 [間隔] 值。 例如，如果您希望觸發程序每隔 15 分鐘輪詢一次，則將 [頻率] 設定為 [分鐘] 並將 [間隔] 設定為 [15]。 
   
    ![](./media/connectors-create-api-office365-outlook/calendar-settings.png)
4. **儲存**您的變更 (工具列的左上角)。 邏輯應用程式將會儲存，而且可能會自動啟用。

## <a name="use-an-action"></a>使用動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。 [深入了解動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)。

1. 選取加號。 您會看到幾個選擇︰[新增動作]、[新增條件] 或其中一個 [其他] 選項。
   
    ![](./media/connectors-create-api-office365-outlook/add-action.png)
2. 選擇 [新增動作] 。
3. 在文字方塊中，輸入 “office 365” 以取得所有可用動作的清單。
   
    ![](./media/connectors-create-api-office365-outlook/office365-actions.png) 
4. 在本例中，選擇 [Office 365 Outlook - 建立連絡人]。 如果連線已存在，則選擇 [資料夾識別碼]、[名字] 和其他屬性︰  
   
    ![](./media/connectors-create-api-office365-outlook/office365-sampleaction.png)
   
    如果系統提示您輸入連線資訊，請輸入詳細資料以建立連線。 本主題的[建立連線](connectors-create-api-office365-outlook.md#create-the-connection)一節會說明這些屬性。 
   
   > [!NOTE]
   > 在此範例中，我們會在 Office 365 Outlook 中建立新連絡人。 您可以使用另一個觸發程序的輸出來建立連絡人。 例如，新增 SalesForce「當物件建立時」觸發程序。 然後新增 Office 365 Outlook「建立連絡人」動作，以使用 SalesForce 欄位在 Office 365 中建立新的連絡人。 
   > 
   > 
5. **儲存**您的變更 (工具列的左上角)。 邏輯應用程式將會儲存，而且可能會自動啟用。

## <a name="connector-specific-details"></a>連接器特定的詳細資料

檢視 Swagger 中定義的任何觸發程序和動作，另請參閱[連接器詳細資料](/connectors/office365connector/)的所有限制。 

## <a name="next-steps"></a>後續步驟
[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 請到我們的 [API 清單](apis-list.md)探索 Logic Apps 中其他可用的連接器。

