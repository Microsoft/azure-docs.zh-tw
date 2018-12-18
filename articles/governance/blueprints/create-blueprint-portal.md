---
title: 在入口網站中建立 Azure 藍圖
description: Azure 藍圖可用來建立、定義和部署成品。
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: quickstart
ms.service: blueprints
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: 7af0d4819d4044f4cccc43cde1cffe7dff7982a7
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056432"
---
# <a name="define-and-assign-an-azure-blueprint-in-the-portal"></a>在入口網站中定義和指派 Azure 藍圖

了解如何在 Azure 中建立及指派藍圖，有助於組織定義一致性的常見模式，且能根據 Resource Manager 範本、原則、安全性等，開發出可重複使用並能快速部署的設定。 在本教學課程中，您將了解如何使用 Azure 藍圖在您的組織中處理藍圖的建立、發佈和指派等常見工作，例如：

> [!div class="checklist"]
> - 建立新藍圖並新增各種支援成品
> - 變更仍在**草稿**狀態的現有藍圖
> - 將藍圖標示為**已發佈**，代表藍圖已可供指派
> - 將藍圖指派給現有的訂用帳戶
> - 檢查指派的藍圖的狀態和進度
> - 移除已指派給訂用帳戶的藍圖

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free)。

## <a name="create-a-blueprint"></a>建立藍圖

定義合規性標準模式的第一個步驟，即是以可用的資源規劃藍圖。 在此範例中，會新建立名為「MyBlueprint」的藍圖，用以設定訂用帳戶的角色和原則指派、新增資源群組、在新資源群組上建立 Resource Manager 的範本和角色指派。

1. 在左窗格中按一下 [所有服務]，再搜尋並選取 [原則]，即可在 Azure 入口網站中啟動 Azure 藍圖服務。 請在 [原則] 頁面上，按一下 [藍圖]。

1. 在頁面左邊選取 [藍圖定義]，然後按一下頁面頂端的 [+ 建立藍圖] 按鈕。

   - 或是在 [開始使用] 頁面按一下 [建立]，即可直接建立藍圖。

   ![建立藍圖](./media/create-blueprint-portal/create-blueprint-button.png)

1. 填妥藍圖的 [藍圖名稱]，例如「MyBlueprint」(僅限使用字母和數字，最多 48 個字元，不可加入空格或特殊字元)，但目前請將 [藍圖描述] 留白。  在 [定義位置] 方塊中，按一下右邊的省略符號，選取想要儲存藍圖的[管理群組](../management-groups/overview.md)，並按一下 [選取]。

   > [!NOTE]
   > 藍圖定義只能儲存至管理群組。 若要建立第一個管理群組，請按照[下列步驟](../management-groups/create.md)進行。

1. 確認資訊正確無誤 (之後無法變更 [藍圖名稱] 和 [定義位置] 欄位)，然後按一下頁面底部的 [下一步：成品]或頁面頂端的 [成品] 索引標籤。

1. 新增訂用帳戶的角色指派：以滑鼠左鍵按一下 [訂用帳戶] 底下的 [+ 新增成品...] 資料列，即會在瀏覽器右側開啟 [新增成品] 視窗。 [成品類型] 請選取 [角色指派]。 請選取 [角色] 底下的 [參與者]，然後在 [新增使用者、應用程式或群組] 欄位勾選代表**動態參數**的核取方塊。 按一下 [新增] 將此成品新增至藍圖。

   ![成品 - 角色指派](./media/create-blueprint-portal/add-role-assignment.png)

   > [!NOTE]
   > 大部分_成品_均可支援參數。 在藍圖建立期間，獲得指派值的參數是**靜態參數**。 如果是在藍圖指派期間受指派的參數，則為**動態參數**。 如需詳細資訊，請參閱[藍圖參數](./concepts/parameters.md)。

1. 新增訂用帳戶的原則指派：以滑鼠左鍵按一下 [訂用帳戶] 底下的 [+ 新增成品...] 資料列。 [成品類型] 請選取 [原則指派]。 將 [類型] 變更為 [內建]，然後在 [搜尋] 中輸入「標籤」。 按一下 [搜尋] 進行篩選。 按一下 [將標籤及其預設值套用至資源群組]，加以選取。 按一下 [新增] 將此成品新增至藍圖。

1. 按一下 [將標籤及其預設值套用至資源群組] 原則指派的資料列。 在藍圖定義中提供成品參數的視窗隨即開啟，可用來根據此藍圖設定所有指派的參數 (**靜態參數**)，而不需要在指派期間設調 (**動態參數**)。 例如，想要在藍圖指派其間使用**動態參數**，可以保留預設值，並按一下 [取消]。

1. 在訂用帳戶新增資源群組：以滑鼠左鍵按一下 [訂用帳戶] 底下的 [+ 新增成品...] 資料列。 [成品類型] 請選取 [資源群組]。 將 [資源群組名稱] 和 [位置] 欄位留白，但是請務必勾選每個屬性的核取方塊，將其設定為**動態參數**。 按一下 [新增] 將此成品新增至藍圖。

1. 在資源群組下新增範本：以滑鼠左鍵按一下 [ResourceGroup] 項目底下的 [+ 新增成品...]  資料列。 [成品類型] 請選擇「Azure Resource Manager 範本 」，並將 [成品顯示名稱] 設為「StorageAccount」，將 [描述] 留白。 在編輯器方塊的 [範本] 索引標籤中，貼上以下的 Resource Manager 範本。 貼上範本之後，按一下 [參數] 索引標籤，並注意是否有自動偵測到範本參數 **storageAccountType** 並填入預設值 **Standard_LRS**，但設定為**動態參數**。 取消勾選核取方塊，並注意下拉式清單是否僅有 **allowedValues** 底下的 Resource Manager 範本所含的值。 勾選方塊，將其設回**動態參數**。 按一下 [新增] 將此成品新增至藍圖。

   > [!IMPORTANT]
   > 如果要匯入範本，請確定檔案僅有 JSON，不含 HTML。 指向 GitHub 上的 URL 時，請確定您是按下 [RAW] 取得單純的 JSON 檔案，而不是以 HTML 包裝用於在 GitHub 上顯示的檔案。 如果匯入的範本並非單純的 JSON，會發生錯誤。

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
           "storageAccountType": {
               "type": "string",
               "defaultValue": "Standard_LRS",
               "allowedValues": [
                   "Standard_LRS",
                   "Standard_GRS",
                   "Standard_ZRS",
                   "Premium_LRS"
               ],
               "metadata": {
                   "description": "Storage Account type"
               }
           }
       },
       "variables": {
           "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
       },
       "resources": [{
           "type": "Microsoft.Storage/storageAccounts",
           "name": "[variables('storageAccountName')]",
           "apiVersion": "2016-01-01",
           "location": "[resourceGroup().location]",
           "sku": {
               "name": "[parameters('storageAccountType')]"
           },
           "kind": "Storage",
           "properties": {}
       }],
       "outputs": {
           "storageAccountName": {
               "type": "string",
               "value": "[variables('storageAccountName')]"
           }
       }
   }
   ```

   ![成品 - Resource Manager 範本](./media/create-blueprint-portal/add-resource-manager-template.png)

1. 完成的藍圖應該如下所示： 請注意，每個成品的 [參數] 資料行都會出現「已填入 _x_ 個參數，共 _y_ 個」。 每次指派藍圖時，均會設定**動態參數**，且角色指派的單一**靜態參數**已先設定完畢。

   ![完成的藍圖](./media/create-blueprint-portal/completed-blueprint.png)

1. 加入所有計劃中的成品之後，按一下頁面底部的 [儲存草稿]。

## <a name="edit-a-blueprint"></a>編輯藍圖

在[建立藍圖](#create-a-blueprint) 時，並未提供描述，也未在新的資源群組內新增角色指派。 執行下列步驟可修正這兩者：

1. 在左邊的頁面選取 [藍圖定義]。

1. 在藍圖清單中，以滑鼠右鍵按一下您先前建立的藍圖，然後選取 [編輯藍圖]。

1. 在 [藍圖描述] 中，提供藍圖及其構成成品的相關資訊。  在此例中，請輸入類似如下的內容：「此藍圖設定的是訂用帳戶的標記原則和角色指派，且會建立資源群組，並將資源範本和角色指派部署到該資源群組。」

1. 按一下頁面底端的 [下一步：成品] 或頁面頂端的 [成品]。

1. 在資源群組底下新增角色指派：以滑鼠左鍵按一下 [資源群組] 項目底下的 [+ 新增成品...] 資料列。 [成品類型] 請選取 [角色指派]。 在 [角色] 底下，選取 [擁有者]，並取消勾選 [新增使用者、應用程式或群組] 欄位，然後搜尋並選取所要新增的使用者、應用程式或群組。 這會是**靜態參數**，每次指派此藍圖時均會使用。 按一下 [新增] 將此成品新增至藍圖。

   ![成品 - 角色指派 #2](./media/create-blueprint-portal/add-role-assignment-2.png)

1. 完成的藍圖應該如下所示： 請注意，新增的角色指派會顯示**已填入 1 個參數，共 1 個**，表示這是**靜態參數**。

   ![完成的藍圖 #2](./media/create-blueprint-portal/completed-blueprint-2.png)

1. 更新之後，按一下 [儲存草稿]。

## <a name="publish-a-blueprint"></a>發佈藍圖

將所有規劃好的成品新增到藍圖之後，即可發佈藍圖。
發佈後即可將藍圖指派給訂用帳戶。

1. 在左邊的頁面選取 [藍圖定義]。

1. 在藍圖清單中，以滑鼠右鍵按一下您先前建立的藍圖，然後選取 [發佈藍圖]。

1. 在開啟的對話方塊，填妥**版本** (僅限字母、數字和連字號，長度上限為 20 個字元)，例如「v1」，並**變更附註** (選擇性)，例如「首次發佈」。

1. 按一下頁面底端的 [發佈]。

## <a name="assign-a-blueprint"></a>指派藍圖

發佈藍圖後，即可將藍圖指派給訂用帳戶。 將您建立的藍圖指派給管理群組階層的其中一個訂用帳戶。

1. 在左邊的頁面選取 [藍圖定義]。

1. 在藍圖清單中，以滑鼠右鍵按一下您先前建立的藍圖 (或以滑鼠左鍵按一下省略符號)，然後選取 [指派藍圖]。

1. 在 [指派藍圖] 頁面上，從 [訂用帳戶] 下拉式清單選取要部署此藍圖的訂用帳戶。

   > [!NOTE]
   > 每個選取的每個訂用帳戶均會建立指派，如此可在稍後變更單一訂用帳戶的指派，而不需強制變更其餘選取的訂用帳戶。

1. [指派的名稱] 請填入此次指派的專屬名稱。

1. 在 [位置] 中，選取將建立受控識別的區域。 Azure 藍圖會使用此受控識別來部署指派的藍圖之中的所有成品。 若要深入了解，請參閱[適用於 Azure 資源的受控識別](../../active-directory/managed-identities-azure-resources/overview.md)。

1. 請將「v1」項目上**已發佈**版本的 [藍圖定義版本] 下拉式清單保留原樣 (預設為最新的**已發佈**版本)。

1. [鎖定指派] 請保留 [不要鎖定] 的預設值。 如需詳細資訊，請參閱[藍圖資源鎖定](./concepts/resource-locking.md)。

1. 對於訂用帳戶等級的角色指派「**[使用者群組或應用程式名稱]: 參與者**」，請搜尋並選取使用者、應用程式或群組。

1. 訂用帳戶等級的原則指派，則請將 [標記名稱] 設為「CostCenter」，並將 [標記值] 設為「ContosoIT」。

1. 對於「Resourcegroup」，請填妥「StorageAccount」的 [名稱]，並在 [位置] 的下拉式清單中選擇 [美國東部 2]。

   > [!NOTE]
   > 藍圖定義期間在資源群組下新增的每個成品，均是為了配合其將部署的資源群組或物件所建立的。 不接受參數的成品，或是在指派時並未定義任何參數的成品，僅會列出內容資訊。

1. 在 Azure Resource Manager 範本「StorageAccount」中，**storageAccountType** 參數請選擇「Standard_GRS」。

1. 請閱讀頁面底端的 [資訊] 方塊，然後按一下 [指派]。

## <a name="track-deployment-of-a-blueprint"></a>追蹤藍圖的部署情況

將藍圖指派到一個或多個訂用帳戶時，會發生兩件事：

- 每個指派到藍圖的訂用帳戶，會有藍圖新增至其 [指派藍圖]
- 藍圖定義的所有成品開始進行部署

請在藍圖被指派到訂用帳戶之後，確認部署的進度。

1. 從左邊的頁面選取 [指派藍圖]。

1. 在藍圖的清單中，以滑鼠右鍵按一下您先前指派的藍圖，然後選取 [檢視指派詳細資料]。

   ![檢視指派詳細資料](./media/create-blueprint-portal/view-assignment-details.png)

1. 請在 [部署詳細資料] 頁面上，確認已成功部署所有成品，且部署期間並未出現任何錯誤。 如果發生錯誤，請參閱[疑難排解藍圖](./troubleshoot/general.md)的步驟，判斷發生錯誤的原因。

## <a name="unassign-a-blueprint"></a>取消指派藍圖

如果不再需要藍圖，或因為模式、原則、設計有所更新，而已經用較新的藍圖取代舊藍圖，可以將藍圖從訂用帳戶中移除。 移除藍圖時，會將指派為該藍圖一部份的成品保留下來。 若要移除藍圖指派，請按照下列步驟操作：

1. 從左邊的頁面選取 [指派藍圖]。

1. 在藍圖的清單中，選取將取消指派的藍圖，然後按一下頁面頂端的 [取消指派藍圖] 按鈕。

1. 閱讀確認訊息，然後按一下 [確定]。

## <a name="delete-a-blueprint"></a>刪除藍圖

1. 在左邊的頁面選取 [藍圖定義]。

1. 以滑鼠右鍵按一下要刪除的藍圖，並選取 [刪除藍圖]，然後按一下確認對話方塊中的 [是]。

> [!NOTE]
> 以此方法刪除藍圖，也會將所選藍圖的**已發佈版本**全數刪除。 若要刪除單一版本，請開啟藍圖，按一下 [已發佈版本] 索引標籤，選取並按一下要刪除的版本，然後按一下 [刪除此版本]。 此外，需先刪除藍圖的所有指派項目，否則無法刪除具有指派項目的藍圖。

## <a name="next-steps"></a>後續步驟

- 深入了解[藍圖生命週期](./concepts/lifecycle.md) (英文)
- 了解如何使用[靜態和動態參數](./concepts/parameters.md) (英文)
- 了解如何自訂[藍圖排序順序](./concepts/sequencing-order.md) (英文)
- 了解如何使用[藍圖資源鎖定](./concepts/resource-locking.md) (英文)
- 瞭解如何[更新現有的指派](./how-to/update-existing-assignments.md) (英文)
- 使用[一般疑難排解](./troubleshoot/general.md)來解決藍圖指派期間發生的問題