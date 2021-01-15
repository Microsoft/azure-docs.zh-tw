---
title: 將資料遭到外泄限制為 Azure 儲存體-Azure CLI
description: 在本文中，您將瞭解如何使用 Azure CLI，以虛擬網路服務端點原則限制和限制虛擬網路資料遭到外泄，以 Azure 儲存體資源。
services: virtual-network
documentationcenter: virtual-network
author: rdhillon
manager: ''
editor: ''
tags: azure-resource-manager
Customer intent: I want only specific Azure Storage account to be allowed access from a virtual network subnet.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: rdhillon
ms.custom: ''
ms.openlocfilehash: 9ce1e320a93a834a938ce95f3931d885d2214faa
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98216852"
---
# <a name="manage-data-exfiltration-to-azure-storage-accounts-with-virtual-network-service-endpoint-policies-using-the-azure-cli"></a>使用 Azure CLI 以虛擬網路服務端點原則管理資料遭到外泄來 Azure 儲存體帳戶

虛擬網路服務端點原則可讓您從虛擬網路內的服務端點，將存取控制套用至 Azure 儲存體帳戶。 這是保護工作負載安全的關鍵，可管理允許的儲存體帳戶，以及允許資料遭到外泄的位置。
在本文中，您將學會如何：

* 建立虛擬網路並新增子網。
* 啟用 Azure 儲存體的服務端點。
* 建立兩個 Azure 儲存體帳戶，並允許從上述建立的子網進行網路存取。
* 建立服務端點原則，只允許存取其中一個儲存體帳戶。
* 將虛擬機器 (VM) 部署至子網。
* 確認從子網存取允許的儲存體帳戶。
* 確認從子網拒絕存取非允許的儲存體帳戶。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- 本文需要 2.0.28 版或更新版本的 Azure CLI。 如果您是使用 Azure Cloud Shell，就已安裝最新版本。

## <a name="create-a-virtual-network"></a>建立虛擬網路

建立虛擬網路之前，您必須為虛擬網路以及在本文中建立的所有其他資源，建立資源群組。 使用 [az group create](/cli/azure/group) 來建立資源群組。 下列範例會在 eastus  位置建立名為 myResourceGroup  的資源群組。

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

使用 [az network vnet create](/cli/azure/network/vnet) 建立具有一個子網路的虛擬網路。

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Private \
  --subnet-prefix 10.0.0.0/24
```

## <a name="enable-a-service-endpoint"></a>啟用服務端點 

在此範例中，會針對 *私人* 子網建立 *Microsoft* 的服務端點： 

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.0.0/24 \
  --service-endpoints Microsoft.Storage
```

## <a name="restrict-network-access-for-a-subnet"></a>限制子網路的網路存取

使用 [az network nsg create](/cli/azure/network/nsg) 建立網路安全性群組。 下列範例會建立名為 myNsgPrivate 的網路安全性群組。

```azurecli-interactive
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsgPrivate
```

請使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet) 將網路安全性群組與「私人」子網路建立關聯。 下列範例會將 myNsgPrivate 網路安全性群組與「私人」子網路建立關聯：

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Private \
  --resource-group myResourceGroup \
  --network-security-group myNsgPrivate
```

使用 [az network nsg rule create](/cli/azure/network/nsg/rule) 來建立安全性規則。 下列規則允許對指派給 Azure 儲存體服務之公用 IP 位址的輸出存取： 

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
```

每個網路安全性群組包含數個 [預設安全性規則](./network-security-groups-overview.md#default-security-rules)。 接下來的規則會覆寫允許對所有公用 IP 位址進行輸出存取的預設安全性規則。 `destination-address-prefix "Internet"`選項會拒絕對所有公用 IP 位址的輸出存取。 上一個規則會因其具有較高優先順序而覆寫這項規則，從而允許對 Azure 儲存體之公用 IP 位址的存取。

```azurecli-interactive
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
```

下列規則允許從任何地方輸入子網的 SSH 流量。 此規則會覆寫拒絕來自網際網路之所有輸入流量的預設安全性規則。 允許 SSH 到子網，以便在稍後的步驟中測試連線能力。

```azurecli-interactive
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

## <a name="restrict-network-access-to-azure-storage-accounts"></a>限制 Azure 儲存體帳戶的網路存取

本節列出的步驟可讓您從虛擬網路中的指定子網，透過服務端點來限制 Azure 儲存體帳戶的網路存取。

### <a name="create-a-storage-account"></a>建立儲存體帳戶

建立兩個使用 [az storage account create](/cli/azure/storage/account)的 Azure 儲存體帳戶。

```azurecli-interactive
storageAcctName1="allowedstorageacc"

az storage account create \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2

storageAcctName2="notallowedstorageacc"

az storage account create \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2
```

建立儲存體帳戶之後，將儲存體帳戶的連接字串取出至具有 [az 儲存體帳戶顯示連接字串](/cli/azure/storage/account)的變數中。 在稍後步驟中，會使用連接字串來建立檔案共用。

```azurecli-interactive
saConnectionString1=$(az storage account show-connection-string \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)

saConnectionString2=$(az storage account show-connection-string \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)
```

<a name="account-key"></a>檢視變數的內容，並記下輸出中傳回的 **AccountKey** 值，因為在稍後步驟中會用到它。

```azurecli-interactive
echo $saConnectionString1

echo $saConnectionString2
```

### <a name="create-a-file-share-in-the-storage-account"></a>在儲存體帳戶中建立檔案共用

使用 [az storage share create](/cli/azure/storage/share) 在儲存體帳戶中建立檔案共用。 在稍後步驟中，會裝載此檔案共用，以確認其網路存取。

```azurecli-interactive
az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString1 > /dev/null

az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString2 > /dev/null
```

### <a name="deny-all-network-access-to-the-storage-account"></a>拒絕所有的儲存體帳戶網路存取

根據預設，儲存體帳戶會接受來自任何網路用戶端的網路連線。 若要限制對選取網路的存取，請使用 [az storage account update](/cli/azure/storage/account) 將預設動作變更為「拒絕」。 一旦網路存取遭到拒絕後，就無法從任何網路存取儲存體帳戶。

```azurecli-interactive
az storage account update \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --default-action Deny

az storage account update \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --default-action Deny
```

### <a name="enable-network-access-from-virtual-network-subnet"></a>啟用虛擬網路子網的網路存取

使用 [az storage account network-rule add](/cli/azure/storage/account/network-rule) 允許從「私人」子網路對儲存體帳戶的網路存取。

```azurecli-interactive
az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName1 \
  --vnet-name myVirtualNetwork \
  --subnet Private

az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName2 \
  --vnet-name myVirtualNetwork \
  --subnet Private
```

## <a name="apply-policy-to-allow-access-to-valid-storage-account"></a>套用原則以允許存取有效的儲存體帳戶

Azure 服務端點原則僅適用于 Azure 儲存體。 因此，我們會針對此範例設定，在此子網上啟用 Microsoft 的服務端點 *。*

服務端點原則會套用至服務端點。 我們將從建立服務端點原則開始。 接著，我們會在此原則下建立原則定義，以針對此子網核准 Azure 儲存體帳戶

建立服務端點原則

```azurecli-interactive
az network service-endpoint policy create \
  --resource-group myResourceGroup \
  --name mysepolicy \
  --location eastus
```

將允許的儲存體帳戶的資源 URI 儲存在變數中。 在執行下列命令之前，請 *\<your-subscription-id>* 將取代為您的訂用帳戶識別碼的實際值。

```azurecli-interactive
$serviceResourceId="/subscriptions/<your-subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/allowedstorageacc"
```

建立 & 新增原則定義，以允許上述 Azure 儲存體帳戶新增至服務端點原則

```azurecli-interactive
az network service-endpoint policy-definition create \
  --resource-group myResourceGroup \
  --policy-name mysepolicy \
  --name mypolicydefinition \
  --service "Microsoft.Storage" \
  --service-resources $serviceResourceId
```

並更新虛擬網路子網，以將在上一個步驟中建立的服務端點原則與其相關聯

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --service-endpoints Microsoft.Storage \
  --service-endpoint-policy mysepolicy
```

## <a name="validate-access-restriction-to-azure-storage-accounts"></a>驗證 Azure 儲存體帳戶的存取限制

### <a name="create-the-virtual-machine"></a>建立虛擬機器

若要測試對儲存體帳戶的網路存取，請將 VM 部署至子網。

在具有 [az VM create](/cli/azure/vm)的 *私人* 子網中建立 VM。 如果預設金鑰位置中還沒有 SSH 金鑰，此命令將會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。

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

### <a name="confirm-access-to-storage-account"></a>確認對儲存體帳戶的存取

使用 SSH 連線到 myVmPrivate VM。 取代 *\<publicIpAddress>* 為 *myVmPrivate* VM 的公用 IP 位址。

```bash 
ssh <publicIpAddress>
```

建立裝載點的資料夾：

```bash
sudo mkdir /mnt/MyAzureFileShare1
```

將 Azure 檔案共用裝載至您所建立的目錄。 在執行下列命令之前，請將取代 *\<storage-account-key>* 為 **$saConnectionString 1** 的 *AccountKey* 值。

```bash
sudo mount --types cifs //allowedstorageacc.file.core.windows.net/my-file-share /mnt/MyAzureFileShare1 --options vers=3.0,username=allowedstorageacc,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

您會收到 `user@myVmPrivate:~$` 提示字元。 Azure 檔案共用已成功裝載到 /mnt/MyAzureFileShare。

### <a name="confirm-access-is-denied-to-storage-account"></a>確認對儲存體帳戶的存取遭到拒絕

從相同的 VM *myVmPrivate* 中，建立掛接點的目錄：

```bash
sudo mkdir /mnt/MyAzureFileShare2
```

嘗試從儲存體帳戶 *notallowedstorageacc* 將 Azure 檔案共用掛接到您所建立的目錄。 本文假設您已部署最新版本的 Ubuntu。 如果您是使用舊版的 Ubuntu，請參閱 [Linux 上的裝載](../storage/files/storage-how-to-use-files-linux.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，以取得有關裝載檔案共用的其他指示。 

在執行下列命令之前，請將取代 *\<storage-account-key>* 為 **$saConnectionString 2** 中的 *AccountKey* 值。

```bash
sudo mount --types cifs //notallowedstorageacc.file.core.windows.net/my-file-share /mnt/MyAzureFileShare2 --options vers=3.0,username=notallowedstorageacc,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

存取遭到拒絕，且您收到 `mount error(13): Permission denied` 錯誤，因為此儲存體帳戶不在我們套用至子網的服務端點原則的允許清單中。 

結束對 myVmPublic VM 的 SSH 工作階段。

## <a name="clean-up-resources"></a>清除資源

請使用 [az group delete](/cli/azure) 來移除不再需要的資源群組以及其所包含的所有資源。

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在本文中，您已透過 Azure 虛擬網路服務端點將服務端點原則套用至 Azure 儲存體。 您已建立 Azure 儲存體帳戶，且僅限特定儲存體帳戶的網路存取權 (，因而拒絕其他) 自虛擬網路子網的其他儲存體帳戶。 若要深入瞭解服務端點原則，請參閱 [服務端點原則總覽](virtual-network-service-endpoint-policies-overview.md)。