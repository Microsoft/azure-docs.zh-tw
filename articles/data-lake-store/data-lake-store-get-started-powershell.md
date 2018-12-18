---
title: 透過 PowerShell 開始使用 Azure Data Lake Storage Gen1 | Microsoft Docs
description: 使用 Azure PowerShell 建立資料 Azure Data Lake Storage Gen1 帳戶，並執行基本作業
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: nitinme
ms.openlocfilehash: 7a465559bd4e46777f67121e9b3c7d2b0b8a0a22
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986331"
---
# <a name="get-started-with-azure-data-lake-storage-gen1-using-azure-powershell"></a>使用 Azure PowerShell 開始使用 Azure Data Lake Storage Gen1
> [!div class="op_single_selector"]
> * [入口網站](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [Azure CLI](data-lake-store-get-started-cli-2.0.md)
>
>

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

了解如何使用 Azure PowerShell 建立 Azure Data Lake Storage Gen1 帳戶並執行基本作業，例如建立資料夾、上傳和下載資料檔案、刪除您的帳戶等等。如需有關 Data Lake Storage Gen1 的詳細資訊，請參閱 [Data Lake Storage Gen1 概觀](data-lake-store-overview.md)。

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure PowerShell 1.0 或更新版本**。 請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

## <a name="authentication"></a>驗證
本文搭配使用較簡單的驗證方法與 Data Lake Storage Gen1，系統會提示您輸入 Azure 帳號認證。 Data Lake Storage Gen1 帳戶和檔案系統的存取層級則由已登入使用者的存取層級所控管。 不過，還有其他方法可向 Data Lake Storage Gen1 進行驗證：**使用者驗證**或**服務對服務驗證**。 如需有關如何驗證的指示和詳細資訊，請參閱[使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)或[服務對服務驗證](data-lake-store-authenticate-using-active-directory.md)。

## <a name="create-a-data-lake-storage-gen1-account"></a>建立 Data Lake Storage Gen1 帳戶
1. 從桌面開啟新的 Windows PowerShell 視窗。 輸入下列程式碼片段登入 Azure 帳戶、設定訂用帳戶，然後註冊 Data Lake Storage Gen1 提供者。 系統提示您登入時，請確定您會使用其中一個訂用帳戶管理員/擁有者身分登入：

        # Log in to your Azure account
        Connect-AzureRmAccount

        # List all the subscriptions associated to your account
        Get-AzureRmSubscription

        # Select a subscription
        Set-AzureRmContext -SubscriptionId <subscription ID>

        # Register for Azure Data Lake Storage Gen1
        Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
2. Data Lake Storage Gen1 帳戶與 Azure 資源群組相關聯。 從建立 Azure 資源群組開始。

        $resourceGroupName = "<your new resource group name>"
        New-AzureRmResourceGroup -Name $resourceGroupName -Location "East US 2"

    ![建立 Azure 資源群組](./media/data-lake-store-get-started-powershell/ADL.PS.CreateResourceGroup.png "建立 Azure 資源群組")
3. 建立 Data Lake Storage Gen1 帳戶。 您指定的名稱必須只包含小寫字母和數字。

        $dataLakeStorageGen1Name = "<your new Data Lake Storage Gen1 account name>"
        New-AzureRmDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStorageGen1Name -Location "East US 2"

    ![建立 Data Lake Storage Gen1 帳戶](./media/data-lake-store-get-started-powershell/ADL.PS.CreateADLAcc.png "建立 Data Lake Storage Gen1 帳戶")
4. 確認已成功建立帳戶。

        Test-AzureRmDataLakeStoreAccount -Name $dataLakeStorageGen1Name

    此 Cmdlet 的輸出應為 **True**。

## <a name="create-directory-structures-in-your-data-lake-storage-gen1-account"></a>在您的 Data Lake Storage Gen1 帳戶中建立目錄結構
您可以在您的 Data Lake Storage Gen1 帳戶下建立用於管理與儲存資料的目錄。

1. 指定根目錄。

        $myrootdir = "/"
2. 在指定的根目錄下建立名為 **mynewdirectory** 的新目錄。

        New-AzureRmDataLakeStoreItem -Folder -AccountName $dataLakeStorageGen1Name -Path $myrootdir/mynewdirectory
3. 確認已成功建立新目錄。

        Get-AzureRmDataLakeStoreChildItem -AccountName $dataLakeStorageGen1Name -Path $myrootdir

    輸出應如下列螢幕擷取畫面所示：

    ![確認目錄](./media/data-lake-store-get-started-powershell/ADL.PS.Verify.Dir.Creation.png "確認目錄")

## <a name="upload-data-to-your-data-lake-storage-gen1-account"></a>將資料上傳到您的 Data Lake Storage Gen1 帳戶
您可以在根層級直接將資料上傳至 Data Lake Storage Gen1，或上傳至您在帳戶內建立的目錄。 本節的程式碼範例說明如何將一些範例資料上傳至您在上一節中建立的目錄 (**mynewdirectory**)。

如果您正在尋找一些可上傳的範例資料，您可以從 **Azure Data Lake Git 存放庫** 取得 [Ambulance Data](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData)資料夾。 下載檔案並將它儲存在電腦的本機目錄上，例如 C:\sampledata\。

    Import-AzureRmDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Path "C:\sampledata\vehicle1_09142014.csv" -Destination $myrootdir\mynewdirectory\vehicle1_09142014.csv


## <a name="rename-download-and-delete-data-from-your-data-lake-storage-gen1-account"></a>重新命名、下載與刪除 Data Lake Storage Gen1 帳戶中的資料
若要重新命名檔案，請使用下列命令：

    Move-AzureRmDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Path $myrootdir\mynewdirectory\vehicle1_09142014.csv -Destination $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv

若要下載檔案，請使用下列命令：

    Export-AzureRmDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Path $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv -Destination "C:\sampledata\vehicle1_09142014_Copy.csv"

若要刪除檔案，請使用下列命令：

    Remove-AzureRmDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Paths $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv

出現提示時，請輸入 **Y** 刪除項目。 如果您要刪除多個檔案，可以提供所有的路徑並以逗號分隔。

    Remove-AzureRmDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Paths $myrootdir\mynewdirectory\vehicle1_09142014.csv, $myrootdir\mynewdirectoryvehicle1_09142014_Copy.csv

## <a name="delete-your-data-lake-storage-gen1-account"></a>刪除您的 Data Lake Storage Gen1 帳戶
使用下列命令刪除您的 Data Lake Storage Gen1 帳戶。

    Remove-AzureRmDataLakeStoreAccount -Name $dataLakeStorageGen1Name

出現提示時，請輸入 **Y** 刪除帳戶。

## <a name="next-steps"></a>後續步驟
* [使用 PowerShell 和 Azure Data Lake Storage Gen1 進行效能微調之方針](data-lake-store-performance-tuning-powershell.md)
* [使用 Azure Data Lake Storage Gen1 處理巨量資料需求](data-lake-store-data-scenarios.md) 
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
* [將 Azure Data Lake Analytics 與 Data Lake Storage Gen1 搭配使用](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [將 Azure HDInsight 與 Data Lake Storage Gen1 搭配使用](data-lake-store-hdinsight-hadoop-use-portal.md)
