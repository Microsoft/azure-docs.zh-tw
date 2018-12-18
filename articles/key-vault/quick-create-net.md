---
title: 快速入門：使用 Node Web 應用程式從 Azure Key Vault 設定及擷取祕密 | Microsoft Docs
description: 快速入門：使用 Node Web 應用程式從 Azure Key Vault 設定及擷取祕密
services: key-vault
author: prashanthyv
manager: sumedhb
ms.service: key-vault
ms.topic: quickstart
ms.date: 09/12/2018
ms.author: barclayn
ms.custom: mvc
ms.openlocfilehash: 7f71e92513aedb1eb9c394c1e8f547173cfb4dbe
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45604173"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-by-using-a-net-web-app"></a>快速入門：使用 .NET Web 應用程式從 Azure Key Vault 設定及擷取祕密

在此快速入門中，您將執行讓 Azure Web 應用程式使用 Azure 資源的受控識別從 Azure Key Vault 讀取資訊所需的步驟。 您會了解如何：

> [!div class="checklist"]
> * 建立金鑰保存庫。
> * 將秘密儲存在金鑰保存庫中。
> * 從金鑰保存庫擷取祕密。
> * 建立 Azure Web 應用程式。
> * 啟用 Web 應用程式的[受控識別](../active-directory/managed-identities-azure-resources/overview.md)。
> * 授與 Web 應用程式從金鑰保存庫讀取資料所需的權限。

在我們繼續之前，請閱讀[基本概念](key-vault-whatis.md#basic-concepts)。

>[!NOTE]
>Key Vault 是一個中央存放庫，可透過程式設計方式儲存秘密。 但若要這樣做，應用程式和使用者必須要先向 Key Vault 進行驗證，也就是出具祕密。 為了遵循安全性最佳做法，第一個秘密必須要定期輪替。 
>
>使用 [Azure 資源的受控識別](../active-directory/managed-identities-azure-resources/overview.md)時，在 Azure 中執行的應用程式將會獲得一個由 Azure 自動管理的身分識別。 這有助於解決*祕密導入問題*，如此，使用者和應用程式即可遵循最佳做法，且不需要擔心輪替第一個祕密的問題。

## <a name="prerequisites"></a>必要條件

* 在 Windows 上：
  * [Visual Studio 2017 15.7.3 版或更新版本](https://www.microsoft.com/net/download/windows)，具有下列工作負載：
    * ASP.NET 和 Web 開發
    * .NET Core 跨平台開發
  * [.NET Core 2.1 SDK 或更新版本](https://www.microsoft.com/net/download/windows)

* 在 Mac 上：
  * 請參閱 [Visual Studio for Mac 的新功能](https://visualstudio.microsoft.com/vs/mac/)。

* 所有平台：
  * Git ([下載](https://git-scm.com/downloads))。
  * Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
  * [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 2.0.4 版或更新版本。 此工具適用於 Windows、Mac 和 Linux。

## <a name="log-in-to-azure"></a>登入 Azure

若要使用 Azure CLI 登入 Azure，請輸入：

```azurecli
az login
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

選取資源群組名稱，並填入預留位置。
下列範例會在美國東部位置建立一個資源群組：

```azurecli
# To list locations: az account list-locations --output table
az group create --name "<YourResourceGroupName>" --location "East US"
```

此文章將一律使用您剛才建立的資源群組。

## <a name="create-a-key-vault"></a>建立金鑰保存庫

接下來，您會在上一個步驟中建立的資源群組中建立金鑰保存庫。 請提供下列資訊：

* 金鑰保存庫名稱：此名稱必須是 3-24 個字元的字串，且只能包含 0-9、a-z、A-Z 和 -。
* 資源群組名稱。
* 位置：**美國東部**。

```azurecli
az keyvault create --name "<YourKeyVaultName>" --resource-group "<YourResourceGroupName>" --location "East US"
```

此時，您的 Azure 帳戶是唯一獲得授權在此新保存庫上執行任何作業的帳戶。

## <a name="add-a-secret-to-the-key-vault"></a>將秘密新增至金鑰保存庫

我們將新增密碼，以協助說明其運作方式。 您可以儲存 SQL 連接字串，或任何您必須安全保存但可供應用程式使用的其他資訊。

輸入下列命令，以在名為 **AppSecret** 的金鑰保存庫中建立秘密。 此秘密會儲存值 **MySecret**。

```azurecli
az keyvault secret set --vault-name "<YourKeyVaultName>" --name "AppSecret" --value "MySecret"
```

若要以純文字檢視包含在祕密中的值：

```azurecli
az keyvault secret show --name "AppSecret" --vault-name "<YourKeyVaultName>"
```

此命令會顯示祕密資訊，包括 URI。 完成這些步驟之後，您應該會有祕密在金鑰保存庫中的 URI。 請記下此資訊。 您在後續的步驟中將需要此資訊。

## <a name="clone-the-repo"></a>複製存放庫

複製存放庫，以建立可用來編輯來源的本機複本。 執行以下命令：

```
git clone https://github.com/Azure-Samples/key-vault-dotnet-core-quickstart.git
```

## <a name="open-and-edit-the-solution"></a>開啟及編輯解決方案

編輯 program.cs 檔案，以使用您的特定金鑰保存庫名稱來執行範例：

1. 瀏覽至 key-vault-dotnet-core-quickstart 資料夾。
2. 在 Visual Studio 2017 中開啟 key-vault-dotnet-core-quickstart.sln 檔案。
3. 開啟 Program.cs 檔案，並使用您先前建立的金鑰保存庫名稱更新預留位置 *YourKeyVaultName*。

這個解決方案會使用 [AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) 和 [KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault) NuGet 程式庫。

## <a name="run-the-app"></a>執行應用程式

在 Visual Studio 2017 的主功能表中，選取 [偵錯] > [啟動但不偵錯]。 在瀏覽器出現時，移至 [關於] 頁面。 **AppSecret** 的值隨即顯示。

## <a name="publish-the-web-application-to-azure"></a>將 Web 應用程式發佈至 Azure

將此應用程式發佈至 Azure，以 Web 應用程式的形式即時查看，並確認您可以擷取祕密值：

1. 在 Visual Studio 中，選取 **key-vault-dotnet-core-quickstart** 專案。
2. 選取 [發佈] > [開始]。
3. 建立新的 **App Service**，然後選取 [發佈]。
4. 將應用程式名稱變更為 **keyvaultdotnetcorequickstart**。
5. 選取 [建立] 。

>[!VIDEO https://sec.ch9.ms/ch9/e93d/a6ac417f-2e63-4125-a37a-8f34bf0fe93d/KeyVault_high.mp4]

## <a name="enable-a-managed-identity-for-the-web-app"></a>啟用 Web 應用程式的受控識別

Azure Key Vault 可安全地儲存認證和其他金鑰及密碼，但是您的程式碼必須向 Key Vault 進行驗證，才可取得這些項目。 [Azure 資源受控識別概觀](../active-directory/managed-identities-azure-resources/overview.md)可在 Azure Active Directory (Azure AD) 中將受控識別自動提供給 Azure 服務，而降低解決此問題的難度。 您可以使用此身分識別來完成任何支援 Azure AD 驗證的服務驗證 (包括 Key Vault)，不需要任何您程式碼中的認證。

1. 返回 Azure CLI。
2. 執行 assign-identity 命令來建立此應用程式的識別：

   ```azurecli
   az webapp identity assign --name "keyvaultdotnetcorequickstart" --resource-group "<YourResourceGroupName>"
   ```

>[!NOTE]
>此程式中的命令等同於前往入口網站，並在 Web 應用程式屬性中將 [身分識別/系統指派] 設定切換為 [開啟]。

## <a name="assign-permissions-to-your-application-to-read-secrets-from-key-vault"></a>將權限指派給您的應用程式，以便從 Key Vault 讀取秘密

當您將應用程式發佈至 Azure 時，請記下輸出。 其格式應為：
        
        {
          "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "type": "SystemAssigned"
        }
        
然後，使用金鑰保存庫的名稱和 **PrincipalId** 的值來執行此命令：

```azurecli

az keyvault set-policy --name '<YourKeyVaultName>' --object-id <PrincipalId> --secret-permissions get list

```

現在當您執行應用程式時，您應該會看到擷取的秘密值。 在上面的命令中，您會被賦予 App Service 的身分識別 (MSI) 權限以在您的 Key Vault 上執行**取得**與**列出**作業

## <a name="next-steps"></a>後續步驟

* [Azure Key Vault 首頁](https://azure.microsoft.com/services/key-vault/)
* [Azure Key Vault 文件](https://docs.microsoft.com/azure/key-vault/)
* [適用於 .NET 的 Azure SDK](https://github.com/Azure/azure-sdk-for-net)
* [Azure REST API 參考](https://docs.microsoft.com/rest/api/keyvault/)
