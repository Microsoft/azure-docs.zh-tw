---
title: 將受控資料磁碟連結到 Windows VM - Azure | Microsoft Docs
description: 如何在 Azure 入口網站中使用 Resource Manager 部署模型，將新的受控資料磁碟連結到 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 12/13/2017
ms.author: cynthn
ms.openlocfilehash: 14721b2f2bc7913c2b7eadfc5ee801a223201ea9
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30913411"
---
# <a name="how-to-attach-a-managed-data-disk-to-a-windows-vm-in-the-azure-portal"></a>如何在 Azure 入口網站中將受控資料磁碟連結到 Windows VM

本文示範如何在 Azure 入口網站中將新的受控資料磁碟連結到 Windows 虛擬機器。 這麼做之前，請先檢閱下列提示：

* 虛擬機器的大小會控制您可以連接的資料磁碟數目。 如需詳細資訊，請參閱 [虛擬機器的大小](sizes.md)。
* 對於新的磁碟，當您連接的時候 Azure 會建立該磁碟，所以您不需要建立它。

您也可以[使用 Powershell 連接資料磁碟](attach-disk-ps.md)。



## <a name="add-a-data-disk"></a>新增資料磁碟
1. 在左側功能表中，按一下 [虛擬機器]。
2. 然後從清單中選取虛擬機器。
3. 在虛擬機器頁面上，按一下 [磁碟]。
4. 在 [磁碟] 頁面上，按一下 [+ 新增資料磁碟]。
5. 在新磁碟的下拉式清單中，選取 [建立磁碟]。
6. 在 [建立受控磁碟] 頁面中，輸入磁碟名稱並視需要調整其他設定。 完成之後，請按一下 [建立]。
7. 在 [磁碟] 頁面中，按一下 [儲存] 以儲存 VM 的新磁碟設定。
6. 在 Azure 建立磁碟並將其連接至虛擬機器之後，該新磁碟就會列在虛擬機器之磁碟設定中的 [資料磁碟] 底下。


## <a name="initialize-a-new-data-disk"></a>初始化新的資料磁碟

1. 連線至 VM。
1. 按一下 VM 內部的 [開始] 功能表，然後輸入 **diskmgmt.msc** 並按下 **Enter** 鍵。 磁碟管理嵌入式管理單元隨即開啟。
2. 磁碟管理會辨識出您擁有新的、未初始化的磁碟，且會隨即顯示 [初始化磁碟] 視窗。
3. 請確定已選取新磁碟，並按一下 [確定] 將磁碟初始化。
4. 新的磁碟會顯示為 [未配置]。 以滑鼠右鍵按一下磁碟上的任何位置，並選取 [新增簡單磁碟區]。 [新增簡單磁碟區精靈] 會隨即開啟。
5. 逐一執行精靈的所有步驟，保留所有預設值，並在完成後選取 [完成]。
6. 關閉磁碟管理。
7. 您會看到快顯視窗，說明您必須先將新的磁碟格式化之後才能使用。 按一下 [格式化磁碟]。
8. 在 [格式化新磁碟] 對話方塊中，檢查設定，然後按一下 [開始]。
9. 系統會向您顯示警告，說明格式化磁碟會清除所有資料，請按一下 [確定]。
10. 格式化完成之後，按一下 [確定]。

## <a name="use-trim-with-standard-storage"></a>搭配使用 TRIM 與標準儲存體

如果您使用標準儲存體 (HDD)，您應該啟用 TRIM。 TRIM 會捨棄磁碟上未使用的區塊，因此您只需支付實際使用的儲存體。 如果您建立大型檔案，然後將它們刪除，這便可節省成本。 

您可以執行此命令來檢查 TRIM 設定。 在您的 Windows VM 上開啟命令提示字元並輸入︰

```
fsutil behavior query DisableDeleteNotify
```

如果命令傳回 0，則已正確啟用 TRIM。 如果傳回 1，請執行下列命令來啟用 TRIM：
```
fsutil behavior set DisableDeleteNotify 0
```

從磁碟中刪除資料之後，您可以使用 TRIM 來執行重組，以確保 TRIM 作業正確排清：

```
defrag.exe <volume:> -l
```

您也可以確保已透過格式化磁碟區，修剪整個磁碟區。

## <a name="next-steps"></a>後續步驟
如果您的應用程式需要使用 D: 磁碟機來儲存資料，您可以[變更 Windows 暫存磁碟的磁碟機代號](change-drive-letter.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。
