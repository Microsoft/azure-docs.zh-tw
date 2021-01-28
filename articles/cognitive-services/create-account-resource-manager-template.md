---
title: 使用 ARM 範本建立 Azure 認知服務資源 | Microsoft Docs
description: 使用 ARM 範本建立 Azure 認知服務資源。
keywords: 認知服務, 認知解決方案, 認知智慧, 認知人工智慧
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: quickstart
ms.date: 09/14/2020
ms.author: aahi
ms.custom: subject-armqs, devx-track-azurecli
ms.openlocfilehash: 856fd86b08dfe74925e682661df0d20692839bd5
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98928656"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-an-arm-template"></a>快速入門：使用 ARM 範本建立認知服務資源

本快速入門說明如何使用 Azure Resource Manager 範本 (ARM 範本) 建立認知服務。

Azure 認知服務是可搭配 REST API 和用戶端程式庫 SDK 的雲端式服務，可協助開發人員建置認知智慧應用程式，且無須直接人工智慧 (AI) 或資料科學技術或知識。 Azure 認知服務可讓開發人員使用認知解決方案，輕鬆地將認知功能新增至其應用程式，以查看、聆聽、說出、了解，甚至是開始的原因。

使用 Azure Resource Manager 範本 (ARM 範本) 建立資源。 此多服務資源可讓您：

* 使用單一金鑰和端點存取多個 Azure 認知服務。
* 合併您所使用服務的帳單。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果您的環境符合必要條件，而且您很熟悉 ARM 範本，請選取 [部署至 Azure] 按鈕。 範本會在 Azure 入口網站中開啟。

[![將您的認知服務部署至 Azure](../media/template-deployments/deploy-to-azure.svg "將您的認知服務部署至 Azure")](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cognitive-services-universalkey%2Fazuredeploy.json)

## <a name="prerequisites"></a>必要條件

* 如果您沒有 Azure 訂用帳戶，請[建立免費帳戶](https://azure.microsoft.com/free/cognitive-services)。

## <a name="review-the-template"></a>檢閱範本

本快速入門中使用的範本是來自 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/101-cognitive-services-universalkey/)。

:::code language="json" source="~/quickstart-templates/101-cognitive-services-universalkey/azuredeploy.json":::

此範本中已定義一項 Azure 資源：
* [Microsoft.CognitiveServices/accounts](/azure/templates/microsoft.cognitiveservices/accounts)：建立認知服務資源。

## <a name="deploy-the-template"></a>部署範本

# <a name="azure-portal"></a>[Azure 入口網站](#tab/portal)

1. 按一下 [部署至 Azure] 按鈕。

    [![部署至 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cognitive-services-universalkey%2Fazuredeploy.json)

2. 輸入下列值。

    |值  |描述  |
    |---------|---------|
    | **訂用帳戶** | 選取 Azure 訂用帳戶。 |
    | **資源群組** | 選取 [新建]，輸入資源群組的唯一名稱，然後按一下 [確認]。 |
    | **區域** | 選取區域。  例如，**美國東部** |
    | **認知服務名稱** | 以您資源的唯一名稱取代。 當您在下一節中驗證部署時，會需要用到此名稱。 |
    | **位置** | 取代為上方使用的區域。 |
    | **Sku** | 您資源的[定價層](https://azure.microsoft.com/pricing/details/cognitive-services/)。 |

    :::image type="content" source="media/arm-template/universal-key-portal-template.png" alt-text="資源建立畫面。":::

3. 選取 [檢閱 + 建立]，然後選取 [建立]。 資源順利完成部署之後，[移至資源] 按鈕將會反白顯示。

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

> [!NOTE]
> `az deployment group` create 需要 Azure CLI 2.6 版或更新版本。 顯示版本類型 `az --version`。 如需詳細資訊，請參閱[文件](/cli/azure/deployment/group)。

[在您的本機電腦上](/cli/azure/install-azure-cli)使用 Azure 命令列介面 (CLI) 執行下列指令碼，或在瀏覽器中使用 [試試看] 按鈕。 輸入新資源群組的名稱和位置 (例如 `centralus`)，ARM 範本將用來部署其中的認知服務資源。 請記住您使用的名稱。 稍後您會使用該名稱來驗證部署。


```azurecli-interactive
read -p "Enter a name for your new resource group:" resourceGroupName &&
read -p "Enter the location (i.e. centralus):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-cognitive-services-universalkey/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

---

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]


## <a name="review-deployed-resources"></a>檢閱已部署的資源

# <a name="portal"></a>[入口網站](#tab/portal)

當部署完成時，您可以按一下 [移至資源] 按鈕，以查看您的新資源。 您也可以藉由下列方式來尋找資源群組：

1. 從左側導覽功能表中選取 [資源群組]。
2. 選取資源群組名稱。

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

使用 Azure CLI 執行下列指令碼，並輸入您稍早建立的資源群組名稱。

```azurecli-interactive
echo "Enter the resource group where the Cognitive Services resource exists:" &&
read resourceGroupName &&
az cognitiveservices account list -g $resourceGroupName
```

---


## <a name="clean-up-resources"></a>清除資源

如果您想要清除和移除認知服務訂用帳戶，則可以刪除資源或資源群組。 刪除資源群組也會刪除與群組中的任何其他資源。

# <a name="azure-portal"></a>[Azure 入口網站](#tab/portal)

1. 在 Azure 入口網站中，展開左側功能表以開啟服務的功能表，然後選擇 [資源群組] 以顯示資源群組的清單。
2. 找出要刪除的資源位於哪一個資源群組
3. 以滑鼠右鍵按一下資源群組清單。 選取 [刪除資源群組] 並且確認。

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

使用 Azure CLI 執行下列指令碼，並輸入您稍早建立的資源群組名稱。

```azurecli-interactive
echo "Enter the resource group name, for deletion:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

## <a name="next-steps"></a>後續步驟

* [驗證 Azure 認知服務要求](authentication.md)
* [什麼是 Azure 認知服務？](./what-are-cognitive-services.md)
* [自然語言支援](language-support.md)
* [Docker 容器支援](cognitive-services-container-support.md)
