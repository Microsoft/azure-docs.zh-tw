---
title: 使用 SMB 在 Linux VM 上掛接 Azure 檔案儲存體 | Microsoft Docs
description: 如何透過 Azure CLI 使用 SMB 在 Linux VM 上掛接 Azure 檔案儲存體
services: virtual-machines-linux
documentationcenter: virtual-machines-linux
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/28/2018
ms.author: cynthn
ms.openlocfilehash: 7cd7f0f37f0d351d1d50d4c15e7132f072b5125d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46982200"
---
# <a name="mount-azure-file-storage-on-linux-vms-using-smb"></a>使用 SMB 在 Linux VM 上掛接 Azure 檔案儲存體

本文說明如何透過 Azure CLI，利用 SMB 掛接在 Linux VM 上使用 Azure 檔案儲存體服務。 Azure 檔案儲存體可在雲端中使用標準的 SMB 通訊協定提供檔案共用。 

檔案儲存體可在使用標準 SMB 通訊協定的雲端中提供檔案共用。 您可以從任何支援 SMB 3.0 的作業系統掛接檔案共用。 在 Linux 上使用 SMB 掛接時，您可以輕鬆地將備份放到有 SLA 支援的強大、永久封存儲存體位置。

將檔案從 VM 移至裝載在檔案儲存體上的 SMB 掛接，可方便您對記錄進行偵錯。 相同的 SMB 共用可以掛接在 Mac、Linux 或 Windows 工作站的本機上。 SMB 並非即時串流 Linux 或應用程式記錄的最佳解決方案，因為 SMB 通訊協定的建置目的不是要處理如此繁重的記錄職責。 專用的整合記錄層級工具，像是 Fluentd，是比透過 SMB 收集 Linux 和應用程式記錄輸出更好的選擇。

本指南要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 **az --version** 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 


## <a name="create-a-resource-group"></a>建立資源群組

在「美國東部」位置建立名為 myResourceGroup 的資源群組。

```bash
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

使用 [az storage account create](/cli/azure/storage/account#create)，在您所建立的資源群組中建立新的儲存體帳戶。 此範例會建立名為 *mySTORAGEACCT<random number>* 的儲存體帳戶，並且在變數 **STORAGEACCT** 中放入該儲存體帳戶的名稱。 儲存體帳戶名稱必須是唯一的，請使用 `$RANDOM` 在結尾附加一個數字，使其成為唯一名稱。

```bash
STORAGEACCT=$(az storage account create \
    --resource-group "myResourceGroup" \
    --name "mystorageacct$RANDOM" \
    --location eastus \
    --sku Standard_LRS \
    --query "name" | tr -d '"')
```

## <a name="get-the-storage-key"></a>取得儲存體金鑰

在建立儲存體帳戶時，帳戶金鑰會成對建立，因此您可以輪替金鑰，而不會干擾到服務。 當您將金鑰切換為金鑰組中的第二個金鑰時，您會建立新的金鑰組。 新的儲存體帳戶金鑰一律會成對建立，所以您永遠有至少一個準備切換到的未使用儲存體帳戶金鑰。

使用 [az storage account keys list](/cli/azure/storage/account/keys#list) 檢視儲存體帳戶金鑰。 此範例會在 **STORAGEKEY** 變數中儲存金鑰 1 的值。

```bash
STORAGEKEY=$(az storage account keys list \
    --resource-group "myResourceGroup" \
    --account-name $STORAGEACCT \
    --query "[0].value" | tr -d '"')
```

## <a name="create-a-file-share"></a>建立檔案共用

使用 [az storage share create](/cli/azure/storage/share#create) 建立檔案儲存體共用。 

共用名稱必須全部使用小寫字母、數字和單一連字號，但開頭不可以是連字號。 如需有關為檔案共用與檔案命名的完整詳細資料，請參閱 [命名和參考共用、目錄、檔案及中繼資料](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata)。

此範例會建立名為 myshare 的共用 (具有 10-GiB 配額)。 

```bash
az storage share create --name myshare \
    --quota 10 \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY
```

## <a name="create-a-mount-point"></a>建立掛接點

若要在 Linux 電腦上掛接 Azure 檔案共用，您必須確定您已安裝 **cifs-utils** 套件。 如需安裝指示，請參閱[在您的 Linux 散發套件上安裝 cifs-utils 套件](../../storage/files/storage-how-to-use-files-linux.md#install-cifs-utils)。

Azure 檔案服務會使用 SMB 通訊協定，其透過 TCP 通訊埠 445 進行通訊。  如果您在掛接 Azure 檔案共用時發生問題，請確定您的防火牆不會封鎖 TCP 通訊埠 445。


```bash
mkdir -p /mnt/MyAzureFileShare
```

## <a name="mount-the-share"></a>掛接共用

將 Azure 檔案共用裝載在本機目錄。 

```bash
sudo mount -t cifs //$STORAGEACCT.file.core.windows.net/myshare /mnt/MyAzureFileShare -o vers=3.0,username=$STORAGEACCT,password=$STORAGEKEY,dir_mode=0777,file_mode=0777,serverino
```



## <a name="persist-the-mount"></a>保存掛接

當您重新開機 Linux VM 時，掛接的 SMB 共用會在關機期間取消掛接。 若要在開機時重新掛接 SMB 共用，請新增一行至 Linux /etc/fstab。 Linux 會使用 fstab 檔案來列出它在開機程序期間所必須掛接的檔案系統。 新增 SMB 共用可確保檔案儲存體共用是 Linux VM 的永久掛接檔案系統。 當您使用 cloud-init 時，便可以將檔案儲存體 SMB 共用新增至新的 VM。

```bash
//myaccountname.file.core.windows.net/mystorageshare /mnt/mymountpoint cifs vers=3.0,username=mystorageaccount,password=myStorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777
```
為了提高生產環境的安全性，您應該將您的認證儲存在 fstab 之外。

## <a name="next-steps"></a>後續步驟

- [在建立期間使用 cloud-init 自訂 Linux VM](using-cloud-init.md)
- [在 Linux VM 中新增磁碟](add-disk.md)
- [使用 Azure CLI 將 Linux VM 上的磁碟加密](encrypt-disks.md)

