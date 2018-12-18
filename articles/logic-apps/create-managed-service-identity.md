---
title: 存取並驗證而不登入 - Azure Logic Apps | Microsoft Docs
description: 建立受控識別，讓您的邏輯應用程式不需要您的認證，就能驗證並存取其他 Azure Active Directory (Azure AD) 租用戶中的資源
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, LADocs
services: logic-apps
ms.service: logic-apps
ms.suite: integration
ms.topic: article
ms.date: 09/24/2018
ms.openlocfilehash: fb1c31e6e7c075e20191a4e51d7b1a9323f3b979
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973961"
---
# <a name="access-resources-and-authenticate-as-managed-identities-in-azure-logic-apps"></a>存取資源並在 Azure Logic Apps 中以受控識別的方式驗證

若要存取其他 Azure Active Directory (Azure AD) 租用戶中的資源並驗證您的身分識別而不登入，您可以建立邏輯應用程式使用的[受控識別](../active-directory/managed-identities-azure-resources/overview.md)而不使用您的認證。 Azure 會為您管理此身分識別，並協助保護您的認證，因為您不需要提供或輪替使用祕密。 此文章說明如何為您的邏輯應用程式建立並使用受控識別。 如需詳細資訊，請參閱[管理 Azure 資源的身分識別](../app-service/app-service-managed-service-identity.md)。

> [!NOTE]
> 先前稱為「受控服務識別」(MSI) 的服務，其新名稱為「適用於 Azure 資源的受控識別」。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶，或如果沒有訂用帳戶，您可以<a href="https://azure.microsoft.com/free/" target="_blank">免費註冊 Azure 帳戶</a>。

* 您要在其中使用受控識別的邏輯應用程式。 如果您沒有邏輯應用程式，請參閱[建立第一個邏輯應用程式工作流程](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

<a name="create-identity"></a>

## <a name="create-managed-identity"></a>建立受控識別

您可以透過 Azure 入口網站、Azure Resource Manager 範本或 Azure PowerShell 來為您的邏輯應用程式建立或啟用受控識別。 

### <a name="azure-portal"></a>Azure 入口網站

若要透過 Azure 入口網站為您的邏輯應用程式建立受控識別，請在邏輯應用程式的工作流程設定中開啟 [向 Azure Active Directory 註冊] 設定。

1. 在 [Azure 入口網站](https://portal.azure.com)的邏輯應用程式設計工具中，開啟邏輯應用程式。

1. 請遵循下列步驟： 

   1. 在邏輯應用程式功能表的 [設定] 下，選取 [工作流程設定]。 

   1. 在 [受控服務識別]**** > 
   [向 Azure Active Directory 註冊]**** 下，選擇 [開啟]****。

   1. 當您完成時，在工具列上選擇 [儲存]。

      ![開啟受控識別設定](./media/create-managed-service-identity/turn-on-managed-service-identity.png)

      Azure 現在針對您邏輯應用程式的受控識別顯示這些屬性和值：

      ![主體識別碼與租用戶識別碼的 GUID](./media/create-managed-service-identity/principal-tenant-id.png)

      | 屬性 | 值 | 說明 | 
      |----------|-------|-------------| 
      | **主體識別碼** | <*principal-ID-GUID*> | 代表 Azure AD 租用戶中之邏輯應用程式的全域唯一識別碼 (GUID) | 
      | **租用戶識別碼** | <*Azure-AD-tenant--ID-GUID*> | 代表您的邏輯應用程式已是其成員之 Azure AD 租用戶的全域唯一識別碼 (GUID)。 在 Azure AD 租用戶中，服務主體會有與邏輯應用程式執行個體相同的名稱。 | 
      ||| 

### <a name="deployment-template"></a>部署範本

若要將建立並部署邏輯應用程式等資源的作業自動化，您可以設定 Azure Resource Manager 範本。 如需詳細資訊，請參閱[使用 Azure Resource Manager 範本建立並部署邏輯應用程式](../logic-apps/logic-apps-create-deploy-azure-resource-manager-templates.md)。 

若要透過範本為您的邏輯應用程式建立受控識別，請在部署範本中將 **identity** 元素與 **type** 屬性新增到您的邏輯應用程式工作流程定義。 設些設定表示 Azure 會為您建立並管理這些身分識別：

```json
"identity": {
    "type": "SystemAssigned"
}
```

例如，您的邏輯應用程式可能看起來像這個版本：

```json
{
   "apiVersion": "2016-06-01", 
   "type": "Microsoft.logic/workflows", 
   "name": "[variables('logicappName')]", 
   "location": "[resourceGroup().location]", 
   "identity": { 
      "type": "SystemAssigned" 
   }, 
   "properties": { 
      "definition": { 
         "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#", 
          "actions": {}, 
          "parameters": {}, 
          "triggers": {}, 
          "contentVersion": "1.0.0.0", 
          "outputs": {} 
     }, 
     "parameters": {}, 
     "dependsOn": [] 
}
```

當 Azure 建立您的邏輯應用程式時，該邏輯應用程式的工作流程定義會包含這些額外的屬性：

```json
"identity": {
    "type": "SystemAssigned",
    "principalId": "<principal-ID-GUID>",
    "tenantId": "<Azure-AD-tenant-ID>-GUID"
}
```

| 屬性 | 值 | 說明 | 
|----------|-------|-------------|
| **principalId** | <*principal-ID-GUID*> | 代表 Azure AD 租用戶中之邏輯應用程式的全域唯一識別碼 (GUID) | 
| tenantId | <*Azure-AD-tenant--ID-GUID*> | 代表邏輯應用程式現在已是其成員之 Azure AD 租用戶的全域唯一識別碼 (GUID)。 在 Azure AD 租用戶中，服務主體會有與邏輯應用程式執行個體相同的名稱。 | 
||| 

<a name="access-other-resources"></a>

## <a name="access-resources-with-managed-identity"></a>使用受控識別存取資源

當您為您的邏輯應用程式建立受控識別之後，您可以[將該身分識別存取提供給其他資源](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md)。 您可以將該受控識別用於驗證，就像其他[服務主體](../active-directory/develop/app-objects-and-service-principals.md)一樣。 

例如，假設您已經設定一個有受控識別與其他資源之存取權的邏輯應用程式。 您現在可以新增 HTTP 動作，讓您的邏輯應用程式可以傳送 HTTP 要求或呼叫該資源。 

1. 在您的邏輯應用程式中，新增 **HTTP** 動作。 

1. 提供該動作的必要詳細資料，例如要求**方法**與您要呼叫之資源的 **URI** 位置。

1. 在 HTTP 動作中，選擇 [顯示進階選項]。 

1. 從 [驗證] 清單中，選取 [受控服務識別]，然後它會顯示供您設定的 [受眾] 屬性：

   ![選取 [受控服務識別]](./media/create-managed-service-identity/select-managed-service-identity.png)

1. 繼續按照您想要的方式建置邏輯應用程式。

<a name="remove-identity"></a>

## <a name="remove-managed-identity"></a>移除受控識別

若要在您的邏輯應用程式上停用受控識別，您可以依照透過 Azure 入口網站、Azure Resource Manager 部署範本或 Azure PowerShell 建立身分識別的類似步驟來停用。 

當您刪除邏輯應用程式時，Azure 會自動將您邏輯應用程式的系統指派身分識別從 Azure AD 移除。

### <a name="azure-portal"></a>Azure 入口網站

1. 在邏輯應用程式設計工具中，開啟您的邏輯應用程式。

1. 依照下列步驟執行： 

   1. 在邏輯應用程式功能表的 [設定] 下，選取 [工作流程設定]。 
   
   1. 在 [受控服務識別] 下，針對 [向 Azure Active Directory 註冊] 屬性選擇 [關閉]。

   1. 當您完成時，在工具列上選擇 [儲存]。

      ![關閉受控識別設定](./media/create-managed-service-identity/turn-off-managed-service-identity.png)

### <a name="deployment-template"></a>部署範本

如果您是使用 Azure Resource Manager 部署範本建立邏輯應用程式的受控識別，請將 `"identity"` 元素的 `"type"` 屬性設定為 `"None"`。 這個動作也會將主體識別碼從 Azure AD 刪除。 

```json
"identity": {
    "type": "None"
}
```

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。
