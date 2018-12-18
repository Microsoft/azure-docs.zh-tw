---
title: Azure Stack 儲存體簡介
description: 了解 Azure Stack 儲存體
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.assetid: 092aba28-04bc-44c0-90e1-e79d82f4ff42
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 05/21/2018
ms.author: mabrigg
ms.openlocfilehash: d97a5f8aff57f4bbfd7d5222a87d258fa5c92da8
ms.sourcegitcommit: 680964b75f7fff2f0517b7a0d43e01a9ee3da445
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34604381"
---
# <a name="introduction-to-azure-stack-storage"></a>Azure Stack 儲存體簡介

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

## <a name="overview"></a>概觀

Azure Stack 儲存體是一組雲端儲存體服務，包括與 Azure 儲存體服務一致的 Blob、資料表和佇列。

## <a name="azure-stack-storage-services"></a>Azure Stack 儲存體服務

Azure Stack 儲存體提供下列三項服務：

- **Blob 儲存體**

    Blob 儲存體可儲存非結構化物件資料。 Blob 可以是任何類型的文字或二進位資料，例如文件、媒體檔案或應用程式安裝程式。

- **資料表儲存體**

    資料表儲存體可儲存結構化的資料集。 表格儲存體屬於 NoSQL 索引鍵屬性資料儲存，可允許快速開發和迅速存取大量資料。

- **佇列儲存體**

    佇列儲存體針對工作流程處理及雲端服務元件間的通訊，提供可靠的訊息服務。

Azure Stack 儲存體帳戶是可讓您存取 Azure Stack 儲存體服務的安全帳戶。 您的儲存體帳戶提供儲存體資源的唯一命名空間。 下圖顯示儲存體帳戶中 Azure Stack 儲存體資源之間的關係：

![Azure Stack 儲存體概觀](media/azure-stack-storage-overview/AzureStackStorageOverview.png)

### <a name="blob-storage"></a>Blob 儲存體

對於擁有大量非結構化物件資料要儲存於雲端的使用者，Blob 儲存體提供有效且可調整的解決方案。 您可以使用 Blob 儲存體來儲存如下所示的內容：

- 文件
- 社交資料 (例如照片、視訊、音樂和部落格)
- 檔案、電腦、資料庫和裝置的備份
- Web 應用程式的影像和文字
- 雲端應用程式的組態資料
- 巨量資料 (例如記錄和其他大型資料集)

每個 Blob 會組織成一個容器。 容器也提供對物件群組指派安全原則的實用方式。 儲存體帳戶可包含任意數目的容器，而容器可包含任意數目的 Blob，最高可達儲存體帳戶的限制。

Blob 儲存體提供三種類型的 Blob：

- **區塊 Blob**

    區塊 Blob 已針對串流和儲存雲端物件進行最佳化，是儲存文件、媒體檔案、備份等的不錯選擇。

- **附加 Blob**

    附加 Blob 和區塊 Blob 類似，但已針對附加作業最佳化。 附加 Blob 只能透過在結尾新增新的區塊來更新。 對於如記錄等僅需要將新資料寫入 Blob 結尾的情況，附加 Blob 是不錯的選擇。

- **分頁 Blob**

    分頁 Blob 已針對代表 IaaS 磁碟與支援隨機寫入進行最佳化，且大小可以高達 1 TB。 Azure Stack 虛擬機器連結的 IaaS 磁碟是以分頁 Blob 方式儲存的 VHD。

### <a name="table-storage"></a>表格儲存體

新式應用程式通常會要求以超越前幾代軟體需求的延伸性和彈性儲存資料。 資料表儲存體提供高可用性且可大幅擴充的儲存體，方便您的應用程式自動擴充以滿足使用者需求。 表格儲存體屬於 Microsoft 的 NoSQL 索引鍵屬性儲存，它的無結構描述設計讓它有別於傳統的關聯式資料庫。 透過無結構描述資料儲存，便可輕易隨著應用程式發展需求改寫資料。 表格儲存體非常容易使用，方便開發人員可以快速建立應用程式。

表格儲存體屬於索引鍵屬性儲存，代表資料表中的每個值會與具型別的屬性名稱一起儲存。 屬性名稱可用來篩選與指定選取條件。 一組屬性及其值就構成一個實體。 因為表格儲存體沒有結構描述，因此相同資料表中的兩個實體可包含不同屬性集合，且這些屬性可以是不同類型。

您可以使用表格儲存體來儲存具彈性的資料集，例如 Web 應用程式的使用者資料、通訊錄、裝置資訊，以及服務所需的任何其他中繼資料類型。 在現今的網際網路架構應用程式中，NoSQL 資料庫 (如資料表儲存體) 提供傳統關聯式資料庫的熱門替代方式。

儲存體帳戶可包含任意數目的資料表，而資料表可包含任意數目的實體，最高可達儲存體帳戶的容量限制。

### <a name="queue-storage"></a>佇列儲存體

設計擴充性的應用程式時，會經常分離應用程式元件，以便進行個別擴充。 佇列儲存體針對應用程式元件間的非同步通訊，提供可靠的訊息服務解決方案，無論應用程式元件是在雲端、桌面、內部部署伺服器或行動裝置上執行。 佇列儲存體也支援管理非同步工作並建置處理工作流程。

儲存體帳戶可包含任意數目的佇列，而佇列可包含任意數目的訊息，最高可達儲存體帳戶的容量限制。 個別訊息的大小可能高達 64 KB。

## <a name="next-steps"></a>後續步驟

- [與 Azure 一致的儲存體：差異與注意事項](azure-stack-acs-differences.md)

- 若要深入了解 Azure 儲存體，請參閱 [Microsoft Azure 儲存體簡介](../../storage/common/storage-introduction.md)
