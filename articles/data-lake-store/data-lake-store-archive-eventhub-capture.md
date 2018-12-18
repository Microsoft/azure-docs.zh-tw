---
title: 從事件中樞擷取資料並放入 Azure Data Lake Storage Gen1 | Microsoft Docs
description: 使用 Azure Data Lake Storage Gen1 從事件中樞擷取資料
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 0bb870b54099fce9f7f6cfd1666be1b6393c5d07
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391431"
---
# <a name="use-azure-data-lake-storage-gen1-to-capture-data-from-event-hubs"></a>使用 Azure Data Lake Storage Gen1 從事件中樞擷取資料

了解如何使用 Azure Data Lake Storage Gen1 來擷取 Azure 事件中樞所收到的資料。

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* **Azure Data Lake Storage Gen1 帳戶**。 如需有關如何建立帳戶的指示，請參閱[開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)。

*  **事件中樞命名空間**。 如需相關指示，請參閱[建立事件中樞命名空間](../event-hubs/event-hubs-create.md#create-an-event-hubs-namespace)。 請確定 Data Lake Storage Gen1 帳戶和事件中樞命名空間位於相同的 Azure 訂用帳戶中。


## <a name="assign-permissions-to-event-hubs"></a>將權限指派給事件中樞

在本節中，您會在帳戶中建立一個資料夾，以便從事件中樞擷取資料並放入其中。 您也會將權限指派給事件中樞，以便它將資料寫入至 Data Lake Storage Gen1 帳戶。 

1. 開啟您要從事件中樞擷取資料並放入其中的 Data Lake Storage Gen1 帳戶，然後按一下**資料總管**。

    ![Data Lake Storage Gen1 資料總管](./media/data-lake-store-archive-eventhub-capture/data-lake-store-open-data-explorer.png "Lake Storage Gen1 資料總管")

1.  按一下 [新增資料夾]，然後輸入您要擷取資料的目的地資料夾名稱。

    ![在 Data Lake Storage Gen1 中建立新的資料夾](./media/data-lake-store-archive-eventhub-capture/data-lake-store-create-new-folder.png "在 Data Lake Storage Gen1 中建立新的資料夾")

1. 在 Data Lake Storage Gen1 根目錄指派權限。 

    a. 按一下**資料總管**，選取 Data Lake Storage Gen1 帳戶的根目錄，然後按一下 [存取]。

    ![為 Data Lake Storage Gen1 根目錄指派權限](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-permissions-to-root.png "為 Data Lake Storage Gen1 根目錄指派權限")

    b. 在 [存取] 下，依序按一下 [新增] 和 [選取使用者或群組]，然後搜尋 `Microsoft.EventHubs`。 

    ![為 Data Lake Storage Gen1 根目錄指派權限](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp.png "為 Data Lake Storage Gen1 根目錄指派權限")
    
    按一下 [選取] 。

    c. 在 [指派權限] 下，按一下 [選取權限]。 將 [權限] 設定為 [執行]。 將 [新增至] 設定為 [此資料夾及所有子系]。 將 [新增為] 設定為 [存取權限項目及預設權限項目]。

    > [!IMPORTANT]
    > 建立新的資料夾階層 (用來擷取 Azure 事件中樞所接收的資料) 時，這個簡單的方法可確保您能夠存取目的地資料夾。  不過，如果最上層資料夾有許多子檔案和資料夾，為其所有子系新增權限可能需要很長的時間。  如果根資料夾包含大量的檔案和資料夾，為最終目的地資料夾的路徑中的每個資料夾個別新增對 `Microsoft.EventHubs` 的 **Execute** 權限，可能更快。 

    ![為 Data Lake Storage Gen1 根目錄指派權限](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp1.png "為 Data Lake Storage Gen1 根目錄指派權限")

    按一下 [確定]。

1. 為 Data Lake Storage Gen1 帳戶下要擷取資料的目的地資料夾指派權限。

    a. 按一下**資料總管**，選取 Data Lake Storage Gen1 帳戶中的資料夾，然後按一下 [存取]。

    ![為 Data Lake Storage Gen1 資料夾指派權限](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-permissions-to-folder.png "為 Data Lake Storage Gen1 資料夾指派權限")

    b. 在 [存取] 下，依序按一下 [新增] 和 [選取使用者或群組]，然後搜尋 `Microsoft.EventHubs`。 

    ![為 Data Lake Storage Gen1 資料夾指派權限](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp.png "為 Data Lake Storage Gen1 資料夾指派權限")
    
    按一下 [選取] 。

    c. 在 [指派權限] 下，按一下 [選取權限]。 將 [權限] 設定為 [讀取]、[寫入] 和 [執行]。 將 [新增至] 設定為 [此資料夾及所有子系]。 最後，將 [新增為] 設定為 [存取權限項目及預設權限項目]。

    ![為 Data Lake Storage Gen1 資料夾指派權限](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp-folder.png "為 Data Lake Storage Gen1 資料夾指派權限")
    
    按一下 [確定]。 

## <a name="configure-event-hubs-to-capture-data-to-data-lake-storage-gen1"></a>設定事件中樞，以將資料擷取到 Data Lake Storage Gen1

在本節中，您會在事件中樞命名空間內建立事件中樞。 您也會設定事件中樞，以將資料擷取到 Azure Data Lake Storage Gen1 帳戶。 本節假設您已建立事件中樞命名空間。

1. 從事件中樞命名空間的 [概觀] 窗格，按一下 [+ 事件中樞]。

    ![建立事件中樞](./media/data-lake-store-archive-eventhub-capture/data-lake-store-create-event-hub.png "建立事件中樞")

1. 提供下列值來設定事件中樞，以將資料擷取到 Data Lake Storage Gen1。

    ![建立事件中樞](./media/data-lake-store-archive-eventhub-capture/data-lake-store-configure-eventhub.png "建立事件中樞")

    a. 提供事件中樞的名稱。
    
    b. 針對此教學課程，將 [分割區計數] 和 [訊息保留期] 設定為預設值。
    
    c. 將 [擷取] 設定為 [開啟]。 設定 [時間範圍] \(擷取頻率) 和 [大小範圍] \(擷取的資料大小)。 
    
    d. 針對 [Capture Provider] \(擷取提供者\)，選取 [Azure Data Lake Store]，然後選取您稍早建立的 Data Lake Storage Gen1。 針對 [Data Lake Path] \(Data Lake 路徑\)，輸入您在 Data Lake Storage Gen1 帳戶中建立的資料夾名稱。 您只需要提供資料夾的相對路徑。

    e. 將 [擷取檔案名稱格式範例] 保留為預設值。 此選項會掌管在擷取資料夾下建立的資料夾結構。

    f. 按一下頁面底部的 [新增] 。

## <a name="test-the-setup"></a>測試設定

您現在可以將資料傳送至 Azure 事件中樞來測試解決方案。 請遵循[將事件傳送至 Azure 事件中樞](../event-hubs/event-hubs-dotnet-framework-getstarted-send.md)的指示。 一旦您開始傳送資料，您就會看到 Data Lake Storage Gen1 中反映的資料使用您指定的資料夾結構。 例如，您會在 Data Lake Storage Gen1 中看到資料夾結構，如下列螢幕擷取畫面所示。

![Data Lake Storage Gen1 中的事件中樞資料範例](./media/data-lake-store-archive-eventhub-capture/data-lake-store-eventhub-data-sample.png "Data Lake Storage Gen1 中的事件中樞資料範例")

> [!NOTE]
> 即使您沒有傳入事件中樞的訊息，事件中樞也會將只有標頭的空檔案寫入至 Data Lake Storage Gen1 帳戶。 這些檔案會依您在建立事件中樞時所提供的相同時間間隔來寫入。
> 
>

## <a name="analyze-data-in-data-lake-storage-gen1"></a>分析 Data Lake Storage Gen1 中的資料

一旦資料位於 Data Lake Storage Gen1，您就可以執行分析作業來處理資料並進行大量運算。 請參閱 [USQL Avro 範例](https://github.com/Azure/usql/tree/master/Examples/AvroExamples)，以了解如何使用 Azure Data Lake Analytics 來執行此動作。
  

## <a name="see-also"></a>另請參閱
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
* [將資料從 Azure 儲存體 Blob 複製到 Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
