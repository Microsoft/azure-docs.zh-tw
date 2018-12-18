---
title: Azure Data Lake Store 跨區域移轉 | Microsoft Docs
description: 了解 Azure Data Lake Store 跨區域移轉指引。
services: data-lake-store
documentationcenter: ''
author: swums
manager: amitkul
editor: swums
ms.assetid: ebde7b9f-2e51-4d43-b7ab-566417221335
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 01/27/2017
ms.author: stewu
ms.openlocfilehash: 1199eca457c3f06fdd6a4b68a05da3210ea9a2c9
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34197234"
---
# <a name="migrate-data-lake-store-across-regions"></a>跨區域移轉 Data Lake Store

隨著 Azure Data Lake Store 在新區域開放使用，您可能會選擇進行一次性移轉，以利用某個新區域。 了解在規劃和完成移轉時的一些考量。

## <a name="prerequisites"></a>先決條件

* **Azure 訂用帳戶**。 如需詳細資訊，請參閱[立即建立免費的 Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/)。
* **兩個不同區域中的 Data Lake Store 帳戶**。 如需詳細資訊，請參閱[開始使用 Azure Data Lake Store](data-lake-store-get-started-portal.md)。
* **Azure Data Factory**。 如需詳細資訊，請參閱 [Azure Data Factory 簡介](../data-factory/introduction.md)。


## <a name="migration-considerations"></a>移轉考量

首先，針對在 Azure Data Lake Store 中寫入、讀取或處理資料的應用程式，找出最合適的移轉策略。 當您選擇策略時，請考慮應用程式的可用性需求，以及移轉期間內發生的停機時間。 例如，最簡單的方法可能是使用「隨即轉移」雲端移轉模型。 採用採個方法，您會在所有資料複製到新區域時，暫停現有區域中的應用程式。 複製程序完成後，您可以在新區域中繼續執行應用程式，然後刪除舊的 Azure Data Lake Store 帳戶。 移轉期間需要停機。

若要減少停機時間，您可以立即開始在新的區域中內嵌新資料。 當您擁有所需的最少資料時，請在新區域中執行應用程式。 在背景中，繼續從現有的 Azure Data Lake Store 帳戶，將較舊的資料複製到新區域中的新 Data Lake Store 帳戶。 使用這個方法，您可以在轉換到新區域時只造成最少的停機時間。 複製好所有較舊的資料後，請刪除舊的 Data Lake Store 帳戶。

規劃移轉時要考量的其他重要詳細資料如下︰

* **資料量**。 資料量 (以 GB 為單位的檔案和資料夾等) 會影響移轉時所需的時間和資源。

* **Data Lake Store 帳戶名稱**。 新區域中的新帳戶名稱必須是全域唯一的。 例如，在美國東部 2 的舊 Data Lake Store 帳戶名稱可能是 contosoeastus2.azuredatalakestore.net。 您可能會將歐盟北部的新 Data Lake Store 帳戶命名為 contosonortheu.azuredatalakestore.net。

* **工具**。 建議您使用 [Azure Data Factory 複製活動](../data-factory/connector-azure-data-lake-store.md)來複製 Data Lake Store 檔案。 Data Factory 支援高效能與可靠性的資料移動。 請記住，Data Factory 只會複製資料夾階層和檔案內容。 您必須將用於舊帳戶的任何存取控制清單 (ACL) 手動套用到新帳戶。 如需詳細資訊 (包括最佳狀況案例的效能目標)，請參閱[複製活動的效能及微調指南](../data-factory/copy-activity-performance.md)。 如果您想要更快速地複製資料，您可能需要使用其他雲端資料移動單位。 AdlCopy 等其他工具不支援在區域之間複製資料。  

* **頻寬費用**。 適用[頻寬費用](https://azure.microsoft.com/pricing/details/bandwidth/)，因為資料會傳出 Azure 區域。

* **資料的 ACL**。 請對檔案和資料夾套用 ACL 以保護新區域內的資料。 如需詳細資訊，請參閱[在 Azure Data Lake Store 中保護資料](data-lake-store-secure-data.md)。 建議您使用移轉來更新和調整 ACL。 您想要使用的設定可能類似目前的設定。 您可以使用 Azure 入口網站、[PowerShell Cmdlet](/powershell/module/azurerm.datalakestore/get-azurermdatalakestoreitempermission) 或 SDK 檢視來套用到任何檔案的 ACL。  

* **分析服務的位置**。 為獲得最佳效能，分析服務 (例如 Azure Data Lake Analytics 或 Azure HDInsight) 應位於與資料相同的區域中。  

## <a name="next-steps"></a>後續步驟
* [Azure Data Lake Store 概觀](data-lake-store-overview.md)
