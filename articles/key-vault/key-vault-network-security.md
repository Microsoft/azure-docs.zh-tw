---
ms.assetid: ''
title: 設定 Azure Key Vault 防火牆和虛擬網路
description: 用於設定 Key Vault 防火牆和虛擬網路的逐步指示
services: key-vault
author: amitbapat
manager: mbaldwin
ms.service: key-vault
ms.topic: conceptual
ms.workload: identity
ms.date: 08/31/2018
ms.author: ambapat
ms.openlocfilehash: 6315434c1e8acc82e02f5c9e5ae8ab2d1cacc887
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302048"
---
# <a name="configure-azure-key-vault-firewalls-and-virtual-networks"></a>設定 Azure Key Vault 防火牆和虛擬網路

本指南說明設定 Key Vault 防火牆和虛擬網路的逐步指示，以限制金鑰保存庫的存取權。 [Key Vault 的虛擬網路服務端點](key-vault-overview-vnet-service-endpoints.md)可讓您將金鑰保存庫的存取限制於指定的虛擬網路和/或一組 IPv4 (網際網路通訊協定第 4 版) 位址範圍。

> [!IMPORTANT]
> 防火牆和虛擬網路規則一旦生效，只有在呼叫端要求源自允許的虛擬網路或 IPV4 位址範圍時，才可以執行所有的 Key Vault [資料平面](../key-vault/key-vault-secure-your-key-vault.md#data-plane-access-control)作業。 這也適用於從 Azure 入口網站存取金鑰保存庫。 雖然使用者可以從 Azure 入口網站瀏覽金鑰保存庫，但是如果其用戶端電腦不在允許的清單中，他們就無法列出金鑰/祕密/憑證。 這也會影響其他 Azure 服務的 [金鑰保存庫選擇器]。 使用者可以看到金鑰保存庫清單，但是如果防火牆規則阻止其用戶端電腦，則不會列出金鑰。

## <a name="azure-portal"></a>Azure 入口網站

1. 巡覽至您要保護的金鑰保存庫。
2. 按一下 [防火牆和虛擬網路]。
3. 按一下 [允許存取來源] 下方的 [選取的網路]。
4. 若要將現有的虛擬網路新增至防火牆和虛擬網路規則，請按一下 [+ 新增現有的虛擬網路]。
5. 在快顯的新刀鋒視窗中，選取您要允許存取此金鑰保存庫的訂用帳戶、虛擬網路和子網路。 如果您選取的虛擬網路和子網路尚未啟用服務端點，您會看到訊息指出「下列網路尚未啟用服務端點...」。 確認您要針對所列的虛擬網路和子網路啟用服務端點之後，按一下 [啟用]。 可能需要 15 分鐘的時間才會生效。
6. 您可以也新增虛擬網路和子網路，然後按一下 [+ 新增虛擬網路] 並遵循提示，為新建立的虛擬網路和子網路啟用服務端點。
7. 在 [IP 網路] 下方，您可以輸入採用 [CIDR (無類別網域間路由選擇) 表示法](https://tools.ietf.org/html/rfc4632)的 IPv4 位址範圍或個別的 IP 位址，以新增 IPv4 位址範圍。
8. 按一下 [檔案] 。

## <a name="azure-cli-20"></a>Azure CLI 2.0

1. [安裝 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli) 並[登入](https://docs.microsoft.com/cli/azure/authenticate-azure-cli)。

2. 列出可用的虛擬網路規則，如果您還沒有為此金鑰保存庫設定任何規則，清單會是空的。
```azurecli
az keyvault network-rule list --resource-group myresourcegroup --name mykeyvault
```

3. 在現有的虛擬網路和子網路上啟用 Key Vault 的服務端點
```azurecli
az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.KeyVault"
```

4. 為虛擬網路和子網路新增網路規則
```azurecli
subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
az keyvault network-rule add --resource-group "demo9311" --name "demo9311premium" --subnet $subnetid
```

5. 新增允許流量來自的 IP 位址範圍
```azurecli
az keyvault network-rule add --resource-group "myresourcegroup" --name "mykeyvault" --ip-address "191.10.18.0/24"
```

6. 如果有任何信任的服務需要存取此金鑰保存庫，請將設定 [略過] 設定為 AzureServices
```azurecli
az keyvault update --resource-group "myresourcegroup" --name "mykeyvault" --bypass AzureServices
```

7. 最後一個重要步驟，就是將預設動作設定為 [拒絕] 來開啟網路規則
```azurecli
az keyvault update --resource-group "myresourcegroup" --name "mekeyvault" --default-action Deny
```

## <a name="azure-powershell"></a>Azure PowerShell

1. 安裝最新的 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) 並[登入](https://docs.microsoft.com/powershell/azure/authenticate-azureps)。

2. 列出可用的虛擬網路規則，如果您還沒有為此金鑰保存庫設定任何規則，清單會是空的。
```PowerShell
(Get-AzureRmKeyVault -VaultName "mykeyvault").NetworkAcls
```

3. 在現有的虛擬網路和子網路上啟用 Key Vault 的服務端點
```PowerShell
Get-AzureRmVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzureRmVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.1.1.0/24" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzureRmVirtualNetwork
```

4. 為虛擬網路和子網路新增網路規則
```PowerShell
$subnet = Get-AzureRmVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzureRmVirtualNetworkSubnetConfig -Name "mysubnet"
Add-AzureRmKeyVaultNetworkRule -VaultName "mykeyvault" -VirtualNetworkResourceId $subnet.Id
```

5. 新增允許流量來自的 IP 位址範圍
```PowerShell
Add-AzureRmKeyVaultNetworkRule -VaultName "mykeyvault" -IpAddressRange "16.17.18.0/24"
```

6. 如果有任何信任的服務需要存取此金鑰保存庫，請將設定 [略過] 設定為 AzureServices
```PowerShell
Update-AzureRmKeyVaultNetworkRuleSet -VaultName "mykeyvault" -Bypass AzureServices
```

7. 最後一個重要步驟，就是將預設動作設定為 [拒絕] 來開啟網路規則
```PowerShell
Update-AzureRmKeyVaultNetworkRuleSet -VaultName "mykeyvault" -DefaultAction Deny
```

## <a name="references"></a>參考

* Azure CLI 2.0 命令 - [az keyvault network-rule](https://docs.microsoft.com/cli/azure/keyvault/network-rule?view=azure-cli-latest)
* Azure PowerShell Cmdlet - [Get-AzureRmKeyVault](https://docs.microsoft.com/powershell/module/azurerm.keyvault/get-azurermkeyvault)、[Add-AzureRmKeyVaultNetworkRule](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Add-AzureRmKeyVaultNetworkRule)、[Remove-AzureRmKeyVaultNetworkRule](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Remove-AzureRmKeyVaultNetworkRule)、[Update-AzureRmKeyVaultNetworkRuleSet](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Update-AzureRmKeyVaultNetworkRuleSet)

## <a name="next-steps"></a>後續步驟

* [Key Vault 的虛擬網路服務端點](key-vault-overview-vnet-service-endpoints.md)
* [保護您的金鑰保存庫](key-vault-secure-your-key-vault.md)