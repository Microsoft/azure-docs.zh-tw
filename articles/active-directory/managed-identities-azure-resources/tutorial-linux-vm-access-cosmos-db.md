---
title: 使用 Linux VM 系統指派的受控識別來存取 Azure Cosmos DB
description: 本教學課程會逐步引導您使用 Linux VM 系統指派的受控識別，以存取 Azure Cosmos DB。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/09/2018
ms.author: daveba
ms.openlocfilehash: ff41bbe796b561234ce5d09abfc8eb5da0e0a6b5
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220308"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-cosmos-db"></a>教學課程：使用 Linux VM 系統指派的受控識別來存取 Azure Cosmos DB 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]


本教學課程說明如何將系統指派的受控識別用於 Linux 虛擬機器 (VM)，以存取 Azure Cosmos DB。 您會了解如何：

> [!div class="checklist"]
> * 建立 Cosmos DB 帳戶
> * 在 Cosmos DB 帳戶中建立集合
> * 將存取 Azure Cosmos DB 執行個體的權利，授予系統指派的受控識別
> * 擷取 Linux VM 系統所指派受控識別的 `principalID`
> * 取得存取權杖，並使用它來呼叫 Azure Resource Manager
> * 從 Azure Resource Manager 取得存取金鑰以進行 Cosmos DB 呼叫

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Linux 虛擬機器](/azure/virtual-machines/linux/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

若要執行本教學課程中的 CLI 指令碼範例，您有兩個選項：

- 透過 Azure 入口網站或是每個程式碼區塊右上角的 [試試看] 按鈕，使用 [Azure Cloud Shell](~/articles/cloud-shell/overview.md)。
- 如果您需要使用本機 CLI 主控台，請[安裝最新版的 CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli) (2.0.23 或更新版本)。

## <a name="create-a-cosmos-db-account"></a>建立 Cosmos DB 帳戶 

如果您還沒有 Cosmos DB 帳戶，請加以建立。 您可以跳過此步驟，直接使用現有的 Cosmos DB 帳戶。 

1. 按一下 Azure 入口網站左上角的 [+/建立新服務] 按鈕。
2. 按一下 [資料庫]，然後按 [Azure Cosmos DB]，新的 [新增帳戶] 面板隨即顯示。
3. 輸入 Cosmos DB 帳戶的 [識別碼]，您稍後將會用到。  
4. **API** 應設定為 "SQL"。 本教學課程中所述的方法可用於其他可用的 API 類型，但本教學課程中的步驟適用於 SQL API。
5. 確定 [訂用帳戶] 和 [資源群組] 符合您在上一個步驟中建立 VM 時指定的值。  選取有可用 Cosmos DB 的 [位置]。
6. 按一下頁面底部的 [新增] 。

## <a name="create-a-collection-in-the-cosmos-db-account"></a>在 Cosmos DB 帳戶中建立集合

接下來，在 Cosmos DB 帳戶中新增您可以在後續步驟中查詢的資料收集。

1. 導覽至您新建立的 Cosmos DB 帳戶。
2. 在 [概觀] 索引標籤上按一下 [+/新增集合] 按鈕，[新增集合] 面板隨即顯示。
3. 為集合指定資料庫識別碼和集合識別碼、選取儲存容量、輸入分割區索引鍵、輸入輸送量值，然後按一下 [確定]。  在本教學課程中，以 "Test" 作為資料庫識別碼和集合識別碼，並選取固定的儲存容量和最小輸送量 (400 RU/s)，即足堪使用。  

## <a name="retrieve-the-principalid-of-the-linux-vms-system-assigned-managed-identity"></a>擷取 Linux VM 系統所指派受控識別的 `principalID`

在下一節中，若要從 Resource Manager 存取 Cosmos DB 帳戶存取金鑰，您必須擷取 Linux VM 系統所指派受控識別的 `principalID`。  請務必以您自己的值取代 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>` (VM 所在的資源群組) 和 `<VM NAME>`。

```azurecli-interactive
az resource show --id /subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAMe> --api-version 2017-12-01
```
回應包含系統所指派受控識別的詳細資料 (請記下 principalID，下一節會用到)：

```bash  
{
    "id": "/subscriptions/<SUBSCRIPTION ID>/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAMe>",
  "identity": {
    "principalId": "6891c322-314a-4e85-b129-52cf2daf47bd",
    "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533f8",
    "type": "SystemAssigned"
 }

```
## <a name="grant-your-linux-vms-system-assigned-identity-access-to-the-cosmos-db-account-access-keys"></a>將存取 Cosmos DB 帳戶存取金鑰的權利，授予 Linux VM 系統指派的身分識別

Cosmos DB 原生並不支援 Azure AD 驗證。 不過，您可以使用受控識別，從 Resource Manager 中擷取 Cosmos DB 存取金鑰，然後使用該金鑰來存取 Cosmos DB。 在此步驟中，您會將存取 Cosmos DB 帳戶的權利，授予系統指派的受控識別。

若要在 Azure Resource Manager 中使用 Azure CLI，將存取 Cosmos DB 帳戶的權利授予系統指派的受控識別，請更新您環境的 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>` 和 `<COSMOS DB ACCOUNT NAME>` 值。 將 `<MI PRINCIPALID>` 換成[擷取 Linux VM MI 的 principalID](#retrieve-the-principalID-of-the-linux-VM's-system-assigned-identity) 中，由 `az resource show` 命令所傳回的 `principalId` 屬性。  使用存取金鑰時，Cosmos DB 支援兩種層級的資料細微性：對帳戶的讀取/寫入存取，以及對帳戶的唯讀存取。  如果您想要取得帳戶的讀取/寫入金鑰，請指派 `DocumentDB Account Contributor` 角色；如果要取得帳戶的唯讀金鑰，請指派 `Cosmos DB Account Reader Role` 角色：

```azurecli-interactive
az role assignment create --assignee <MI PRINCIPALID> --role '<ROLE NAME>' --scope "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMODS DB ACCOUNT NAME>"
```

回應會包含已建立的角色指派之詳細資料：

```
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>/providers/Microsoft.Authorization/roleAssignments/5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "name": "5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "properties": {
    "principalId": "c0833082-6cc3-4a26-a1b1-c4b5f90a981f",
    "roleDefinitionId": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Authorization/roleDefinitions/fbdf93bf-df7d-467e-a4d2-9458aa1360c8",
    "scope": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>"
  },
  "resourceGroup": "<RESOURCE GROUP>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="get-an-access-token-using-the-linux-vms-system-assigned-managed-identity-and-use-it-to-call-azure-resource-manager"></a>使用 Linux VM 系統指派的受控識別來取得存取權杖，以用來呼叫 Azure Resource Manager

其餘課程要從稍早建立的 VM 繼續進行。

若要完成這些步驟，您需要 SSH 用戶端。 如果您使用 Windows，您可以在[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/install_guide)中使用 SSH 用戶端。 如果您需要設定 SSH 用戶端金鑰的協助，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](../../virtual-machines/linux/ssh-from-windows.md)，或[如何在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組](../../virtual-machines/linux/mac-create-ssh-keys.md)。

1. 在 Azure 入口網站中，瀏覽至 [虛擬機器]，移至您的 Linux 虛擬機器，然後在 [概觀] 頁面中，按一下頂端的 [連線]。 複製字串以連線到您的 VM。 
2. 使用 SSH 用戶端連線到 VM。  
3. 接下來，系統會提示您輸入建立 Linux VM 時新增的 [密碼]。 您應該可以順利登入。  
4. 使用 CURL 取得 Azure Resource Manager 的存取權杖： 
     
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true   
    ```
 
    > [!NOTE]
    > 在前面的要求中，"resource" 參數的值必須完全符合 Azure AD 的預期。 當使用 Azure Resource Manager 資源 ID 時，必須在 URI 中包含結尾的斜線。
    > 在下列回應中，為求簡單明瞭，已縮短 access_token 元素。
    
    ```bash
    {"access_token":"eyJ0eXAiOi...",
     "expires_in":"3599",
     "expires_on":"1518503375",
     "not_before":"1518499475",
     "resource":"https://management.azure.com/",
     "token_type":"Bearer",
     "client_id":"1ef89848-e14b-465f-8780-bf541d325cd5"}
     ```
    
## <a name="get-access-keys-from-azure-resource-manager-to-make-cosmos-db-calls"></a>從 Azure Resource Manager 取得存取金鑰以進行 Cosmos DB 呼叫  

現在，請使用在上一節中擷取的存取權杖，使用 CURL 來呼叫資源管理員，以擷取 Cosmos DB 帳戶存取金鑰。 取得存取金鑰後，我們即可查詢 Cosmos DB。 別忘了以您自己的值取代 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>` 和 `<COSMOS DB ACCOUNT NAME>` 參數的值。 將 `<ACCESS TOKEN>` 值取代為您先前擷取的存取權杖。  如果您想要擷取讀取/寫入金鑰，請使用金鑰作業類型 `listKeys`。  如果您想要擷取唯讀金鑰，請使用金鑰作業類型 `readonlykeys`：

```bash 
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT NAME>/<KEY OPERATION TYPE>?api-version=2016-03-31' -X POST -d "" -H "Authorization: Bearer <ACCESS TOKEN>" 
```

> [!NOTE]
> 前述 URL 的文字區分大小寫，因此請確定您使用的資源群組大小寫有正確相應。 此外，務必知道這是 POST 要求而不是 GET 要求，並確定您傳遞的值符合 -d 的長度限制，且 -d 可能是 NULL。  

CURL 回應會提供金鑰清單。  例如，如果您收到唯讀金鑰：  

```bash 
{"primaryReadonlyMasterKey":"bWpDxS...dzQ==",
"secondaryReadonlyMasterKey":"38v5ns...7bA=="}
```

現在，您已有 Cosmos DB 帳戶的存取金鑰，您可以將其傳至 Cosmos DB SDK，並進行呼叫以存取帳戶。  如需快速範例，您可以將存取金鑰傳至 Azure CLI。  您可以透過 Azure 入口網站，從 Cosmos DB 帳戶刀鋒視窗上的 [概觀] 索引標籤取得 <COSMOS DB CONNECTION URL>。  請將 <ACCESS KEY> 取代為您先前取得的值：

```bash
az cosmosdb collection show -c <COLLECTION ID> -d <DATABASE ID> --url-connection "<COSMOS DB CONNECTION URL>" --key <ACCESS KEY>
```

此 CLI 命令會傳回集合的詳細資料：

```bash
{
  "collection": {
    "_conflicts": "conflicts/",
    "_docs": "docs/",
    "_etag": "\"00006700-0000-0000-0000-5a8271e90000\"",
    "_rid": "Es5SAM2FDwA=",
    "_self": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/",
    "_sprocs": "sprocs/",
    "_triggers": "triggers/",
    "_ts": 1518498281,
    "_udfs": "udfs/",
    "id": "Test",
    "indexingPolicy": {
      "automatic": true,
      "excludedPaths": [],
      "includedPaths": [
        {
          "indexes": [
            {
              "dataType": "Number",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "String",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "Point",
              "kind": "Spatial"
            }
          ],
          "path": "/*"
        }
      ],
      "indexingMode": "consistent"
    }
  },
  "offer": {
    "_etag": "\"00006800-0000-0000-0000-5a8271ea0000\"",
    "_rid": "f4V+",
    "_self": "offers/f4V+/",
    "_ts": 1518498282,
    "content": {
      "offerIsRUPerMinuteThroughputEnabled": false,
      "offerThroughput": 400
    },
    "id": "f4V+",
    "offerResourceId": "Es5SAM2FDwA=",
    "offerType": "Invalid",
    "offerVersion": "V2",
    "resource": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/"
  }
}
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何在 Linux 虛擬機器上使用系統指派的受控識別，來存取 Cosmos DB。  若要深入了解 Cosmos DB，請參閱：

> [!div class="nextstepaction"]
>[Azure Cosmos DB 概觀](/azure/cosmos-db/introduction)

