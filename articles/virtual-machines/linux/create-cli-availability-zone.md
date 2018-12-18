---
title: 使用 Azure CLI 建立分區 Linux VM | Microsoft Docs
description: 使用 Azure CLI 在可用性區域中建立 Linux VM
services: virtual-machines-linux
documentationcenter: virtual-machines
author: dlepow
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/05/2018
ms.author: danlep
ms.custom: ''
ms.openlocfilehash: c202379f236bcd2fea05ad9d135096bc724898e7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956416"
---
# <a name="create-a-linux-virtual-machine-in-an-availability-zone-with-the-azure-cli"></a>使用 Azure CLI 在可用性區域中建立 Linux 虛擬機器

本文逐步引導您使用 Azure CLI 在可用性區域中建立 Linux VM。 [可用性區域](../../availability-zones/az-overview.md)是指 Azure 區域內實際上分隔的區域。 萬一整個資料中心失敗或遺失，使用可用性區域可保護您的應用程式和資料免於受害。

若要使用可用性區域，請在[支援的 Azure 區域](../../availability-zones/az-overview.md#regions-that-support-availability-zones)中建立虛擬機器。

請確定您已安裝最新的 [Azure CLI](/cli/azure/install-az-cli2) 並使用 [az login](/cli/azure/reference-index#az_login) 登入 Azure 帳戶。


## <a name="check-vm-sku-availability"></a>檢查 VM SKU 可用性
VM 大小或 SKU 的可用性可能因地區和區域而有所不同。 為了協助您規劃如何利用可用性區域，您可以依照 Azure 地區和區域列出可用的 VM SKU。 這項功能可確保您選擇適當的 VM 大小，並取得所需的跨區域復原功能。 如需有關不同 VM 類型和大小的詳細資訊，請參閱 [VM 大小概觀](sizes.md)。

您可以使用 [az vm list-skus](/cli/azure/vm#az_vm_list_skus) 命令檢視可用的 VM SKU。 下列範例會列出 eastus2 地區中可用的 VM SKU：

```azurecli
az vm list-skus --location eastus2 --output table
```

其輸出類似下列扼要的範例，會顯示每個 VM 大小適用的可用性區域：

```azurecli
ResourceType      Locations  Name               [...]    Tier       Size     Zones
----------------  ---------  -----------------           ---------  -------  -------
virtualMachines   eastus2    Standard_DS1_v2             Standard   DS1_v2   1,2,3
virtualMachines   eastus2    Standard_DS2_v2             Standard   DS2_v2   1,2,3
[...]
virtualMachines   eastus2    Standard_F1s                Standard   F1s      1,2,3
virtualMachines   eastus2    Standard_F2s                Standard   F2s      1,2,3
[...]
virtualMachines   eastus2    Standard_D2s_v3             Standard   D2_v3    1,2,3
virtualMachines   eastus2    Standard_D4s_v3             Standard   D4_v3    1,2,3
[...]
virtualMachines   eastus2    Standard_E2_v3              Standard   E2_v3    1,2,3
virtualMachines   eastus2    Standard_E4_v3              Standard   E4_v3    1,2,3
```


## <a name="create-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。  

Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 資源群組必須在虛擬機器之前建立。 在此範例中，會在 *eastus2* 區域中建立名為 *myResourceGroupVM* 的資源群組。 美國東部 2 是其中一個支援可用性區域的 Azure 區域。

```azurecli 
az group create --name myResourceGroupVM --location eastus2
```

在建立或修改 VM 時，會指定資源群組，在本文的整個過程中都可以看到此操作。

## <a name="create-virtual-machine"></a>建立虛擬機器

使用 [az vm create](/cli/azure/vm#az_vm_create) 命令建立虛擬機器。 

建立虛擬機器時，有數個可用的選項，例如作業系統映像、磁碟大小及系統管理認證。 在此範例中，是使用 myVM 名稱來建立執行 Ubuntu Server 的虛擬機器。 VM 是在可用性區域 *1* 中所建立。 依預設，會以 *Standard_DS1_v2* 大小建立 VM。

```azurecli-interactive 
az vm create --resource-group myResourceGroupVM --name myVM --location eastus2 --image UbuntuLTS --generate-ssh-keys --zone 1
```

系統可能需要幾分鐘的時間來建立 VM。 建立 VM 之後，Azure CLI 就會輸出 VM 的相關資訊。 請記下 `zones` 值，該值代表 VM 正在其中執行的可用性區域。 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus2",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroupVM",
  "zones": "1"
}
```

## <a name="confirm-zone-for-managed-disk-and-ip-address"></a>確認受控磁碟和 IP 位址的區域

在可用性區域中部署 VM 時，會在相同的可用性區域中部署 VM 的受控磁碟。 根據預設，也會在該區域中建立公用 IP 位址。 在下列範例中會取得這些資源的相關資訊。

若要確認 VM 的受控磁碟位於可用性區域，請使用 [az vm show](/cli/azure/vm#az_vm_show) 命令來傳回磁碟識別碼。在此範例中，磁碟會儲存在變數中並使用於稍後的步驟。 

```azurecli-interactive
osdiskname=$(az vm show -g myResourceGroupVM -n myVM --query "storageProfile.osDisk.name" -o tsv)
```
現在您可以取得受控磁碟的相關資訊：

```azurecli-interactive
az disk show --resource-group myResourceGroupVM --name $osdiskname
```

輸出會顯示與 VM 相同的可用性區域中的受控磁碟：

```azurecli
{
  "creationData": {
    "createOption": "FromImage",
    "imageReference": {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westeurope/Publishers/Canonical/ArtifactTypes/VMImage/Offers/UbuntuServer/Skus/16.04-LTS/Versions/latest",
      "lun": null
    },
    "sourceResourceId": null,
    "sourceUri": null,
    "storageAccountId": null
  },
  "diskSizeGb": 30,
  "encryptionSettings": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/disks/osdisk_761c570dab",
  "location": "eastus2",
  "managedBy": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "name": "myVM_osdisk_761c570dab",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroupVM",
  "sku": {
    "name": "Premium_LRS",
    "tier": "Premium"
  },
  "tags": {},
  "timeCreated": "2018-03-05T22:16:06.892752+00:00",
  "type": "Microsoft.Compute/disks",
  "zones": [
    "1"
  ]
}
```

使用 [az vm list-ip-addresses](/cli/azure/vm#az_vm_list_ip_addresses) 命令，以傳回 *myVM* 中公用 IP 位址資源的名稱。 在此範例中，名稱會儲存在變數中並使用於稍後的步驟。

```azurecli
ipaddressname=$(az vm list-ip-addresses -g myResourceGroupVM -n myVM --query "[].virtualMachine.network.publicIpAddresses[].name" -o tsv)
```

現在您可以取得 IP 位址的相關資訊：

```azurecli
az network public-ip show --resource-group myResourceGroupVM --name $ipaddressname
```

輸出會顯示與 VM 相同的可用性區域中的 IP 位址：

```azurecli
{
  "dnsSettings": null,
  "etag": "W/\"b7ad25eb-3191-4c8f-9cec-c5e4a3a37d35\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/publicIPAddresses/myVMPublicIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "52.174.34.95",
  "ipConfiguration": {
    "etag": null,
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM",
    "name": null,
    "privateIpAddress": null,
    "privateIpAllocationMethod": null,
    "provisioningState": null,
    "publicIpAddress": null,
    "resourceGroup": "myResourceGroupVM",
    "subnet": null
  },
  "location": "eastUS2",
  "name": "myVMPublicIP",
  "provisioningState": "Succeeded",
  "publicIpAddressVersion": "IPv4",
  "publicIpAllocationMethod": "Dynamic",
  "resourceGroup": "myResourceGroupVM",
  "resourceGuid": "8c70a073-09be-4504-0000-000000000000",
  "tags": {},
  "type": "Microsoft.Network/publicIPAddresses",
  "zones": [
    "1"
  ]
}
```

## <a name="next-steps"></a>後續步驟

在本文中，您已學到如何在可用性區域中建立 VM。 深入了解 Azure VM 的[區域和可用性](regions-and-availability.md)。




