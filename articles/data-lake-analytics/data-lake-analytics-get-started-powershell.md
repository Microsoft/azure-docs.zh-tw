---
title: 使用 Azure PowerShell 開始使用 Azure Data Lake Analytics
description: 使用 Azure PowerShell 來建立 Azure Data Lake Analytics 帳戶和提交 U-SQL 作業。
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 8a4e901e-9656-4a60-90d0-d78ff2f00656
ms.topic: conceptual
ms.date: 05/04/2017
ms.openlocfilehash: 4b4fa05164db402122efc745302dffe8c130b97b
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048063"
---
# <a name="get-started-with-azure-data-lake-analytics-using-azure-powershell"></a>使用 Azure PowerShell 開始使用 Azure Data Lake Analytics
[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

了解如何使用 Azure PowerShell 建立 Azure Data Lake Analytics 帳戶，然後提交和執行 U-SQL 作業。 如需有關 Data Lake Analytics 的詳細資訊，請參閱 [Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)。

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前，您必須具備下列資訊：

* **Azure Data Lake Analytics 帳戶**。 請參閱[開始使用 Data Lake Analytics](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-get-started-portal)。
* **具有 Azure PowerShell 的工作站**。 請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

## <a name="log-in-to-azure"></a>登入 Azure

本教學課程假設您已熟悉如何使用 Azure PowerShell。 特別是，您需要了解如何登入 Azure。 如果您需要協助，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)。

若要使用訂用帳戶名稱登入：

```
Connect-AzureRmAccount -SubscriptionName "ContosoSubscription"
```

除了訂用帳戶名稱之外，您也可以使用訂用帳戶識別碼來登入：

```
Connect-AzureRmAccount -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

如果成功，這個命令的輸出看起來會類似下列文字：

```
Environment           : AzureCloud
Account               : joe@contoso.com
TenantId              : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionId        : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionName      : ContosoSubscription
CurrentStorageAccount :
```

## <a name="preparing-for-the-tutorial"></a>準備教學課程

本教學課程中的 PowerShell 程式碼片段會使用這些變數來儲存此資訊：

```
$rg = "<ResourceGroupName>"
$adls = "<DataLakeStoreAccountName>"
$adla = "<DataLakeAnalyticsAccountName>"
$location = "East US 2"
```

## <a name="get-information-about-a-data-lake-analytics-account"></a>取得 Data Lake Analytics 帳戶的相關資訊

```
Get-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla  
```

## <a name="submit-a-u-sql-job"></a>提交 U-SQL 作業

建立 PowerShell 變數，可保留 U-SQL 指令碼。

```
$script = @"
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();

"@
```

使用 `Submit-AdlJob` Cmdlet 和 `-Script` 參數提交指令碼文字。

```
$job = Submit-AdlJob -Account $adla -Name "My Job" �Script $script
```

或者，您也可以使用 `-ScriptPath` 參數提交指令檔：

```
$filename = "d:\test.usql"
$script | out-File $filename
$job = Submit-AdlJob -Account $adla -Name "My Job" �ScriptPath $filename
```

使用 `Get-AdlJob` 取得作業狀態。 

```
$job = Get-AdlJob -Account $adla -JobId $job.JobId
```

使用 `Wait-AdlJob` Cmdlet，而不是一再呼叫 Get-AdlJob 直到作業完成。

```
Wait-AdlJob -Account $adla -JobId $job.JobId
```

使用 `Export-AdlStoreItem` 下載輸出檔案。

```
Export-AdlStoreItem -Account $adls -Path "/data.csv" -Destination "C:\data.csv"
```

## <a name="see-also"></a>另請參閱
* 若要使用其他工具檢視同一個教學課程，請按一下頁面最上方的索引標籤選取器。
* 若要了解 U-SQL，請參閱 [開始使用 Azure Data Lake Analytics U-SQL 語言](data-lake-analytics-u-sql-get-started.md)。
* 針對管理工作，請參閱 [使用 Azure 入口網站管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-portal.md)。
