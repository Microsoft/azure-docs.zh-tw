---
title: 在 Azure 虛擬網路中新增或移除子網委派
titlesuffix: Azure Virtual Network
description: 瞭解如何在 Azure 中新增或移除服務的委派子網。
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/06/2019
ms.author: kumud
ms.openlocfilehash: 2bb80ba421617d5fd1699826deda00e56f1e43af
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98943677"
---
# <a name="add-or-remove-a-subnet-delegation"></a>新增或移除子網委派

子網路委派提供明確的權限給服務以在部署服務時使用唯一識別碼在子網路中建立服務特定資源。 本文說明如何新增或移除 Azure 服務的委派子網。

## <a name="portal"></a>入口網站

### <a name="sign-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 登入 Azure 入口網站。

### <a name="create-the-virtual-network"></a>建立虛擬網路

在本節中，您會建立虛擬網路和子網，您稍後將會將其委派給 Azure 服務。

1. 在畫面的左上方，選取 [**建立資源**  >  **網路**  >  **虛擬網路**]。
1. 在 [建立虛擬網路] 中，輸入或選取這項資訊：

    | 設定 | 值 |
    | ------- | ----- |
    | 名稱 | 輸入 *MyVirtualNetwork*。 |
    | 位址空間 | 輸入 *10.0.0.0/16*。 |
    | 訂用帳戶 | 選取您的訂用帳戶。|
    | 資源群組 | 選取 [新建]，輸入 *myResourceGroup*，然後選取 [確定]。 |
    | Location | 選取 [ **EastUS**]。|
    | 子網路 - 名稱 | 輸入 *>mysubnet*。 |
    | 子網路 - 位址範圍 | 輸入 *10.0.0.0/24*。 |
    |||
1. 將其餘部分保留為預設值，然後選取 [ **建立**]。

### <a name="permissions"></a>權限

如果您未建立想要委派給 Azure 服務的子網，則需要下列許可權： `Microsoft.Network/virtualNetworks/subnets/write` 。

內建的 [網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) 角色也包含必要的許可權。

### <a name="delegate-a-subnet-to-an-azure-service"></a>將子網委派給 Azure 服務

在本節中，您會將您在上一節中建立的子網委派給 Azure 服務。

1. 在入口網站的搜尋列中，輸入 *myVirtualNetwork*。 當搜尋結果中出現 **myVirtualNetwork** 時加以選取。
2. 在搜尋結果中，選取 [ *myVirtualNetwork*]。
3. 在 [**設定**] 底下選取 [**子網**]，然後選取 [ **>mysubnet**]。
4. 在 [ *>mysubnet* ] 頁面上，針對 [ **子網委派** ] 清單，選取 [ **將子網委派給服務** ] 下所列的服務 (例如 **DBforPostgreSQL/serversv2**) 。  

### <a name="remove-subnet-delegation-from-an-azure-service"></a>從 Azure 服務移除子網委派

1. 在入口網站的搜尋列中，輸入 *myVirtualNetwork*。 當搜尋結果中出現 **myVirtualNetwork** 時加以選取。
2. 在搜尋結果中，選取 [ *myVirtualNetwork*]。
3. 在 [**設定**] 底下選取 [**子網**]，然後選取 [ **>mysubnet**]。
4. 在 [ *>mysubnet* ] 頁面的 [**子網委派**] 清單中，從 [**委派子網至服務**] 底下列出的服務中選取 [**無**]。 

## <a name="azure-cli"></a>Azure CLI

備妥環境以使用 Azure CLI。

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- 本文需要 2.0.28 版或更新版本的 Azure CLI。 如果您是使用 Azure Cloud Shell，就已安裝最新版本。

### <a name="create-a-resource-group"></a>建立資源群組
使用 [az group create](/cli/azure/group) 來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例會在 eastus  位置建立名為 myResourceGroup  的資源群組：

```azurecli-interactive

  az group create \
    --name myResourceGroup \
    --location eastus

```

### <a name="create-a-virtual-network"></a>建立虛擬網路
使用 [az network vnet create](/cli/azure/network/vnet)，在 **myResourceGroup** 中建立名為 **myVNet**、具有子網路 **mySubnet** 的虛擬網路。

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myVnet \
    --address-prefix 10.0.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 10.0.0.0/24
```
### <a name="permissions"></a>權限

如果您未建立想要委派給 Azure 服務的子網，則需要下列許可權： `Microsoft.Network/virtualNetworks/subnets/write` 。

內建的 [網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) 角色也包含必要的許可權。

### <a name="delegate-a-subnet-to-an-azure-service"></a>將子網委派給 Azure 服務

在本節中，您會將您在上一節中建立的子網委派給 Azure 服務。 

使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) ，以 Azure 服務的委派更新名為 **>mysubnet** 的子網。  在此範例中，會使用 **DBforPostgreSQL/serversv2** 作為範例委派：

```azurecli-interactive
  az network vnet subnet update \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --delegations Microsoft.DBforPostgreSQL/serversv2
```

若要確認已套用委派，請使用 [az network vnet subnet show](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-show)。 確認服務已委派給屬性 **serviceName** 下的子網：

```azurecli-interactive
  az network vnet subnet show \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --query delegations
```

```json
[
  {
    "actions": [
      "Microsoft.Network/virtualNetworks/subnets/join/action"
    ],
    "etag": "W/\"8a8bf16a-38cf-409f-9434-fe3b5ab9ae54\"",
    "id": "/subscriptions/3bf09329-ca61-4fee-88cb-7e30b9ee305b/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet/delegations/0",
    "name": "0",
    "provisioningState": "Succeeded",
    "resourceGroup": "myResourceGroup",
    "serviceName": "Microsoft.DBforPostgreSQL/serversv2",
    "type": "Microsoft.Network/virtualNetworks/subnets/delegations"
  }
]
```

### <a name="remove-subnet-delegation-from-an-azure-service"></a>從 Azure 服務移除子網委派

使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) 從名為 **>mysubnet** 的子網中移除委派：

```azurecli-interactive
  az network vnet subnet update \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --remove delegations
```
若要確認已移除委派，請使用 [az network vnet subnet show](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-show)。 確認服務已從屬性 **serviceName** 的子網中移除：

```azurecli-interactive
  az network vnet subnet show \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --query delegations
```
命令的輸出為 null 括弧：
```json
[]
```

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="connect-to-azure"></a>連線到 Azure

```azurepowershell-interactive
  Connect-AzAccount
```

### <a name="create-a-resource-group"></a>建立資源群組
使用 [New-AzResourceGroup](/cli/azure/group) 來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例會在 eastus  位置建立名為 myResourceGroup  的資源群組：

```azurepowershell-interactive
  New-AzResourceGroup -Name myResourceGroup -Location eastus
```
### <a name="create-virtual-network"></a>建立虛擬網路

使用 [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork)，在 **myResourceGroup** 中建立名為 **myVnet** 的虛擬網路，並使用 [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) 建立名為 **mySubnet** 的子網路。 虛擬網路的 IP 位址空間是 **10.0.0.0/16**。 虛擬網路內的子網為 **10.0.0.0/24**。  

```azurepowershell-interactive
  $subnet = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix "10.0.0.0/24"

  New-AzVirtualNetwork -Name myVnet -ResourceGroupName myResourceGroup -Location eastus -AddressPrefix "10.0.0.0/16" -Subnet $subnet
```
### <a name="permissions"></a>權限

如果您未建立想要委派給 Azure 服務的子網，則需要下列許可權： `Microsoft.Network/virtualNetworks/subnets/write` 。

內建的 [網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) 角色也包含必要的許可權。

### <a name="delegate-a-subnet-to-an-azure-service"></a>將子網委派給 Azure 服務

在本節中，您會將您在上一節中建立的子網委派給 Azure 服務。 

使用 [AzDelegation](/powershell/module/az.network/add-azdelegation) ，將名為 **>mysubnet** 的子網，以名為 **MyDelegation** 的委派更新至 Azure 服務。  在此範例中，會使用 **DBforPostgreSQL/serversv2** 作為範例委派：

```azurepowershell-interactive
  $vnet = Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName "myResourceGroup"
  $subnet = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $vnet
  $subnet = Add-AzDelegation -Name "myDelegation" -ServiceName "Microsoft.DBforPostgreSQL/serversv2" -Subnet $subnet
  Set-AzVirtualNetwork -VirtualNetwork $vnet
```
使用 [AzDelegation](/powershell/module/az.network/get-azdelegation) 來驗證委派：

```azurepowershell-interactive
  $subnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup" | Get-AzVirtualNetworkSubnetConfig -Name "mySubnet"
  Get-AzDelegation -Name "myDelegation" -Subnet $subnet

  ProvisioningState : Succeeded
  ServiceName       : Microsoft.DBforPostgreSQL/serversv2
  Actions           : {Microsoft.Network/virtualNetworks/subnets/join/action}
  Name              : myDelegation
  Etag              : W/"9cba4b0e-2ceb-444b-b553-454f8da07d8a"
  Id                : /subscriptions/3bf09329-ca61-4fee-88cb-7e30b9ee305b/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet/delegations/myDelegation

```
### <a name="remove-subnet-delegation-from-an-azure-service"></a>從 Azure 服務移除子網委派

使用 [AzDelegation](/powershell/module/az.network/remove-azdelegation) 從名為 **>mysubnet** 的子網中移除委派：

```azurepowershell-interactive
  $vnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
  $subnet = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $vnet
  $subnet = Remove-AzDelegation -Name "myDelegation" -Subnet $subnet
  Set-AzVirtualNetwork -VirtualNetwork $vnet
```
使用 [AzDelegation](/powershell/module/az.network/get-azdelegation) 確認已移除委派：

```azurepowershell-interactive
  $subnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup" | Get-AzVirtualNetworkSubnetConfig -Name "mySubnet"
  Get-AzDelegation -Name "myDelegation" -Subnet $subnet

  Get-AzDelegation: Sequence contains no matching element

```

## <a name="next-steps"></a>後續步驟
- 瞭解如何 [在 Azure 中管理子網](virtual-network-manage-subnet.md)。
