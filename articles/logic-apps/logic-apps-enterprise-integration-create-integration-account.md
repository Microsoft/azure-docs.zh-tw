---
title: 建立和管理 B2B 解決方案的整合帳戶 - Azure Logic Apps | Microsoft Docs
description: 建立、連結、移動和刪除企業整合及 B2B 解決方案與 Azure Logic Apps 的整合帳戶
services: logic-apps
documentationcenter: ''
author: ecfan
manager: jeconnoc
editor: ''
ms.assetid: d3ad9e99-a9ee-477b-81bf-0881e11e632f
ms.service: logic-apps
ms.workload: logic-apps
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 04/30/2018
ms.author: estfan
ms.openlocfilehash: 2a1fe501386884e02657d4b6cbef58ffc533fa33
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35297983"
---
# <a name="create-and-manage-integration-accounts-for-b2b-solutions-with-logic-apps"></a>建立和管理 B2B 解決方案與 Logic Apps 的整合帳戶

在能夠使用 [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 建置[企業整合及 B2B 解決方案](../logic-apps/logic-apps-enterprise-integration-overview.md)之前，您必須先具備整合帳戶，這是您用來建立、儲存和管理 B2B 成品 (例如交易夥伴、合約、對應、結構描述、憑證等等) 的位置。 您必須[連結整合帳戶](#link-account)到邏輯應用程式，邏輯應用程式才能處理整合帳戶中的成品，並使用 Logic Apps B2B 連接器，例如 XML 驗證。 若要連結它們，整合帳戶和邏輯應用程式都必須具有「相同的」 Azure 位置或區域。

這篇文章會示範如何執行這些工作：

* 建立整合帳戶。
* 將整合帳戶連結至邏輯應用程式。
* 將整合帳戶移到另一個 Azure 資源群組或訂用帳戶。
* 刪除整合帳戶。

如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

使用您的 Azure 帳戶認證登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。

## <a name="create-integration-account"></a>建立整合帳戶

1. 從主要 Azure 功能表中選取 [所有服務]。 在搜尋方塊中輸入「整合帳戶」作為篩選器，然後選取 [整合帳戶]。

   ![尋找整合帳戶](./media/logic-apps-enterprise-integration-create-integration-account/create-integration-account.png)

2. 在 [整合帳戶] 底下選擇 [新增]。

   ![選擇 [新增] 以建立整合帳戶](./media/logic-apps-enterprise-integration-create-integration-account/add-integration-account.png)

3. 提供整合帳戶的相關資訊： 

   ![提供整合帳戶的詳細資料](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-details.png)

   | 屬性 | 必要 | 範例值 | 說明 | 
   |----------|----------|---------------|-------------|
   | Name | yes | test-integration-account | 整合帳戶的名稱。 在此範例中，請使用指定的名稱。 | 
   | 訂用帳戶 | yes | <*Azure-subscription-name*> | 要使用的 Azure 訂用帳戶的名稱 | 
   | 資源群組 | yes | test-integration-account-rg | 用來組織相關資源之 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱。 在此範例中，請使用指定的名稱建立新的資源群組。 | 
   | 定價層 | yes | 免費 | 您想要使用的定價層。 在此範例中，選取 [免費]，但如需詳細資訊，請參閱 [Logic Apps 限制和設定](../logic-apps/logic-apps-limits-and-config.md)和[Logic Apps 價格](https://azure.microsoft.com/pricing/details/logic-apps/)。 | 
   | 位置 | yes | 美國西部 | 用來存放整合帳戶資訊的區域。 請選取與邏輯應用程式相同的位置，或在與整合帳戶相同的位置中建立邏輯應用程式。 | 
   | Log Analytics | 否 | 關閉 | 保留診斷記錄的 [關閉] 設定。 | 
   ||||| 

4. 當您準備就緒，選取 [釘選到儀表板]，然後選擇 [建立]。

   Azure 將您的整合帳戶部署到選取的位置之後 (這通常會在一分鐘內完成)，Azure 就會開啟該整合帳戶。

   ![Azure 會開啟您的整合帳戶](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-created.png)

現在，您必須將整合帳戶連結至邏輯應用程式，然後邏輯應用程式才能使用整合帳戶。

<a name="link-account"></a>

## <a name="link-to-logic-app"></a>連結至邏輯應用程式

若要讓您的邏輯應用程式存取包含 B2B 成品 (例如交易夥伴、合約、地圖及結構描述) 的整合帳戶，請將整合帳戶連結到邏輯應用程式。 

> [!NOTE]
> 您的整合帳戶和邏輯應用程式必須存在於相同的區域。

1. 在 Azure 入口網站中，尋找並開啟邏輯應用程式。

2. 在邏輯應用程式功能表的 [設定] 底下，選取 [工作流程設定]。 在 [選取整合帳戶] 清單中，選取要連結到邏輯應用程式的整合帳戶。

   ![選取您的整合帳戶](./media/logic-apps-enterprise-integration-create-integration-account/linkaccount-2.png)

3. 若要完成連結，請選擇 [儲存]。

   ![選取您的整合帳戶](./media/logic-apps-enterprise-integration-create-integration-account/linkaccount-3.png)

   當您的整合帳戶成功連結時，Azure 就會顯示確認訊息。 

   ![Azure 會確認成功的連結](./media/logic-apps-enterprise-integration-create-integration-account/linkaccount-5.png)

現在，邏輯應用程式可以使用整合帳戶中的任何和所有成品以及 B2B 連接器，例如 XML 驗證和一般檔案編碼或解碼。  

## <a name="unlink-from-logic-app"></a>從邏輯應用程式取消連結

若要將邏輯應用程式連結至另一個整合帳戶，或者不再利用邏輯應用程式使用整合帳戶，您可以透過 Azure 資源總管刪除連結。

1. 在您的瀏覽器中，移至 <a href="https://resources.azure.com" target="_blank">Azure 資源總管 (https://resources.azure.com)</a>。 請確定您已使用相同的 Azure 認證登入。

   ![Azure 資源總管](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer.png)

2. 在搜尋方塊中，輸入邏輯應用程式的名稱，然後尋找並選取邏輯應用程式。

   ![尋找並選取邏輯應用程式](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-find-logic-app.png)

3. 在瀏覽器標題列中，選擇 [讀取/寫入]。

   ![開啟「讀取/寫入」模式](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-choose-read-write-mode.png)

4. 在 [資料] 索引標籤上，選擇 [編輯]。

   ![在 [資料] 索引標籤上選擇 [編輯]](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-choose-edit.png)

5. 在編輯器中，尋找整合帳戶的 `integrationAccount` 屬性，並刪除該屬性，其格式如下：

   ```json
   "integrationAccount": {
      "name": "<integration-account-name>",
      "id": "<integration-account-resource-ID>",
      "type": "Microsoft.Logic/integrationAccounts"  
   },
   ```

   例如︰

   ![尋找 "integrationAccount" 屬性定義](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-delete-integration-account.png)

6. 在 [資料] 索引標籤上，選擇 [Put] 以儲存變更。 

   ![選擇 "Put" 以儲存變更](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-save-changes.png)

7. 在 Azure 入口網站中邏輯應用程式的 [工作流程設定] 底下，確認 [整合帳戶] 屬性現在顯示為空白。

   ![確認整合帳戶未連結](./media/logic-apps-enterprise-integration-create-integration-account/unlinked-account.png)

## <a name="move-integration-account"></a>移動整合帳戶

您可以將整合帳戶移到另一個 Azure 訂用帳戶或資源群組。

1. 在主要 Azure 功能表上，選取 [所有服務]。 在搜尋方塊中輸入「整合帳戶」作為篩選器，然後選取 [整合帳戶]。

   ![尋找您的整合帳戶](./media/logic-apps-enterprise-integration-create-integration-account/create-integration-account.png)

2. 在 [整合帳戶] 底下，選取您想要移動的整合帳戶。 在整合帳戶功能表的 [設定] 底下，選擇 [屬性]。

   ![在 [設定] 底下選擇 [屬性]](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-properties.png)

3. 變更整合帳戶的 Azure 資源群組或訂用帳戶。

   ![選擇 [變更資源群組] 或 [變更訂用帳戶]](./media/logic-apps-enterprise-integration-create-integration-account/change-resource-group-subscription.png)

4. 完成後，請確定您使用成品的新資源識別碼來更新任何和所有的指令碼。  

## <a name="delete-integration-account"></a>刪除整合帳戶

1. 在主要 Azure 功能表上，選取 [所有服務]。 在搜尋方塊中輸入「整合帳戶」作為篩選器，然後選取 [整合帳戶]。

   ![尋找您的整合帳戶](./media/logic-apps-enterprise-integration-create-integration-account/create-integration-account.png)

2. 在 [整合帳戶] 底下，選取您想要刪除的整合帳戶。 在整合帳戶功能表中選擇 [概觀]，然後選擇 [刪除]。 

   ![選取整合帳戶。 在[概觀] 頁面上選擇 [刪除]](./media/logic-apps-enterprise-integration-create-integration-account/delete-integration-account.png)

3. 若要確認您想要刪除整合帳戶，請選擇 [是]。

   ![若要確認刪除，請選擇 [是]](./media/logic-apps-enterprise-integration-create-integration-account/confirm-delete.png)

## <a name="next-steps"></a>後續步驟

* [建立交易夥伴](../logic-apps/logic-apps-enterprise-integration-partners.md)
* [建立合約](../logic-apps/logic-apps-enterprise-integration-agreements.md)
