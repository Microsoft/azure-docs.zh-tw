---
title: 使用 GitHub Actions 部署 Resource Manager 範本
description: 說明如何使用 GitHub Actions)  (ARM 範本部署 Azure Resource Manager 範本。
ms.topic: conceptual
ms.date: 10/13/2020
ms.custom: github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 67d4ac51e3e1f84f6a9acd0fc94d5818355d3954
ms.sourcegitcommit: 3c3ec8cd21f2b0671bcd2230fc22e4b4adb11ce7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2021
ms.locfileid: "98762079"
---
# <a name="deploy-arm-templates-by-using-github-actions"></a>使用 GitHub Actions 部署 ARM 範本

[GitHub Actions](https://docs.github.com/en/actions) 是 GitHub 中的一組功能，可讓您在儲存程式碼及共同處理提取要求和問題的相同位置，自動化軟體發展工作流程。

使用 [ [部署 Azure Resource Manager 範本] 動作](https://github.com/marketplace/actions/deploy-azure-resource-manager-arm-template) ，自動將 Azure Resource Manager 範本 (ARM 範本部署到 Azure) 。

## <a name="prerequisites"></a>必要條件

- 具有有效訂用帳戶的 Azure 帳戶。 [免費建立帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
- GitHub 帳戶。 如果您沒有帳戶，請[免費](https://github.com/join)註冊。
    - 用於儲存 Resource Manager 範本和工作流程檔案的 GitHub 存放庫。 若要建立一個，請參閱[建立新的存放庫](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository) \(英文\)。


## <a name="workflow-file-overview"></a>工作流程檔案概觀

工作流程是由您存放庫內 `/.github/workflows/` 路徑中的 YAML (. yml) 檔案所定義的。 此定義包含組成工作流程的各種步驟與參數。

檔案內有兩個區段：

|區段  |工作  |
|---------|---------|
|**驗證** | 1.定義服務主體。 <br /> 2.建立 GitHub 祕密。 |
|**部署** | 1. 部署 Resource Manager 範本。 |

## <a name="generate-deployment-credentials"></a>產生部署認證


您可以使用 [Azure CLI](/cli/azure/) 中的 [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac&preserve-view=true) 命令來建立[服務主體](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object)。 請使用 Azure 入口網站中的 [Azure Cloud Shell](https://shell.azure.com/)，或選取 [試試看] 按鈕來執行此命令。

如果您還沒有資源群組，請建立一個。

```azurecli-interactive
    az group create -n {MyResourceGroup}
```

請將預留位置 `myApp` 取代為您的應用程式名稱。

```azurecli-interactive
   az ad sp create-for-rbac --name {myApp} --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{MyResourceGroup} --sdk-auth
```

在上述範例中，請將預留位置取代為您的訂用帳戶識別碼和資源群組名稱。 輸出是一個 JSON 物件，內有角色指派認證可讓您存取 App Service 應用程式，如下所示。 複製此 JSON 物件以供後續使用。 您只需要具有 `clientId`、`clientSecret`、`subscriptionId` 和 `tenantId` 值的區段。

```output
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> 授與最小存取權永遠是最佳作法。 上述範例中的範圍僅限於資源群組。



## <a name="configure-the-github-secrets"></a>設定 GitHub 祕密

您必須為您的 Azure 認證、資源群組和訂用帳戶建立秘密。

1. 在 [GitHub](https://github.com/) 中，瀏覽您的存放庫。

1. 選取 [設定] > [秘密] > [新增秘密]。

1. 將得自 Azure CLI 命令的整個 JSON 輸出貼到祕密的 [值] 欄位中。 將祕密命名為 `AZURE_CREDENTIALS`。

1. 建立另一個名為 `AZURE_RG` 的秘密。 將資源群組的名稱新增至秘密的值欄位， (範例： `myResourceGroup`) 。

1. 建立名為的其他秘密 `AZURE_SUBSCRIPTION` 。 將您的訂用帳戶識別碼新增至秘密的值欄位， (範例： `90fd3f9d-4c61-432d-99ba-1273f236afa2`) 。

## <a name="add-resource-manager-template"></a>新增 Resource Manager 範本

將 Resource Manager 範本新增至您的 GitHub 存放庫。 此範本會建立儲存體帳戶。

```url
https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

您可以將檔案放置於存放庫中的任何位置。 下一節中的工作流程範例假設範本檔案命名為 **azuredeploy.json**，並儲存在存放庫的根目錄。

## <a name="create-workflow"></a>建立工作流程

工作流程檔案必須儲存在存放庫根目錄的 [ **github/** workflow] 資料夾中。 工作流程副檔名可以是 **.yml** 或 **.yaml**。

1. 從您的 GitHub 存放庫，選取頂端功能表中的 [動作]。
1. 選取 [新增工作流程]。
1. 選取 [自行設定工作流程]。
1. 如果您偏好使用 **main.yml** 以外的其他名稱，請將工作流程檔案重新命名。 例如：**deployStorageAccount.yml**。
1. 以下列內容取代 yml 檔案的內容：

    ```yml
    on: [push]
    name: Azure ARM
    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:

          # Checkout code
        - uses: actions/checkout@main

          # Log into Azure
        - uses: azure/login@v1
          with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

          # Deploy ARM template
        - name: Run ARM deploy
          uses: azure/arm-deploy@v1
          with:
            subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
            resourceGroupName: ${{ secrets.AZURE_RG }}
            template: ./azuredeploy.json
            parameters: storageAccountType=Standard_LRS

          # output containerName variable from template
        - run: echo ${{ steps.deploy.outputs.containerName }}
    ```
    > [!NOTE]
    > 您可以改為在 ARM 部署動作中指定 JSON 格式參數檔案 (範例： `.azuredeploy.parameters.json`) 。

    工作流程檔案的第一個區段包括：

    - **名稱**：工作流程的名稱。
    - **on**：觸發工作流程的 GitHub 事件名稱。 當主要分支上有一個推播事件時，工作流程就會觸發，其中至少會修改兩個指定檔案中的其中一個。 這兩個檔案為工作流程檔案和範本檔案。

1. 選取 [開始認可]。
1. 選取 **[直接認可至主要分支**]。
1. 選取 [認可新檔案] (或 [認可變更])。

由於工作流程會設定為透過要更新的工作流程檔案或範本檔案來觸發，因此，工作流程會在您認可變更後立即啟動。

## <a name="check-workflow-status"></a>檢查工作流程狀態

1. 選取 [ **動作** ] 索引標籤。您會看到列出的 [ **建立 deployStorageAccount] yml** 工作流程。 執行工作流程需要1-2 分鐘的時間。
1. 選取工作流程以將其開啟。
1. 從功能表選取 [ **執行 ARM 部署** ] 以確認部署。

## <a name="clean-up-resources"></a>清除資源
當您的資源群組和儲存機制不再需要時，請刪除資源群組和您的 GitHub 存放庫，以清除您所部署的資源。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [建立您的第一個 ARM 範本](./template-tutorial-create-first-template.md)

> [!div class="nextstepaction"]
> [學習模組：使用 GitHub Actions 將 ARM 範本的部署自動化](/learn/modules/deploy-templates-command-line-github-actions/)
