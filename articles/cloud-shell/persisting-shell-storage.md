---
title: 保存 Azure Cloud Shell 中的 Bash 檔案 | Microsoft Docs
description: 逐步解說 Azure Cloud Shell 中的 Bash 如何保存檔案。
services: azure
documentationcenter: ''
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 09/04/2018
ms.author: juluk
ms.openlocfilehash: 606b36be4a2bbeff8dd226f41341d60e23f0d988
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44163272"
---
[!INCLUDE [PersistingStorage-introblock](../../includes/cloud-shell-persisting-shell-storage-introblock.md)]

## <a name="how-cloud-shell-storage-works"></a>Cloud Shell 儲存體的運作方式 
Cloud Shell 透過下列兩種方法來保存檔案： 
* 建立 `$Home` 目錄的磁碟映像，以保存該目錄內的所有內容。 此磁碟映像會在您指定的檔案共用中儲存為 `acc_<User>.img` (位於 `fileshare.storage.windows.net/fileshare/.cloudconsole/acc_<User>.img`)，並自動同步變更。 
* 在您的 `$Home` 目錄中，將指定的檔案共用掛接為 `clouddrive`，以便直接與檔案共用互動。 `/Home/<User>/clouddrive` 對應至 `fileshare.storage.windows.net/fileshare`。
 
> [!NOTE]
> `$Home` 目錄中的所有檔案 (例如 SSH 金鑰) 會都保存於已掛接檔案共用中儲存的使用者磁碟映像中。 當您在 `$Home` 目錄和已掛接的檔案共用中保存資訊時，請套用最佳做法。

## <a name="bash-specific-commands"></a>Bash 特有的命令

### <a name="use-the-clouddrive-command"></a>使用 `clouddrive` 命令
Cloud Shell 中的 Bash 可讓您執行稱為 `clouddrive` 的命令，以手動更新掛接至 Cloud Shell 的檔案共用。
![執行 "clouddrive" 命令](media/persisting-shell-storage/clouddrive-h.png)

### <a name="mount-a-new-clouddrive"></a>掛接新的 clouddrive

#### <a name="prerequisites-for-manual-mounting"></a>手動掛接的先決條件
您可以使用 `clouddrive mount` 命令來更新與 Cloud Shell 關聯的檔案共用。

如果您要掛接現有的檔案共用，儲存體帳戶必須位在所選的 Cloud Shell 區域中： 藉由從 Bash 執行 `env` 和檢查 `ACC_LOCATION` 來擷取位置。

#### <a name="the-clouddrive-mount-command"></a>`clouddrive mount` 命令

> [!NOTE]
> 如果您要掛接新的檔案共用，則系統會建立您 `$Home` 目錄的新使用者映像。 您的先前 `$Home` 映像會保留在先前的檔案共用中。

執行 `clouddrive mount` 命令搭配下列參數：

```
clouddrive mount -s mySubscription -g myRG -n storageAccountName -f fileShareName
```

若要檢視更多詳細資料，請執行 `clouddrive mount -h`，如下所示：

![執行 `clouddrive mount` 命令](media/persisting-shell-storage/mount-h.png)

### <a name="unmount-clouddrive"></a>卸載 clouddrive
您可以隨時將掛接至 Cloud Shell 的檔案共用卸載。 因為 Cloud Shell 需要使用掛接的檔案共用，所以系統會在下一個工作階段提示您建立和掛接另一個檔案共用。

1. 執行 `clouddrive unmount`。
2. 了解並確認提示。

除非手動刪除，否則您的檔案共用將會繼續存在。 Cloud Shell 在後續的工作階段中將不再搜尋此檔案共用。 若要檢視更多詳細資料，請執行 `clouddrive unmount -h`，如下所示：

![執行 `clouddrive unmount` 命令](media/persisting-shell-storage/unmount-h.png)

> [!WARNING]
> 雖然執行此命令不會刪除任何資源，但手動刪除對應至 Cloud Shell 的資源群組、儲存體帳戶或檔案共用，將會清除您的 `$Home` 目錄磁碟映像及檔案共用中的任何檔案。 此動作無法復原。

### <a name="list-clouddrive"></a>列出 `clouddrive`
若要探索掛接為 `clouddrive` 的檔案共用，請執行 `df` 命令。 

clouddrive 檔案路徑會在 URL 中顯示您的儲存體帳戶名稱和檔案共用。 例如， `//storageaccountname.file.core.windows.net/filesharename`

```
justin@Azure:~$ df
Filesystem                                          1K-blocks   Used  Available Use% Mounted on
overlay                                             29711408 5577940   24117084  19% /
tmpfs                                                 986716       0     986716   0% /dev
tmpfs                                                 986716       0     986716   0% /sys/fs/cgroup
/dev/sda1                                           29711408 5577940   24117084  19% /etc/hosts
shm                                                    65536       0      65536   0% /dev/shm
//mystoragename.file.core.windows.net/fileshareName 5368709120    64 5368709056   1% /home/justin/clouddrive
justin@Azure:~$
```
## <a name="powershell-specific-commands"></a>PowerShell 特有的命令

### <a name="list-clouddrive-azure-file-shares"></a>列出 `clouddrive` Azure 檔案共用
`Get-CloudDrive` Cmdlet 會擷取 Cloud Shell 中 `clouddrive` 目前所掛接的 Azure 檔案共用資訊。 <br>
![Running Get-CloudDrive](media/persisting-shell-storage-powershell/Get-Clouddrive.png)

### <a name="unmount-clouddrive"></a>卸載 `clouddrive`
您可以隨時將掛接至 Cloud Shell 的 Azure 檔案共用卸載。 如果已移除 Azure 檔案共用，則系統會在下一個工作階段提示您建立和掛接新的 Azure 檔案共用。

`Dismount-CloudDrive` Cmdlet 會從目前儲存體帳戶卸載 Azure 檔案共用。 卸載 `clouddrive` 會終止目前工作階段。 系統會提示使用者在下一個工作階段期間建立和掛接新的 Azure 檔案共用。
![執行 Dismount-CloudDrive](media/persisting-shell-storage-powershell/Dismount-Clouddrive.png)

[!INCLUDE [PersistingStorage-endblock](../../includes/cloud-shell-persisting-shell-storage-endblock.md)]

## <a name="next-steps"></a>後續步驟
[Cloud Shell 中 Bash 的快速入門](quickstart.md) <br>
[Cloud Shell 中 PowerShell 的快速入門](quickstart-powershell.md) <br>
[了解 Microsoft Azure 檔案儲存體](https://docs.microsoft.com/azure/storage/storage-introduction#file-storage) <br>
[了解儲存體標籤](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags) <br>
