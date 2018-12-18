---
title: 使用 Azure 傳統 CLI 建立 Linux VM | Microsoft Docs
description: 使用 Azure 傳統 CLI 在 Azure 上建立 Linux VM
services: virtual-machines-linux
documentationcenter: ''
author: vlivech
manager: jeconnoc
editor: ''
ms.assetid: facb1115-2b4e-4ef3-9905-330e42beb686
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/15/2016
ms.author: v-livech
ms.openlocfilehash: 94362256dfb5b8deff233f7c575d0720f29eec5c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987845"
---
# <a name="create-a-linux-vm-using-the-azure-classic-cli"></a>使用 Azure 傳統 CLI 建立 Linux VM

本文示範如何使用 Azure 命令列介面 (CLI) 中的 `azure vm quick-create` 命令，在 Azure 上快速部署 Linux 虛擬機器 (VM)。 `quick-create` 命令會將 VM 部署在基本且安全的基礎結構內，可讓您快速地建立原型或測試概念。

> [!NOTE]
若要使用 Azure CLI 來建立 VM，請參閱[使用 Azure CLI 來建立 VM](../windows/quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

您也可以使用 [Azure 入口網站](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)來快速部署 Linux VM。

本文需要 [SSH 公開金鑰和私密金鑰檔案](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="quick-commands"></a>快速命令

以下範例示範如何部署 CoreOS VM，並附加您的安全殼層 (SSH) 金鑰 (您的引數可能會不同)：

```azurecli
azure vm quick-create -M ~/.ssh/id_rsa.pub -Q CoreOS
```

## <a name="detailed-walkthrough"></a>詳細的逐步解說

下列逐步解說包含要部署的 UbuntuLTS VM，說明每個步驟的執行內容。

## <a name="vm-quick-create-aliases"></a>VM quick-create 別名

使用對應到常見作業系統散發版本的 Azure CLI 別名，可以快速選擇版本。 下表列出別名 (依 Azure CLI 0.10 版)。 所有使用 `quick-create` 建立的 VM 預設都是由固態硬碟 (SSD) 儲存空間支援，SSD 提供更快的佈建及高效能磁碟存取。 (這些別名代表 Azure 上可用 OS 散發版本的一小部分。 [在 PowerShell 中](../windows/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)、[Web 上](https://azure.microsoft.com/marketplace/virtual-machines/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)搜尋映像以在 Azure Marketplace 中尋找更多映像，或者[可以上傳您自己的自訂映像](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。)

| Alias | 發行者 | 供應項目 | SKU | 版本 |
|:--- |:--- |:--- |:--- |:--- |
| CentOS |OpenLogic |CentOS |7.2 |最新 |
| CoreOS |CoreOS |CoreOS |Stable |最新 |
| Debian |credativ |Debian |8 |最新 |
| openSUSE |SUSE |openSUSE |13.2 |最新 |
| RHEL |Red Hat |RHEL |7.2 |最新 |
| UbuntuLTS |Canonical |Ubuntu Server |14.04.4-LTS |最新 |

以下範例使用 **ImageURN** 選項 (`-Q`) 的 `UbuntuLTS` 別名來部署 Ubuntu 14.04.4 LTS Server。

先前的 `quick-create` 範例只呼叫 `-M` 旗標來識別要上傳的 SSH 公開金鑰，且同時停用 SSH 密碼，因此系統會提示您輸入下列引數：

* 資源群組名稱 (任何字串一般都適用於您的第一個 Azure 資源群組)
* VM 名稱
* 位置 (`westus` 或 `westeurope` 都是不錯的預設值)
* Linux (讓 Azure 知道您要的作業系統)
* username

下列範例會指定所有值，因此不會有進一步的提示。 只要您有 ssh-rsa 格式公開金鑰檔案的 `~/.ssh/id_rsa.pub`，它就會如預期運作：

```azurecli
azure vm quick-create \
  --resource-group myResourceGroup \
  --name myVM \
  --location westus \
  --os-type Linux \
  --admin-username myAdminUser \
  --ssh-publickey-file ~/.ssh/id_rsa.pub \
  --image-urn UbuntuLTS
```

輸出應該看起來像下列的輸出區塊：

```azurecli
info:    Executing command vm quick-create
+ Listing virtual machine sizes available in the location "westus"
+ Looking up the VM "myVM"
info:    Verifying the public key SSH file: /Users/ahmet/.ssh/id_rsa.pub
info:    Using the VM Size "Standard_DS1"
info:    The [OS, Data] Disk or image configuration requires storage account
+ Looking up the storage account cli16330708391032639673
+ Looking up the NIC "examp-westu-1633070839-nic"
info:    An nic with given name "examp-westu-1633070839-nic" not found, creating a new one
+ Looking up the virtual network "examp-westu-1633070839-vnet"
info:    Preparing to create new virtual network and subnet
/ Creating a new virtual network "examp-westu-1633070839-vnet" [address prefix: "10.0.0.0/16"] with subnet "examp-westu-1633070839-snet" [address prefix: "10.+.1.0/24"]
+ Looking up the virtual network "examp-westu-1633070839-vnet"
+ Looking up the subnet "examp-westu-1633070839-snet" under the virtual network "examp-westu-1633070839-vnet"
info:    Found public ip parameters, trying to setup PublicIP profile
+ Looking up the public ip "examp-westu-1633070839-pip"
info:    PublicIP with given name "examp-westu-1633070839-pip" not found, creating a new one
+ Creating public ip "examp-westu-1633070839-pip"
+ Looking up the public ip "examp-westu-1633070839-pip"
+ Creating NIC "examp-westu-1633070839-nic"
+ Looking up the NIC "examp-westu-1633070839-nic"
+ Looking up the storage account clisto1710997031examplev
+ Creating VM "myVM"
+ Looking up the VM "myVM"
+ Looking up the NIC "examp-westu-1633070839-nic"
+ Looking up the public ip "examp-westu-1633070839-pip"
data:    Id                              :/subscriptions/2<--snip-->d/resourceGroups/exampleResourceGroup/providers/Microsoft.Compute/virtualMachines/exampleVMName
data:    ProvisioningState               :Succeeded
data:    Name                            :exampleVMName
data:    Location                        :westus
data:    Type                            :Microsoft.Compute/virtualMachines
data:
data:    Hardware Profile:
data:      Size                          :Standard_DS1
data:
data:    Storage Profile:
data:      Image reference:
data:        Publisher                   :Canonical
data:        Offer                       :UbuntuServer
data:        Sku                         :14.04.4-LTS
data:        Version                     :latest
data:
data:      OS Disk:
data:        OSType                      :Linux
data:        Name                        :clic7fadb847357e9cf-os-1473374894359
data:        Caching                     :ReadWrite
data:        CreateOption                :FromImage
data:        Vhd:
data:          Uri                       :https://cli16330708391032639673.blob.core.windows.net/vhds/clic7fadb847357e9cf-os-1473374894359.vhd
data:
data:    OS Profile:
data:      Computer Name                 :myVM
data:      User Name                     :myAdminUser
data:      Linux Configuration:
data:        Disable Password Auth       :true
data:
data:    Network Profile:
data:      Network Interfaces:
data:        Network Interface #1:
data:          Primary                   :true
data:          MAC Address               :00-0D-3A-33-42-FB
data:          Provisioning State        :Succeeded
data:          Name                      :examp-westu-1633070839-nic
data:          Location                  :westus
data:            Public IP address       :138.91.247.29
data:            FQDN                    :examp-westu-1633070839-pip.westus.cloudapp.azure.com
data:
data:    Diagnostics Profile:
data:      BootDiagnostics Enabled       :true
data:      BootDiagnostics StorageUri    :https://clisto1710997031examplev.blob.core.windows.net/
data:
data:      Diagnostics Instance View:
info:    vm quick-create command OK
```

## <a name="log-in-to-the-new-vm"></a>登入新的 VM
使用輸出中所列的公用 IP 位址登入您的 VM。 您也可以使用其中列出的完整網域名稱 (FQDN)：

```bash
ssh -i ~/.ssh/id_rsa.pub ahmet@138.91.247.29
```

登入程序應該類似下列輸出區塊：

```bash
Warning: Permanently added '138.91.247.29' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.19.0-65-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Thu Sep  8 22:50:57 UTC 2016

  System load: 0.63              Memory usage: 2%   Processes:       81
  Usage of /:  39.6% of 1.94GB   Swap usage:   0%   Users logged in: 0

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

myAdminUser@myVM:~$
```

## <a name="next-steps"></a>後續步驟
`azure vm quick-create` 命令是快速部署 VM 的方法，讓您可以登入 bash 殼層並開始使用。 不過，使用 `vm quick-create` 不會提供您進一步的控制權，也無法讓您能夠建立更複雜的環境。  若要部署為您的基礎結構自訂的 Linux VM，您可以依照下列任一篇文章執行：

* [直接使用 Azure CLI 命令，建立自訂的 Linux VM 環境](create-cli-complete.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [使用範本在 Azure 上建立 SSH 保護的 Linux VM](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

您也可以[搭配使用 `docker-machine` Azure 驅動程式與各種命令以快速建立當作 Docker 主機的 Linux VM](docker-machine.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
