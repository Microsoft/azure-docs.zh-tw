---
title: Azure Data Lake Storage Gen1 的災害復原指導方針 | Microsoft Docs
description: Azure Data Lake Storage Gen1 的災害復原指導方針
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 02/21/2018
ms.author: nitinme
ms.openlocfilehash: 44c1dc3e3f6c2c9af52a6e9c9320d4a8ba63b4d0
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127103"
---
# <a name="disaster-recovery-guidance-for-data-in-azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1 中資料的災害復原指導方針

Azure Data Lake Storage Gen1 提供本地備援儲存體 (LRS)。 因此，透過自動化複本，Data Lake Storage Gen1 帳戶中的資料可彈性應對資料中心內發生的暫時性硬體故障。 這可確保持久性與高可用性，符合 Data Lake Storage Gen1 的 SLA。 此文章提供指導方針，說明如何進一步保護資料，使其不受罕見的全區停電或意外刪除事件影響。

## <a name="disaster-recovery-guidance"></a>災害復原指導方針
每個客戶備妥自己的災害復原計畫是相當重要的。 請參閱此文章的資訊來擬定您的災害復原計劃。 這裡有一些資源可協助您建立自己的計畫。

* [Azure 應用程式的災害復原和高可用性](../resiliency/resiliency-disaster-recovery-high-availability-azure-applications.md)
* [Azure 復原技術指導](../resiliency/resiliency-technical-guidance.md)

### <a name="best-practices"></a>最佳做法
我們建議您以符合災害復原計畫需求的頻率，將重要資料複製到位於其他區域的另一個 Data Lake Storage Gen1 帳戶。 您可以使用各種方法來複製資料，包括 [ADLCopy](data-lake-store-copy-data-azure-storage-blob.md)、[Azure PowerShell](data-lake-store-get-started-powershell.md) 或 [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md)。 Azure Data Factory 這個服務很適合用來反覆建立和部署資料移動管線。

發生區域性停電時，您就可以從資料所複製到的區域存取資料。 您可以監視 [Azure 服務健全狀況儀表板](https://azure.microsoft.com/status/)以確定全球各地 Azure 服務的狀態。

## <a name="data-corruption-or-accidental-deletion-recovery-guidance"></a>資料損毀或意外刪除復原指導方針
雖然 Data Lake Storage Gen1 可透過自動化複本提供資料復原功能，但這無法防止應用程式 (或開發人員/使用者) 的資料遭到損毀或意外刪除。

### <a name="best-practices"></a>最佳做法
若要防止意外刪除，建議您先為 Data Lake Storage Gen1 帳戶設定正確的存取原則。  這包括套用 [Azure 資源鎖定](../azure-resource-manager/resource-group-lock-resources.md)來鎖定重要資源，以及套用採用可用 [Data Lake Storage Gen1 安全性功能](data-lake-store-security-overview.md)的帳戶與檔案層級存取控制。 此外，我們也建議您使用 [ADLCopy](data-lake-store-copy-data-azure-storage-blob.md)、[Azure PowerShell](data-lake-store-get-started-powershell.md) 或 [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md)，在另一個 Data Lake Storage Gen1 帳戶、資料夾或 Azure 訂用帳戶定期建立重要資料的複本。  這可用來從資料損毀或刪除事件中復原。 Azure Data Factory 此服務很適合用來反覆建立和部署資料移動管線。

組織也可以為其 Data Lake Storage Gen1 帳戶啟用[診斷記錄](data-lake-store-diagnostic-logs.md)，以收集資料存取稽核線索來提供刪除或更新檔案之可疑人員的相關資訊。

## <a name="next-steps"></a>後續步驟
* [開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)

