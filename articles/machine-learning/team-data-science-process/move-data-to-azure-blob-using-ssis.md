---
title: 使用 SSIS 連接器從 Azure Blob 儲存體來回移動資料 | Microsoft Docs
description: 使用 SSIS 連接器從 Azure Blob 儲存體來回移動資料。
services: machine-learning,storage
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 96a1b5fb-34d1-4b9b-8d99-2bb8289e0398
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: 1a0d8106c6a48494a938ee7a9da6f43f467516f6
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394012"
---
# <a name="move-data-to-or-from-azure-blob-storage-using-ssis-connectors"></a>使用 SSIS 連接器從 Azure Blob 儲存體來回移動資料
[SQL Server Integration Services Feature Pack for Azure](https://msdn.microsoft.com/library/mt146770.aspx) 中的元件可供連線至 Azure、在 Azure 與內部部署資源來源之間傳輸資料，以及處理儲存在 Azure 中的資料。

[!INCLUDE [blob-storage-tool-selector](../../../includes/machine-learning-blob-storage-tool-selector.md)]

客戶將內部部署資料移至雲端後，便能從任何 Azure 服務存取該資料，以利用 Azure 技術套件的完整功能。 例如，可用在 Azure 機器學習服務或 HDInsight 叢集中。

這通常是 [SQL](sql-walkthrough.md) 與 [HDInsight](hive-walkthrough.md) 逐步解說的第一步。

有關使用 SSIS 完成混合式資料整合案例中常見業務需求的典型案例討論，請參閱 [使用 Azure 適用的 SQL Server 整合服務功能套件事半功倍](http://blogs.msdn.com/b/ssis/archive/2015/06/25/doing-more-with-sql-server-integration-services-feature-pack-for-azure.aspx) 部落格。

> [!NOTE]
> 如需 Azure Blob 儲存體的完整介紹，請參閱 [Azure Blob 基本概念](../../storage/blobs/storage-dotnet-how-to-use-blobs.md)和 [Azure Blob 服務](https://msdn.microsoft.com/library/azure/dd179376.aspx)。
> 
> 

## <a name="prerequisites"></a>必要條件
若要執行本文所述的工作，您必須有設好的 Azure 訂用帳戶與 Azure 儲存體帳戶。 您必須知道您的 Azure 儲存體帳戶名稱與帳戶金鑰，才能上傳或下載資料。

* 若要設定 **Azure 訂用帳戶**，請參閱 [免費試用一個月](https://azure.microsoft.com/pricing/free-trial/)。
* 如需建立 **儲存體帳戶** 以及取得帳戶和金鑰資訊的指示，請參閱 [關於 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md)。

若要使用 **SSIS 連接器**，您必須下載:

* **SQL Server 2014 或 2016 標準版 (或更新版本)**：安裝包含 SQL Server 整合服務。
* **適用於 Azure 的 Microsoft SQL Server 2014 或 2016 整合服務 Feature Pack**：您可以個別從 [SQL Server 2014 整合服務](http://www.microsoft.com/download/details.aspx?id=47366)和 [SQL Server 2016 整合服務頁面](https://www.microsoft.com/download/details.aspx?id=49492)下載。

> [!NOTE]
> SSIS 會隨同 SQL Server 安裝，但未包含在 Express 版本中。 如需 SQL Server 各種版本中包含哪些應用程式的相關資訊，請參閱 [SQL Server 版本](http://www.microsoft.com/en-us/server-cloud/products/sql-server-editions/)
> 
> 

如需 SSIS 的訓練教材，請參閱 [SSIS 實務訓練](https://www.microsoft.com/sql-server/training-certification)

如需如何使用 SISS 啟動與執行，以建置簡單擷取、轉換與載入 (ETL) 封裝的相關資訊，請參閱 [SSIS 教學課程：建立簡易 ETL 封裝](https://msdn.microsoft.com/library/ms169917.aspx)。

## <a name="download-nyc-taxi-dataset"></a>下載紐約計程車資料集
此處描述的範例使用供大眾使用的資料集 -- [紐約計程車路線](http://www.andresmh.com/nyctaxitrips/) 資料集。 此資料集收集了 2013 年紐約 1 億 7 千 3 百萬筆計程車行程資料。 資料有兩種：路線詳細資料及車費資料。 由於每個月都有一個檔案，因此總共有 24 個檔案，每個檔案未壓縮的大小約 2GB。

## <a name="upload-data-to-azure-blob-storage"></a>將資料上傳至 Azure Blob 儲存體
為了使用 SSIS 功能封裝將資料從內部部署移至 Azure Blob 儲存體，我們使用 [**Azure Blob 上傳工作**](https://msdn.microsoft.com/library/mt146776.aspx)執行個體，如這裡所示：

![configure-data-science-vm](./media/move-data-to-azure-blob-using-ssis/ssis-azure-blob-upload-task.png)

此工作使用的參數如下所述：

| 欄位 | 說明 |
| --- | --- |
| **AzureStorageConnection** |指定現有的或建立新的 Azure 儲存體連線管理員，以參照 Azure 儲存體帳戶；該帳戶指向託管 Blob 檔案的位置。 |
| **BlobContainer** |指定會將上傳的檔案保存為 Blob 的 Blob 容器名稱。 |
| **BlobDirectory** |指定上傳的檔案會儲存在哪一個 Blob 目錄中以儲存成區塊 Blob。 Blob 目錄是虛擬的階層式結構。 如果 Blob 已經存在，則會被取代。 |
| **LocalDirectory** |指定包含要上傳的檔案的本機目錄。 |
| **FileName** |指定名稱篩選器以選取採用指定名稱模式的檔案。 例如，MySheet\*.xls\* 包括如 MySheet001.xls 與 MySheetABC.xlsx 等檔案 |
| **TimeRangeFrom/TimeRangeTo** |指定時間範圍篩選器。 會包含在 *TimeRangeFrom* 後和 *TimeRangeTo* 前修改的檔案。 |

> [!NOTE]
> **AzureStorageConnection** 認證必須更正且 **BlobContainer** 必須存在，才能嘗試傳輸。
> 
> 

## <a name="download-data-from-azure-blob-storage"></a>從 Azure Blob 儲存體下載資料
若要透過 SSIS 將資料從 Azure Blob 儲存體下載到內部部署儲存體，請使用 [Azure Blob 下載工作](https://msdn.microsoft.com/library/mt146779.aspx)的執行個體。

## <a name="more-advanced-ssis-azure-scenarios"></a>較進階的 SSIS-Azure 案例
SSIS 功能套件藉由將工作封裝在一起，處理更複雜的流程。 例如，Blob 資料可直接饋送到 HDInsight 叢集，其輸出可以回頭下載到 Blob，然後下載到內部部署儲存體。 SSIS 可使用其他 SSIS 連接器在 HDInsight 叢集上執行 Hive 與 Pig 工作：

* 若要使用 SSIS 在 Azure HDInsight 叢集上執行 Hive 指令碼，請使用 [Azure HDInsight Hive 工作](https://msdn.microsoft.com/library/mt146771.aspx)。
* 若要使用 SSIS 在 Azure HDInsight 叢集上執行 Pig 指令碼，請使用 [Azure HDInsight Pig 工作](https://msdn.microsoft.com/library/mt146781.aspx)。

