---
title: 在 Azure Linux 虛擬機器上設定 Oracle ASM | Microsoft Docs
description: 快速在您的 Azure 環境中啟動並執行 Oracle ASM。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: romitgirdhar
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/02/2018
ms.author: rogirdh
ms.openlocfilehash: 236809336975eec94d7decd9822fc9143ae19bfb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981009"
---
# <a name="set-up-oracle-asm-on-an-azure-linux-virtual-machine"></a>在 Azure Linux 虛擬機器上設定 Oracle ASM  

Azure 虛擬機器提供完全可設定且彈性的計算環境。 本教學課程涵蓋基本的 Azure 虛擬機器部署，並結合 Oracle 自動儲存體管理 (ASM) 的安裝和設定。  您會了解如何：

> [!div class="checklist"]
> * 建立和連線至 Oracle 資料庫 VM
> * 安裝和設定 Oracle 自動儲存體管理
> * 安裝和設定 Oracle Grid Infrastructure
> * 初始化 Oracle ASM 安裝
> * 建立受 ASM 管理的 Oracle DB


[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="prepare-the-environment"></a>準備環境

### <a name="create-a-resource-group"></a>建立資源群組

若要建立資源群組，請使用 [az group create](/cli/azure/group#az_group_create) 命令。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 在此範例中，系統會在 eastus 區域中建立名為 myResourceGroup 的資源群組。

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

### <a name="create-a-vm"></a>建立 VM

若要建立以 Oracle 資料庫映像為基礎的虛擬機器，並將其設定為使用 Oracle ASM，請使用 [az vm create](/cli/azure/vm#az_vm_create) 命令。 

下列範例會建立名為 myVM 的 VM，其大小為 Standard_DS2_v2，並附加四個容量各 50 GB 的資料磁碟。 如果預設的金鑰位置還沒有 SSH 金鑰的話，此範例也會建立這些金鑰。  若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。  

   ```azurecli-interactive
   az vm create --resource-group myResourceGroup \
    --name myVM \
    --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
    --size Standard_DS2_v2 \
    --generate-ssh-keys \
    --data-disk-sizes-gb 50 50 50 50
   ```

在您建立 VM 後，Azure CLI 會顯示類似下列範例的資訊。 請記下 `publicIpAddress` 的值。 您必須使用此位址來存取 VM。

   ```azurecli
   {
     "fqdns": "",
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
     "location": "eastus",
     "macAddress": "00-0D-3A-36-2F-56",
     "powerState": "VM running",
     "privateIpAddress": "10.0.0.4",
     "publicIpAddress": "13.64.104.241",
     "resourceGroup": "myResourceGroup"
   }
   ```

### <a name="connect-to-the-vm"></a>連接至 VM

若要對 VM 建立 SSH 工作階段並進行其他設定，請使用下列命令。 以 VM 的 `publicIpAddress` 值取代 IP 位址。

```bash 
ssh <publicIpAddress>
```

## <a name="install-oracle-asm"></a>安裝 Oracle ASM

若要安裝 Oracle ASM，請完成下列步驟。 

如需關於安裝 Oracle ASM 的詳細資訊，請參閱 [Oracle ASMLib Downloads for Oracle Linux 6](http://www.oracle.com/technetwork/server-storage/linux/asmlib/ol6-1709075.html)。  

1. 您必須以根使用者身分登入才能繼續安裝 ASM：

   ```bash
   sudo su -
   ```
   
2. 執行這些其他命令以安裝 Oracle ASM 元件：

   ```bash
    yum list | grep oracleasm 
    yum -y install kmod-oracleasm.x86_64 
    yum -y install oracleasm-support.x86_64 
    wget http://download.oracle.com/otn_software/asmlib/oracleasmlib-2.0.12-1.el6.x86_64.rpm 
    yum -y install oracleasmlib-2.0.12-1.el6.x86_64.rpm 
    rm -f oracleasmlib-2.0.12-1.el6.x86_64.rpm
   ```

3. 確認您已安裝 Oracle ASM：

   ```bash
   rpm -qa |grep oracleasm
   ```

    此命令的輸出會列出下列元件：

    ```bash
   oracleasm-support-2.1.10-4.el6.x86_64
   kmod-oracleasm-2.0.8-15.el6_9.x86_64
   oracleasmlib-2.0.12-1.el6.x86_64
    ```

4. ASM 需要特定的使用者和角色才能正常運作。 下列命令會建立必要的使用者帳戶和群組： 

   ```bash
    groupadd -g 54345 asmadmin 
    groupadd -g 54346 asmdba 
    groupadd -g 54347 asmoper 
    useradd -u 3000 -g oinstall -G dba,asmadmin,asmdba,asmoper grid 
    usermod -g oinstall -G dba,asmdba,asmadmin oracle
   ```

5. 確認已正確建立使用者和群組：

   ```bash
   id grid
   ```

    此命令的輸出會列出下列使用者和群組：

    ```bash
    uid=3000(grid) gid=54321(oinstall) groups=54321(oinstall),54322(dba),54345(asmadmin),54346(asmdba),54347(asmoper)
    ```
 
6. 為使用者 grid 建立資料夾，並變更擁有者：

   ```bash
   mkdir /u01/app/grid 
   chown grid:oinstall /u01/app/grid
   ```

## <a name="set-up-oracle-asm"></a>設定 Oracle ASM

在此教學課程中，預設使用者是「grid」，預設群組是「asmadmin」。 確定「oracle」使用者屬於 asmadmin 群組。 若要設定 Oracle ASM 的安裝，請完成下列步驟︰

1. 設定 Oracle ASM 程式庫的驅動程式包含定義預設使用者 (grid) 和預設群組 (asmadmin)，以及設定在開機時啟動磁碟機 (選擇 y) 並在開機時掃描磁碟 (選擇 y)。 您必須回答下列命令中的提示：

   ```bash
   /usr/sbin/oracleasm configure -i
   ```

   此命令的輸出應類似下列結果，並停止於要回答的提示。

    ```bash
   Configuring the Oracle ASM library driver.

   This will configure the on-boot properties of the Oracle ASM library
   driver. The following questions will determine whether the driver is
   loaded on boot and what permissions it will have. The current values
   will be shown in brackets ('[]'). Hitting <ENTER> without typing an
   answer will keep that current value. Ctrl-C will abort.

   Default user to own the driver interface []: grid
   Default group to own the driver interface []: asmadmin
   Start Oracle ASM library driver on boot (y/n) [n]: y
   Scan for Oracle ASM disks on boot (y/n) [y]: y
   Writing Oracle ASM library driver configuration: done
   ```

2. 檢視磁碟組態︰
   ```bash
   cat /proc/partitions
   ```

   此命令的輸出應類似下列可用磁碟清單

   ```bash
   8       16   14680064 sdb
   8       17   14678976 sdb1
   8        0   52428800 sda
   8        1     512000 sda1
   8        2   51915776 sda2
   8       48   52428800 sdd
   8       64   52428800 sde
   8       80   52428800 sdf
   8       32   52428800 sdc
   11       0       1152 sr0
   ```

3. 透過執行下列命令格式化磁碟 /dev/sdc，並透過下列項目回答提示：
   - *n* 適用於新的磁碟分割
   - *p* 為主要磁碟分割
   - *1* 以選取第一個磁碟分割
   - 按 `enter` 為預設第一個磁柱
   - 按 `enter` 為預設最後一個磁柱
   - 按 *w* 將變更寫入磁碟分割表格Azure Resource Manager  

   ```bash
   fdisk /dev/sdc
   ```
   
   使用先前提供的答案，fdisk 命令的輸出如下所示：

   ```bash
   Device contains not a valid DOS partition table, or Sun, SGI or OSF disklabel
   Building a new DOS disklabel with disk identifier 0xf865c6ca.
   Changes will remain in memory only, until you decide to write them.
   After that, of course, the previous content won't be recoverable.

   Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

   The device presents a logical sector size that is smaller than
   the physical sector size. Aligning to a physical sector (or optimal
   I/O) size boundary is recommended, or performance may be impacted.

   WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
           switch off the mode (command 'c') and change display units to
           sectors (command 'u').

   Command (m for help): n
   Command action
     e   extended
     p   primary partition (1-4)
   p
   Partition number (1-4): 1
   First cylinder (1-6527, default 1):
   Using default value 1
   Last cylinder, +cylinders or +size{K,M,G} (1-6527, default 6527):
   Using default value 6527

   Command (m for help): w
   The partition table has been altered!

   Calling ioctl() to re-read partition table.
   Syncing disks.
   ```

4. 針對 `/dev/sdd`、`/dev/sde` 和 `/dev/sdf` 重複上述 fdisk 命令。

5. 檢查磁碟組態︰

   ```bash
   cat /proc/partitions
   ```

   此命令的輸出如下所示：

   ```bash
   major minor  #blocks  name

     8       16   14680064 sdb
     8       17   14678976 sdb1
     8       32   52428800 sdc
     8       33   52428096 sdc1
     8       48   52428800 sdd
     8       49   52428096 sdd1
     8       64   52428800 sde
     8       65   52428096 sde1
     8       80   52428800 sdf
     8       81   52428096 sdf1
     8        0   52428800 sda
     8        1     512000 sda1
     8        2   51915776 sda2
     11       0    1048575 sr0
   ```

6. 檢查 Oracle ASM 服務狀態，並啟動 Oracle ASM 服務：

   ```bash
   service oracleasm status 
   service oracleasm start
   ```

   此命令的輸出如下所示：
   
   ```bash
   Checking if ASM is loaded: no
   Checking if /dev/oracleasm is mounted: no
   Initializing the Oracle ASMLib driver:                     [  OK  ]
   Scanning the system for Oracle ASMLib disks:               [  OK  ]
   ```

7. 建立 Oracle ASM 磁碟︰

   ```bash
   service oracleasm createdisk ASMSP /dev/sdc1 
   service oracleasm createdisk DATA /dev/sdd1 
   service oracleasm createdisk DATA1 /dev/sde1 
   service oracleasm createdisk FRA /dev/sdf1
   ```    

   此命令的輸出如下所示：

   ```bash
   Marking disk "ASMSP" as an ASM disk:                       [  OK  ]
   Marking disk "DATA" as an ASM disk:                        [  OK  ]
   Marking disk "DATA1" as an ASM disk:                       [  OK  ]
   Marking disk "FRA" as an ASM disk:                         [  OK  ]
   ```

8. 列出 Oracle ASM 磁碟︰

   ```bash
   service oracleasm listdisks
   ```   

   此命令的輸出會列出下列 Oracle ASM 磁碟：

   ```bash
    ASMSP
    DATA
    DATA1
    FRA
   ```

9. 變更 Root、Oracle 和 Grid 使用者的密碼。 由於您稍後會在安裝期間使用這些新密碼，因此**請將其記下**。

   ```bash
   passwd oracle 
   passwd grid 
   passwd root
   ```

10. 變更資料夾權限︰

   ```bash
   chmod -R 775 /opt 
   chown grid:oinstall /opt 
   chown oracle:oinstall /dev/sdc1 
   chown oracle:oinstall /dev/sdd1 
   chown oracle:oinstall /dev/sde1 
   chown oracle:oinstall /dev/sdf1 
   chmod 600 /dev/sdc1 
   chmod 600 /dev/sdd1 
   chmod 600 /dev/sde1 
   chmod 600 /dev/sdf1
   ```

## <a name="download-and-prepare-oracle-grid-infrastructure"></a>下載並準備 Oracle Grid Infrastructure

若要下載並準備 Oracle Grid Infrastructure 軟體，請完成下列步驟︰

1. 從 [Oracle ASM 下載頁面](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/database12c-linux-download-2240591.html)下載 Oracle Grid Infrastructure。 

   在 **Oracle Database 12c Release 1 Grid Infrastructure (12.1.0.2.0) for Linux x86-64** 下載標題底下，下載兩個 .zip 檔案。

2. 將 .zip 檔下載到用戶端電腦之後，您可以使用「安全複製通訊協定 (SCP)」，將這些檔案複製到 VM：

   ```bash
   scp *.zip <publicIpAddress>:.
   ```

3. 透過 SSH 連接回 Oracle VM，以將 .zip 檔案移至 /opt 資料夾。 接著，變更檔案的擁有者︰

   ```bash
   ssh <publicIPAddress>
   sudo mv ./*.zip /opt
   cd /opt
   sudo chown grid:oinstall linuxamd64_12102_grid_1of2.zip
   sudo chown grid:oinstall linuxamd64_12102_grid_2of2.zip
   ```

4. 將檔案解壓縮。 (如果您尚未安裝 Linux 解壓縮工具，請加以安裝。)
   
   ```bash
   sudo yum install unzip
   sudo unzip linuxamd64_12102_grid_1of2.zip
   sudo unzip linuxamd64_12102_grid_2of2.zip
   ```

5. 變更權限：
   
   ```bash
   sudo chown -R grid:oinstall /opt/grid
   ```

6. 更新設定的交換空間。 Oracle Grid 元件需要至少 6.8 GB 的交換空間才能安裝 Grid。 Azure 中 Oracle Linux 映像的預設分頁檔大小只有 2048 MB。 您必須增加 `/etc/waagent.conf` 檔案中 `ResourceDisk.SwapSizeMB` 的值，並重新啟動 WALinuxAgent 服務，更新的設定才會生效。 由於其為唯讀檔案，因此您必須變更檔案權限才能啟用寫入權限。

   ```bash
   sudo chmod 777 /etc/waagent.conf  
   vi /etc/waagent.conf
   ```
   
   搜尋 `ResourceDisk.SwapSizeMB`，並將該值變更為 **8192**。 您必須按 `insert` 進入插入模式，輸入 **8192** 值，然後按 `esc` 返回命令模式。 若要寫入變更並結束檔案，請輸入 `:wq` 並按 `enter`。
   
   > [!NOTE]
   > 強烈建議您一律使用 `WALinuxAgent` 設定交換空間，讓空間一律建立在本機暫時磁碟 (暫存磁碟) 上，以獲得最佳效能。 如需詳細資訊，請參閱[如何在 Linux Azure 虛擬機器中新增分頁檔](https://support.microsoft.com/en-us/help/4010058/how-to-add-a-swap-file-in-linux-azure-virtual-machines)。

## <a name="prepare-your-local-client-and-vm-to-run-x11"></a>讓本機用戶端和 VM 準備執行 x11
設定 Oracle ASM 時，您需要圖形化介面才能完成安裝和設定。 我們會使用 x11 通訊協定簡化此安裝。 如果您正在使用已啟用且已設定 X11 功能的用戶端系統 (Mac 或 Linux)，則可以跳過這項特定於 Windows 電腦的組態和設定。 

1. 將 [PuTTY](http://www.putty.org/) 和 [Xming](https://xming.en.softonic.com/) 下載到 Windows 電腦。 繼續進行之前，您必須以預設值完成這兩個應用程式的安裝。

2. 安裝 PuTTY 之後，請開啟命令提示字元，變更至 PuTTY 資料夾 (例如，C:\Program Files\PuTTY)，並執行 `puttygen.exe` 以產生金鑰。

3. 在 PuTTY 金鑰產生器中︰
   
   1. 選取 `Generate` 按鈕以產生金鑰。
   2. 複製金鑰的內容 (Ctrl+C)。
   3. 選取 `Save private key` 按鈕。
   4. 忽略與透過複雜密碼保護金鑰相關的警告，然後選取 `OK`。

   ![PuTTY 金鑰產生器的螢幕擷取畫面](./media/oracle-asm/puttykeygen.png)

4. 在您的 VM 中，執行下列命令︰

   ```bash
   sudo su - grid
   mkdir .ssh 
   cd .ssh
   ```

5. 建立名為 `authorized_keys` 的檔案。 在此檔案中貼上金鑰的內容，然後儲存檔案。

   > [!NOTE]
   > 金鑰中必須包含字串 `ssh-rsa`。 此外，金鑰的內容必須是單行文字。
   >  

6. 在用戶端系統上，啟動 PuTTY。 在 [類別] 窗格中，移至 [連線] > [SSH] > [驗證]。在 [用於驗證的私密金鑰檔] 方塊中，瀏覽至您稍早產生的金鑰。

   ![SSH 驗證選項的螢幕擷取畫面](./media/oracle-asm/setprivatekey.png)

7. 在 [類別] 窗格中，移至 [連線] > [SSH] > [X11]。 選取 [啟用 X11 轉送] 核取方塊。

   ![SSH X11 轉送選項的螢幕擷取畫面](./media/oracle-asm/enablex11.png)

8. 在 [類別] 窗格中，移至 [工作階段]。 在 [主機名稱] 對話方塊中，輸入 Oracle ASM VM `<publicIPaddress>`，填入新的 `Saved Session` 名稱，然後按一下 `Save`。  儲存之後，請按一下 `open` 以連線至 Oracle ASM 虛擬機器。  第一次連線時，系統會警告您登錄中不會快取遠端系統。 按一下 `yes` 新增該項目並繼續。

   ![PuTTY 工作階段選項的螢幕擷取畫面](./media/oracle-asm/puttysession.png)

## <a name="install-oracle-grid-infrastructure"></a>安裝 Oracle Grid Infrastructure

若要安裝 Oracle Grid Infrastructure，請完成下列步驟：

1. 以 **grid** 的身分登入。 (您應該能夠順利登入，而不會收到需要輸入密碼的提示)。 

   > [!NOTE]
   > 如果您正在執行 Windows，開始安裝之前，請確定您已啟動 Xming。

   ```bash
   cd /opt/grid
   ./runInstaller
   ```

   Oracle Grid Infrastructure 12c Release 1 安裝程式會隨即開啟  (安裝程式可能需要幾分鐘的時間才會啟動)。

2. 在 [選取安裝選項] 頁面上，選取 [為獨立伺服器安裝和設定 Oracle Grid Infrastructure]。

   ![安裝程式之 [選取安裝選項] 頁面的螢幕擷取畫面](./media/oracle-asm/install01.png)

3. 在 [選取產品語言] 頁面上，請確定已選取 [英文] 或您想要使用的語言。  按一下 `next`。

4. 在 [建立 ASM 磁碟群組] 頁面上︰
   - 輸入磁碟群組的名稱。
   - 在 [備援] 底下選取 [外部]。
   - 在 [配置單位大小] 底下選取 [4]。
   - 在 [新增磁碟] 底下選取 [ORCLASMSP]。
   - 按一下 `next`。

5. 在 [指定 ASM 密碼] 頁面上選取 [對這些帳戶使用相同密碼] 選項，然後輸入密碼。

   ![安裝程式之 [指定 ASM 密碼] 頁面的螢幕擷取畫面](./media/oracle-asm/install04.png)

6. 在 [指定管理選項] 頁面上，您可以選擇設定 EM Cloud Control。 我們會略過此選項 - 按一下 `next` 以繼續。 

7. 在 [特殊權限作業系統群組] 頁面上使用預設設定。 按一下 `next` 以繼續。

8. 在 [指定安裝位置] 頁面上，使用預設設定。 按一下 `next` 以繼續。

9. 在 [建立清查] 頁面上，將清查目錄變更至 `/u01/app/grid/oraInventory`。 按一下 `next` 以繼續。

   ![安裝程式之 [建立清查] 頁面的螢幕擷取畫面](./media/oracle-asm/install08.png)

10. 在 [Root 指令碼執行組態] 頁面上，選取 [自動執行組態指令碼] 核取方塊。 接著，選取 [使用 root 使用者認證] 選項，然後輸入 root 使用者密碼。

    ![安裝程式之 [Root 指令碼執行組態] 頁面的螢幕擷取畫面](./media/oracle-asm/install09.png)

11. 在 [執行必要條件檢查] 頁面上，目前的設定會失敗並顯示錯誤。 這是預期中的行為。 選取 `Fix & Check Again`。

12. 在 [修復指令碼] 對話方塊中，按一下 `OK`。

13. 在 [摘要] 頁面上，檢閱您選取的設定，然後按一下 `Install`。

    ![安裝程式之 [摘要] 頁面的螢幕擷取畫面](./media/oracle-asm/install12.png)

14. 隨即顯示警告對話方塊，通知您必須以特殊權限使用者身分執行組態指令碼。 按一下 `Yes` 以繼續。

15. 在 [完成] 頁面上，按一下 `Close` 以完成安裝。

## <a name="set-up-your-oracle-asm-installation"></a>設定 Oracle ASM 安裝

若要設定 Oracle ASM 的安裝，請完成下列步驟︰

1. 請確定您仍從 X11 工作階段以 **grid** 的身分登入。 您可能需要按 `enter` 以恢復終端機。 然後啟動 Oracle 自動儲存體管理組態助理：

   ```bash
   cd /u01/app/grid/product/12.1.0/grid/bin
   ./asmca
   ```

   Oracle ASM Configuration Assistant 隨即開啟。

2. 在 [設定 ASM: 磁碟群組] 對話方塊中，按一下 `Create` 按鈕，然後按一下 `Show Advanced Options`。

3. 在 [建立磁碟群組] 對話方塊中：

   - 輸入磁碟群組名稱 **DATA**。
   - 在 [選取成員磁碟] 底下選取 [ORCL_DATA] 和 [ORCL_DATA1]。
   - 在 [配置單位大小] 底下選取 [4]。
   - 按一下 `ok` 以建立磁碟群組。
   - 按一下 `ok` 以關閉確認視窗。

   ![[建立磁碟群組] 對話方塊的螢幕擷取畫面](./media/oracle-asm/asm02.png)

4. 在 [設定 ASM: 磁碟群組] 對話方塊中，按一下 `Create` 按鈕，然後按一下 `Show Advanced Options`。

5. 在 [建立磁碟群組] 對話方塊中：

   - 輸入磁碟群組名稱 **FRA**。
   - 在 [備援] 底下選取 [外部 (無)]。
   - 在 [選取成員磁碟] 底下選取 [ORCL_FRA]。
   - 在 [配置單位大小] 底下選取 [4]。
   - 按一下 `ok` 以建立磁碟群組。
   - 按一下 `ok` 以關閉確認視窗。

   ![[建立磁碟群組] 對話方塊的螢幕擷取畫面](./media/oracle-asm/asm04.png)

6. 選取 [結束] 以關閉 ASM Configuration Assistant。

   ![[設定 ASM: 磁碟群組] 對話方塊與 [結束] 按鈕的螢幕擷取畫面](./media/oracle-asm/asm05.png)

## <a name="create-the-database"></a>建立資料庫

Azure Marketplace 映像上已安裝 Oracle 資料庫軟體。 若要建立資料庫，請完成下列步驟：

1. 將使用者切換為 Oracle 超級使用者，然後將接聽程式初始化以啟用記錄功能：

   ```bash
   su - oracle
   cd /u01/app/oracle/product/12.1.0/dbhome_1/bin
   ./dbca
   ```
   Database Configuration Assistant 隨即開啟。

2. 在 [資料庫作業] 頁面上，按一下 `Create Database`。

3. 在 [建立模式] 頁面上︰

   - 輸入資料庫名稱。
   - 在 [儲存體類型] 中，請確定已選取 [自動儲存體管理 (ASM)]。
   - 在 [資料庫檔案位置] 中，請使用預設的 ASM 建議位置。
   - 在 [快速復原區域] 中，請使用預設的 ASM 建議位置。
   - 輸入 [管理密碼] 和 [確認密碼]。
   - 確定已選取 `create as container database`。
   - 輸入 `pluggable database name` 值。

4. 在 [摘要] 頁面上，檢閱您選取的設定，然後按一下 `Finish` 以建立資料庫。

   ![[摘要] 頁面的螢幕擷取畫面](./media/oracle-asm/createdb03.png)

5. 已建立資料庫。 在 [完成] 頁面上，您可以選擇解除鎖定其他帳戶以使用此資料庫並變更密碼。 如果您想要進行此操作，請選取 [密碼管理]，否則請按一下 `close`。

## <a name="delete-the-vm"></a>刪除 VM

您已從 Azure Marketplace 在 Oracle DB 映像上成功設定 Oracle 自動儲存體管理。  如果您不再需要此 VM，則可以使用下列命令移除資源群組、VM 和所有相關資源：

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

[教學課程：設定 Oracle DataGuard](configure-oracle-dataguard.md)

[教學課程：設定 Oracle GoldenGate](Configure-oracle-golden-gate.md)

請檢閱[建置 Oracle DB](oracle-design.md)
