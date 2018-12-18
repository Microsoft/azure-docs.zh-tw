---
title: 使用 Azure CLI 上傳自訂 Linux 磁碟 | Microsoft Docs
description: 使用 Resource Manager 部署模型和 Azure CLI 來建立虛擬硬碟 (VHD) 並上傳至 Azure
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: a8c7818f-eb65-409e-aa91-ce5ae975c564
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 07/10/2017
ms.author: cynthn
ms.openlocfilehash: 1926f0bcf7efca786e97bd973601888e5a8d4463
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966498"
---
# <a name="upload-and-create-a-linux-vm-from-custom-disk-with-the-azure-cli"></a>使用 Azure CLI 從自訂磁碟上傳並建立 Linux VM

本文說明如何使用 Azure CLI，將虛擬硬碟 (VHD) 上傳至 Azure 儲存體帳戶，並從這個自訂磁碟建立 Linux VM。 這項功能可讓您安裝和設定 Linux 散發版本以符合您的需求，然後使用該 VHD 快速建立 Azure 虛擬機器 (VM)。

本主題會將儲存體帳戶用於最終的 VHD，但您也可以使用[受控磁碟](upload-vhd.md)來執行這些步驟。 

## <a name="quick-commands"></a>快速命令
如果您需要快速完成作業，下列章節詳細說明將 VHD 上傳至 Azure 的基本命令。 每個步驟的詳細資訊和內容可在文件其他地方找到，[從這裡開始](#requirements)。

請確定您已安裝最新的 [Azure CLI](/cli/azure/install-az-cli2) 並使用 [az login](/cli/azure/reference-index#az_login) 登入 Azure 帳戶。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 `myResourceGroup`、`mystorageaccount` 和 `mydisks`。

首先，使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。 下列範例會在 `WestUs` 位置建立名為 `myResourceGroup` 的資源群組：

```azurecli
az group create --name myResourceGroup --location westus
```

使用 [az storage account create](/cli/azure/storage/account#az_storage_account_create) 建立儲存體帳戶以存放您的虛擬磁碟。 下列範例會建立名為 `mystorageaccount` 的儲存體帳戶：

```azurecli
az storage account create --resource-group myResourceGroup --location westus \
  --name mystorageaccount --kind Storage --sku Standard_LRS
```

使用 [az storage account keys list](/cli/azure/storage/account/keys#az_storage_account_keys_list) 列出您的儲存體帳戶存取金鑰。 記下 `key1`：

```azurecli
az storage account keys list --resource-group myResourceGroup --account-name mystorageaccount
```

使用透過 [az storage container create](/cli/azure/storage/container#az_storage_container_create) 取得的儲存體金鑰在您的儲存體帳戶內建立容器。 下列範例會使用來自 `key1` 的儲存體金鑰值，建立名為 `mydisks` 的容器：

```azurecli
az storage container create --account-name mystorageaccount \
    --account-key key1 --name mydisks
```

最後，使用 [az storage blob upload](/cli/azure/storage/blob#az_storage_blob_upload) 將您的 VHD 上傳至您所建立的容器。 在 `/path/to/disk/mydisk.vhd` 下方為您的 VHD 指定本機路徑：

```azurecli
az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd
```

使用 [az vm create](/cli/azure/vm#az_vm_create)，為您的磁碟指定 URI (`--image`)。 下列範例會使用先前上傳的虛擬磁碟來建立名為 `myVM` 的 VM：

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --storage-account mystorageaccount --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --image https://mystorageaccount.blob.core.windows.net/mydisk/myDisks.vhd \
    --use-unmanaged-disk
```

目的地儲存體帳戶必須與您上傳虛擬磁碟的目的地帳戶相同。 您也需要指定或依據提示回答 **az vm create** 命令所需的所有其他參數，例如虛擬網路、公用 IP 位址、使用者名稱及 SSH 金鑰。 您可以深入了解[可用的 CLI Resource Manager 參數](../azure-cli-arm-commands.md#azure-vm-commands-to-manage-your-azure-virtual-machines)。

## <a name="requirements"></a>需求
若要完成下列步驟，您需要：

* **以 .vhd 檔案安裝的 Linux 作業系統** - 以 VHD 格式將 [Azure 背書的 Linux 散發套件](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (或參閱[非背書散發套件的資訊](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)) 安裝到虛擬磁碟。 有多項工具可用來建立 VM 和 VHD：
  * 安裝和設定 [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) 或 [KVM](http://www.linux-kvm.org/page/RunningKVM)，並小心使用 VHD 做為您的映像格式。 如有需要，您可以使用 `qemu-img convert` 來[轉換映像](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats)。
  * 您也可以在 [Windows 10](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install) 或 [Windows Server 2012/2012 R2](https://technet.microsoft.com/library/hh846766.aspx) 上使用 Hyper-V。

> [!NOTE]
> Azure 不支援較新的 VHDX 格式。 當您建立 VM 時，請指定 VHD 做為格式。 如有需要，您可以使用 [`qemu-img convert`](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) 或 [`Convert-VHD`](https://technet.microsoft.com/library/hh848454.aspx) PowerShell Cmdlet 將 VHDX 磁碟轉換為 VHD。 此外，Azure 不支援上傳動態 VHD，因此您必須將此類磁碟轉換成靜態 VHD 再上傳。 您可以在上傳至 Azure 的期間使用 [適用於 GO 的 Azure VHD 公用程式](https://github.com/Microsoft/azure-vhd-utils-for-go) 之類的工具來轉換動態磁碟。
> 
> 

* 從自訂磁碟建立的 VM 必須位於與磁碟本身相同的儲存體帳戶中
  * 建立儲存體帳戶和容器來存放您的自訂磁碟和所建立的 VM
  * 建立所有 VM 之後，您即可放心地刪除您的磁碟

請確定您已安裝最新的 [Azure CLI](/cli/azure/install-az-cli2) 並使用 [az login](/cli/azure/reference-index#az_login) 登入 Azure 帳戶。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 `myResourceGroup`、`mystorageaccount` 和 `mydisks`。

<a id="prepimage"> </a>

## <a name="prepare-the-disk-to-be-uploaded"></a>準備要上傳的磁碟
Azure 支援各種 Linux 散發套件 (請參閱 [背書的散發套件](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json))。 下列文章會逐步引導您了解如何準備 Azure 上支援的各種 Linux 散發套件：

* **[CentOS 型散發套件](create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Debian Linux](debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Oracle Linux](oracle-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Red Hat Enterprise Linux](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[SLES 和 openSUSE](suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Ubuntu](create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[其他：非背書散發套件](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**

如需有關為 Azure 準備 Linux 映像的更多一般秘訣，另請參閱 **[Linux 安裝注意事項](create-upload-generic.md#general-linux-installation-notes)**。

> [!NOTE]
> 只有在搭配[經 Azure 背書之 Linux 散發套件](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中＜支援的版本＞下指定的組態詳細資料來使用其中一個經背書的散發套件時，[Azure 平台 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/) 才適用於執行 Linux 的虛擬機器。
> 
> 

## <a name="create-a-resource-group"></a>建立資源群組
資源群組會以邏輯方式將所有 Azure 資源 (例如虛擬網路和儲存體) 結合在一起以支援您的虛擬機器。 如需有關資源群組的詳細資訊，請參閱[資源群組概觀](../../azure-resource-manager/resource-group-overview.md)。 在上傳您的自訂磁碟並建立 VM 之前，您必須先使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。

下列範例會在 `westus` 位置建立名為 `myResourceGroup` 的資源群組：

```azurecli
az group create --name myResourceGroup --location westus
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

使用 [az storage account create](/cli/azure/storage/account#az_storage_account_create) 為自訂磁碟和 VM 建立儲存體帳戶。 您從自訂磁碟建立、具有非受控磁碟的任何 VM 都必須位於與該磁碟相同的儲存體帳戶中。 

下列範例會在先前建立的資源群組中建立名為 `mystorageaccount` 的儲存體帳戶：

```azurecli
az storage account create --resource-group myResourceGroup --location westus \
  --name mystorageaccount --kind Storage --sku Standard_LRS
```

## <a name="list-storage-account-keys"></a>列出儲存體帳戶金鑰
Azure 會為每個儲存體帳戶產生兩個 512 位元的存取金鑰。 對儲存體帳戶進行驗證時 (例如為了執行寫入作業)，就會使用這些存取金鑰。 從 [這裡](../../storage/common/storage-account-manage.md#access-keys)深入了解如何管理對儲存體的存取。 您使用 [az storage account keys list](/cli/azure/storage/account/keys#az_storage_account_keys_list) 來檢視存取金鑰。

檢視您所建立儲存體帳戶的存取金鑰：

```azurecli
az storage account keys list --resource-group myResourceGroup --account-name mystorageaccount
```

輸出如下：

```azurecli
info:    Executing command storage account keys list
+ Getting storage account keys
data:    Name  Key                                                                                       Permissions
data:    ----  ----------------------------------------------------------------------------------------  -----------
data:    key1  d4XAvZzlGAgWdvhlWfkZ9q4k9bYZkXkuPCJ15NTsQOeDeowCDAdB80r9zA/tUINApdSGQ94H9zkszYyxpe8erw==  Full
data:    key2  Ww0T7g4UyYLaBnLYcxIOTVziGAAHvU+wpwuPvK4ZG0CDFwu/mAxS/YYvAQGHocq1w7/3HcalbnfxtFdqoXOw8g==  Full
info:    storage account keys list command OK
```
請記下 `key1` ，因為在接下來的步驟中，您將使用它來與您的儲存體帳戶互動。

## <a name="create-a-storage-container"></a>建立儲存體容器
您可以在儲存體帳戶內建立容器來組織您的磁碟，方法與您建立不同目錄來以邏輯方式組織本機檔案系統相同。 儲存體帳戶可以包含任意數目的容器。 使用 [az storage container create](/cli/azure/storage/container#az_storage_container_create) 來建立容器。

下列範例會建立名為 `mydisks` 的容器：

```azurecli
az storage container create \
    --account-name mystorageaccount \
    --name mydisks
```

## <a name="upload-vhd"></a>上傳 VHD
現在您可以使用 [az storage blob upload](/cli/azure/storage/blob#az_storage_blob_upload) 上傳您的自訂磁碟。 您上傳並將您的自訂磁碟儲存為分頁 Blob。

指定您的存取金鑰、您在上一個步驟中建立的容器，然後指定您本機電腦上自訂磁碟的路徑：

```azurecli
az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd
```

## <a name="create-the-vm"></a>建立 VM
若要使用非受控磁碟來建立 VM，請使用 [az vm create](/cli/azure/vm#az_vm_create) 指定磁碟的 URI (`--image`)。 下列範例會使用先前上傳的虛擬磁碟來建立名為 `myVM` 的 VM：

您需搭配 [az vm create](/cli/azure/vm#az_vm_create) 指定 `--image` 參數，以指向您的自訂磁碟。 請確保 `--storage-account` 與儲存您自訂磁碟的儲存體帳戶相符。 您不需使用與自訂磁碟相同的容器來儲存您的 VM。 上傳您的自訂磁碟之前，請確定會使用與先前步驟中相同的方式來建立任何額外的容器。

下列範例會從您的自訂磁碟建立名為 `myVM` 的 VM：

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --storage-account mystorageaccount --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --image https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd \
    --use-unmanaged-disk
```

您仍需指定或依據提示回答 **az vm create** 命令所需的所有其他參數，例如使用者名稱及 SSH 金鑰。


## <a name="resource-manager-template"></a>Resource Manager 範本
Azure Resource Manager 範本是「JavaScript 物件標記法」(JSON) 檔案，定義了您想要建置的環境。 範本會細分成不同的資源提供者，例如計算或網路。 您可以使用現有的範本或自行撰寫範本。 深入了解如何 [使用 Resource Manager 和範本](../../azure-resource-manager/resource-group-overview.md)。

在範本的 `Microsoft.Compute/virtualMachines` 提供者內，您會有一個包含 VM 組態詳細資料的 `storageProfile` 節點。 兩個要編輯的主要參數為 `image` 和 `vhd` URI，分別指向您的自訂磁碟和新 VM 的虛擬磁碟。 以下顯示使用自訂磁碟的 JSON 範例：

```json
"storageProfile": {
          "osDisk": {
            "name": "myVM",
            "osType": "Linux",
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "image": {
              "uri": "https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd"
            },
            "vhd": {
              "uri": "https://mystorageaccount.blob.core.windows.net/vhds/newvmname.vhd"
            }
          }
```

您可以使用[這個現有的範本以從自訂映像建立 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image) 或閱讀[建立您自己的 Azure Resource Manager 範本](../../azure-resource-manager/resource-group-authoring-templates.md)。 

設定範本之後，請使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create) 來建立您的 VM。 請使用 `--template-uri` 參數來指定您 JSON 範本的 URI︰

```azurecli
az group deployment create --resource-group myNewResourceGroup \
  --template-uri https://uri.to.template/mytemplate.json
```

如果您已在電腦本機儲存 JSON 檔案，則可以改用 `--template-file` 參數︰

```azurecli
az group deployment create --resource-group myNewResourceGroup \
  --template-file /path/to/mytemplate.json
```


## <a name="next-steps"></a>後續步驟
在您備妥並上傳自訂虛擬磁碟之後，您可以深入了解如何 [使用 Resource Manager 和範本](../../azure-resource-manager/resource-group-overview.md)。 您也可以 [將資料磁碟新增](add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 至您的新 VM。 如果您需要存取在您的 VM 上執行的應用程式，請務必 [開啟連接埠和端點](nsg-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

