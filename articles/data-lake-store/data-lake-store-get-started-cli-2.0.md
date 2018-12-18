---
title: 使用 Azure CLI 來開始使用 Azure Data Lake Storage Gen1 | Microsoft Docs
description: 使用 Azure CLI 建立 Data Lake Storage Gen1 帳戶並執行基本作業
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: nitinme
ms.openlocfilehash: d8232b1e29a3d2585e79cf56d0f180a5084fd13e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979061"
---
# <a name="get-started-with-azure-data-lake-store-using-azure-cli"></a>使用 Azure CLI 開始使用 Azure Data Lake Store
> [!div class="op_single_selector"]
> * [入口網站](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [Azure CLI](data-lake-store-get-started-cli-2.0.md)
>
> 

了解如何使用 Azure CLI 建立 Azure Data Lake Storage Gen1 帳戶並執行基本作業，例如建立資料夾、上傳及下載資料檔案、刪除您的帳戶等等。如需有關 Data Lake Storage Gen1 的詳細資訊，請參閱 [Data Lake Storage Gen1 概觀](data-lake-store-overview.md)。

Azure CLI 是管理 Azure 資源的 Azure 命令列體驗。 它可以用於 macOS、Linux 和 Windows。 如需詳細資訊，請參閱 [Azure CLI 概觀](https://docs.microsoft.com/cli/azure)。 您也可以查看 [Azure Data Lake Storage Gen1 CLI 參考](https://docs.microsoft.com/cli/azure/dls)以取得命令和語法的完整清單。


## <a name="prerequisites"></a>必要條件
開始閱讀本文之前，您必須符合下列必要條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* **Azure CLI** - 如需相關指示，請參閱[安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。

## <a name="authentication"></a>驗證

此文章使用較簡單的驗證方法搭配 Data Lake Storage Gen1，其中您會以使用者身分登入。 Data Lake Storage Gen1 帳戶和檔案系統的存取層級則由已登入使用者的存取層級所控管。 不過，還有其他方法可向 Data Lake Storage Gen1 進行驗證：**使用者驗證**或**服務對服務驗證**。 如需有關如何驗證的指示和詳細資訊，請參閱[使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)或[服務對服務驗證](data-lake-store-authenticate-using-active-directory.md)。


## <a name="log-in-to-your-azure-subscription"></a>登入您的 Azure 訂用帳戶

1. 登入您的 Azure 訂用帳戶。

    ```azurecli
    az login
    ```

    您會在下一個步驟中取得要使用的程式碼。 請使用網頁瀏覽器開啟頁面 https://aka.ms/devicelogin 並輸入代碼進行驗證。 系統會提示您使用您的認證登入。

2. 一旦您登入後，視窗會列出與您帳戶相關聯的所有 Azure 訂用帳戶。 使用下列命令可使用特定訂用帳戶。
   
    ```azurecli
    az account set --subscription <subscription id> 
    ```

## <a name="create-an-azure-data-lake-storage-gen1-account"></a>建立 Azure Data Lake Storage Gen1 帳戶

1. 建立新的資源群組。 在下列命令中，提供您想要使用的參數值。 如果位置名稱包含空格，請將它放在引號中。 例如 "East US 2"。 
   
    ```azurecli
    az group create --location "East US 2" --name myresourcegroup
    ```

2. 建立 Data Lake Storage Gen1 帳戶。
   
    ```azurecli
    az dls account create --account mydatalakestoragegen1 --resource-group myresourcegroup
    ```

## <a name="create-folders-in-a-data-lake-storage-gen1-account"></a>在 Data Lake Storage Gen1 帳戶中建立資料夾

您可以在您的 Azure Data Lake Storage Gen1 帳戶下建立資料夾，以管理及存放資料。 使用下列命令在 Data Lake Storage Gen1 帳戶的根目錄建立名為 **mynewfolder** 的資料夾。

```azurecli
az dls fs create --account mydatalakestoragegen1 --path /mynewfolder --folder
```

> [!NOTE]
> `--folder` 參數可確保命令會建立一個資料夾。 如果這個參數不存在，命令會在 Data Lake Storage Gen1 帳戶的根目錄中建立稱為 mynewfolder 的空白檔案。
> 
>

## <a name="upload-data-to-a-data-lake-storage-gen1-account"></a>將資料上傳到 Data Lake Storage Gen1 帳戶

您可以將資料直接上傳到 Data Lake Storage Gen1 的根目錄層級，或上傳到您在該帳戶內建立的資料夾。 下列程式碼片段示範如何將一些範例資料上傳至您在上一節中建立的資料夾 (**mynewfolder**)。

如果您正在尋找一些可上傳的範例資料，您可以從 **Azure Data Lake Git 存放庫** 取得 [Ambulance Data](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData)資料夾。 下載檔案並將它儲存在電腦的本機目錄上，例如 C:\sampledata\。

```azurecli
az dls fs upload --account mydatalakestoragegen1 --source-path "C:\SampleData\AmbulanceData\vehicle1_09142014.csv" --destination-path "/mynewfolder/vehicle1_09142014.csv"
```

> [!NOTE]
> 針對目的地，您必須指定包含檔案名稱的完整路徑。
> 
>


## <a name="list-files-in-a-data-lake-storage-gen1-account"></a>列出 Data Lake Storage Gen1 帳戶中的檔案

使用下列命令列出 Data Lake Storage Gen1 帳戶中的檔案。

```azurecli
az dls fs list --account mydatalakestoragegen1 --path /mynewfolder
```

此命令的輸出應類似這樣：

    [
        {
            "accessTime": 1491323529542,
            "aclBit": false,
            "blockSize": 268435456,
            "group": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
            "length": 1589881,
            "modificationTime": 1491323531638,
            "msExpirationTime": 0,
            "name": "mynewfolder/vehicle1_09142014.csv",
            "owner": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
            "pathSuffix": "vehicle1_09142014.csv",
            "permission": "770",
            "replication": 1,
            "type": "FILE"
        }
    ]

## <a name="rename-download-and-delete-data-from-a-data-lake-storage-gen1-account"></a>重新命名、下載及刪除 Data Lake Storage Gen1 帳戶中的資料 

* **若要重新命名檔案**，請使用下列命令：
  
    ```azurecli
    az dls fs move --account mydatalakestoragegen1 --source-path /mynewfolder/vehicle1_09142014.csv --destination-path /mynewfolder/vehicle1_09142014_copy.csv
    ```

* **若要下載檔案**，請使用下列命令。 請確定您指定的目的地路徑已存在。
  
    ```azurecli     
    az dls fs download --account mydatalakestoragegen1 --source-path /mynewfolder/vehicle1_09142014_copy.csv --destination-path "C:\mysampledata\vehicle1_09142014_copy.csv"
    ```

    > [!NOTE]
    > 如果不存在，命令就會建立目的資料夾。
    > 
    >

* **若要刪除檔案**，請使用下列命令：
  
    ```azurecli
    az dls fs delete --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014_copy.csv
    ```

    如果您想要在單一命令中一起刪除 **mynewfolder** 資料夾和 **vehicle1_09142014_copy.csv** 檔案，請使用 --recurse 參數

    ```azurecli
    az dls fs delete --account mydatalakestoragegen1 --path /mynewfolder --recurse
    ```

## <a name="work-with-permissions-and-acls-for-a-data-lake-storage-gen1-account"></a>處理 Data Lake Storage Gen1 帳戶的權限和 ACL

在此節中，您將了解如何管理 ACL 和使用 Azure CLI 的權限。 如需 ACL 如何在 Azure Data Lake Storage Gen1 中實作的詳細討論，請參閱 [Azure Data Lake Storage Gen1 中的存取控制](data-lake-store-access-control.md)。

* **若要更新檔案/資料夾的擁有者**，請使用下列命令︰

    ```azurecli
    az dls fs access set-owner --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014.csv --group 80a3ed5f-959e-4696-ba3c-d3c8b2db6766 --owner 6361e05d-c381-4275-a932-5535806bb323
    ```

* **若要更新檔案/資料夾的權限**，請使用下列命令︰

    ```azurecli
    az dls fs access set-permission --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014.csv --permission 777
    ```
    
* **若要取得指定路徑中的 ACL**，請使用下列命令︰

    ```azurecli
    az dls fs access show --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014.csv
    ```

    輸出應該類似如下範例：

        {
            "entries": [
            "user::rwx",
            "group::rwx",
            "other::---"
          ],
          "group": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
          "owner": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
          "permission": "770",
          "stickyBit": false
        }

* **若要設定 ACL 的項目**，請使用下列命令︰

    ```azurecli
    az dls fs access set-entry --account mydatalakestoragegen1 --path /mynewfolder --acl-spec user:6360e05d-c381-4275-a932-5535806bb323:-w-
    ```

* **若要移除 ACL 的項目**，請使用下列命令︰

    ```azurecli
    az dls fs access remove-entry --account mydatalakestoragegen1 --path /mynewfolder --acl-spec user:6360e05d-c381-4275-a932-5535806bb323
    ```

* **若要移除整個預設的 ACL**，請使用下列命令︰

    ```azurecli
    az dls fs access remove-all --account mydatalakestoragegen1 --path /mynewfolder --default-acl
    ```

* **若要移除整個非預設的 ACL**，請使用下列命令︰

    ```azurecli
    az dls fs access remove-all --account mydatalakestoragegen1 --path /mynewfolder
    ```
    
## <a name="delete-a-data-lake-storage-gen1-account"></a>刪除 Data Lake Storage Gen1 帳戶
使用下列命令刪除您的 Data Lake Storage Gen1 帳戶。

```azurecli
az dls account delete --account mydatalakestoragegen1
```

出現提示時，請輸入 **Y** 刪除帳戶。

## <a name="next-steps"></a>後續步驟
* [使用 Azure Data Lake Storage Gen1 處理巨量資料需求](data-lake-store-data-scenarios.md) 
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
* [搭配 Data Lake Storage Gen1 使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [搭配 Data Lake Storage Gen1 使用 Azure HDInsight](data-lake-store-hdinsight-hadoop-use-portal.md)
