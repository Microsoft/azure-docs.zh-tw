---
title: Azure PowerShell 指令碼範例 - 在 Windows 上使用 AzCopy 在儲存體帳戶間移轉 blob | Microsoft Docs
description: 使用 AzCopy，將一個 Azure 儲存體帳戶的 Blob 內容複製到另一個。
services: storage
documentationcenter: na
author: roygara
manager: jeconnoc
ms.custom: mvc
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: sample
ms.date: 02/01/2018
ms.author: rogarana
ms.openlocfilehash: 970315c5d597d691454f9dea0a76f2c0dc4a40ec
ms.sourcegitcommit: eeb5daebf10564ec110a4e83874db0fb9f9f8061
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/03/2018
ms.locfileid: "29360712"
---
# <a name="migrate-blobs-across-storage-accounts-using-azcopy-on-windows"></a>在 Windows 上使用 AzCopy 在儲存體帳戶間移轉 blob

這個範例會將所有 blob 物件從使用者提供的來源儲存體帳戶，複製到使用者提供的目標儲存體帳戶。 

這是藉由使用 `Get-AzureStorageContainer` 命令來完成，該命令會列出儲存體帳戶中的所有容器。 此範例接著會發出 AzCopy 命令，將每個容器從來源儲存體帳戶複製到目的地儲存體帳戶。 如果發生任何失敗，此範例會重試 $retryTimes 次 (預設值為 3，而且可以使用 `-RetryTimes` 參數修改)。 如果每次重試都發生失敗，使用者可以重新執行指令碼，方法是使用 `-LastSuccessContainerName` 參數，提供具有最後成功複製容器的範例。 此範例接著會繼續從該點複製容器。

此範例需要 Azure PowerShell 儲存體模組 **4.0.2** 版或更新版本。 您可以使用 `Get-Module -ListAvailable Azure.storage` 檢查已安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

此範例也需要最新版本的 [AzCopy on Windows](http://aka.ms/downloadazcopy)。 預設安裝目錄是 `C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\`

這個範例會採用來源儲存體帳戶名稱和金鑰、目標儲存體帳戶名稱和金鑰，以及 AzCopy.exe 的完整檔案路徑 (如果它不是安裝在預設目錄)。

此範例的輸入範例如下：

如果 AzCopy 是安裝在預設目錄：
```PowerShell
srcStorageAccountName: ExampleSourceStorageAccountName
srcStorageAccountKey: ExampleSourceStorageAccountKey
DestStorageAccountName: ExampleTargetStorageAccountName
DestStorageAccountKey: ExampleTargetStorageAccountKey
```

如果 AzCopy 不是安裝在預設目錄：

```Powershell
srcStorageAccountName: ExampleSourceStorageAccountName
srcStorageAccountKey: ExampleSourceStorageAccountKey
DestStorageAccountName: ExampleTargetStorageAccountName
DestStorageAccountKey: ExampleTargetStorageAccountKey
AzCopyPath: C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\AzCopy.exe
```

如果發生失敗，而且範例必須從特定的容器重新執行： 

`.\copyScript.ps1 -LastSuccessContainerName myContainerName`

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/storage/migrate-blobs-between-accounts/migrate-blobs-between-accounts.ps1 "Migrate blobs between storage accounts.")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令，將資料從一個儲存體帳戶複製到另一個。 下表中的每個項目都會連結至特定命令的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureStorageContainer](/powershell/module/azure.storage/Get-AzureStorageContainer) | 傳回與儲存體帳戶關聯的容器。 |
| [New-AzureStorageContext](/powershell/module/azure.storage/New-AzureStorageContext) | 建立 Azure 儲存體內容。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

在 [Azure Blob 儲存體的 PowerShell 範例](../blobs/storage-samples-blobs-powershell.md)中，提供了其他儲存體 PowerShell 指令碼範例。
