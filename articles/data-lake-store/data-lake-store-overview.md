---
title: Azure Data Lake Storage Gen1 概觀 | Microsoft Docs
description: 了解什麼是 Data Lake Storage Gen1 (以前稱為 Azure Data Lake Store) 以及它提供其他資料存放區所沒有的價值
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: nitinme
ms.openlocfilehash: b733a0bec20c7abae5df41acd74284bdf75bca2c
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46124128"
---
# <a name="overview-of-azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1 概觀

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Azure Data Lake Storage Gen1 是容納巨量資料分析工作負載的企業級超大規模存放庫。 Azure Data Lake 可讓您在單一位置擷取任何大小、類型和擷取速度的資料，以便進行運作和探究分析。

> [!TIP]
> 使用 [Data Lake Storage Gen1 學習路徑](https://azure.microsoft.com/documentation/learning-paths/data-lake-store-self-guided-training/)開始探索 Data Lake Storage Gen1 服務。
> 
> 

使用與 WebHDFS 相容的 REST API，可以從 Hadoop (HDInsight 叢集所提供) 存取 Data Lake Storage Gen1。 它專為預存資料分析而設計，並針對資料分析案例而調整效能。 根據預設，它包含真實企業使用案例不可或缺的所有企業級功能：安全性、管理性、延展性、可靠性和可用性。

![Azure Data Lake](./media/data-lake-store-overview/data-lake-store-concept.png)

Data Lake Storage Gen1 的一些重要功能包括下列項目。

### <a name="built-for-hadoop"></a>專為 Hadoop 而建置
Data Lake Storage Gen1 是與 Hadoop 分散式檔案系統 (HDFS) 相容的 Apache Hadoop 檔案系統，可搭配 Hadoop 生態系統使用。  採用 WebHDFS API 的現有 HDInsight 應用程式或服務可以輕易地與 Data Lake Storage Gen 1 整合。 Data Lake Storage Gen1 也會公開適用於應用程式的 WebHDFS 相容 REST 介面

使用 Hadoop 分析架構 (例如 MapReduce 或 Hive)，可以輕鬆地分析 Data Lake Storage Gen1 中儲存的資料。 您可以佈建並設定 Microsoft Azure HDInsight 叢集，以直接存取 Data Lake Storage Gen1 中儲存的資料。

### <a name="unlimited-storage-petabyte-files"></a>無限制的儲存空間、PB 大小的檔案
Data Lake Storage Gen1 提供無限制的儲存空間，適合用來儲存各種資料以供分析。 對於帳戶大小、檔案大小，或資料湖中可儲存的資料量，均無任何限制。 個別檔案的大小可從 KB 到 PB，很適合用來儲存任何類型的資料。 透過製作多個複本來長期儲存資料，而資料可以儲存在資料湖中的持續時間並沒有限制。

### <a name="performance-tuned-for-big-data-analytics"></a>針對巨量資料分析調整效能
Data Lake Storage Gen1 專為執行大型分析系統而建置，而此類系統需要龐大輸送量才能查詢及分析大量資料。 資料湖會將檔案的各個部分散於數個個別的儲存體伺服器。 這可改善以平行方式讀取檔案以便執行資料分析時的輸送量。

### <a name="enterprise-ready-highly-available-and-secure"></a>符合企業需求：高度可用且安全
Data Lake Storage Gen1 提供符合業界標準的可用性與可靠性。 您的資料資產可透過製作備援複本來長期儲存，以防範任何未預期的失敗。 企業可以在其解決方案中使用 Data Lake Storage Gen1，以成為其現有資料平台的重要部分。

Data Lake Storage Gen1 也可對預存資料提供企業級安全性。 如需詳細資訊，請參閱[在 Azure Data Lake Storage Gen1 中保護資料](#DataLakeStoreSecurity)。

### <a name="all-data"></a>所有資料
Data Lake Storage Gen1 能以原生格式 (原樣) 儲存任何資料，而不需要任何先前的轉換。 載入資料前，Data Lake Storage Gen1 不需要定義結構描述，而是留待個別的分析架構在分析時解譯資料及定義結構描述。 Data Lake Storage Gen1 能夠儲存任意大小與格式的檔案，因此可以處理結構化、半結構化與非結構化資料。

Data Lake Storage Gen1 的資料容器基本上是資料夾與檔案。 您可以使用 SDK、Azure 入口網站與 Azure Powershell 來操作儲存的資料。 只要您使用這些介面和適當容器將資料放入存放區中，就可以儲存任何類型的資料。 Data Lake Storage Gen1 不會根據其儲存的資料類型來對資料執行任何特殊處理。

## <a name="DataLakeStoreSecurity"></a>在 Data Lake Storage Gen1 中保護資料
Data Lake Storage Gen1 會使用 Azure Active Directory 進行驗證，並使用存取控制清單 (ACL) 來管理對資料的存取。

| 功能 | 說明 |
| --- | --- |
| 驗證 |Data Lake Storage Gen1 整合了 Azure Active Directory (AAD)，可對 Data Lake Storage Gen1 中儲存的所有資料進行身分識別與存取管理。 由於整合的結果，Data Lake Storage Gen1 受惠於所有 AAD 功能，包括 Multi-Factor Authentication、條件式存取、角色型存取控制、應用程式使用情況監視、安全性監視與警示等。Data Lake Storage Gen1 支援 OAuth 2.0 通訊協定以便在 REST 介面中進行驗證。 請參閱 [Data Lake Storage Gen1 驗證](data-lakes-store-authentication-using-azure-active-directory.md)|
| 存取控制 |Data Lake Storage Gen1 透過支援 WebHDFS 通訊協定所公開的 POSIX 樣式權限，以提供存取控制。 ACL 可在根資料夾、子資料夾與個別檔案上啟用。 如需 ACL 如何在 Data Lake Storage Gen1 的環境中運作的詳細資訊，請參閱 [Data Lake Storage Gen1 中的存取控制](data-lake-store-access-control.md)。 |
| 加密 |Data Lake Storage Gen1 也會為帳戶中儲存的資料提供加密功能。 您會在建立 Data Lake Storage Gen1 帳戶時指定加密設定。 您可以選擇將資料加密，或選擇不使用加密。 如需詳細資訊，請參閱 [Data Lake Storage Gen1 的加密](data-lake-store-encryption.md)。 如需如何提供加密相關設定的指示，請參閱[使用 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)。 |

想要深入了解如何在 Data Lake Storage Gen1 中保護資料？ 請依照下列連結執行。

* 如需有關如何在 Data Lake Storage Gen1 中保護資料的指示，請參閱[在 Azure Data Lake Storage Gen1 中保護資料](data-lake-store-secure-data.md)。
* 偏好影片？ [觀看此影片](https://mix.office.com/watch/1q2mgzh9nn5lx)以了解如何保護 Data Lake Storage Gen1 中儲存的資料。

## <a name="applications-compatible-with-data-lake-storage-gen1"></a>與 Data Lake Storage Gen1 相容的應用程式
Data Lake Storage Gen1 與 Hadoop 生態系統中的大部分開放原始碼元件相容。 此外，它還與其他 Azure 服務完美整合。 這讓 Data Lake Storage Gen1 成為符合您資料儲存需求的最佳選項。 請依照下面的連結，深入了解 Data Lake Storage Gen1 如何搭配開放原始碼元件與其他 Azure 服務使用。

* 如需可與 Data Lake Storage Gen1 互通的開放原始碼應用程式清單，請參閱[與 Azure Data Lake Storage Gen1 相容的應用程式與服務](data-lake-store-compatible-oss-other-applications.md)。
* 請參閱[與其他 Azure 服務整合](data-lake-store-integrate-with-other-services.md)，以了解 Data Lake Storage Gen1 如何搭配其他 Azure 服務使用，以促成更廣泛的案例。
* 請參閱[使用 Data Lake Storage Gen1 的案例](data-lake-store-data-scenarios.md)，以了解如何在擷取資料、處理資料、下載資料和視覺化資料等案例中使用 Data Lake Storage Gen1。

## <a name="what-is-data-lake-storage-gen1-file-system-adl"></a>什麼是 Data Lake Storage Gen1 檔案系統 (adl://)？
在 Hadoop 環境中可透過新的檔案系統 (AzureDataLakeFilesystem (adl://)) 存取 Data Lake Storage Gen1 (透過 HDInsight 叢集提供)。 使用 adl:// 的應用程式和服務能夠利用 WebHDFS 中目前無法使用的進一步效能最佳化。 因此，Data Lake Storage Gen1 讓您具有彈性：可透過使用 adl:// 的建議選項獲得最佳效能，或繼續直接使用 WebHDFS API 來維護現有的程式碼。 Azure HDInsight 充分利用 AzureDataLakeFilesystem 來提供 Data Lake Storage Gen1 的最佳效能。

您可以使用 `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net`，在 Data Lake Storage Gen1 中存取您的資料。 如需有關如何在 Data Lake Storage Gen1 中存取資料的詳細資訊，請參閱[檢視預存資料的屬性](data-lake-store-get-started-portal.md#properties)

## <a name="next-steps"></a>後續步驟

* [使用 Azure 入口網站開始使用 Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* [使用 .NET SDK 開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-net-sdk.md)
* [搭配 Data Lake Storage Gen1 使用 Azure HDInsight](data-lake-store-hdinsight-hadoop-use-portal.md)