---
title: 使用 PowerShell 和 Data Lake Store 進行效能微調之方針 | Microsoft Docs
description: 如何在使用 Azure PowerShell 搭配 Data Lake Store 時改善效能的秘訣
services: data-lake-store
documentationcenter: ''
author: stewu
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 01/09/2018
ms.author: stewu
ms.openlocfilehash: 7b19972ed4a75ac899a4b78b28ab36ba305a5a64
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34198645"
---
# <a name="performance-tuning-guidance-for-using-powershell-with-azure-data-lake-store"></a>使用 PowerShell 和 Azure Data Lake Store 進行效能微調之方針

本文列出可以微調的屬性，以在使用 PowerShell 搭配 Data Lake Store 運作時獲得最佳效能︰

## <a name="performance-related-properties"></a>效能相關的屬性

| 屬性            | 預設值 | 說明 |
|---------------------|---------|-------------|
| PerFileThreadCount  | 10      | 這個參數可讓您選擇可用於上傳或下載每個檔案的平行執行緒數目。 此數字代表每個檔案可以配置的執行緒數目上限，但視您的案例而定，您可能會收到比較少的執行緒 (例如︰如果您要上傳 1-KB 檔案，即使您要求 20 個執行緒，但您將取得一個執行緒)。  |
| ConcurrentFileCount | 10      | 這個參數特別用於上傳或下載資料夾。 這個參數會決定可以上傳或下載的並行檔案數目。 此數字代表可以一次上傳或下載的並行檔案數目上限，但視您的案例而定，可能會收到比較少的並行檔案 (例如︰如果您要上傳兩個檔案，即使您要求 15 個檔案，但您將取得兩個並行檔案)。 |

**範例**

此命令會使用每個檔案 20 個執行緒和 100 個並行檔案，將檔案從 Azure Data Lake Store 下載至使用者的本機磁碟機。

    Export-AzureRmDataLakeStoreItem -AccountName <Data Lake Store account name> -PerFileThreadCount 20-ConcurrentFileCount 100 -Path /Powershell/100GB/ -Destination C:\Performance\ -Force -Recurse

## <a name="how-do-i-determine-the-value-for-these-properties"></a>如何判斷這些屬性的值？

您的下一個問題可能是如何判斷要提供給效能相關屬性的值。 以下是一些您可以使用的指引。

* **步驟 1︰決定總執行緒計數** - 您應該從計算要使用的總執行緒計數著手。 一般來說，您應該針對每個實體核心使用六個執行緒。

        Total thread count = total physical cores * 6

    **範例**

    假設您正從有 16 個核心的 D14 VM 執行 PowerShell 命令

        Total thread count = 16 cores * 6 = 96 threads


* **步驟 2︰計算 PerFileThreadCount** - 我們會根據檔案的大小計算 PerFileThreadCount。 對於小於 2.5 GB 的檔案，不需要變更此參數，因為預設值為 10 就已足夠。 對於大於 2.5 GB 的檔案，您應該使用 10 個執行緒作為第一個 2.5 GB 的基底，並且為每增加額外 256-MB 的檔案大小新增 1 個執行緒。 如果您要複製包含各種檔案大小的資料夾，請考慮將其分組為相似的檔案大小。 檔案大小不相近可能會導致非最佳的效能。 如果不可能分組為相似的檔案大小，您應該根據最大檔案大小設定 PerFileThreadCount。

        PerFileThreadCount = 10 threads for the first 2.5 GB + 1 thread for each additional 256 MB increase in file size

    **範例**

    假設您有 100 個 1 GB 到 10 GB 的檔案，我們使用 10 GB 作為等式的最大檔案大小，該等式會如下所示。

        PerFileThreadCount = 10 + ((10 GB - 2.5 GB) / 256 MB) = 40 threads

* **步驟 3︰計算 ConcurrentFilecount** - 使用總執行緒計數和 PerFileThreadCount 來計算以下列等式作為基礎的 ConcurrentFileCount：

        Total thread count = PerFileThreadCount * ConcurrentFileCount

    **範例**

    以我們使用的範例值為基礎

        96 = 40 * ConcurrentFileCount

    因此，**ConcurrentFileCount** 是 **2.4**, ，我們可以四捨五入為 **2**。

## <a name="further-tuning"></a>進一步微調

您可能需要進一步微調，因為有各種檔案大小要處理。 如果所有或大部分檔案都比較大且比較接近 10-GB 的範圍，就很適合上述的計算。 相反地，如果有許多不同的檔案大小且很多檔案比較小，您可以減少 PerFileThreadCount。 藉由減少 PerFileThreadCount，我們即可增加 ConcurrentFileCount。 所以，如果假設 5-GB 範圍中的大部分檔案比較小，我們可以重新進行計算︰

    PerFileThreadCount = 10 + ((5 GB - 2.5 GB) / 256 MB) = 20

因此，**ConcurrentFileCount** 會變成 96/20，也就是 4.8，可四捨五入為 **4**。

您可以根據檔案大小的分佈情形，將 **PerFileThreadCount** 變大和變小，繼續微調這些設定。

### <a name="limitation"></a>限制

* **檔案數目小於 ConcurrentFileCount**︰如果您要上傳的檔案數目小於您計算的 **ConcurrentFileCount**，則應該減少 **ConcurrentFileCount** 使其等於檔案數目。 您可以使用任何剩餘的執行緒來增加 **PerFileThreadCount**。

* **執行緒太多**︰如果您增加太多執行緒計數，但未增加您的叢集大小，您會有效能降低的風險。 在 CPU 上進行環境切換時，可能會有爭用問題。

* **並行處理量不足**︰如果並行處理量不足，您的叢集可能太小。 您可以增加叢集中的節點數目，以提供更多的並行處理量。

* **節流錯誤**︰如果並行處理量太高，您可能會看到節流錯誤。 如果您看到節流錯誤，則應該減少並行處理量或與我們連絡。

## <a name="next-steps"></a>後續步驟
* [使用 Azure Data Lake Store 處理巨量資料需求](data-lake-store-data-scenarios.md) 
* [保護 Data Lake Store 中的資料](data-lake-store-secure-data.md)
* [搭配 Data Lake Store 使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [搭配資料湖存放區使用 Azure HDInsight](data-lake-store-hdinsight-hadoop-use-portal.md)

