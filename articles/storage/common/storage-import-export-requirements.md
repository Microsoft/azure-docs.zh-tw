---
title: Azure 匯入/匯出服務需求 | Microsoft Docs
description: 了解 Azure 匯入/匯出服務的軟體和硬體需求。
author: alkohli
services: storage
ms.service: storage
ms.topic: article
ms.date: 07/19/2018
ms.author: alkohli
ms.component: common
ms.openlocfilehash: 10e8fb6ac5bcce278de3924ebd3a0d9f90392217
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528049"
---
# <a name="azure-importexport-system-requirements"></a>Azure 匯入/匯出系統需求

本文說明 Azure 匯入/匯出服務的重要需求。 建議您先仔細檢閱資訊，再使用匯入/匯出服務，然後在作業期間，視需要回顧參考。

## <a name="supported-operating-systems"></a>受支援的作業系統

若要使用 WAImportExport 工具來準備硬碟，支援下列**支援 BitLocker 磁碟機加密的 64 位元作業系統**。


|平台 |版本 |
|---------|---------|
|Windows     | Windows 7 Enterprise、Windows 7 Ultimate <br> Windows 8 Pro、Windows 8 Enterprise、Windows 8.1 Pro、Windows 8.1 Enterprise <br> Windows 10        |
|Windows Server     |Windows Server 2008 R2 <br> Windows Server 2012、Windows Server 2012 R2         |


## <a name="supported-storage-accounts"></a>支援的儲存體帳戶

Azure 匯入/匯出服務支援下列 [Azure 儲存體帳戶](storage-account-options.md)。
- 一般用途 v1 儲存體帳戶 (傳統或 Azure Resource Manager 部署)
- Blob 儲存體帳戶
- 一般用途 v2 儲存體帳戶

每項工作都只能從僅只一個儲存體帳戶收送資料。 換句話說，單一匯入/匯出作業不能跨越多個儲存體帳戶。 如需建立新儲存體帳戶的詳細資訊，請參閱 [如何建立儲存體帳戶](storage-create-storage-account.md#create-a-storage-account)(英文)。

> [!IMPORTANT] 
> Azure 匯入/匯出服務不支援已啟用[虛擬網路服務端點](../../virtual-network/virtual-network-service-endpoints-overview.md)功能的儲存體帳戶。 

## <a name="supported-storage-types"></a>支援的儲存體類型

以下是使用 Azure 匯入/匯出服務時支援的儲存類型清單。


|工作 (Job)  |儲存體服務 |支援  |不支援  |
|---------|---------|---------|---------|
|Import     |  Azure Blob 儲存體 <br><br> Azure 檔案儲存體       | 支援區塊 Blob 與分頁 Blob <br><br> 支援檔案服務          |
|匯出     |   Azure Blob 儲存體       | 支援區塊 Blob、分頁 Blob 及附加 Blob         | 不支援 Azure 檔案服務


## <a name="supported-hardware"></a>支援的硬體 

對於 Azure 匯入/匯出服務，您需要支援的磁碟才能複製資料。

### <a name="supported-disks"></a>支援的磁碟

以下是支援與匯入/匯出服務搭配使用的儲存類型清單。


|磁碟類型  |大小  |支援 |不支援  |
|---------|---------|---------|---------|
|SSD    |   2.5 吋      |         |         |
|HDD     |  2.5 吋<br>3.5 吋       |SATA II、SATA III         |具有內建 USB 介面卡的外接式 HDD <br> 外接式 HDD 機殼內的磁碟         |


單一匯入/匯出作業可以有：
- 最多 10 個 HDD/SSD。
- 混用任何尺寸的 HDD/SSD。

您可以將大量的磁碟機分散到多個作業，而且可建立的作業數目並無限制。 若為匯入工作，則只會處理磁碟機上的第一個資料磁碟區。 此資料磁碟區必須以 NTFS 格式化。

當使用 WAImportExport 工具來準備硬碟和複製資料時，您可以使用外接式 USB 轉接器。 大部分現成可用的 USB 3.0 或更新版配接器應都可使用。 


## <a name="next-steps"></a>後續步驟

* [設定 WAImportExport 工具](storage-import-export-tool-how-to.md)
* [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)
* [Azure 匯入匯出 REST API 範例 (英文)](https://azure.microsoft.com/documentation/samples/storage-dotnet-import-export-job-management/)

