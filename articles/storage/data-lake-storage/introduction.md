---
title: Azure Data Lake Storage Gen2 預覽版介紹
description: 提供 Azure Data Lake Storage Gen2 預覽版概觀
services: storage
author: jamesbak
ms.service: storage
ms.topic: article
ms.date: 06/27/2018
ms.author: jamesbak
ms.component: data-lake-storage-gen2
ms.openlocfilehash: 072573b16fbeebac1ec942b0be508cf901b5cd27
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42144197"
---
# <a name="introduction-to-azure-data-lake-storage-gen2-preview"></a>Azure Data Lake Storage Gen2 預覽版介紹

Azure Data Lake Storage Gen2 預覽版是一組巨量資料分析的專屬功能，內建於 [Azure Blob 儲存體](../blobs/storage-blobs-introduction.md)上。 此功能可讓您使用檔案系統和物件儲存範例連接您的資料。 這使得 Data Lake Storage Gen2 成為唯一的雲端式多重強制回應儲存體服務，可讓您擷取所有資料的分析值。

Data Lake Storage Gen2 具有分析資料完整生命週期所需的所有品質。 這是融合我們現有的兩項儲存體服務之功能所產生。 [Azure Data Lake Storage Gen1](../../data-lake-store/index.md) 的功能 (例如檔案系統語意、檔案層級安全性和級別) 結合了 [Azure Blob 儲存體](../blobs/storage-blobs-introduction.md)的低成本、分層式儲存體、高可用性/災害復原功能及大型 SDK/工具生態系統。 在 Data Lake Storage Gen2 中，所有物件儲存體的品質都維持不變，另外還增加了經過分析工作負載最佳化的檔案系統介面優點。

## <a name="designed-for-enterprise-big-data-analytics"></a>針對企業巨量資料分析所設計

Data Lake Storage Gen2 是在 Azure 上打造企業 Data Lake (EDL) 的基礎儲存體服務。 Data Lake Storage Gen2 從一開始就設計為服務數 PB 的資訊，同時可以維持數百 GB 的輸送量，可讓您輕鬆地管理大量資料。

Data Lake Storage Gen2 的基本功能是將[階層式命名空間](./namespace.md)加入 Blob 儲存體服務；該服務可將物件/檔案整理成階層式目錄，以進行高效能資料存取。 階層式命名空間也可讓 Data Lake Storage Gen2 同時支援物件存放區和檔案系統範本。 例如，一般物件存放區命名慣例是在名稱中使用斜線來模仿階層式資料夾結構。 Data Lake Storage Gen2 會使這樣的結構成真。 重新命名或刪除目錄等操作會成為目錄中單一不可部分完成的中繼資料作業，而不是列舉及處理共用目錄名稱前置詞的所有物件。

以往雲端式分析必須在效能、管理及安全性方面妥協。 Data Lake Storage Gen2 透過下列方式解決這些方面的問題：

- **效能**經過最佳化，因為您不需要複製或轉換資料作為分析的必要條件。 階層式命名空間大幅提高目錄管理作業的效能，從而提高整體作業效能。

- **管理**更容易，因為您可以透過目錄和子目錄整理和操作檔案。

- **安全性**是強制的，因為您可以在資料夾或個人檔案上定義 POSIX 權限。

- **符合成本效益**，因為 Data Lake Storage Gen2 建立在低成本的 [Azure Blob 儲存體](../blobs/storage-blobs-introduction.md)上。 額外功能進一步降低了在 Azure 上執行巨量資料分析的擁有權總成本。

## <a name="key-features-of-data-lake-storage-gen2"></a>Data Lake Storage Gen2 的主要功能

> [!NOTE]
> 在 Data Lake Storage Gen2 公開預覽期間，以下所列部分功能的可用性可能會有所不同。 在預覽計畫期間發布新功能和區域時，將會傳達此資訊。
> [註冊](https://aka.ms/adlsgen2signup) Data Lake Storage Gen2 的公開預覽。  

- **Hadoop 相容存取**：Data Lake Storage Gen2 可讓您管理及存取資料，就如同使用 [Hadoop 分散式檔案系統 (HDFS)](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html) 一樣。 全新 [ABFS 驅動程式](./abfs-driver.md)可在所有 Apache Hadoop 環境中使用，包括 [Azure HDInsight](../../hdinsight/index.yml) 和 [Azure Databricks](../../azure-databricks/index.yml)，以存取儲存在 Data Lake Storage Gen2 中的資料。

- **POSIX 權限的超集合**：Data Lake Gen2 的安全性模型可完全支援 ACL 和 POSIX 權限，以及一些 Data Lake Storage Gen2 特有的額外細微姓。 這些設定可透過系統管理工具或 Hive 和 Spark 這類架構來配置。

- **符合成本效益**：Data Lake Storage Gen2 具有低成本儲存體容量和異動功能。 隨著資料在整個生命週期中進行轉換，計費率會有所更改，透過 [Azure Blob 儲存體生命週期](../common/storage-lifecycle-managment-concepts.md)等內建功能將成本降到最低。

- **可搭配使用 Blob 儲存體工具、架構及應用程式**：Data Lake Storage Gen2 可繼續搭配使用現今仍有的各種 Blob 儲存體工具、架構及應用程式。

- **已最佳化的驅動程式**：此 `abfs` 驅動程式已特別針對巨量資料分析[完成最佳化](./abfs-driver.md)。 相應的 REST API 透過`dfs`端點`dfs.core.windows.net`呈現。

## <a name="scalability"></a>延展性

無論您是透過 Data Lake Storage Gen2 或 Blob 儲存體介面存取，Azure 儲存體都可以隨設計調整。 而且能夠儲存和使用*數 EB 的資料*。 這樣的儲存量可用於在每秒高輸入/輸出作業 (IOPS) 時以每秒 GB (Gbps) 為單位測量的輸送量。 除了持續性之外，處理作業是在近常數的每個要求延遲時執行的，這些延遲是在服務、帳戶及檔案層級上所測得。

## <a name="cost-effectiveness"></a>符合成本效益

在 Azure Blob 儲存體上建立 Data Lake Storage Gen2 的眾多好處之一，在於儲存容量和異動的[成本低](https://azure.microsoft.com/pricing/details/storage)。 Data Lake Storage Gen2 和其他雲端儲存體服務不同之處在於可降低成本，因為在執行分析之前不需要移動或轉換資料。

此外，例如[階層式命名空間](./namespace.md)等功能可大幅提升許多分析作業的整體效能。 效能提升即表示處理數量相同的資料時，所需的計算能力較少，因此可降低端對端分析工作的擁有權總成本 (TCO)。

## <a name="next-steps"></a>後續步驟

下列文章說明 Data Lake Storage Gen2 的一些主要概念，並詳述如何儲存、存取與管理資料，以及取得資料見解：

* [階層式命名空間](./namespace.md)
* [建立儲存體帳戶](./quickstart-create-account.md)
* [建立搭配 Azure Data Lake Storage Gen2 的 HDInsight 叢集](./quickstart-create-connect-hdi-cluster.md)
* [在 Azure Databricks 中使用 Azure Data Lake Storage Gen2 帳戶](./quickstart-create-databricks-account.md) 