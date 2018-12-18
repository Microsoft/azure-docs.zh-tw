---
title: 透過 Azure CLI 使用 Linux 針對 VM 進行疑難排解 | Microsoft Docs
description: 了解如何使用 Azure CLI 將 OS 磁碟連接至復原 VM，以針對 Linux VM 問題進行疑難排解
services: virtual-machines-linux
documentationCenter: ''
authors: genlin
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/16/2017
ms.author: genli
ms.openlocfilehash: 2f10a231c4edc3e01d3f8c5f7f4db1854f83044d
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49392397"
---
# <a name="troubleshoot-a-linux-vm-by-attaching-the-os-disk-to-a-recovery-vm-with-the-azure-cli"></a>透過 Azure CLI 將 OS 磁碟連結到復原 VM，以對 Linux VM 進行疑難排解
如果 Linux 虛擬機器 (VM) 發生開機或磁碟錯誤，您可能需要對虛擬硬碟本身執行疑難排解步驟。 常見的例子是 `/etc/fstab` 中的項目無效，導致 VM 無法成功開機。 本文詳細說明如何使用 Azure CLI 將虛擬硬碟連接至另一個 Linux VM，以修正任何錯誤，然後重新建立原始 VM。 


## <a name="recovery-process-overview"></a>復原程序概觀
疑難排解程序如下所示︰

1. 刪除遇到問題的 VM，保住虛擬硬碟。
2. 將虛擬硬碟連結和掛接至另一個 Linux VM，以進行疑難排解。
3. 連接至疑難排解 VM。 編輯檔案或執行任何工具來修正原始虛擬硬碟的問題。
4. 從疑難排解 VM 卸載並中斷連結虛擬硬碟。
5. 使用原始虛擬硬碟建立 VM。

若為使用受控磁碟的 VM，請參閱[藉由連結新的 OS 磁碟來針對受控磁碟 VM 進行疑難排解](#troubleshoot-a-managed-disk-vm-by-attaching-a-new-os-disk)。

若要執行這些疑難排解步驟，您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並且使用 [az login](/cli/azure/reference-index#az_login) 登入 Azure 帳戶。

在下列範例中，請以您自己的值取代參數名稱。 範例參數名稱包含 `myResourceGroup`、`mystorageaccount` 和 `myVM`。


## <a name="determine-boot-issues"></a>判斷開機問題
檢查序列輸出來判斷 VM 為何無法正常開機。 常見的例子是 `/etc/fstab` 中的項目無效，或因為刪除或移動基礎虛擬硬碟。

使用 [az vm boot-diagnostics get-boot-log](/cli/azure/vm/boot-diagnostics#az_vm_boot_diagnostics_get_boot_log) 取得與開機記錄。 下列範例會從資源群組 `myResourceGroup` 中的 VM `myVM` 取得序列輸出：

```azurecli
az vm boot-diagnostics get-boot-log --resource-group myResourceGroup --name myVM
```

檢閱序列輸出來判斷 VM 為何無法開機。 如果序列輸出未提供任何指示，您可能需要將虛擬硬碟連接至疑難排解 VM，然後檢閱 `/var/log` 中的記錄檔。


## <a name="view-existing-virtual-hard-disk-details"></a>檢視現有的虛擬硬碟詳細資料
您需要先識別 OS 磁碟的 URI，才能將虛擬硬碟 (VHD) 連結至另一個 VM。 

使用 [az vm show](/cli/azure/vm#az_vm_show) 檢視 VM 的相關資訊。 使用 `--query` 旗標擷取 OS 磁碟的 URI。 下列範例會取得資源群組 `myResourceGroup` 中 VM `myVM` 的磁碟資訊：

```azurecli
az vm show --resource-group myResourceGroup --name myVM \
    --query [storageProfile.osDisk.vhd.uri] --output tsv
```

URI 類似於 **https://mystorageaccount.blob.core.windows.net/vhds/myVM.vhd**。

## <a name="delete-existing-vm"></a>刪除現有的 VM
虛擬硬碟和 VM 在 Azure 中是兩個不同的資源。 虛擬硬碟中儲存作業系統本身、應用程式和設定。 VM 本身只是定義大小或位置的中繼資料，還會參考資源，例如虛擬硬碟或虛擬網路介面卡 (NIC)。 每個虛擬硬碟連結至 VM 時會獲派租用。 雖然即使 VM 正在執行時也可以連結和中斷連結資料磁碟，但除非刪除 VM 資源，否則無法中斷連結 OS 磁碟。 即使 VM 處於已停止和解除配置的狀態，租用仍會持續讓 OS 磁碟與 VM 產生關聯。

復原 VM 的第一個步驟是刪除 VM 資源本身。 刪除 VM 時，虛擬硬碟會留在儲存體帳戶中。 刪除 VM 之後，您需要將虛擬硬碟連結至另一個 VM，以進行疑難排解並解決錯誤。

使用 [az vm delete](/cli/azure/vm#az_vm_delete) 刪除 VM。 下列範例會從資源群組 `myResourceGroup` 中刪除 VM `myVM`：

```azurecli
az vm delete --resource-group myResourceGroup --name myVM 
```

請等到 VM 完成刪除之後，再將虛擬硬碟連結至另一個 VM。 在虛擬硬碟上，將它與 VM 產生關聯的租用必須先釋放，您才能將虛擬硬碟連結至另一個 VM。


## <a name="attach-existing-virtual-hard-disk-to-another-vm"></a>將現有的虛擬硬碟連結至另一個 VM
在接下來幾個步驟中，您將使用另一個 VM 進行疑難排解。 您將現有的虛擬硬碟連結至這個疑難排解 VM，以瀏覽並編輯磁碟的內容。 舉例來說，此程序可讓您更正任何設定錯誤，或檢閱其他應用程式記錄檔或系統記錄檔。 選擇或建立另一個 VM 以進行疑難排解。

使用 [az vm unmanaged-disk attach](/cli/azure/vm/unmanaged-disk#az_vm_unmanaged_disk_attach) 連結現有虛擬硬碟。 當您連結現有的虛擬硬碟時，請指定先前的 `az vm show` 命令取得的磁碟 URI。 下列範例會將現有的虛擬硬碟連結至資源群組 `myResourceGroup` 中的疑難排解 VM `myVMRecovery`：

```azurecli
az vm unmanaged-disk attach --resource-group myResourceGroup --vm-name myVMRecovery \
    --vhd-uri https://mystorageaccount.blob.core.windows.net/vhds/myVM.vhd
```


## <a name="mount-the-attached-data-disk"></a>掛接已連結的資料磁碟

> [!NOTE]
> 下列範例詳細說明 Ubuntu VM 上所需的步驟。 如果您使用不同的 Linux 發行版本，例如 Red Hat Enterprise Linux 或 SUSE，則記錄檔位置和 `mount` 命令可能稍微不同。 請參閱您的特定發行版本的文件，以了解命令中相應的變更。

1. 使用適當的認證以 SSH 登入疑難排解 VM。 如果此磁碟是第一個連結至疑難排解 VM 的資料磁碟，則磁碟很可能連結至 `/dev/sdc`。 使用 `dmseg` 來檢視已連結的磁碟︰

    ```bash
    dmesg | grep SCSI
    ```

    輸出類似於下列範例：

    ```bash
    [    0.294784] SCSI subsystem initialized
    [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
    [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
    [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
    [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
    ```

    在上述範例中，OS 磁碟位於 `/dev/sda`，而提供給每個 VM 的暫存磁碟位於 `/dev/sdb`。 如果您有多個資料磁碟，它們應該是位於 `/dev/sdd`、`/dev/sde`，依此類推。

2. 建立目錄來掛接現有的虛擬硬碟。 下列範例會建立名為 `troubleshootingdisk` 的目錄：

    ```bash
    sudo mkdir /mnt/troubleshootingdisk
    ```

3. 如果您在現有的虛擬硬碟上有多個磁碟分割，請掛接所需的磁碟分割。 下列範例會將第一個主要磁碟分割掛接在 `/dev/sdc1`：

    ```bash
    sudo mount /dev/sdc1 /mnt/troubleshootingdisk
    ```

    > [!NOTE]
    > 最佳做法是使用虛擬硬碟的通用唯一識別碼 (UUID)，將資料磁碟掛接在 Azure 中的 VM。 在這個簡短的疑難排解案例中，不需要使用 UUID 來掛接虛擬硬碟。 但在正常使用情況下，如果編輯 `/etc/fstab` 來使用裝置名稱掛接虛擬硬碟，而不是使用 UUID，可能會造成 VM 無法開機。


## <a name="fix-issues-on-original-virtual-hard-disk"></a>修正原始虛擬硬碟的問題
已掛接現有的虛擬硬碟掛，您現在可以視需要執行任何維護和疑難排解步驟。 解決問題之後，請繼續進行下列步驟。


## <a name="unmount-and-detach-original-virtual-hard-disk"></a>卸載並中斷連結原始虛擬硬碟
一旦解決錯誤，您就要從疑難排解 VM 卸載中斷連結並現有的虛擬硬碟。 直到將虛擬硬碟連結至疑難排解 VM 的租用釋放，您才能將虛擬硬碟用於其他任何 VM。

1. 從疑難排解 VM 的 SSH 工作階段，卸載現有的虛擬硬碟。 首先離開掛接點的上層目錄︰

    ```bash
    cd /
    ```

    現在卸載現有的虛擬硬碟。 下列範例會卸載位於 `/dev/sdc1` 的裝置：

    ```bash
    sudo umount /dev/sdc1
    ```

2. 現在從 VM 中斷連結虛擬硬碟。 結束疑難排解 VM 的 SSH 工作階段。 使用 [az vm unmanaged-disk list](/cli/azure/vm/unmanaged-disk#az_vm_unmanaged_disk_list) 列出連結至疑難排解 VM 的資料磁碟。 下列範例會列出連結至資源群組 `myResourceGroup` 中 VM `myVMRecovery` 的資料磁碟：

    ```azurecli
    azure vm unmanaged-disk list --resource-group myResourceGroup --vm-name myVMRecovery \
        --query '[].{Disk:vhd.uri}' --output table
    ```

    記下現有虛擬硬碟的名稱。 例如，URI 為 **https://mystorageaccount.blob.core.windows.net/vhds/myVM.vhd** 的磁碟名稱為 **myVHD**。 

    使用 [az vm unmanaged-disk detach](/cli/azure/vm/unmanaged-disk#az_vm_unmanaged_disk_detach) 從 VM 卸載資料磁碟。 下列範例會將磁碟 `myVHD` 從資源群組 `myResourceGroup` 中的 VM `myVMRecovery` 刪除：

    ```azurecli
    az vm unmanaged-disk detach --resource-group myResourceGroup --vm-name myVMRecovery \
        --name myVHD
    ```


## <a name="create-vm-from-original-hard-disk"></a>從原始硬碟建立 VM
若要從原始虛擬硬碟建立 VM，請使用[這個 Azure Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-specialized-vhd)。 實際的 JSON 範本位於下列連結︰

- https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json

此範本會使用先前以命令取得的 VHD URI 來部署 VM。 使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create) 部署範本。 提供原始 VHD 的 URI，然後指定 OS 類型、VM 大小和 VM 名稱，如下所示︰

```azurecli
az group deployment create --resource-group myResourceGroup --name myDeployment \
  --parameters '{"osDiskVhdUri": {"value": "https://mystorageaccount.blob.core.windows.net/vhds/myVM.vhd"},
    "osType": {"value": "Linux"},
    "vmSize": {"value": "Standard_DS1_v2"},
    "vmName": {"value": "myDeployedVM"}}' \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-specialized-vhd/azuredeploy.json
```

## <a name="re-enable-boot-diagnostics"></a>重新啟用開機診斷
當您從現有的虛擬硬碟建立 VM 時，可能不會自動啟用開機診斷。 使用 [az vm boot-diagnostics enable](/cli/azure/vm/boot-diagnostics#az_vm_boot_diagnostics_enable) 啟用開機診斷。 下列範例會在資源群組 `myResourceGroup` 中的 VM `myDeployedVM` 上啟用診斷擴充：

```azurecli
az vm boot-diagnostics enable --resource-group myResourceGroup --name myDeployedVM
```

## <a name="troubleshoot-a-managed-disk-vm-by-attaching-a-new-os-disk"></a>藉由連結新的 OS 磁碟來針對受控磁碟 VM 進行疑難排解
1. 停止受影響的 VM。
2. 針對受控磁碟 VM 的 OS 磁碟[建立受控磁碟快照集](../linux/snapshot-copy-managed-disk.md)。
3. [從快照集建立受控磁碟](../scripts/virtual-machines-windows-powershell-sample-create-managed-disk-from-snapshot.md)。
4. [連結作為 VM 資料磁碟的受控磁碟](../windows/attach-disk-ps.md)。
5. [將步驟 4 的資料磁碟變更為 OS 磁碟](../windows/os-disk-swap.md)。

## <a name="next-steps"></a>後續步驟
如果連接至 VM 時發生問題，請參閱[針對 Azure VM 的 SSH 連接進行疑難排解](troubleshoot-ssh-connection.md)。 如果存取 VM 上執行的應用程式時發生問題，請參閱[針對 Linux VM 上的應用程式連線問題進行疑難排解](troubleshoot-app-connection.md)。

