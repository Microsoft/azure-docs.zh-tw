---
title: 限制對 PaaS 資源的網路存取 - Azure CLI | Microsoft Docs
description: 在本文中，您會了解如何透過使用 Azure CLI 和虛擬網路服務端點來限制對 Azure 資源 (例如 Azure 儲存體和 Azure SQL Database) 的網路存取。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure-services
ms.date: 03/14/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: ce4ff3fe2917d4dc34718fccc740223df0c52e8e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970935"
---
# <a name="restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-the-azure-cli"></a>透過使用 Azure CLI 的虛擬網路服務端點來限制對 PaaS 資源的網路存取

虛擬網路服務端點可讓您限制某些 Azure 服務資源對虛擬網路子網路的網路存取。 您也可以移除對資源的網際網路存取。 服務端點提供從您虛擬網路到受支援 Azure 服務的直接連線，讓您可以使用虛擬網路的私人位址空間來存取 Azure 服務。 透過服務端點預定流向 Azure 資源的流量，一律會留在 Microsoft Azure 骨幹網路中。 在本文中，您將了解：

* 建立具有一個子網路的虛擬網路
* 新增子網路，並啟用服務端點
* 建立 Azure 資源，並僅允許從子網路對其進行網路存取
* 將虛擬機器 (VM) 部署到每個子網路
* 確認從子網路對資源的存取
* 確認從子網路和網際網路對資源的存取遭到拒絕

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.28 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="create-a-virtual-network"></a>建立虛擬網路

建立虛擬網路之前，您必須為虛擬網路以及在本文中建立的所有其他資源，建立資源群組。 使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立具有一個子網路的虛擬網路。

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Public \
  --subnet-prefix 10.0.0.0/24
```

## <a name="enable-a-service-endpoint"></a>啟用服務端點 

您只能針對支援服務端點的服務啟用服務端點。 使用 [az network vnet list-endpoint-services](/cli/azure/network/vnet#az_network_vnet_list_endpoint_services) 來檢視 Azure 位置中所提供之已啟用服務端點的服務。 下列範例會傳回 eastus 區域中所提供且已啟用服務端點的服務。 隨著更多 Azure 服務啟用服務端點，所傳回的服務清單會隨著時間成長。

```azurecli-interactive
az network vnet list-endpoint-services \
  --location eastus \
  --out table
``` 

使用 [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) 在虛擬網路中建立其他的子網路。 在此範例中，系統會針對子網路建立 Microsoft.Storage 的服務端點： 

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.1.0/24 \
  --service-endpoints Microsoft.Storage
```

## <a name="restrict-network-access-for-a-subnet"></a>限制子網路的網路存取

使用 [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) 建立網路安全性群組。 下列範例會建立名為 myNsgPrivate 的網路安全性群組。

```azurecli-interactive
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsgPrivate
```

請使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update) 將網路安全性群組與「私人」子網路建立關聯。 下列範例會將 myNsgPrivate 網路安全性群組與「私人」子網路建立關聯：

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Private \
  --resource-group myResourceGroup \
  --network-security-group myNsgPrivate
```

使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) 來建立安全性規則。 下列規則允許對指派給 Azure 儲存體服務之公用 IP 位址的輸出存取： 

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Allow-Storage-All \
  --access Allow \
  --protocol "*" \
  --direction Outbound \
  --priority 100 \
  --source-address-prefix "VirtualNetwork" \
  --source-port-range "*" \
  --destination-address-prefix "Storage" \
  --destination-port-range "*"

Each network security group contains several [default security rules](security-overview.md#default-security-rules). The rule that follows overrides a default security rule that allows outbound access to all public IP addresses. The `destination-address-prefix "Internet"` option denies outbound access to all public IP addresses. The previous rule overrides this rule, due to its higher priority, which allows access to the public IP addresses of Azure Storage.

az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Deny-Internet-All \
  --access Deny \
  --protocol "*" \
  --direction Outbound \
  --priority 110 \
  --source-address-prefix "VirtualNetwork" \
  --source-port-range "*" \
  --destination-address-prefix "Internet" \
  --destination-port-range "*"

The following rule allows SSH traffic inbound to the subnet from anywhere. The rule overrides a default security rule that denies all inbound traffic from the internet. SSH is allowed to the subnet so that connectivity can be tested in a later step.

az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 120 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "VirtualNetwork" \
  --destination-port-range "22"
```

## <a name="restrict-network-access-to-a-resource"></a>限制對資源的網路存取

如果資源是透過已針對服務端點啟用的 Azure 服務建立，則限制其網路存取的必要步驟會因為服務不同而有所差異。 請參閱個別服務的文件，以了解每個服務的特定步驟。 本文的其餘部分包含限制 Azure 儲存體帳戶網路存取的步驟 (以範例形式說明)。

### <a name="create-a-storage-account"></a>建立儲存體帳戶

使用 [az storage account create](/cli/azure/storage/account#az_storage_account_create) 建立 Azure 儲存體帳戶。 請將 `<replace-with-your-unique-storage-account-name>` 取代為在所有 Azure 位置間具有唯一性、長度介於 3-24 個字元，且僅使用數字和小寫字母的名稱。

```azurecli-interactive
storageAcctName="<replace-with-your-unique-storage-account-name>"

az storage account create \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2
```

建立儲存體帳戶之後，使用 [az storage account show-connection-string](/cli/azure/storage/account#az_storage_account_show_connection_string) 將儲存體帳戶的連接字串取出至變數中。 在稍後步驟中，會使用連接字串來建立檔案共用。

```azurecli-interactive
saConnectionString=$(az storage account show-connection-string \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)
```

<a name="account-key"></a>檢視變數的內容，並記下輸出中傳回的 **AccountKey** 值，因為在稍後步驟中會用到它。

```azurecli-interactive
echo $saConnectionString
```

### <a name="create-a-file-share-in-the-storage-account"></a>在儲存體帳戶中建立檔案共用

使用 [az storage share create](/cli/azure/storage/share#az_storage_share_create) 在儲存體帳戶中建立檔案共用。 在稍後步驟中，會裝載此檔案共用，以確認其網路存取。

```azurecli-interactive
az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString > /dev/null
```

### <a name="deny-all-network-access-to-a-storage-account"></a>拒絕所有對儲存體帳戶的網路存取

根據預設，儲存體帳戶會接受來自任何網路用戶端的網路連線。 若要限制對選取網路的存取，請使用 [az storage account update](/cli/azure/storage/account#az_storage_account_update) 將預設動作變更為「拒絕」。 一旦網路存取遭到拒絕後，就無法從任何網路存取儲存體帳戶。

```azurecli-interactive
az storage account update \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --default-action Deny
```

### <a name="enable-network-access-from-a-subnet"></a>啟用子網路的網路存取

使用 [az storage account network-rule add](/cli/azure/storage/account/network-rule#az_storage_account_network_rule_add) 允許從「私人」子網路對儲存體帳戶的網路存取。

```azurecli-interactive
az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName \
  --vnet-name myVirtualNetwork \
  --subnet Private
```
## <a name="create-virtual-machines"></a>建立虛擬機器

若要測試對儲存體帳戶的網路存取，請將 VM 部署至每個子網路。

### <a name="create-the-first-virtual-machine"></a>建立第一部虛擬機器

使用 [az vm create](/cli/azure/vm#az_vm_create) 在「公用」子網路中建立虛擬機器。 如果預設金鑰位置中還沒有 SSH 金鑰，此命令將會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPublic \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Public \
  --generate-ssh-keys
```

建立 VM 需要幾分鐘的時間。 建立虛擬機器之後，Azure CLI 會顯示類似下列範例的資訊： 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmPublic",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

請記下傳回之輸出中的 **publicIpAddress**。 在稍後的步驟中，這個位址可用來從網際網路存取虛擬機器。

### <a name="create-the-second-virtual-machine"></a>建立第二部虛擬機器

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPrivate \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Private \
  --generate-ssh-keys
```

建立 VM 需要幾分鐘的時間。 在建立之後，請記下傳回之輸出中的 **publicIpAddress**。 在稍後的步驟中，這個位址可用來從網際網路存取虛擬機器。

## <a name="confirm-access-to-storage-account"></a>確認對儲存體帳戶的存取

使用 SSH 連線到 myVmPrivate VM。 以 myVmPrivate VM 的公用 IP 位址取代 <publicIpAddress>。

```bash 
ssh <publicIpAddress>
```

建立裝載點的資料夾：

```bash
sudo mkdir /mnt/MyAzureFileShare
```

將 Azure 檔案共用裝載至您所建立的目錄。 執行下列命令之前，請將 `<storage-account-name>` 取代為帳戶名稱，並將 `<storage-account-key>` 取代為您在[建立儲存體帳戶](#create-a-storage-account)中取出的金鑰。

```bash
sudo mount --types cifs //<storage-account-name>.file.core.windows.net/my-file-share /mnt/MyAzureFileShare --options vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

您會收到 `user@myVmPrivate:~$` 提示字元。 Azure 檔案共用已成功裝載到 /mnt/MyAzureFileShare。

確認 VM 沒有任何以其他公用 IP 位址為目標的輸出連線：

```bash
ping bing.com -c 4
```

您不會收到回應，因為與「私人」子網路相關聯的網路安全性群組只允許對指派給 Azure 儲存體服務的位址進行輸出存取，此位址以外的公用 IP 位址一律不允許。

結束對 myVmPrivate VM 的 SSH 工作階段。

## <a name="confirm-access-is-denied-to-storage-account"></a>確認對儲存體帳戶的存取遭到拒絕

使用下列命令來建立使用 myVmPublic VM 的 SSH 工作階段。 以 myVmPublic VM 的公用 IP 位址取代 `<publicIpAddress>`： 

```bash 
ssh <publicIpAddress>
```

建立裝載點的目錄：

```bash
sudo mkdir /mnt/MyAzureFileShare
```

嘗試將 Azure 檔案共用裝載至您所建立的目錄。 本文假設您已部署最新版本的 Ubuntu。 如果您是使用舊版的 Ubuntu，請參閱 [Linux 上的裝載](../storage/files/storage-how-to-use-files-linux.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，以取得有關裝載檔案共用的其他指示。 執行下列命令之前，請將 `<storage-account-name>` 取代為帳戶名稱，並將 `<storage-account-key>` 取代為您在[建立儲存體帳戶](#create-a-storage-account)中取出的金鑰：

```bash
sudo mount --types cifs //storage-account-name>.file.core.windows.net/my-file-share /mnt/MyAzureFileShare --options vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

存取遭到拒絕，且您收到 `mount error(13): Permission denied` 錯誤，因為 myVmPublic VM 是部署在「公用」子網路中。 「公用」子網路沒有已啟用的服務端點可供 Azure 儲存體使用，且儲存體帳戶只允許「私人」子網路而不允許「公用」子網路的網路存取。

結束對 myVmPublic VM 的 SSH 工作階段。

嘗試從您的電腦使用 [az storage share list](/cli/azure/storage/share?view=azure-cli-latest#az_storage_share_list) 來檢視儲存體帳戶中的共用。 將 `<account-name>` 和 `<account-key>` 取代為儲存體帳戶名稱和[建立儲存體帳戶](#create-a-storage-account)中的金鑰：

```azurecli-interactive
az storage share list \
  --account-name <account-name> \
  --account-key <account-key>
```

存取遭到拒絕，且您收到「此要求未獲授權執行此作業」錯誤，因為您的電腦不在 MyVirtualNetwork 虛擬網路的「私人」子網路中。

## <a name="clean-up-resources"></a>清除資源

請使用 [az group delete](/cli/azure#az_group_delete) 來移除不再需要的資源群組以及其所包含的所有資源。

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在本文中，您已啟用虛擬網路子網路的服務端點。 您已了解可以針對使用多個 Azure 服務部署的資源啟用服務端點。 您已建立 Azure 儲存體帳戶，並將儲存體帳戶的網路存取限制為只能存取虛擬網路子網路內的資源。 若要深入了解服務端點，請參閱[服務端點概觀](virtual-network-service-endpoints-overview.md)和[管理子網路](virtual-network-manage-subnet.md)。

如果您的帳戶中有多個虛擬網路，您可以同時連線兩個虛擬網路，讓每個虛擬網路內的資源都可互相進行通訊。 若要了解作法，請參閱[連線虛擬網路](tutorial-connect-virtual-networks-cli.md)。