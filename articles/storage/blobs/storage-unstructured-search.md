---
title: 搜尋 Azure 雲端儲存體中的非結構化資料
description: 使用 Azure 搜尋服務來搜尋非結構化資料。
author: roygara
services: storage
ms.service: storage
ms.topic: tutorial
ms.date: 10/12/2017
ms.author: rogarana
ms.custom: mvc
ms.openlocfilehash: 902009d7807b1ce340000c271350af1c37231d77
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181187"
---
# <a name="tutorial-search-unstructured-data-in-cloud-storage"></a>教學課程：搜尋雲端儲存體中的非結構化資料

在本教學課程中，您會了解如何使用利用 Azure Blob 中所儲存資料的 [Azure 搜尋服務](../../search/search-what-is-azure-search.md)來搜尋非結構化資料。 非結構化資料是未依預先定義方式組織或沒有資料模型的資料。 範例是 .txt 檔案。

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立資源群組
> * 建立儲存體帳戶
> * 建立容器
> * 將資料上傳至容器
> * 透過入口網站建立搜尋服務
> * 使用搜尋服務來搜尋容器

## <a name="download-the-sample"></a>下載範例

目前已為您備妥範例資料集。 **下載 [clinical-trials.zip](https://github.com/Azure-Samples/storage-blob-integration-with-cdn-search-hdi/raw/master/clinical-trials.zip)**，並將它解壓縮至它本身的資料夾。

這個範例包含取自 [clinicaltrials.gov](https://clinicaltrials.gov/ct2/results) 的文字檔。 您可以使用它們作為範例文字檔，以使用 Azure 進行搜尋。

## <a name="log-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](http://portal.azure.com)。

## <a name="create-a-storage-account"></a>建立儲存體帳戶

儲存體帳戶提供唯一的位置來儲存和存取您的 Azure 儲存體資料物件。

目前，有兩種類型的儲存體帳戶：**Blob** 和**一般用途**。 在本教學課程中，您會建立**一般用途**儲存體帳戶。

如果您不熟悉建立一般用途儲存體帳戶的程序，則其建立方式如下：

1. 從左側的功能表中，選取 [儲存體帳戶]，然後選取 [新增]。

2. 輸入儲存體帳戶的唯一名稱。 

3. 選取 [Resource Manager] 作為 [部署模型]，然後從 [帳戶種類] 下拉式清單中選取 [一般用途]。

4. 從 [複寫] 下拉式清單中，選取 [本地備援儲存體 (LRS)]。

5. 在 [資源群組] 下方，選取 [新建]，然後輸入唯一的名稱。

6. 選取適當的訂用帳戶。

7. 選擇位置，然後選取 [建立]。

  ![非結構化搜尋](media/storage-unstructured-search/storagepanes2.png)

## <a name="create-a-container"></a>建立容器

容器類似資料夾，用來儲存 Blob。

在本教學課程中，您會使用單一容器來儲存取自 clinicaltrials.gov 的文字檔。

1. 在 Azure 入口網站中巡覽至您的儲存體帳戶。

2. 選取 [Blob 服務] 下方的 [瀏覽 Blob]。

3. 新增容器。

4. 將容器命名為 "data"，並選取 [容器] 作為公用存取層級。

5. 選取 [確定] 以建立容器。 

  ![非結構化搜尋](media/storage-unstructured-search/storageactinfo.png)

## <a name="upload-the-example-data"></a>上傳範例資料

您現在已經有容器，可以將範例資料上傳至其中。

1. 選取容器，然後選取 [上傳]。

2. 選取 [檔案] 欄位旁邊的藍色資料夾圖示，並瀏覽至您解壓縮範例資料所在的本機資料夾。

3. 選取所有解壓縮的檔案，然後選取 [開啟]

4. 選取 [上傳] 以開始上傳程序。

  ![非結構化搜尋](media/storage-unstructured-search/upload.png)

上傳程序可能需要一些時間。

完成之後，請巡覽回您的 data 容器，確認文字檔已上傳。

  ![非結構化搜尋](media/storage-unstructured-search/clinicalfolder.png)

## <a name="create-a-search-service"></a>建立搜尋服務

Azure 搜尋服務是搜尋即服務雲端解決方案，可為開發人員提供 API 和工具，以透過資料在 Web、行動和企業應用程式中增添豐富的搜尋體驗。

如果您不熟悉建立搜尋服務的程序，則其建立方式如下：

1. 在 Azure 入口網站中巡覽至您的儲存體帳戶。

2. 向下捲動，並按一下 [Blob 服務] 下方的 [新增 Azure 搜尋服務]。

3. 在 [匯入資料] 下方，選取 [挑選您的服務]。

4. 選取 [按一下這裡建立新搜尋服務]。

5. 在 [新增搜尋服務] 的 [URL] 欄位中，輸入您搜尋服務的唯一名稱。

6. 在 [資源群組] 下方，選取 [使用現有項目]，然後選擇您稍早建立的資源群組。

7. 在 [定價層] 中，選取 [免費] 層，然後按一下 [選取]。

8. 選取 [建立] 以建立搜尋服務。

  ![非結構化搜尋](media/storage-unstructured-search/createsearch2.png)

## <a name="connect-your-search-service-to-your-container"></a>將您的搜尋服務連接到您的容器

您現在已有搜尋服務，可以將它附加到您的 Blob 儲存體。 本節逐步解說完成選擇資料來源、建立索引以及建立索引子的程序。

1. 瀏覽至儲存體帳戶。

2. 選取 [Blob 服務] 下方的 [新增 Azure 搜尋服務]。

3. 選取 [匯入資料] 內的 [搜尋服務]，然後按一下您在上一節中建立的搜尋服務。 這會開啟 [新增資料來源]。

### <a name="new-data-source"></a>新增資料來源

  資料來源指定要編製索引的資料以及如何存取資料。 同一個搜尋服務可以多次使用資料來源。

1. 輸入資料來源的名稱。 在 [要擷取的資料] 下方，選取 [內容與中繼資料]。 資料來源指定要對 Blob 的哪些部分編製索引。
    
    a. 在您自己的未來案例中，也可以選取 [僅限儲存體中繼資料]。 如果您想要限制要編製索引為標準 Blob 屬性或使用者定義屬性的資料，則會進行該選擇。
    
    b. 您也可以選擇 [所有中繼資料] 以取得這兩個標準 Blob 屬性，以及「所有」內容類型特定中繼資料。 

2. 因為您正在使用的 Blob 是文字檔，所以請將 [剖析模式] 設為 [文字]。
    
    a. 在您自己的未來案例中，可能想要根據 Blob 內容選取[其他剖析模式](../../search/search-howto-indexing-azure-blob-storage.md)。

  ![非結構化搜尋](media/storage-unstructured-search/datasources.png)

3. 選取 [儲存體容器]，列出可用的儲存體帳戶。

4. 選取儲存體帳戶，然後選取您先前建立的容器。

5. 按一下 [選取] 返回 [新增資料來源]，然後選取 [確定] 繼續。

  ![非結構化搜尋](media/storage-unstructured-search/datacontainer.png)

### <a name="configure-the-index"></a>設定索引

  索引是可搜尋資料來源中的欄位集合。 索引是搜尋服務如何知道應該使用哪些方式搜尋資料。

1. 在 [匯入資料] 中，選取 [自訂目標索引]。
 
2. 在 [索引名稱] 欄位中，輸入索引名稱。

3. 選取 **metadata_storage_name** 下方之 [可擷取] 屬性的核取方塊。

  ![非結構化搜尋](media/storage-unstructured-search/valuestoselect.png)

4. 按一下 [確定]，以啟動 [建立索引子]。

索引的參數以及提供這些參數的屬性十分重要。 這些參數指定要儲存「什麼」資料，而屬性指定「如何」儲存該資料。

[欄位名稱] 資料行包含這些參數。 下表提供可用屬性和其描述的清單。

### <a name="field-attributes"></a>欄位屬性
| 屬性 | 說明 |
| --- | --- |
| *金鑰* |字串，提供每一份文件的唯一識別碼，用於查閱文件。 每個索引必須有一個索引鍵。 只有一個欄位可以做為索引鍵，而且其類型必須設定為 Edm.String。 |
| *Retrievable* |指定搜尋結果中是否可傳回某欄位。 |
| *Filterable* |允許欄位用於篩選查詢。 |
| *Sortable* |允許查詢使用此欄位排序搜尋結果。 |
| *Facetable* |允許欄位用於使用者自我引導篩選的多面向導覽結構中。 通常，欄位若包含您可以用來將多份文件群組在一起的重複值 (例如，落在單一品牌或服務類別目錄下的多份文件)，最適合做為 Facet。 |
| *Searchable* |將欄位標記為可供全文檢索。 |


### <a name="create-an-indexer"></a>建立索引子
    
  索引子會以搜尋索引連接資料來源，並提供排程以對資料進行編製索引。

1. 在 [名稱] 欄位中輸入名稱，然後選取 [確定]。

  ![非結構化搜尋](media/storage-unstructured-search/exindexer.png)

2. 您會返回 [匯入資料]，並選取 [確定] 完成連線程序。

您現在已成功將 Blob 連接至您的搜尋服務。 入口網站需要幾分鐘的時間，才會顯示已填入索引。 不過，搜尋服務會立即開始編製索引，讓您可以立即開始搜尋。

## <a name="search-your-text-files"></a>搜尋文字檔

若要搜尋您的檔案，請在您新建之搜尋服務的索引內開啟搜尋總管。

下列步驟示範在何處尋找搜尋總管，並提供一些範例查詢：

1. 覽至所有資源，並尋找您新建的搜尋服務。

  ![非結構化搜尋](media/storage-unstructured-search/exampleurl.png)

3. 選取索引，以將它開啟。 

  ![非結構化搜尋](media/storage-unstructured-search/overview.png)

4. 選取 [搜尋總管] 以開啟搜尋總管，您可以在其中對資料進行即時查詢。

  ![非結構化搜尋](media/storage-unstructured-search/indexespane.png)

5. 查詢字串欄位為空時，請選取 [搜尋]。 空的查詢會傳回 Blob 中的「所有」資料。

  ![非結構化搜尋](media/storage-unstructured-search/emptySearch.png)

### <a name="full-text-search"></a>全文檢索搜尋 

在 [查詢字串] 欄位中，輸入 "Myopia"，然後選取 [搜尋]。 起始搜尋檔案內容，並傳回其子集，其中包含"Myopia" 這個單字。

  ![非結構化搜尋](media/storage-unstructured-search/secondMyopia.png) 

### <a name="system-properties-search"></a>系統屬性搜尋

您也可以建立查詢，以使用 `$select` 參數依系統屬性搜尋。 在查詢字串中輸入 `$select=metadata_storage_name`，並按 Enter ，只傳回該特定欄位。
    
查詢字串將會直接修改 URL，因此不允許有空格。 若要搜尋多個欄位，請使用逗號，例如：`$select=metadata_storage_name,metadata_storage_path`
    
定義索引時，`$select` 參數只能與標示為可擷取的欄位搭配使用。

  ![非結構化搜尋](media/storage-unstructured-search/metadatasearchunstructured.png) 

您現在已完成本教學課程，並且具有一組可搜尋的未結構化資料。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Azure 搜尋服務來搜尋非結構化資料，例如，如何：

> [!div class="checklist"]
> * 建立資源群組
> * 建立儲存體帳戶
> * 建立容器
> * 將資料上傳至容器
> * 建立搜尋服務
> * 使用搜尋服務來搜尋容器

遵循此連結，以深入了解如何使用 Azure 搜尋服務對文件編製索引。

> [!div class="nextstepaction"]
> [使用 Azure 搜尋服務在 Azure Blob 儲存體中對文件編製索引](../../search/search-howto-indexing-azure-blob-storage.md)