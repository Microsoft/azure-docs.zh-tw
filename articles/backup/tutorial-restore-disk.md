---
title: 使用 Azure 備份還原 VM 磁碟
description: 了解如何還原磁碟，並在 Azure 中使用備份與復原服務建立 VM。
services: backup
author: markgalioto
manager: carmonm
tags: azure-resource-manager, virtual-machine-backup
ms.service: backup
ms.topic: tutorial
ms.date: 4/17/2018
ms.author: markgal
ms.custom: mvc
ms.openlocfilehash: 6a6211302fd6810339ea9967dbcae75e1e413964
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987249"
---
# <a name="restore-a-disk-and-create-a-recovered-vm-in-azure"></a>在 Azure 中還原磁碟並建立已復原的 VM
Azure 備份會建立復原點，並儲存在異地備援復原保存庫。 當您從復原點還原時，可以還原整個 VM 或個別檔案。 本文說明如何使用 CLI 還原完整的 VM。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 列出和選取復原點
> * 從復原點還原磁碟
> * 從還原的磁碟建立 VM

如需使用 PowerShell 還原磁碟及建立已復原 VM 的相關資訊，請參閱[使用 PowerShell 備份及還原 Azure VM](backup-azure-vms-automation.md#restore-an-azure-vm)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.18 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 


## <a name="prerequisites"></a>必要條件
本教學課程需要已使用 Azure 備份保護的 Linux VM。 若要模擬意外刪除 VM 和復原程序，您可以在復原點從磁碟建立 VM。 如果您需要已使用 Azure 備份保護的 Linux VM，請參閱[使用 CLI 在 Azure 中備份虛擬機器](quick-backup-vm-cli.md)。


## <a name="backup-overview"></a>備份概觀
當 Azure 起始備份時，VM 上的備份擴充功能會建立時間點快照集。 此備份擴充功能會在要求第一個備份時安裝在 VM 上。 如果進行備份時 VM 不在執行中，Azure 備份也可以建立基礎儲存體的快照集。

根據預設，Azure 備份會建立檔案系統一致的快照集。 Azure 備份建立快照集之後，資料會傳輸至復原服務保存庫。 為了能更有效率，Azure 備份只會找出並傳輸自上次備份之後有變更的資料區塊。

資料傳輸完畢後，系統會移除快照集並建立復原點。


## <a name="list-available-recovery-points"></a>列出可用的復原點
若要還原磁碟，您可選取復原點作為復原資料的來源。 由於預設原則會每天建立復原點並保留 30 天，您可保留一組復原點，讓您選取特定的時間點進行復原。 

若要查看可用的復原點清單，請使用 [az backup recoverypoint list](https://docs.microsoft.com/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-list)。 復原點**名稱**用來復原磁碟。 在本教學課程中，我們需要最近可用的復原點。 `--query [0].name` 參數可選取最近的復原點名稱，如下所示：

```azurecli-interactive
az backup recoverypoint list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --query [0].name \
    --output tsv
```


## <a name="restore-a-vm-disk"></a>還原 VM 磁碟
若要從復原點還原您的磁碟，您可以先建立 Azure 儲存體帳戶。 此儲存體帳戶用來儲存已還原的磁碟。 在其他步驟中，已還原的磁碟用來建立 VM。

1. 若要建立儲存體帳戶，請使用 [az storage account create](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create)。 儲存體帳戶名稱必須全部小寫，並且是全域唯一的。 以自己的唯一名稱取代 *mystorageaccount*：

    ```azurecli-interactive
    az storage account create \
        --resource-group myResourceGroup \
        --name mystorageaccount \
        --sku Standard_LRS
    ```

2. 使用 [az backup restore restore-disks](https://docs.microsoft.com/cli/azure/backup/restore?view=azure-cli-latest#az-backup-restore-restore-disks) 從您的復原點還原磁碟。 以您在上一個命令中建立的儲存體帳戶名稱取代 *mystorageaccount*。 以您在先前 [az backup recoverypoint list](https://docs.microsoft.com/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-list) 命令的輸出中取得的復原點名稱取代 *myRecoveryPointName*：

    ```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName
    ```


## <a name="monitor-the-restore-job"></a>監視還原作業
若要監視還原作業的狀態，請使用 [az backup job list](https://docs.microsoft.com/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-list)：

```azurecli-interactive 
az backup job list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --output table
```

輸出會類似下列範例，其顯示的還原作業為 InProgress︰

```
Name      Operation        Status      Item Name    Start Time UTC       Duration
--------  ---------------  ----------  -----------  -------------------  --------------
7f2ad916  Restore          InProgress  myvm         2017-09-19T19:39:52  0:00:34.520850
a0a8e5e6  Backup           Completed   myvm         2017-09-19T03:09:21  0:15:26.155212
fe5d0414  ConfigureBackup  Completed   myvm         2017-09-19T03:03:57  0:00:31.191807
```

當還原作業的 [狀態] 回報 [已完成] 時，磁碟便已還原到儲存體帳戶。


## <a name="convert-the-restored-disk-to-a-managed-disk"></a>將還原的磁碟轉換成受控磁碟
還原作業會建立非受控磁碟。 若要從磁碟建立 VM，它必須先轉換成受控磁碟。

1. 使用 [az storage account show-connection-string](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-show-connection-string) 取得儲存體帳戶的連接字串。 以您的儲存體帳戶名稱取代 *mystorageaccount*，如下所示：
    
    ```azurecli-interactive
    export AZURE_STORAGE_CONNECTION_STRING=$( az storage account show-connection-string \
        --resource-group myResourceGroup \
        --output tsv \
        --name mystorageaccount )
    ```

2. 您的非受控磁碟會在儲存體帳戶中保護。 下列命令可取得非受控磁碟的相關資訊，並建立名為 *uri* 的變數，當您建立受控磁碟時，該變數使用於下一個步驟。

    ```azurecli-interactive
    container=$(az storage container list --query [0].name -o tsv)
    blob=$(az storage blob list --container-name $container --query [0].name -o tsv)
    uri=$(az storage blob url --container-name $container --name $blob -o tsv)
    ```

3. 您現在可以使用 [az disk create](https://docs.microsoft.com/cli/azure/disk?view=azure-cli-latest#az-disk-create)，從已復原的磁碟建立受控磁碟。 上一個步驟中的 *Uri* 變數會作為受控磁碟的來源。

    ```azurecli-interactive
    az disk create \
        --resource-group myResourceGroup \
        --name myRestoredDisk \
        --source $uri
    ```

4. 由於您現在有來自已還原磁碟的受控磁碟，請使用 [az storage account delete](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-delete) 清除非受控磁碟和儲存體帳戶。 以您的儲存體帳戶名稱取代 *mystorageaccount*，如下所示：

    ```azurecli-interactive
    az storage account delete \
        --resource-group myResourceGroup \
        --name mystorageaccount
    ```


## <a name="create-a-vm-from-the-restored-disk"></a>從還原的磁碟建立 VM
最後一個步驟是從受控磁碟建立 VM。

1. 使用 [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create) 從受控磁碟建立 VM，如下所示：

    ```azurecli-interactive
    az vm create \
        --resource-group myResourceGroup \
        --name myRestoredVM \
        --attach-os-disk myRestoredDisk \
        --os-type linux
    ```

2. 若要確認已從復原的磁碟建立 VM，請使用 [az vm list](/cli/azure/vm?view=azure-cli-latest#az-vm-list) 列出資源群組中的 VM，如下所示：

    ```azurecli-interactive
    az vm list --resource-group myResourceGroup --output table
    ```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已從復原點還原磁碟，並從磁碟建立 VM。 您已了解如何︰

> [!div class="checklist"]
> * 列出和選取復原點
> * 從復原點還原磁碟
> * 從還原的磁碟建立 VM

前進到下一個教學課程，了解如何從復原點還原個別的檔案。

> [!div class="nextstepaction"]
> [在 Azure 中將檔案還原到虛擬機器](tutorial-restore-files.md)

