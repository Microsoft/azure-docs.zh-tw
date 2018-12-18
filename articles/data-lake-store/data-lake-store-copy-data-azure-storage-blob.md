---
title: 將資料從 Azure 儲存體 Blob 複製到 Azure Data Lake Storage Gen1 | Microsoft Docs
description: 使用 AdlCopy 工具將資料從 Azure 儲存體 Blob 複製到 Azure Data Lake Storage Gen1
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: dc273ef8-96ef-47a6-b831-98e8a777a5c1
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 0cf5ace29dabd3a55524fe38403a07e3916ea7d6
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44390887"
---
# <a name="copy-data-from-azure-storage-blobs-to-azure-data-lake-storage-gen1"></a>將資料從 Azure 儲存體 Blob 複製到 Azure Data Lake Storage Gen1
> [!div class="op_single_selector"]
> * [使用 DistCp](data-lake-store-copy-data-wasb-distcp.md)
> * [使用 AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)
>
>

Azure Data Lake Storage Gen1 提供命令列工具 [AdlCopy](http://aka.ms/downloadadlcopy)，以從下列來源複製資料：

* 從 Azure 儲存體 Blob 複製到 Data Lake Storage Gen1。 您無法使用 AdlCopy 將資料從 Data Lake Storage Gen1 複製到 Azure 儲存體 Blob。
* 在兩個 Azure Data Lake Storage Gen1 帳戶之間。

此外，您可以兩個不同的模式使用 AdlCopy 工具：

* **單獨使用**，此工具會使用 Data Lake Storage Gen1 資源來執行工作。
* **使用 Data Lake Analytics 帳戶**，指派給 Data Lake Analytics 帳戶的單位可用來執行複製作業。 當您想要以可預測的方式執行複製工作時，可能會想使用此選項。

## <a name="prerequisites"></a>必要條件
開始閱讀本文之前，您必須符合下列必要條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure 儲存體 Blob** 容器 (其中含有一些資料)。
* **Azure Data Lake Storage Gen1 帳戶**。 如需如何建立帳戶的指示，請參閱[開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* **Azure Data Lake Analytics 帳戶 (選用)** - 如需如何建立 Data Lake Analytics 帳戶的指示，請參閱[開始使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)。
* **AdlCopy 工具**。 從 [http://aka.ms/downloadadlcopy](http://aka.ms/downloadadlcopy) 安裝 AdlCopy 工具。

## <a name="syntax-of-the-adlcopy-tool"></a>AdlCopy 工具的語法
利用下列語法來使用 AdlCopy 工具

    AdlCopy /Source <Blob or Data Lake Storage Gen1 source> /Dest <Data Lake Storage Gen1 destination> /SourceKey <Key for Blob account> /Account <Data Lake Analytics account> /Units <Number of Analytics units> /Pattern

以下將說明語法中的參數：

| 選項 | 說明 |
| --- | --- |
| 來源 |指定來源資料在 Azure 儲存體 Blob 中的位置。 來源可以是 Blob 容器、Blob 或其他 Data Lake Storage Gen1 帳戶。 |
| Dest |指定要複製的 Data Lake Storage Gen1 目的地。 |
| SourceKey |指定 Azure 儲存體 Blob 來源的儲存體存取金鑰。 這只有在來源是 Blob 容器或 Blob 時才是必要的。 |
| 帳戶 |**選用**。 如果您想要使用 Azure Data Lake Analytics 帳戶來執行複製工作，請使用此選項。 如果您在語法中使用 /Account 選項，但未指定 Data Lake Analytics 帳戶，AdlCopy 就會使用預設帳戶來執行工作。 此外，如果您使用此選項，就必須加入來源 (Azure 儲存體 Blob) 和目的地 (Azure Data Lake Storage Gen1) 做為 Data Lake Analytics 帳戶的資料來源。 |
| Units |指定將針對複製工作使用的 Data Lake Analytics 單位數目。 如果您使用 **/Account** 選項來指定 Data Lake Analytics 帳戶，則此為必要選項。 |
| 模式 |指定用來指示要複製哪些 Blob 或檔案的 regex 模式。 AdlCopy 使用區分大小寫的比對。 不指定任何模式時所使用的預設模式是複製所有項目。 不支援指定多個檔案模式。 |

## <a name="use-adlcopy-as-standalone-to-copy-data-from-an-azure-storage-blob"></a>使用 AdlCopy (獨立) 從 Azure 儲存體 Blob 複製資料
1. 開啟命令提示字元，並瀏覽至安裝 AdlCopy 的目錄，通常是 `%HOMEPATH%\Documents\adlcopy`。
2. 執行下列命令，將特定的 Blob 從來源容器複製到 Data Lake Storage Gen1 資料夾：

        AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>

    例如︰

        AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b.log /dest swebhdfs://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==

    >[!NOTE] 
    >上述語法指定要複製到 Data Lake Storage Gen1 帳戶內資料夾中的檔案。 如果指定的資料夾名稱不存在，AdlCopy 工具會建立一個資料夾。

    系統會提示您輸入您 Data Lake Storage Gen1 帳戶所在 Azure 訂用帳戶的認證。 您將看到類似以下的輸出：

        Initializing Copy.
        Copy Started.
        100% data copied.
        Finishing Copy.
        Copy Completed. 1 file copied.

1. 您也可以使用下列命令，將所有的 Blob 從某一個容器複製到 Data Lake Storage Gen1 帳戶：

        AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/ /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>        

    例如︰

        AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest adl://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==

### <a name="performance-considerations"></a>效能考量

如果您從 Azure Blob 儲存體帳戶複製，Blob 儲存體端在複製期間可能會進行節流。 這會降低複製作業的效能。 若要深入了解 Azure Blob 儲存體的限制，請參閱 [Azure 訂用帳戶和服務限制](../azure-subscription-service-limits.md)中的 Azure 儲存體限制。

## <a name="use-adlcopy-as-standalone-to-copy-data-from-another-data-lake-storage-gen1-account"></a>使用 AdlCopy (獨立) 從另一個 Data Lake Storage Gen1 帳戶複製資料
您也可以使用 AdlCopy 在兩個 Data Lake Storage Gen1 帳戶之間複製資料。

1. 開啟命令提示字元，並瀏覽至安裝 AdlCopy 的目錄，通常是 `%HOMEPATH%\Documents\adlcopy`。
2. 執行下列命令，從 Data Lake Store 帳戶將特定檔案複製到另一個 Data Lake Storage Gen1 帳戶。

        AdlCopy /Source adl://<source_adlsg1_account>.azuredatalakestore.net/<path_to_file> /dest adl://<dest_adlsg1_account>.azuredatalakestore.net/<path>/

    例如︰

        AdlCopy /Source adl://mydatastorage.azuredatalakestore.net/mynewfolder/909f2b.log /dest adl://mynewdatalakestorage.azuredatalakestore.net/mynewfolder/

   > [!NOTE]
   > 上述語法會指定要複製到目的地 Data Lake Storage Gen1 帳戶內資料夾中的檔案。 如果指定的資料夾名稱不存在，AdlCopy 工具會建立一個資料夾。
   >
   >

    系統會提示您輸入您 Data Lake Storage Gen1 帳戶所在 Azure 訂用帳戶的認證。 您將看到類似以下的輸出：

        Initializing Copy.
        Copy Started.|
        100% data copied.
        Finishing Copy.
        Copy Completed. 1 file copied.
3. 下列命令會將來源 Data Lake Storage Gen1 帳戶之特定資料夾中的所有檔案，複製到目的地 Data Lake Storage Gen1 帳戶中的資料夾。

        AdlCopy /Source adl://mydatastorage.azuredatalakestore.net/mynewfolder/ /dest adl://mynewdatalakestorage.azuredatalakestore.net/mynewfolder/

### <a name="performance-considerations"></a>效能考量

使用 AdlCopy 作為獨立工具時，複製是在共用的 Azure 受控資源上執行。 您在此環境中的可達到的效能取決於系統負載和可用的資源。 這個模式最適合臨時的少量傳輸。 使用 AdlCopy 作為獨立工具時不需要調整參數。

## <a name="use-adlcopy-with-data-lake-analytics-account-to-copy-data"></a>使用 AdlCopy (利用 Data Lake Analytics 帳戶) 複製資料
您也可以使用 Data Lake Analytics 帳戶來執行 AdlCopy 工作，將資料從 Azure 儲存體 Blob 複製到 Data Lake Storage Gen1。 若要移動的資料大小範圍在 GB 或 TB 內，而且您想要獲得更好且可預期的效能輸送量，您通常會使用此選項。

若要使用您的 Data Lake Analytics 帳戶與 AdlCopy 從 Azure 儲存體 Blob 複製，必須將來源 (Azure 儲存體 Blob) 新增為您 Data Lake Analytics 帳戶的資料來源。 如需將其他資料來源加入至 Data Lake Analytics 帳戶的指示，請參閱[管理 Data Lake Analytics 帳戶資料來源](../data-lake-analytics/data-lake-analytics-manage-use-portal.md#manage-data-sources)。

> [!NOTE]
> 如果您是使用 Data Lake Analytics 帳戶，從 Azure Data Lake Storage Gen1 帳戶複製做為來源，則您不需要將 Data Lake Storage Gen1 帳戶與 Data Lake Analytics 帳戶建立關聯。 當來源為 Azure 儲存體帳戶時，才需要將來源儲存體與 Data Lake Analytics 帳戶產生關聯。
>
>

執行下列命令，使用 Data Lake Analytics 帳戶，從 Azure 儲存體 Blob 複製到 Data Lake Storage Gen1 帳戶：

    AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Account <data_lake_analytics_account> /Units <number_of_data_lake_analytics_units_to_be_used>

例如︰

    AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest swebhdfs://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Account mydatalakeanalyticaccount /Units 2

同樣地，執行下列命令，使用 Data Lake Analytics 帳戶將來源 Data Lake Storage Gen1 帳戶之特定資料夾中的所有檔案複製到目的地 Data Lake Storage Gen1 帳戶中的資料夾：

    AdlCopy /Source adl://mysourcedatalakestorage.azuredatalakestore.net/mynewfolder/ /dest adl://mydestdatastorage.azuredatalakestore.net/mynewfolder/ /Account mydatalakeanalyticaccount /Units 2

### <a name="performance-considerations"></a>效能考量

複製 TB 規模的資料時，使用 AdlCopy 搭配您自己的 Azure Data Lake Analytics 帳戶可達到更好和更可預測的效能。 應該調整的參數為用於複製作業的 Azure Data Lake Analytics 單位數目。 增加單位數目可提高複製作業的效能。 每個要複製的檔案最多可使用一個單位。 指定的單位數目超過要複製的檔案數目時，將不會提高效能。

## <a name="use-adlcopy-to-copy-data-using-pattern-matching"></a>使用 AdlCopy 複製使用模式比對的資料
在本節中，您會了解如何使用 AdlCopy 將來源 (在我們使用 Azure 儲存體 Blob 的下列範例中) 的資料複製到使用模式比對的目的地 Data Lake Storage Gen1 帳戶。 例如，您可以使用下列步驟將所有副檔名為 .csv 的檔案從來源 Blob 複製到目的地。

1. 開啟命令提示字元，並瀏覽至安裝 AdlCopy 的目錄，通常是 `%HOMEPATH%\Documents\adlcopy`。
2. 執行下列命令，將所有副檔名為 *.csv 的檔案，從來源容器的特定 Blob 複製到 Data Lake Storage Gen1 資料夾：

        AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Pattern *.csv

    例如︰

        AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/FoodInspectionData/ /dest adl://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Pattern *.csv

## <a name="billing"></a>計費
* 如果您獨立使用 AdlCopy 工具，而且如果來源 Azure 儲存體帳戶與 Data Lake Storage Gen1 位於不同區域，則您將需支付移動資料的輸出成本。
* 如果您將 AdlCopy 工具與 Data Lake Analytics 帳戶搭配使用，即會套用標準的 [Data Lake Analytics 計費費率](https://azure.microsoft.com/pricing/details/data-lake-analytics/) 。

## <a name="considerations-for-using-adlcopy"></a>使用 AdlCopy 的考量
* AdlCopy (適用於 1.0.5 版) 支援從共同擁有超過數千個檔案和資料夾的來源複製資料。 但是，如果您在複製大型資料集時發生問題，可將檔案/資料夾分散到不同的子資料夾，並改用這些子資料夾的路徑做為來源。

## <a name="performance-considerations-for-using-adlcopy"></a>使用 AdlCopy 時的效能考量

AdlCopy 支援複製包含數千個檔案和資料夾的資料。 不過，如果您在複製大型資料集時遇到問題，您可以將檔案/資料夾分散至較小的子資料夾。 AdlCopy 適用於臨時複製。 如果您嘗試反覆地複製資料，請考慮使用 [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md)，它能夠完整地管理複製作業。

## <a name="release-notes"></a>版本資訊
* 1.0.13 - 如果您在多個 adlcopy 命令中將資料複製到相同的 Data Lake Storage Gen1 帳戶，則您每次執行時已不需要重新輸入認證。 Adlcopy 現在會在多次執行之間快取該資訊。

## <a name="next-steps"></a>後續步驟
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
* [搭配 Data Lake Storage Gen1 使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [搭配 Data Lake Storage Gen1 使用 Azure HDInsight](data-lake-store-hdinsight-hadoop-use-portal.md)
