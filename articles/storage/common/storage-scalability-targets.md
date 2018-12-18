---
title: Azure 儲存體延展性和效能目標 | Microsoft Docs
description: 了解 Azure 儲存體的延展性和效能目標，包括標準和進階儲存體帳戶的容量、要求率以及輸入和輸出頻寬。 了解每一項 Azure 儲存體服務內分割的效能目標。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 10/24/2017
ms.author: rogarana
ms.component: common
ms.openlocfilehash: 1db7219beead4d7ee228baff4c802c105419ba1c
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39524598"
---
# <a name="azure-storage-scalability-and-performance-targets"></a>Azure 儲存體延展性和效能目標
## <a name="overview"></a>概觀
本文說明 Azure 儲存體的延展性和效能主題。 如需其他 Azure 限制的摘要，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](../../azure-subscription-service-limits.md)。

> [!NOTE]
> 所有儲存體帳戶都能在新的一般網路拓撲上執行，並支援本文中說明的延展性和效能目標，無論它們在何時建立。 如需 Azure 儲存體平面網路架構及延展性的詳細資訊，請參閱 [Microsoft Azure 儲存體：具有高度一致性的高可用性雲端儲存體服務](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)。
> 

> [!IMPORTANT]
> 列於此處的延展性和效能目標都是高階目標，但仍可達成。 在所有情況下，您的儲存體帳戶所達到的要求率和頻寬取決於已儲存物件的大小、使用的存取模式、應用程式執行的工作負載類型。 請務必測試您的服務，以判斷效能是否達到您的要求。 如果可能，請避免流量率突增，確保流量在不同分割之間妥善分散。
> 
> 當您的應用程式達到分割區可處理的工作負載限制時，Azure 儲存體會開始傳回錯誤碼 503 (伺服器忙碌) 或錯誤碼 500 (作業逾時) 回應。 如果發生這些錯誤，您的應用程式應該針對重試使用指數輪詢原則。 指數輪詢讓分割的負載減少，也能減輕該分割流量的尖峰。
> 
> 

如果您應用程式的需求超過單一儲存體帳戶的延展性目標，您可以將應用程式建置為使用多個儲存體帳戶。 接著您可以在這些儲存體帳戶之間分割資料物件。 如需批量價格的詳細資訊，請參閱 [Azure 儲存體價格](https://azure.microsoft.com/pricing/details/storage/) 。

## <a name="scalability-targets-for-a-storage-account"></a>儲存體帳戶的延展性目標
[!INCLUDE [azure-storage-limits](../../../includes/azure-storage-limits.md)]

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

## <a name="azure-blob-storage-scale-targets"></a>Azure Blob 儲存體擴展目標
[!INCLUDE [storage-blob-scale-targets](../../../includes/storage-blob-scale-targets.md)]

## <a name="azure-files-scale-targets"></a>Azure 檔案擴展目標
如需 Azure 檔案服務和 Azure 檔案同步的擴展目標與效能目標的詳細資訊，請參閱 [Azure 檔案服務延展性和效能目標](../files/storage-files-scale-targets.md)。

[!INCLUDE [storage-files-scale-targets](../../../includes/storage-files-scale-targets.md)]

### <a name="azure-file-sync-scale-targets"></a>Azure 檔案同步擴展目標
[!INCLUDE [storage-sync-files-scale-targets](../../../includes/storage-sync-files-scale-targets.md)]

## <a name="azure-queue-storage-scale-targets"></a>Azure 佇列儲存體擴展目標
[!INCLUDE [storage-queues-scale-targets](../../../includes/storage-queues-scale-targets.md)]

## <a name="azure-table-storage-scale-targets"></a>Azure 資料表儲存體擴展目標
[!INCLUDE [storage-table-scale-targets](../../../includes/storage-tables-scale-targets.md)]

## <a name="see-also"></a>另請參閱
* [儲存體定價詳細資料](https://azure.microsoft.com/pricing/details/storage/)
* [Azure 訂用帳戶和服務限制、配額與限制](../../azure-subscription-service-limits.md)
* [Azure 儲存體複寫](../storage-redundancy.md)
* [Microsoft Azure 儲存體效能與延展性檢查清單](../storage-performance-checklist.md)
* [Microsoft Azure 儲存體：具有高度一致性的高可用性雲端儲存體服務。](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)

