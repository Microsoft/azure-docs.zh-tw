---
title: Azure 快速入門 - 使用 PowerShell 從 Key Vault 設定及擷取祕密 | Microsoft Docs
description: ''
services: key-vault
author: barclayn
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 1126f665-2e6c-4cca-897e-7d61842e8334
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.custom: mvc
ms.date: 08/28/2018
ms.author: barclayn
ms.openlocfilehash: ae09442c1ebec878e15b5149d0634f8dc3568f7a
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49362413"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-powershell"></a>快速入門：使用 PowerShell 從 Azure Key Vault 設定及擷取祕密

Azure Key Vault 是一項雲端服務，可作為安全的祕密存放區。 您也可以安全地儲存金鑰、密碼、憑證和其他祕密。 如需 Key Vault 的詳細資訊，您可以檢閱[概觀](key-vault-overview.md)。 在本快速入門中，您會使用 PowerShell 來建立金鑰保存庫。 然後，將祕密存放在新建立的保存庫中。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，您必須使用 Azure PowerShell 模組 5.1.1 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

```azurepowershell-interactive
Login-AzureRmAccount
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

```azurepowershell-interactive
New-AzureRmResourceGroup -Name ContosoResourceGroup -Location EastUS
```

## <a name="create-a-key-vault"></a>建立金鑰保存庫

接下來，您會建立 Key Vault。 執行此步驟時，您需要一些資訊：

雖然我們在此快速入門中使用 “Contoso KeyVault2” 作為我們的 Key Vault 名稱，但您必須使用唯一的名稱。

- **保存庫名稱**：Contoso-Vault2。
- **資源群組名稱**：ContosoResourceGroup。
- **位置**：美國東部。

```azurepowershell-interactive
New-AzureRmKeyVault -Name 'Contoso-Vault2' -ResourceGroupName 'ContosoResourceGroup' -Location 'East US'
```

此 Cmdlet 的輸出會顯示新建立金鑰保存庫的屬性。 請記下下列兩個屬性：

* **保存庫名稱**：在此範例中是 **Contoso-Vault2**。 您將在其他金鑰保存庫 Cmdlet 中使用此名稱。
* **保存庫 URI**：在此範例中是 https://contosokeyvault.vault.azure.net/。 透過其 REST API 使用保存庫的應用程式必須使用此 URI。

在保存庫建立後，您的 Azure 帳戶是唯一能夠在這個新保存庫上執行任何作業的帳戶。

![Key Vault 建立命令完成後的輸出](./media/quick-create-powershell/output-after-creating-keyvault.png)

## <a name="adding-a-secret-to-key-vault"></a>將祕密新增至 Key Vault

若要將祕密新增至保存庫，您只需要採取一些步驟。 在此情況下，您會新增應用程式可以使用的密碼。 此密碼稱為 **ExamplePassword** 並在其中儲存 ''**Pa$$w0rd**'' 值。

第一次將 Pa$$w0rd 值轉換為安全的字串時，請輸入：

```azurepowershell-interactive
$secretvalue = ConvertTo-SecureString 'Pa$$w0rd' -AsPlainText -Force
```

然後，輸入下列 PowerShell 命令，以在 Key Vault 中建立稱為 **ExamplePassword** 並具有 **Pa$$w0rd** 值的祕密：

```azurepowershell-interactive
$secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'ExamplePassword' -SecretValue $secretvalue
```

若要以純文字檢視包含在祕密中的值：

```azurepowershell-interactive
(Get-AzureKeyVaultSecret -vaultName "Contosokeyvault" -name "ExamplePassword").SecretValueText
```

現在，您已建立 Key Vault，儲存祕密，並擷取它。

## <a name="clean-up-resources"></a>清除資源

 此集合中的其他快速入門和教學課程會以本快速入門為基礎。 如果您打算繼續進行其他快速入門和教學課程，您可以讓這些資源留在原處。

若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組、Key Vault 和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name ContosoResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已建立 Key Vault，並在其中儲存軟體金鑰。 若要深入了解 Key Vault 以及如何使用它搭配您的應用程式，請繼續進行 Web 應用程式搭配 Key Vault 運作的教學課程。

> [!div class="nextstepaction"]
> 若要了解如何從針對 Azure 資源使用受控識別的 Web 應用程式讀取 Key Vault 中的祕密，請繼續進行以下教學課程[將 Azure Web 應用程式設定為從 Key Vault 讀取祕密](quick-create-net.md)。
