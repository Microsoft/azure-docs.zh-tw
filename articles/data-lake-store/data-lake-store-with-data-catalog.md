---
title: 在 Azure 資料目錄中從 Azure Data Lake Storage Gen1 註冊資料 | Microsoft Docs
description: 在 Azure 資料目錄中從 Azure Data Lake Storage Gen1 註冊資料
services: data-lake-store,data-catalog
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: 3294d91e-a723-41b5-9eca-ace0ee408a4b
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 35fc7b2c713f8d4b88f4a44d9ddef5d92ba4c402
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46294308"
---
# <a name="register-data-from-azure-data-lake-storage-gen1-in-azure-data-catalog"></a>在 Azure 資料目錄中從 Azure Data Lake Storage Gen1 註冊資料
在此文章中，您將了解如何使用 Azure 資料目錄來整合 Azure Data Lake Storage Gen1，使您的資料與資料目錄整合，讓使用者可在組織中進行搜尋。 如需編目資料的詳細資訊，請參閱 [Azure 資料目錄](../data-catalog/data-catalog-what-is-data-catalog.md)。 若要了解您可以使用資料目錄的案例，請參閱 [Azure 資料目錄常見案例](../data-catalog/data-catalog-common-scenarios.md)。

## <a name="prerequisites"></a>先決條件
開始進行此教學課程之前，您必須具備下列條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* 針對 Data Lake Storage Gen1 **啟用您的 Azure 訂用帳戶**。 請參閱[指示](data-lake-store-get-started-portal.md)。
* **Data Lake Storage Gen1 帳戶**。 依照[使用 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md) 的指示執行。 在此教學課程中，會建立名為 **datacatalogstore** 的 Data Lake Storage Gen1 帳戶。

    一旦您建立帳戶之後，請將範例資料集上傳給它。 在此教學課程中，讓我們上傳 **Azure Data Lake Git 存放庫**中 [AmbulanceData](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData/) 資料夾下的所有 .csv 檔案。 您可以使用各種用戶端，例如：[Azure 儲存體總管](http://storageexplorer.com/)，將資料上傳至 Blob 容器。
* **Azure 資料目錄**。 您的組織必須為組織建立 Azure 資料目錄。 每個組織只允許有一個目錄。

## <a name="register-data-lake-storage-gen1-as-a-source-for-data-catalog"></a>註冊 Data Lake Storage Gen1 做為資料目錄的來源

> [!VIDEO https://channel9.msdn.com/Series/AzureDataLake/ADCwithADL/player]

1. 前往 `https://azure.microsoft.com/services/data-catalog`，然後按一下 [開始使用]。
1. 登入 Azure 資料目錄入口網站，再按一下 [發佈資料] 。

    ![註冊資料來源](./media/data-lake-store-with-data-catalog/register-data-source.png "註冊資料來源")
1. 在下一個頁面上，按一下 [啟動應用程式] 。 這樣會下載電腦上的應用程式資訊清單檔案。 按兩下此資訊清單檔案以啟動應用程式。
1. 在 [歡迎使用] 頁面上，按一下 [登入]，並輸入您的認證。

    ![歡迎使用畫面](./media/data-lake-store-with-data-catalog/welcome.screen.png "歡迎使用畫面")
1. 在 [選取資料來源] 頁面上，選取 [Azure Data Lake Store]，然後按一下 [下一步]。

    ![選取資料來源](./media/data-lake-store-with-data-catalog/select-source.png "選取資料來源")
1. 在下一個頁面上，提供您想要在資料目錄中註冊的 Data Lake Storage Gen1 帳戶名稱。 其他選項保留為預設值，然後按一下 [連線] 。

    ![連接到資料來源](./media/data-lake-store-with-data-catalog/connect-to-source.png "連接到資料來源")
1. 下一個頁面可以分成下列區段。

    a. [伺服器階層] 方塊代表 Data Lake Storage Gen1 帳戶資料夾結構。 [$Root] 代表 Data Lake Storage Gen1 帳戶根目錄，而 [AmbulanceData] 則代表在 Data Lake Storage Gen1 帳戶的根目錄中建立的資料夾。

    b. [可用物件] 方塊會列出 [AmbulanceData] 資料夾下的檔案和資料夾。

    c. [要註冊的物件] 方塊會列出您想要在「Azure 資料目錄」中註冊的檔案和資料夾。

    ![檢視資料結構](./media/data-lake-store-with-data-catalog/view-data-structure.png "檢視資料結構")
1. 在此教學課程中，您應該在目錄中註冊所有檔案。 因此，請按一下 (![移動物件](./media/data-lake-store-with-data-catalog/move-objects.png "移動物件")) 按鈕來將所有檔案移至 [要註冊的物件] 方塊。

    因為資料將會在整個組織的資料目錄中註冊，建議新增一些中繼資料，讓您稍後可以用來快速尋找資料。 例如，您可以新增資料擁有者 (例如，上傳資料的人員) 的電子郵件地址，或新增可識別資料的標籤。 下方螢幕擷取畫面顯示您新增至資料的標記。

    ![檢視資料結構](./media/data-lake-store-with-data-catalog/view-selected-data-structure.png "檢視資料結構")

    按一下 [註冊] 。
1. 下列螢幕擷取畫面表示已成功在資料目錄中註冊資料。

    ![註冊完成](./media/data-lake-store-with-data-catalog/registration-complete.png "檢視資料結構")
1. 按一下 [檢視入口網站]  以回到資料目錄入口網站，並確認您現在可以從入口網站存取已註冊的資料。 若要搜尋資料，您可以使用您在註冊資料時使用的標籤。

     ![搜尋目錄中的資料](./media/data-lake-store-with-data-catalog/search-data-in-catalog.png "搜尋目錄中的資料")
1. 您現在可以執行將註釋和文件新增至資料等作業。 如需詳細資訊，請參閱下列連結。

    * [註釋資料目錄中的資料來源](../data-catalog/data-catalog-how-to-annotate.md)
    * [記錄資料目錄中的資料來源](../data-catalog/data-catalog-how-to-documentation.md)

## <a name="see-also"></a>另請參閱
* [註釋資料目錄中的資料來源](../data-catalog/data-catalog-how-to-annotate.md)
* [記錄資料目錄中的資料來源](../data-catalog/data-catalog-how-to-documentation.md)
* [將 Data Lake Storage Gen1 與其他 Azure 服務整合](data-lake-store-integrate-with-other-services.md)
