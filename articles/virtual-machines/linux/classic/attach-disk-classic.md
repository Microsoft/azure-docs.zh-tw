---
title: 將磁碟附加至 Azure 中的 Linux VM | Microsoft Docs
description: 了解如何使用傳統部署模型將資料磁碟連接至 Linux VM，並初始化磁碟，使其可供使用
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: 4901384d-2a6f-4f46-bba0-337a348b7f87
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/09/2017
ms.author: cynthn
ms.openlocfilehash: b5bb3a9353f83cb569988a068f3ca02da85f739c
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37929146"
---
# <a name="how-to-attach-a-data-disk-to-a-linux-virtual-machine"></a>如何將資料磁碟連接至 Linux 虛擬機器
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 參閱如何[使用 Resource Manager 部署模型連接資料磁碟](../add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

您可以將空的磁碟和含有資料的磁碟連接到 Azure VM。 這兩種類型的磁碟都是位於 Azure 儲存體帳戶中的 .vhd 檔案。 就像將任何磁碟新增到 Linux 機器一樣，連接磁碟之後，您必須將磁碟初始化並格式化，它才可供使用。 本文將會詳細說明連接空的磁碟和連接含有資料的磁碟到 VM，以及初始化和格式化新磁碟的方法。

> [!NOTE]
> 最好使用一或多個不同的磁碟來儲存虛擬機器的資料。 當您建立 Azure 虛擬機器時，它會有作業系統磁碟和暫存磁碟。 **請勿使用暫存磁碟來儲存持續資料。** 顧名思義，它只提供暫存儲存空間。 它並不提供備援或備份，因為它不在 Azure 儲存體內。
> 暫存磁碟通常是由 Azure Linux 代理程式管理，並自動掛接到 **/mnt/resource** (或 Ubuntu 映像中的 **/mnt**)。 另一方面，Linux 核心可能會將資料磁碟命名為 `/dev/sdc`之類的名稱，而您必須分割、格式化及掛接此資源。 如需詳細資訊，請參閱 [Azure Linux 代理程式使用者指南][Agent]。
> 
> 

[!INCLUDE [howto-attach-disk-windows-linux](../../../../includes/howto-attach-disk-linux.md)]

## <a name="initialize-a-new-data-disk-in-linux"></a>在 Linux 中初始化新的資料磁碟
1. SSH 連線到您的 VM。 如需詳細資訊，請參閱[如何登入執行 Linux 的虛擬機器][Logon]。
2. 接下來您需要尋找資料磁碟的裝置識別碼以進行初始化。 作法有二：
   
    a) Grep SCSI 裝置中的記錄檔，如以下命令︰
   
    ```bash
    sudo grep SCSI /var/log/messages
    ```
   
    針對最新的 Ubuntu 散發套件，您可能需要使用 `sudo grep SCSI /var/log/syslog`，因為預設可能會停用記錄到 `/var/log/messages` 的功能。
   
    在出現的訊息中，您可以找到最後一個新增之資料磁碟的識別碼。
   
    ![取得磁碟訊息](./media/attach-disk/scsidisklog.png)
   
    或
   
    b) 使用 `lsscsi` 命令來找出裝置識別碼。您可以透過 `yum install lsscsi` (在 Red Hat 式發行版本上) 或 `apt-get install lsscsi` (在 Debian 式發行版本上) 來安裝 `lsscsi`。 您可以透過磁碟的 *LUN* (亦稱為 **邏輯單元編號**) 找到您所需的磁碟。 例如，您可從 `azure vm disk list <virtual-machine>` 輕易看到所附加磁碟的 *lun*，如下所示：

    ```azurecli
    azure vm disk list myVM
    ```

    輸出大致如下：

    ```azurecli
    info:    Executing command vm disk list
    + Fetching disk images
    + Getting virtual machines
    + Getting VM disks
    data:    Lun  Size(GB)  Blob-Name                         OS
    data:    ---  --------  --------------------------------  -----
    data:         30        myVM-2645b8030676c8f8.vhd  Linux
    data:    0    100       myVM-76f7ee1ef0f6dddc.vhd
    info:    vm disk list command OK
    ```
   
    將此資料與相同範例虛擬機器的 `lsscsi` 輸出做比較：
   
    ```bash
    [1:0:0:0]    cd/dvd  Msft     Virtual CD/ROM   1.0   /dev/sr0
    [2:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sda
    [3:0:1:0]    disk    Msft     Virtual Disk     1.0   /dev/sdb
    [5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc
    ```
   
    在每個資料列的 Tuple 中，最後一個數字即為 *LUN*。 如需詳細資訊，請參閱 `man lsscsi` 。
3. 出現提示時，輸入下列命令來建立裝置：
   
    ```bash
    sudo fdisk /dev/sdc
    ```

4. 出現提示時，輸入 **n** 建立磁碟分割。

    ![建立裝置](./media/attach-disk/fdisknewpartition.png)

5. 出現提示時，輸入 **p** 以將磁碟分割設為主要磁碟分割。 輸入 **1** 以將它設為第一個磁碟分割，然後按 Enter 來接受預設的磁柱值。 在某些系統上，它可能會顯示第一個和最後一個磁區的預設值，而不是圓柱圖。 您可以選擇接受這些預設值。

    ![建立磁碟分割](./media/attach-disk/fdisknewpartdetails.png)


6. 輸入 **p** 查看目前分割之磁碟的詳細資料。

    ![列出磁碟資訊](./media/attach-disk/fdiskpartitiondetails.png)


7. 輸入 **w** 寫入磁碟的設定。

    ![寫入磁碟變更](./media/attach-disk/fdiskwritedisk.png)

8. 您現在可以在新的磁碟分割上建立檔案系統。 在裝置識別碼後加上磁碟分割編號 (在以下範例中為 `/dev/sdc1`)。 以下範例會在 /dev/sdc1 上建立 ext4 磁碟分割：
   
    ```bash
    sudo mkfs -t ext4 /dev/sdc1
    ```
   
    ![建立檔案系統](./media/attach-disk/mkfsext4.png)
   
   > [!NOTE]
   > SuSE Linux Enterprise 11 系統針對 ext4 檔案系統只支援唯讀存取。 針對這些系統，建議您將新檔案系統格式化為 ext3，而非 ext4。

9. 建立目錄以掛接新的檔案系統，如下所示：
   
    ```bash
    sudo mkdir /datadrive
    ```

10. 最後您可以掛接磁碟機，如下所示︰
   
    ```bash
    sudo mount /dev/sdc1 /datadrive
    ```
   
    資料磁碟現在可以當做 **/datadrive**來使用。
   
    ![建立目錄和掛接磁碟](./media/attach-disk/mkdirandmount.png)

11. 將新的磁碟機新增至 /etc/fstab：
   
    為了確保重新開機之後自動重新掛接磁碟機，必須將磁碟機新增至 /etc/fstab 檔案。 此外，強烈建議在 /et/fstab 中使用全域唯一識別碼 (Universally Unique IDentifier, UUID) 來參考磁碟機，而不只是裝置名稱 (例如，/dev/sdc1)。 使用 UUID 可避免當作業系統在開機期間偵測到磁碟錯誤時，將不正確的磁碟掛接到指定的位置，又接著將這些裝置識別碼指派給任何其餘的資料磁碟。 若要尋找新磁碟機的 UUID，您可以使用 **blkid** 公用程式：
   
    ```bash
    sudo -i blkid
    ```
   
    輸出大致如下列範例所示：
   
    ```bash
    /dev/sda1: UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4"
    /dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4"
    /dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"
    ```

    > [!NOTE]
    > 不當編輯 **/etc/fstab** 檔案會導致系統無法開機。 如果不確定，請參閱散發套件的文件，以取得如何適當編輯此檔案的相關資訊。 在編輯之前，也建議先備份 /etc/fstab 檔案。

    接下來，在文字編輯器中開啟 **/etc/fstab** 檔案：

    ```bash
    sudo vi /etc/fstab
    ```

    在此範例中，我們會使用先前步驟所建立之新的 **/dev/sdc1** 裝置的 UUID 值，並使用掛接點 **/datadrive**。 在 **/etc/fstab** 檔案的結尾加入以下程式碼：

    ```sh
    UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,nofail   1   2
    ```

    或者，在基於 SuSE Linux 的系統上，您可能需要使用稍微不同的格式：

    ```sh
    /dev/disk/by-uuid/33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext3   defaults,nofail   1   2
    ```

    > [!NOTE]
    > `nofail` 選項可確保即使檔案系統已損毀或磁碟在開機時並不存在，仍然會啟動 VM。 若不使用此選項，您可能會遇到[因為 FSTAB 錯誤所以無法 SSH 到 Linux VM](https://blogs.msdn.microsoft.com/linuxonazure/2016/07/21/cannot-ssh-to-linux-vm-after-adding-data-disk-to-etcfstab-and-rebooting/) 中所述的行為。

    您現在可以將檔案系統取消掛接後再重新掛接 (亦即使用在先前步驟中建立的範例掛接點 `/datadrive`)，來測試是否已正確掛接檔案系統：

    ```bash
    sudo umount /datadrive
    sudo mount /datadrive
    ```

    如果 `mount` 命令發生錯誤，請檢查 /etc/fstab 檔案的語法是否正確。 如果還有建立其他資料磁碟機或磁碟分割，也請分別在 /etc/fstab 中分別輸入它們。

    使用下列命令將磁碟機設為可寫入：

    ```bash
    sudo chmod go+w /datadrive
    ```

    > [!NOTE]
    > 後續移除資料磁碟而不編輯 fstab，可能會造成 VM 無法開機。 如果這是常見的情況，大多數散發套件都會提供 `nofail` 和/或 `nobootwait` fstab 選項，可讓系統在即使開機時無法掛接磁碟的情況下也能開機。 請查閱散發套件的文件，以取得這些參數的相關資訊。

### <a name="trimunmap-support-for-linux-in-azure"></a>Azure 中 Linux 的 TRIM/UNMAP 支援
有些 Linux 核心會支援 TRIM/UNMAP 作業以捨棄磁碟上未使用的區塊。 這些作業主要是在標準儲存體中相當實用，可用來通知 Azure 已刪除的頁面已不再有效而可予以捨棄。 如果您建立大型檔案，然後再將它們刪除，捨棄頁面可以節省成本。

有兩種方式可在 Linux VM 中啟用 TRIM 支援。 像往常一樣，請參閱您的散發套件以了解建議的方法︰

* 在 `/etc/fstab` 中使用 `discard` 掛接選項，例如：

    ```sh
    UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,discard   1   2
    ```

* 在某些情況下，`discard` 選項可能會影響效能。 或者，您也可以從命令列手動執行 `fstrim` 命令，或將它新增到 crontab 來定期執行︰
  
    **Ubuntu**
  
    ```bash
    sudo apt-get install util-linux
    sudo fstrim /datadrive
    ```
  
    **RHEL/CentOS**
  
    ```bash
    sudo yum install util-linux
    sudo fstrim /datadrive
    ```

## <a name="troubleshooting"></a>疑難排解
[!INCLUDE [virtual-machines-linux-lunzero](../../../../includes/virtual-machines-linux-lunzero.md)]

## <a name="next-steps"></a>後續步驟
您可以閱讀下列文章來進一步了解如何使用 Linux VM：

* [如何登入執行 Linux 的虛擬機器][Logon]
* [如何從 Linux 虛擬機器卸離磁碟](detach-disk-classic.md)
* [搭配傳統部署模型使用 Azuer CLI](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)
* [在 Azure 中的 Linux VM 上設定 RAID](../configure-raid.md)
* [設定 Azure 中 Linux VM 的 LVM](../configure-lvm.md)

<!--Link references-->
[Agent]:../../extensions/agent-linux.md
[Logon]:../mac-create-ssh-keys.md
