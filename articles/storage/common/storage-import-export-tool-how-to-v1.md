---
title: 使用 Azure 匯入/匯出工具 - v1 | Microsoft Docs
description: 了解如何使用匯入/匯出工具來準備硬碟機，以進行匯入作業、修復匯入作業或修復匯出作業。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 1/15/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 90b5697da0f134ccb3c9ddb66b5da138c8849189
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39523405"
---
# <a name="using-the-azure-importexport-tool-classic-deployment-model"></a>使用 Azure 匯入/匯出工具 (傳統部署模型)

Azure 匯入/匯出工具 (WAImportExport.exe) 是用來建立及管理 Azure 匯入/匯出服務的作業，可讓您將大量資料傳入或傳出 Azure Blob 儲存體。

本文件適用於 Azure 匯入/匯出工具的傳統部署模型。 如需使用最新版本工具的相關資訊，請參閱[使用 Azure 匯入/匯出工具](../storage-import-export-tool-how-to.md)。

下列文章會示範如何操作：

- 安裝並設定匯入/匯出工具。
- 準備硬碟機進行將資料從您的磁碟機匯入 Azure Blob 儲存體的作業。
- 使用複製記錄檔檢閱作業的狀態。 
- 修復匯入作業。 
- 修復匯出作業。 
- 如果您在程序期間發生問題，請疑難排解 Azure 匯入/匯出工具。 

## <a name="next-steps"></a>後續步驟

* [設定 WAImportExport 工具](../storage-import-export-tool-how-to.md)