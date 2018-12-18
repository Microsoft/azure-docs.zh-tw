---
title: 建立 OpenBSD VM 映像並上傳至 Azure | Microsoft Docs
description: 了解如何建立及上傳包含 OpenBSD 作業系統的虛擬硬碟 (VHD)，以透過 Azure CLI 建立 Azure 虛擬機器
services: virtual-machines-linux
documentationcenter: ''
author: thomas1206
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 1ef30f32-61c1-4ba8-9542-801d7b18e9bf
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/24/2017
ms.author: huishao
ms.openlocfilehash: cbd8e6c1d12fe506e5c31c980b1ec13bb121e75e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958045"
---
# <a name="create-and-upload-an-openbsd-disk-image-to-azure"></a>建立 OpenBSD 磁碟映像並上傳至 Azure
本文說明如何建立及上傳包含 OpenBSD 作業系統的虛擬硬碟 (VHD)。 上傳之後，您可以使用它作為您自己的映像，在 Azure 中透過 Azure CLI 建立虛擬機器 (VM)。


## <a name="prerequisites"></a>必要條件
本文假設您具有下列項目：

* **Azure 訂用帳戶** - 如果您沒有，只需要幾分鐘的時間就可以建立帳戶。 如果您有 MSDN 訂用帳戶，請參閱 [Visual Studio 訂閱者的每月 Azure 點數](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)。 否則，請參閱 [建立免費試用帳戶](https://azure.microsoft.com/pricing/free-trial/)。  
* **Azure CLI** - 請確定您已安裝最新的 [Azure CLI](/cli/azure/install-azure-cli)，並使用 [az login](/cli/azure/reference-index#az_login) 登入 Azure 帳戶。
* **安裝在 .vhd 檔案中的 OpenBSD 作業系統** - 支援的 OpenBSD 作業系統 ([6.1 版 AMD64](https://ftp.openbsd.org/pub/OpenBSD/6.1/amd64/)) 必須已安裝到虛擬硬碟中。 有多項工具可用來建立 .vhd 檔案。 例如，您可以使用虛擬化解決方案 (例如 Hyper-V) 建立 .vhd 檔案，並安裝作業系統。 如需相關指示，請參閱 [安裝 Hyper-V 和建立虛擬機器](http://technet.microsoft.com/library/hh846766.aspx)。


## <a name="prepare-openbsd-image-for-azure"></a>為 Azure 準備 OpenBSD 映像
在您安裝 OpenBSD 作業系統 6.1 的 VM 上 (新增 Hyper-V 支援)，完成下列程序：

1. 如果在安裝期間未啟用 DHCP，請啟用服務，如下所示：

    ```sh    
    echo dhcp > /etc/hostname.hvn0
    ```

2. 設定序列主控台，如下所示：

    ```sh
    echo "stty com0 115200" >> /etc/boot.conf
    echo "set tty com0" >> /etc/boot.conf
    ```

3. 設定套件安裝，如下所示：

    ```sh
    echo "https://ftp.openbsd.org/pub/OpenBSD" > /etc/installurl
    ```
   
4. 在 Azure 中的虛擬機器上，依預設會停用 `root` 使用者。 使用者可以在 OpenBSD VM 上使用 `doas` 命令，以提高的權限來執行命令。 Doas 預設為啟用狀態。 如需詳細資訊，請參閱 [doas.conf](http://man.openbsd.org/doas.conf.5)。 

5. 安裝和設定 Azure 代理程式的必要條件，如下所示：

    ```sh
    pkg_add py-setuptools openssl git
    ln -sf /usr/local/bin/python2.7 /usr/local/bin/python
    ln -sf /usr/local/bin/python2.7-2to3 /usr/local/bin/2to3
    ln -sf /usr/local/bin/python2.7-config /usr/local/bin/python-config
    ln -sf /usr/local/bin/pydoc2.7  /usr/local/bin/pydoc
    ```

6. 最新版的 Azure 代理程式一律可以在 [Github](https://github.com/Azure/WALinuxAgent/releases) 上找到。 安裝代理程式，如下所示：

    ```sh
    git clone https://github.com/Azure/WALinuxAgent 
    cd WALinuxAgent
    python setup.py install
    waagent -register-service
    ```

    > [!IMPORTANT]
    > 安裝 Azure 代理程式之後，最好先確認它正在執行，如下所示︰
    >
    > ```bash
    > ps auxw | grep waagent
    > root     79309  0.0  1.5  9184 15356 p1  S      4:11PM    0:00.46 python /usr/local/sbin/waagent -daemon (python2.7)
    > cat /var/log/waagent.log
    > ```

7. 取消佈建系統以清理系統，使之適合重新佈建。 下列命令也會刪除最後佈建的使用者帳戶和相關聯的資料：

    ```sh
    waagent -deprovision+user -force
    ```

現在您可以關閉您的 VM。


## <a name="prepare-the-vhd"></a>準備 VHD
Azure 不支援 VHDX 格式，只支援 **固定 VHD**。 您可以使用 Hyper-V 管理員或 Powershell [convert-vhd](https://technet.microsoft.com/itpro/powershell/windows/hyper-v/convert-vhd) Cmdlet，將磁碟轉換為固定 VHD 格式。 範例如下。

```powershell
Convert-VHD OpenBSD61.vhdx OpenBSD61.vhd -VHDType Fixed
```

## <a name="create-storage-resources-and-upload"></a>建立儲存體資源並上傳
首先，使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

若要上傳 VHD，請使用 [az storage account create](/cli/azure/storage/account#az_storage_account_create) 建立儲存體帳戶。 儲存體帳戶名稱必須是唯一的，因此請提供您自己的名稱。 下列範例會建立名為 mystorageaccount 的儲存體帳戶：

```azurecli
az storage account create --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus \
    --sku Premium_LRS
```

若要控制儲存體帳戶的存取權，請使用 [az storage account keys list](/cli/azure/storage/account/keys#az_storage_account_keys_list) 取得儲存體金鑰，如下所示：

```azurecli
STORAGE_KEY=$(az storage account keys list \
    --resource-group myResourceGroup \
    --account-name mystorageaccount \
    --query "[?keyName=='key1']  | [0].value" -o tsv)
```

若要以邏輯方式分隔您上傳的 VHD，請使用 [az storage container create](/cli/azure/storage/container#az_storage_container_create) 在儲存體帳戶內建立容器：

```azurecli
az storage container create \
    --name vhds \
    --account-name mystorageaccount \
    --account-key ${STORAGE_KEY}
```

最後，使用 [az storage blob upload](/cli/azure/storage/blob#az_storage_blob_upload) 上傳 VHD，如下所示：

```azurecli
az storage blob upload \
    --container-name vhds \
    --file ./OpenBSD61.vhd \
    --name OpenBSD61.vhd \
    --account-name mystorageaccount \
    --account-key ${STORAGE_KEY}
```


## <a name="create-vm-from-your-vhd"></a>從 VHD 建立 VM
您可以使用[範例指令碼](../scripts/virtual-machines-linux-cli-sample-create-vm-vhd.md) 或直接使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 若要指定您上傳的 OpenBSD VHD，請使用 `--image` 參數，如下所示：

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myOpenBSD61 \
    --image "https://mystorageaccount.blob.core.windows.net/vhds/OpenBSD61.vhd" \
    --os-type linux \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

使用 [az vm list-ip-addresses](/cli/azure/vm#list-ip-addresses) 取得 OpenBSD VM 的 IP 位址，如下所示：

```azurecli
az vm list-ip-addresses --resource-group myResourceGroup --name myOpenBSD61
```

現在您可以像平常一樣 SSH 到您的 OpenBSD VM：
        
```bash
ssh azureuser@<ip address>
```


## <a name="next-steps"></a>後續步驟
如果您想要深入了解 OpenBSD6.1 上的 Hyper-V 支援，請參閱 [OpenBSD 6.1](https://www.openbsd.org/61.html) 和 [hyperv.4](http://man.openbsd.org/hyperv.4)。

如果您想要從受控磁碟建立 VM，請參閱 [az 磁碟](/cli/azure/disk)。 
