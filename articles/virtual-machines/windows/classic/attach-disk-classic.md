---
title: 將磁碟連接到傳統 Azure VM | Microsoft Docs
description: 將資料磁碟連接到以傳統部署模型建立的 Windows 虛擬機器，並予以初始化。
services: virtual-machines-windows, storage
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: be4e3e74-05bc-4527-969f-84f10a1d66a7
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 02/21/2017
ms.author: cynthn
ms.openlocfilehash: 48a6b96bd611fcc8fa8219aeef359419255b8cef
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30918586"
---
# <a name="attach-a-data-disk-to-a-windows-virtual-machine-created-with-the-classic-deployment-model"></a>將資料磁碟連接至以傳統部署模型建立的 Windows 虛擬機器

本文會示範如何使用 Azure 入口網站將傳統部署模型所建立之新的及現有的磁碟連接到 Windows 虛擬機器。



您也可以[在 Azure 入口網站中將資料磁碟連接到 Linux VM](../../linux/attach-disk-portal.md)。

連接磁碟之前，請檢閱下列提示︰

* 虛擬機器的大小會控制您可以連接的資料磁碟數目。 如需詳細資訊，請參閱 [虛擬機器的大小](../../virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

* 若要使用進階儲存體，您需要 DS 系列或 GS 系列的虛擬機器。 您可以使用進階或標準儲存體帳戶的磁碟搭配這些虛擬機器。 僅特定地區可用進階儲存體。 如需詳細資訊，請參閱[進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../premium-storage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

* 對於新的磁碟，當您連接的時候 Azure 會建立該磁碟，所以您不需要建立它。

您也可以[使用 Powershell 連接資料磁碟](../../virtual-machines-windows-attach-disk-ps.md)。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

## <a name="find-the-virtual-machine"></a>尋找虛擬機器
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從儀表板上所列資源選取虛擬機器。
3. 在左窗格的 [設定] 下方，按一下 [磁碟]。

    ![開啟磁碟設定](./media/attach-disk/virtualmachinedisks.png)

接著，依照指示來連接[新的磁碟](#option-1-attach-a-new-disk)或[現有的磁碟](#option-2-attach-an-existing-disk)。

## <a name="option-1-attach-and-initialize-a-new-disk"></a>選項 1：連接新磁碟並進行初始化

1. 在 [磁碟] 刀鋒視窗上，按一下 [連接新項目]。
2. 檢閱預設設定，視需要進行更新，然後按一下 [確定] 。

   ![檢閱磁碟設定](./media/attach-disk/attach-new.png)

3. 在 Azure 建立磁碟並將其連接至虛擬機器之後，該新磁碟就會列在虛擬機器之磁碟設定中的 [資料磁碟] 底下。

### <a name="initialize-a-new-data-disk"></a>初始化新的資料磁碟

1. 連接至虛擬機器。 如需指示，請參閱[如何連接和登入執行 Windows 的 Azure 虛擬機器](../../virtual-machines-windows-connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
2. 登入虛擬機器之後，開啟 [伺服器管理員]。 在左窗格中，選取 [File and Storage Services] 。

    ![開啟伺服器管理員](../media/attach-disk-portal/fileandstorageservices.png)

3. 選取 [磁碟]。
4. [磁碟]  區段會列出磁碟。 多數情況下，虛擬機器會有磁碟 0、磁碟 1 和磁碟 2。 磁碟 0 是作業系統磁碟，磁碟 1 是暫存磁碟，磁碟 2 則是新連接至虛擬機器的資料磁碟。 資料磁碟會將磁碟分割列為**未知**。

 在磁碟上按一下滑鼠右鍵，然後選取 [初始化] 。

5. 初始化磁碟時，您會收到將清除所有資料的通知。 按一下 [是]，認可此警告並初始化磁碟。 完成之後，即會將磁碟分割列為 [GPT]。 再次於磁碟上按一下滑鼠右鍵，然後選取 新增磁碟區 。

6. 使用預設值完成精靈。 當精靈完成時，[ **磁碟區** ] 區段會列出新的磁碟區。 磁碟此時將上線，可用來儲存資料。

    ![成功初始化磁碟區](./media/attach-disk/newdiskafterinitialization.png)

## <a name="option-2-attach-an-existing-disk"></a>選項 2：連接現有磁碟
1. 在 [磁碟] 刀鋒視窗上，按一下 [連接現有項目]。
2. 在 [連接現有磁碟] 底下，按一下 [位置]。

   ![連接現有磁碟](./media/attach-disk/attachexistingdisksettings.png)
3. 在 [儲存體帳戶] 底下，選取持有該 .vhd 檔案的帳戶和容器。

   ![尋找 VHD 位置](./media/attach-disk/existdiskstorageaccountandcontainer.png)

4. 選取 .vhd 檔案。
5. 在 [連結現有磁碟] 底下，您剛才選取的檔案會列在 [VHD 檔案] 底下。 按一下 [SERVICEPRINCIPAL] 。
6. Azure 將磁碟連接至虛擬機器之後，該磁碟會列在虛擬機器磁碟設定中的 [資料磁碟] 下面。

## <a name="use-trim-with-standard-storage"></a>搭配使用 TRIM 與標準儲存體

如果您使用標準儲存體 (HDD)，您應該啟用 TRIM。 TRIM 會捨棄磁碟上未使用的區塊，因此您只需支付實際使用的儲存體。 使用 TRIM 可以節省成本，包括刪除大型檔案時所產生的未使用區塊。

您可以執行此命令來檢查 TRIM 設定。 在您的 Windows VM 上開啟命令提示字元並輸入︰

```
fsutil behavior query DisableDeleteNotify
```

如果命令傳回 0，則已正確啟用 TRIM。 如果傳回 1，請執行下列命令來啟用 TRIM：
```
fsutil behavior set DisableDeleteNotify 0
```

## <a name="next-steps"></a>後續步驟
如果您的應用程式需要使用 D: 磁碟機來儲存資料，您可以[變更 Windows 暫存磁碟的磁碟機代號](../../virtual-machines-windows-change-drive-letter.md)。

## <a name="additional-resources"></a>其他資源
[有關虛擬機器的磁碟和 VHD](../../virtual-machines-linux-about-disks-vhds.md)
