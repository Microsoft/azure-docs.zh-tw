---
title: 建立 Azure Logic Apps 的部署範本 | Microsoft Docs
description: 建立用來部署邏輯應用程式的 Azure Resource Manager 範本
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.assetid: 85928ec6-d7cb-488e-926e-2e5db89508ee
ms.date: 10/18/2016
ms.openlocfilehash: 393543bbb1891e14ed67487aff26a7bda1eebcd5
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304232"
---
# <a name="create-azure-resource-manager-templates-for-deploying-logic-apps"></a>建立用來部署邏輯應用程式的 Azure Resource Manager 範本

建立邏輯應用程式之後，您可能想要將其建立為 Azure Resource Manager 範本。
如此一來，您就可以輕鬆地將邏輯應用程式部署到您可能需要在其中用到它的任何環境或資源群組。
如需 Resource Manager 範本的詳細資訊，請參閱[編寫 Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)和[使用 Azure Resource Manager 範本部署資源](../azure-resource-manager/resource-group-template-deploy.md)。

## <a name="logic-app-deployment-template"></a>邏輯應用程式部署範本

邏輯應用程式有三個基本元件：

* **邏輯應用程式資源**：包含定價方案、位置和工作流程定義等相關資訊。
* **工作流程定義**︰描述邏輯應用程式的工作流程步驟，以及 Logic Apps 引擎應該如何執行工作流程。
您可以在邏輯應用程式的 [程式碼檢視] 視窗中檢視此定義。
在邏輯應用程式資源中，您可以在 `definition` 屬性中找到此定義。
* **連接**：參照到可安全儲存關於任何連接器連接之中繼資料的個別資源，例如連接字串和存取權杖。
在邏輯應用程式資源中，您的邏輯應用程式會參考 `parameters` 區段中的這些資源。

您可以使用 [Azure 資源總管](http://resources.azure.com)等工具，檢視現有邏輯應用程式的這些所有部分。

若要讓邏輯應用程式的範本可與資源群組部署搭配使用，您必須定義資源並視需要參數化。
例如，如果要部署到開發、測試和生產環境，您可能想要在每個環境中使用不同連接字串連至 SQL Database。
或者，您可能想要在不同訂用帳戶或資源群組內部署。  

## <a name="create-a-logic-app-deployment-template"></a>建立邏輯應用程式部署範本

具備有效的邏輯應用程式部署範本的最簡單方法是使用[適用於 Logic Apps 的 Visual Studio 工具](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#prerequisites)。
Visual Studio 工具產生的有效部署範本，可以在任何訂用帳戶或位置。

有一些其他工具可在您建立邏輯應用程式部署範本時提供協助。
您可以手動撰寫，也就是視需要使用此處已討論過的資源來建立參數。
另一個選項是使用 [邏輯應用程式範本建立者](https://github.com/jeffhollan/LogicAppTemplateCreator) PowerShell 模組。 這個開放原始碼模組會先評估邏輯應用程式和它所使用的任何連接，然後以部署所需的參數產生範本資源。
例如，如果您的邏輯應用程式會收到來自 Azure 服務匯流排佇列的訊息並將資料加入至 Azure SQL Database，則此工具會保留所有的協調流程邏輯並將 SQL 和服務匯流排連接字串參數化，如此便可在部署時進行設定。

> [!NOTE]
> 連接必須位於與邏輯應用程式相同的資源群組內。
>
>

### <a name="install-the-logic-app-template-powershell-module"></a>安裝邏輯應用程式範本 PowerShell 模組
最簡單的安裝方式是透過 [PowerShell 資源庫](https://www.powershellgallery.com/packages/LogicAppTemplate/0.1)使用 `Install-Module -Name LogicAppTemplate` 命令。  

您也可以手動安裝 PowerShell 模組：

1. 下載最新版的 [邏輯應用程式範本建立者](https://github.com/jeffhollan/LogicAppTemplateCreator/releases)。  
2. 將此資料夾解壓縮至您的 PowerShell 模組資料夾 (通常是 `%UserProfile%\Documents\WindowsPowerShell\Modules`)。

為了讓模組能使用任何租用戶和訂用帳戶存取權杖，我們建議您搭配 [ARMClient](https://github.com/projectkudu/ARMClient) 命令列工具使用。  這篇[部落格文章 (英文)](http://blog.davidebbo.com/2015/01/azure-resource-manager-client.html) 會更詳細地討論 ARMClient。

### <a name="generate-a-logic-app-template-by-using-powershell"></a>使用 PowerShell 產生邏輯應用程式範本
安裝 PowerShell 之後，您可以使用下列命令來產生範本：

`armclient token $SubscriptionId | Get-LogicAppTemplate -LogicApp MyApp -ResourceGroup MyRG -SubscriptionId $SubscriptionId -Verbose | Out-File C:\template.json`

`$SubscriptionId` 是 Azure 訂用帳戶識別碼。 這行程式碼會先透過 ARMClient 取得存取權杖，然後透過管線將它傳送到 PowerShell 指令碼，接著在 JSON 檔案中建立範本。

## <a name="add-parameters-to-a-logic-app-template"></a>將參數加入至邏輯應用程式範本
建立邏輯應用程式範本之後，您可以繼續加入或修改您可能需要的參數。 例如，如果您的定義包含您打算在單一部署中部署的 Azure 函數或巢狀工作流程的資源識別碼，您可以將更多資源新增至範本，並視需要將識別碼參數化。 相同情況亦適用於您預計要與各資源群組一起部署的自訂 API 或 Swagger 端點的任何參考。

### <a name="add-references-for-dependent-resources-to-visual-studio-deployment-templates"></a>將相依資源的參考新增至 Visual Studio 部署範本

當您想要邏輯應用程式參考相依的資源時，您可以在邏輯應用程式部署範本中使用 [Azure Resource Manager 範本函式](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-functions)。 例如，您可能想要邏輯應用程式參考您要與邏輯應用程式一起部署的 Azure 函式或整合帳戶。 遵循這些有關如何在部署範本中使用參數的指導方針，以便邏輯應用程式設計工具可正確轉譯。 

您可以在這些類型的觸發程序和動作中使用邏輯應用程式參數︰

*   子工作流程
*   函式應用程式
*   APIM 呼叫
*   API 連線執行階段 URL
*   API 連線路徑

您可以使用範本函式，例如 parameters、variables、resourceId、concat 等等。例如，以下是取代 Azure 函式資源識別碼的方式︰

```
"parameters":{
    "functionName": {
        "type":"string",
        "minLength":1,
        "defaultValue":"<FunctionName>"
    }
},
```

以及您想使用參數的位置︰

```
"MyFunction": {
    "type": "Function",
    "inputs": {
        "body":{},
        "function":{
            "id":"[resourceid('Microsoft.Web/sites/functions','functionApp',parameters('functionName'))]"
        }
    },
    "runAfter":{}
}
```
另一個範例是您可以參數化服務匯流排傳送訊息作業：

```
"Send_message": {
    "type": "ApiConnection",
        "inputs": {
            "host": {
                "connection": {
                    "name": "@parameters('$connections')['servicebus']['connectionId']"
                }
            },
            "method": "post",
            "path": "[concat('/@{encodeURIComponent(''', parameters('queueuname'), ''')}/messages')]",
            "body": {
                "ContentData": "@{base64(triggerBody())}"
            },
            "queries": {
                "systemProperties": "None"
            }
        },
        "runAfter": {}
    }
```
> [!NOTE] 
> host.runtimeUrl 是選擇性的，如果存在的話，可以從您的範本中移除。
> 


> [!NOTE] 
> 針對您在使用參數時要運作的邏輯應用程式設計工具，您必須提供預設值，例如︰
> 
> ```
> "parameters": {
>     "IntegrationAccount": {
>     "type":"string",
>     "minLength":1,
>     "defaultValue":"/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Logic/integrationAccounts/<integrationAccountName>"
>     }
> },
> ```

## <a name="add-your-logic-app-to-an-existing-resource-group-project"></a>將邏輯應用程式新增到現有的資源群組專案

如果您有現有的資源群組專案，可以在 [JSON 大綱] 視窗中將邏輯應用程式新增至該專案。 您也可以您先前建立的應用程式共同新增另一個邏輯應用程式。

1. 開啟 `<template>.json` 檔案。

2. 若要開啟 [JSON 大綱] 視窗，請前往 [檢視] > [其他視窗] > [JSON 大綱]。

3. 若要將資源新增至範本檔案，按一下 [JSON 大綱] 視窗頂端的 [新增資源]。 或在 [JSON 大綱] 視窗中，以滑鼠右鍵按一下 [資源]，然後選取 [新增資源]。

    ![[JSON 大綱] 視窗](./media/logic-apps-create-deploy-template/jsonoutline.png)
    
4. 在 [新增資源] 對話方塊中，尋找並選取 [邏輯應用程式]。 為您的邏輯應用程式命名，然後選擇 [新增]。

    ![新增資源](./media/logic-apps-create-deploy-template/addresource.png)


## <a name="deploy-a-logic-app-template"></a>部署邏輯應用程式範本

您可以使用任何工具部署範本，例如 PowerShell、REST API、[Azure DevOps Release Management](#team-services)，以及透過 Azure 入口網站進行範本部署。
此外，為了儲存參數的值，我們也建議您建立[參數檔](../azure-resource-manager/resource-group-template-deploy.md#parameter-files)。
了解如何[使用 Azure Resource Manager 範本和 PowerShell 部署資源](../azure-resource-manager/resource-group-template-deploy.md)或[使用 Azure Resource Manager 範本和 Azure 入口網站部署資源](../azure-resource-manager/resource-group-template-deploy-portal.md)。

### <a name="authorize-oauth-connections"></a>授權 OAuth 連接

部署之後，邏輯應用程式就能搭配有效參數端對端運作。
不過，您仍然必須授權 OAuth 連接，才能產生有效的存取權杖。
若要授權 OAuth 連接，請在 Logic Apps 設計工具中開啟邏輯應用程式，然後授權這些連接。 或者，如果是自動化部署，您可以使用指令碼來同意每個 OAuth 連接。
在 GitHub 上的 [LogicAppConnectionAuth](https://github.com/logicappsio/LogicAppConnectionAuth) 專案下方有一個範例指令碼。

<a name="team-services"></a>
## <a name="azure-devops-release-management"></a>Azure DevOps Release Management

部署和管理環境的常見案例，是搭配使用 Azure DevOps 中的 Release Management 之類的工具與邏輯應用程式部署範本。 Azure DevOps 包含可新增至任何組建或發行管線的[部署 Azure 資源群組](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/DeployAzureResourceGroup)工作。 您必須擁有[服務主體](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/)才能授權部署，進而能夠產生發行管線。

1. 在 Release Management 中，選取 [空白]，如此就能建立空白的管線。

    ![建立空管線][1]

2. 選擇此定義所需的任何資源，很可能會包含手動產生或在建置流程中產生的邏輯應用程式範本。
3. 新增 [Azure 資源群組部署]  工作。
4. 設定 [服務主體](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/)並參考 [範本] 和 [範本參數] 檔案。
5. 視需要針對任何其他環境、自動化測試或核准者，繼續在發行程序中建置步驟。

<!-- Image References -->
[1]: ./media/logic-apps-create-deploy-template/emptyreleasedefinition.png
