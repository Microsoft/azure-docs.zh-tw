---
title: 使用 Azure 備份將檔案還原到 VM
description: 了解如何使用備份與復原服務，在 Azure VM 上執行檔案層級還原。
services: backup
author: markgalioto
manager: carmonm
tags: azure-resource-manager, virtual-machine-backup
ms.service: backup
ms.topic: tutorial
ms.date: 2/14/2018
ms.author: markgal
ms.custom: mvc
ms.openlocfilehash: 2bd5e3e022c0e780b1efede6facb8924fa6cde4f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998180"
---
# <a name="restore-files-to-a-virtual-machine-in-azure"></a>在 Azure 中將檔案還原到虛擬機器
Azure 備份會建立復原點，並儲存在異地備援復原保存庫。 當您從復原點還原時，可以還原整個 VM 或個別檔案。 本文詳細說明如何還原個別檔案。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 列出和選取復原點
> * 將復原點連線到 VM
> * 從復原點還原檔案

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.18 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 


## <a name="prerequisites"></a>必要條件
本教學課程需要已使用 Azure 備份保護的 Linux VM。 若要模擬意外刪除檔案和復原程序，您可以從網頁伺服器刪除頁面。 如果您需要執行網頁伺服器並已使用 Azure 備份保護的 Linux VM，請參閱[使用 CLI 在 Azure 中備份虛擬機器](quick-backup-vm-cli.md)。


## <a name="backup-overview"></a>備份概觀
當 Azure 起始備份時，VM 上的備份擴充功能會建立時間點快照集。 此備份擴充功能會在要求第一個備份時安裝在 VM 上。 如果進行備份時 VM 不在執行中，Azure 備份也可以建立基礎儲存體的快照集。

根據預設，Azure 備份會建立檔案系統一致的快照集。 Azure 備份建立快照集之後，資料會傳輸至復原服務保存庫。 為了能更有效率，Azure 備份只會找出並傳輸自上次備份之後有變更的資料區塊。

資料傳輸完畢後，系統會移除快照集並建立復原點。


## <a name="delete-a-file-from-a-vm"></a>從 VM 刪除檔案
如果您意外刪除或變更檔案，您可以從復原點還原個別檔案。 此程序可讓您瀏覽復原點中備份的檔案，只還原所需的檔案。 在此範例中，我們會從網頁伺服器刪除檔案，以示範檔案層級復原程序。

1. 若要連線到您的 VM，請使用 [az vm show](/cli/azure/vm?view=azure-cli-latest#az-vm-show) 取得 VM 的 IP 位址：

     ```azurecli-interactive
     az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
     ```

2. 若要確認您的網站目前運作正常，請將網頁瀏覽器開啟到 VM 的公用 IP 位址。 保持開啟網頁瀏覽器視窗。

    ![預設 NGINX 網頁](./media/tutorial-restore-files/nginx-working.png)

3. 透過 SSH 連線到您的 VM。 以您在上一個命令中取得的公用 IP 位址取代 *publicIpAddress*：

    ```bash
    ssh publicIpAddress
    ```

4. 從網頁伺服器刪除預設頁面 (*/var/www/html/index.nginx-debian.html*)，如下所示：

    ```bash
    sudo rm /var/www/html/index.nginx-debian.html
    ```

5. 在您的網頁瀏覽器中，重新整理網頁。 網站不會再載入頁面，如下列範例所示：

    ![NGINX 網站不會再載入預設頁面](./media/tutorial-restore-files/nginx-broken.png)

6. 關閉 VM 的 SSH 工作階段，如下所示：

    ```bash
    exit
    ```


## <a name="generate-file-recovery-script"></a>產生檔案復原指令碼
為了還原您的檔案，Azure 備份會提供指令碼，以在作為本機磁碟區連線到復原點的 VM 上執行。 您可以瀏覽此本機磁碟機、將檔案還原到 VM 本身，然後中斷復原點連線。 Azure 備份會根據排程和保留的指派原則，繼續備份您的資料。

1. 若要列出您 VM 的復原點，請使用 [az backup recoverypoint list](https://docs.microsoft.com/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-list)。 在此範例中，我們為 *myRecoveryServicesVault* 中受保護的 VM (名為 *myVM*) 選取最近的復原點：

    ```azurecli-interactive
    az backup recoverypoint list \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --query [0].name \
        --output tsv
    ```

2. 若要取得將復原點連線或掛接到 VM 的指令碼，請使用 [az backup restore files mount-rp](https://docs.microsoft.com/cli/azure/backup/restore/files?view=azure-cli-latest#az-backup-restore-files-mount-rp)。 下列範例會取得 *myRecoveryServicesVault* 中受保護之 VM (名為 *myVM*) 的指令碼。

    以您在上一個命令中取得的復原點名稱取代 *myRecoveryPointName*：

    ```azurecli-interactive
    az backup restore files mount-rp \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --rp-name myRecoveryPointName
    ```

    這會下載指令碼並顯示密碼，如下列範例所示：

    ```
    File downloaded: myVM_we_1571974050985163527.sh. Use password c068a041ce12465
    ```

3. 若要將指令碼傳送至您的 VM，請使用安全複製 (SCP)。 提供已下載指令碼的名稱，並以您 VM 的公用 IP 位址取代 *publicIpAddress*。 確定您在 SCP 命令結尾包含尾端 `:`，如下所示：

    ```bash
    scp myVM_we_1571974050985163527.sh 52.174.241.110:
    ```


## <a name="restore-file-to-your-vm"></a>將檔案還原到您的 VM
將復原指令碼複製到您的 VM 之後，您現在可以連線到復原點並還原檔案。

1. 透過 SSH 連線到您的 VM。 以您 VM 的公用 IP 位址取代 *publicIpAddress*，如下所示：

    ```bash
    ssh publicIpAddress
    ```

2. 為了讓您的指令碼正確執行，請使用 **chmod** 新增執行權限。 輸入您自己的指令碼名稱：

    ```bash
    chmod +x myVM_we_1571974050985163527.sh
    ```

3. 若要掛接復原點，請執行指令碼。 輸入您自己的指令碼名稱：

    ```bash
    ./myVM_we_1571974050985163527.sh
    ```

    當指令碼執行時，系統會提示您輸入密碼以存取復原點。 輸入透過上一個 [az backup restore files mount-rp](https://docs.microsoft.com/cli/azure/backup/restore/files?view=azure-cli-latest#az-backup-restore-files-mount-rp) 命令產生復原指令碼之輸出中所顯示的密碼。

    指令碼的輸出會提供復原點的路徑。 下列範例輸出顯示復原點掛接在 */home/azureuser/myVM-20170919213536/Volume1*：

    ```
    Microsoft Azure VM Backup - File Recovery
    ______________________________________________
    Please enter the password as shown on the portal to securely connect to the recovery point. : c068a041ce12465
    
    Connecting to recovery point using ISCSI service...
    
    Connection succeeded!
    
    Please wait while we attach volumes of the recovery point to this machine...
    
    ************ Volumes of the recovery point and their mount paths on this machine ************
    
    Sr.No.  |  Disk  |  Volume  |  MountPath
    
    1)  | /dev/sdc  |  /dev/sdc1  |  /home/azureuser/myVM-20170919213536/Volume1
    
    ************ Open File Explorer to browse for files. ************
    ```

4. 使用 **cp** 將 NGINX 預設網頁從已掛接的復原點複製回到原始檔案位置。 以您自己的位置取代 */home/azureuser/myVM-20170919213536/Volume1* 掛接點：

    ```bash
    sudo cp /home/azureuser/myVM-20170919213536/Volume1/var/www/html/index.nginx-debian.html /var/www/html/
    ```

6. 在您的網頁瀏覽器中，重新整理網頁。 網站現在會正確載入，如下列範例所示：

    ![NGINX 網站現在會正確載入](./media/tutorial-restore-files/nginx-restored.png)

7. 關閉 VM 的 SSH 工作階段，如下所示：

    ```bash
    exit
    ```

8. 使用 [az backup restore files unmount-rp](https://docs.microsoft.com/cli/azure/backup/restore/files?view=azure-cli-latest#az-backup-restore-files-unmount-rp) 從您的 VM 卸載復原點。 下列範例會從 *myRecoveryServicesVault* 中名為 *myVM* 的 VM 卸載復原點。

    以您在先前命令中取得的復原點名稱取代 *myRecoveryPointName*：
    
    ```azurecli-interactive
    az backup restore files unmount-rp \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --rp-name myRecoveryPointName
    ```

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已將復原點連線到 VM 和網頁伺服器的已還原檔案。 您已了解如何︰

> [!div class="checklist"]
> * 列出和選取復原點
> * 將復原點連線到 VM
> * 從復原點還原檔案

前進到下一個教學課程，以了解如何將 Windows Server 備份到 Azure。

> [!div class="nextstepaction"]
> [將 Windows Server 備份到 Azure](tutorial-backup-windows-server-to-azure.md)

