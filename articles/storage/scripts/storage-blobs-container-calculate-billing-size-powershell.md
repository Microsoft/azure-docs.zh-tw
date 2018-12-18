---
title: Azure PowerShell 指令碼範例 - 計算 Blob 容器的帳單大小總計 | Microsoft Docs
description: 就計費目的計算 Azure Blob 儲存體中的大小總計。
services: storage
documentationcenter: na
author: fhryo-msft
manager: cbrooks
editor: tysonn
ms.assetid: ''
ms.custom: mvc
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: powershell
ms.topic: sample
ms.date: 11/07/2017
ms.author: fryu
ms.openlocfilehash: c37b416578a76e9b12e29d68e413d851796ccc6f
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2017
ms.locfileid: "26368539"
---
# <a name="calculate-the-total-billing-size-of-a-blob-container"></a>計算 Blob 容器的帳單大小總計

此指令碼會就估計成本計費之目的計算 Azure Blob 儲存體中的容器大小。 指令碼會加總容器中的 blob 大小。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> 這個 PowerShell 指令碼會就計費目的計算容器大小。 如果您計算容器大小是要用於其他用途，請參閱[計算 Blob 儲存體容器的大小總計](../scripts/storage-blobs-container-calculate-size-powershell.md)，以了解提供估計值的簡單指令碼。

## <a name="determine-the-size-of-the-blob-container"></a>決定 Blob 容器的大小

Blob 容器的大小總計包含容器本身的大小和容器下所有 Blob 的大小。

以下各節描述對 Blob 容器和 Blob 儲存體容量的計算方式。 在下節中，Len(X) 表示字串中的字元數。

### <a name="blob-containers"></a>Blob 容器

以下計算說明如何估計每個 Blob 容器使用的儲存體數量：

`
48 bytes + Len(ContainerName) * 2 bytes +
For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
For-Each Signed Identifier[512 bytes]
`

以下是分解：
* 每個容器 48 個位元組的額外負荷包含上次修改時間、權限、公用設定，以及一些系統中繼資料。

* 容器名稱會儲存為 Unicode，因此取得字元數目並乘以 2。

* 對於儲存的每個 Blob 容器中繼資料區塊，我們會儲存名稱的長度 (ASCII) 再加上字串值的長度。

* 每一簽署的識別碼 512 個位元組包括簽署的識別碼名稱、開始時間、到期時間和權限。

### <a name="blobs"></a>Blob

以下計算顯示如何估計每個 Blob 使用的儲存體數量。

* 區塊 Blob (基底 Blob 或快照集)：

   `
   124 bytes + Len(BlobName) * 2 bytes +
   For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
   8 bytes + number of committed and uncommitted blocks * Block ID Size in bytes +
   SizeInBytes(data in unique committed data blocks stored) +
   SizeInBytes(data in uncommitted data blocks)
   `

* 分頁 Blob (基底 Blob 或快照集)：

   `
   124 bytes + Len(BlobName) * 2 bytes +
   For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
   number of nonconsecutive page ranges with data * 12 bytes +
   SizeInBytes(data in unique pages stored)
   `

以下是分解：

* Blob 的 124 位元組負荷，其中包含：
    - 上次修改時間
    - 大小
    - Cache-Control
    - Content-Type
    - Content-Language
    - Content-Encoding
    - Content-MD5
    - 權限
    - 快照集資訊
    - 租用
    - 部分系統中繼資料

* Blob 名稱會儲存為 Unicode，因此取得字元數目並乘以 2。

* 針對儲存的每個中繼資料區塊，新增儲存名稱的長度 (儲存為 ASCII) 再加上字串值的長度。

* 對於區塊 Blob：
    * 區塊清單的 8 位元組。
    * 區塊數目 x 依位元組的區塊識別碼大小。
    * 所有認可及未認可之區塊中的資料大小。 
    
    >[!NOTE]
    >使用快照集時，此大小只包含此基底或快照集 Blob 的唯一資料。 如果未認可的區塊經過一週之後未使用，將予以記憶體回收。 此後，這些不會計入帳單。

* 對於分頁 Blob：
    * 具有資料的非連續分頁範圍 x 12 個位元組。 這是呼叫 **GetPageRanges API** 時會看到的唯一分頁範圍的數目。

    * 所有儲存的分頁中資料的大小 (以位元組為單位)。 
    
    >[!NOTE]
    >使用快照集時，此大小只包含要計算的基底 Blob 或快照集 Blob。

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/storage/calculate-container-size/calculate-container-size-ex.ps1 "Calculate container size")]

## <a name="next-steps"></a>後續步驟

- 請參閱[計算 Blob 儲存體容器的大小總計](../scripts/storage-blobs-container-calculate-size-powershell.md)提供容器大小之估計值的簡單指令碼。

- 如需有關 Azure 儲存體計費的詳細資訊，請參閱[了解 Windows Azure 儲存體計費](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/07/08/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity/)。

- 如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-4.4.1)。

- 您可以在 [Azure 儲存體的 PowerShell 範例](../blobs/storage-samples-blobs-powershell.md) 找到其他的儲存體 PowerShell 指令碼範例。
