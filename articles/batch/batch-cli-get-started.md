---
title: 開始使用 Azure CLI for Batch | Microsoft Docs
description: 取得 Azure CLI 中 Batch 命令的快速簡介，以便管理 Azure Batch 服務資源
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: fcd76587-1827-4bc8-a84d-bba1cd980d85
ms.service: batch
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: multiple
ms.workload: big-compute
ms.date: 07/24/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 9b5c1df8776b63fc8ceecfa0377e74c757ba503c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950143"
---
# <a name="manage-batch-resources-with-azure-cli"></a>使用 Azure CLI 管理 Batch 資源

Azure CLI 是管理 Azure 資源的 Azure 命令列體驗。 它可以用於 macOS、Linux 和 Windows。 Azure CLI 已針對從命令列管理 Azure 資源進行最佳化。 您可以使用 Azure CLI 來管理 Azure Batch 帳戶，以及管理集區、作業和工作等資源。 許多使用 Batch API、Azure 入口網站和 Batch PowerShell Cmdlet 執行的工作，也都可以使用 Azure CLI 來撰寫指令碼。

本文概述如何使用搭配 [Azure CLI 2.0 版](https://docs.microsoft.com/cli/azure) 與 Batch。 如需 CLI 與 Azure 搭配使用的概觀，請參閱[開始使用 Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli)。

## <a name="set-up-the-azure-cli"></a>設定 Azure CLI

您可以在 [Azure Cloud Shell](../cloud-shell/overview.md) 中執行最新的 Azure CLI。 若要在本機安裝 Azure CLI，請遵循[安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 中概述的步驟。

> [!TIP]
> 建議您經常更新您的 Azure CLI 安裝以利用服務更新和增強功能。
> 
> 

## <a name="command-help"></a>命令說明

您可以將 `-h` 附加至命令，以在 Azure CLI 中顯示每個命令的說明文字。 略過任何其他選項。 例如︰

* 若要取得 `az` 命令的說明，請輸入︰`az -h`
* 若要取得 CLI 中所有 Batch 命令的清單，請使用︰`az batch -h`
* 若要取得有關建立 Batch 帳戶的說明，請輸入︰ `az batch account create -h`

如有疑問，請使用 `-h` 命令列選項來取得任何 Azure CLI 命令的說明。



此外，請參閱 Azure CLI 參考文件，以取得[適用於 Batch 的 Azure CLI 命令](/cli/azure/batch)詳細資訊。 

## <a name="log-in-and-authenticate"></a>登入和驗證

若要搭配使用 Azure CLI 與 Batch，您需要登入和驗證。 請遵循兩個簡單的步驟︰

1. **登入 Azure。** 登入 Azure 可讓您存取 Azure Resource Manager 命令，包括 [Batch Management 服務](batch-management-dotnet.md)命令。  
2. **登入您的 Batch 帳戶。** 登入您的 Batch 帳戶即可存取 Batch 服務命令。   

### <a name="log-in-to-azure"></a>登入 Azure

如 [登入 Azure CLI](/cli/azure/authenticate-azure-cli) 中的詳細描述，有幾種不同方式可登入 Azure：

1. [以互動方式登入](https://docs.microsoft.com/cli/azure/authenticate-azure-cli#az-authenticate-azure-cli-interactive-log-in)。 當您自行從命令列執行 Azure CLI 命令時以互動方式登入。
2. [使用服務主體來登入](https://docs.microsoft.com/cli/azure/authenticate-azure-cli#az-authenticate-azure-cli-logging-in-with-a-service-principal)。 當您從指令碼或應用程式執行 Azure CLI 命令時，使用服務主體來登入。

基於本文的目的，我們會示範如何以互動方式登入 Azure。 在命令列上輸入 [az login](https://docs.microsoft.com/cli/azure/reference-index#az-login)：

```azurecli
# Log in to Azure and authenticate interactively.
az login
```

`az login` 命令會傳回可讓您用來驗證的權杖，如下所示。 請依照所提供的指示來開啟網頁並權杖提交至 Azure︰

![登入 Azure](./media/batch-cli-get-started/az-login.png)

[範例 Shell 指令碼](#sample-shell-scripts)一節中所列的範例也會顯示如何藉由以互動方式登入 Azure 來啟動 Azure CLI 工作階段。 登入之後，您可以呼叫命令來使用 Batch Management 資源，包括批次帳戶、金鑰、應用程式套件和配額。  

### <a name="log-in-to-your-batch-account"></a>登入您的 Batch 帳戶

若要使用 Azure CLI 來管理 Batch 資源 (例如集區、作業和工作)，您需要登入您的 Batch 帳戶和進行驗證。 若要登入 Batch 服務，請使用 [az batch account login](https://docs.microsoft.com/cli/azure/batch/account#az-batch-account-login) 命令。 

您有兩個對 Batch 帳戶進行驗證的選項︰

- **使用 Azure Active Directory (Azure AD) 驗證** 

    當您搭配使用 Azure CLI 與 Batch 時，使用 Azure AD 進行驗證是預設值，建議使用大部分的情況。 
    
    當您以互動方式登入 Azure 時，如上一節中所述，系統會快取您的認證，因此 Azure CLI 可讓您使用這些相同的認證來登入 Batch 帳戶。 如果您使用服務主體登入 Azure，這些認證也會用來登入您的 Batch 帳戶。

    Azure AD 的優點就是它會提供角色型存取控制 (RBAC)。 使用 RBAC，使用者的存取權取決於他們被指派的角色，而不是他們是否擁有帳戶金鑰。 您可以管理 RBAC 角色，並且讓 Azure AD 處理存取和驗證，而不用管理帳戶金鑰。  

     若要使用 Azure AD 登入 Batch 帳戶，請呼叫 [az batch account login](https://docs.microsoft.com/cli/azure/batch/account#az-batch-account-login) 命令： 

    ```azurecli
    az batch account login -g myresource group -n mybatchaccount
    ```

- **使用共用金鑰驗證**

    [共用金鑰驗證](/rest/api/batchservice/authenticate-requests-to-the-azure-batch-service#authentication-via-shared-key)會使用帳戶存取金鑰來驗證 Batch 服務的 Azure CLI 命令。

    如果您要建立 Azure CLI 指令碼來自動呼叫 Batch 命令，您可以使用共用金鑰驗證或 Azure AD 服務主體。 在某些情況下，使用共用金鑰驗證可能會比建立服務主體簡單。  

    若要使用共用金鑰驗證登入，請在命令列上包含 `--shared-key-auth` 選項︰

    ```azurecli
    az batch account login -g myresourcegroup -n mybatchaccount --shared-key-auth
    ```

[範例 shell 指令碼](#sample-shell-scripts)一節中所列的範例顯示如何使用 Azure AD 和共用金鑰透過 Azure CLI 登入您的 Batch 帳戶。

## <a name="use-azure-batch-cli-extension-commands"></a>使用 Azure Batch CLI 擴充功能命令

藉由使用 Azure Batch CLI 擴充功能，您可以使用 Azure CLI 執行端對端的 Batch 作業，而不需要撰寫程式碼。 擴充功能所支援的 Batch 命令可讓您使用 JSON 範本與 Azure CLI 來建立集區、作業和工作。 您也可以使用擴充功能 CLI 命令將作業輸入檔案上傳至與 Batch 帳戶相關聯的 Azure 儲存體帳戶，以及從它下載作業輸出檔案。 如需詳細資訊，請參閱[使用 Azure Batch CLI 範本和檔案傳輸](batch-cli-templates.md)。

## <a name="script-examples"></a>指令碼範例

請參閱 [CLI 指令碼範例](cli-samples.md)，以使用 Batch 完成一般工作。 這些範例涵蓋許多 Azure CLI 中可供 Batch 用來建立和管理帳戶、集區、作業及工作的命令。 

## <a name="json-files-for-resource-creation"></a>用於建立資源的 JSON 檔案

當您建立 Batch 資源 (如集區和工作) 時，您可以指定包含新資源組態的 JSON 檔案，而不是將它的參數當作命令列選項傳遞。 例如︰

```azurecli
az batch pool create my_batch_pool.json
```

雖然您只可以使用命令列選項建立大多數 Batch 資源，但有些功能需要您指定 JSON 格式的檔案 (內含資源詳細資料)。 例如，如果您想要指定啟動工作的資源檔，則必須使用 JSON 檔案。

若要查看建立資源所需的 JSON 語法，請參閱 [Batch REST API 參考][rest_api]文件。 REST API 參考中的每個「新增資源類型」主題都包含可供建立該資源的範例 JSON 指令碼。 您可以使用這些範例 JSON 指令碼作為 JSON 檔案的範本，以搭配 Azure CLI 使用。 例如，若要查看用於建立集區的 JSON 語法，請參閱[將集區新增至帳戶][rest_add_pool]。

如需可指定 JSON 檔案的範例指令碼，請參閱[使用 Batch 執行作業和工作](./scripts/batch-cli-sample-run-job.md)。

> [!NOTE]
> 如果您在建立資源時指定 JSON 檔案，則會忽略您在命令列上為該資源指定的所有其他參數。
> 
> 

## <a name="efficient-queries-for-batch-resources"></a>有效率的 Batch 資源查詢

每個 Batch 資源類型都支援 `list` 命令，已查詢 Batch 帳戶並列出該類型的資源。 例如，您可以列出您帳戶中的集區和作業中的工作︰

```azurecli
az batch pool list
az batch task list --job-id job001
```

當您透過 `list` 作業查詢 Batch 服務時，您可以指定 OData 子句來限制傳回的資料量。 因為所有篩選發生於伺服器端，只有您要求的資料會跨越網路。 當您執行作業清單時，使用這些子句來節省頻寬 (因而節省時間)。

下表描述 Batch 服務所支援的 OData 子句︰

| 子句 | 說明 |
|---|---|
| `--select-clause [select-clause]` | 傳回每個實體的屬性子集。 |
| `--filter-clause [filter-clause]` | 傳回符合指定之 OData 運算式的實體。 |
| `--expand-clause [expand-clause]` | 取得單一基礎 REST 呼叫中的實體資訊。 expand 子句目前只支援 `stats` 屬性。 |

如需顯示如何使用 OData 子句的範例指令碼，請參閱[使用 Batch 執行作業和工作](./scripts/batch-cli-sample-run-job.md)。

如需使用 OData 子句執行有效率之清單查詢的詳細資訊，請參閱[有效率地查詢 Azure Batch 服務](batch-efficient-list-queries.md)。

## <a name="troubleshooting-tips"></a>疑難排解秘訣

當您針對 Azure CLI 問題進行疑難排解時，下列秘訣可能有所幫助︰

* 使用 `-h` 取得任何 CLI 命令的 **說明文字**
* 使用 `-v` 和 `-vv` 來顯示 **verbose** 命令輸出。 包含 `-vv` 旗標後，Azure CLI 會顯示實際的 REST 要求和回應。 這些參數方便用於顯示完整的錯誤輸出。
* 您可以使用 `--json` 選項檢視 **JSON 格式的命令輸出**。 例如， `az batch pool show pool001 --json` 會以 JSON 格式顯示 pool001 的屬性。 您可以接著複製並修改此輸出，以便用於 `--json-file` (請參閱本文前面的 [JSON 檔案](#json-files) )。
<!---Loc Comment: Please, check link [JSON files] since it's not redirecting to any location.--->

## <a name="next-steps"></a>後續步驟

* 如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
* 如需 Batch 資源的詳細資訊，請參閱[適用於開發人員的 Azure Batch 概觀](batch-api-basics.md)。
* 如需有關使用 Batch 範本，在不需要撰寫程式碼的情況下來建立集區、作業和工作的詳細資訊，請參閱[使用 Azure Batch CLI 範本和檔案傳輸](batch-cli-templates.md)。

[github_readme]: https://github.com/Azure/azure-xplat-cli/blob/dev/README.md
[rest_api]: https://msdn.microsoft.com/library/azure/dn820158.aspx
[rest_add_pool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
