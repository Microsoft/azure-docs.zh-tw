---
title: 使用 PowerShell 監視和管理 Azure 串流分析作業
description: 本文說明如何使用 Azure PowerShell 和 Cmdlet 來監視及管理 Azure 串流分析作業。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/28/2017
ms.openlocfilehash: 6812298bc90446a7b7e136ec81a67641c2a9e18f
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701603"
---
# <a name="monitor-and-manage-stream-analytics-jobs-with-azure-powershell-cmdlets"></a>透過 Azure PowerShell Cmdlet 監視和管理串流分析工作
了解如何透過執行基本串流分析工作的 Azure PowerShell Cmdlet 及 PowerShell 指令碼，來監視及管理串流分析的資源。

## <a name="prerequisites-for-running-azure-powershell-cmdlets-for-stream-analytics"></a>執行 Azure PowerShell Cmdlet 進行串流分析的必要條件
* 在您的訂用帳戶中建立「Azure 資源群組」。 下列是 PowerShell 指令碼範例。 如需 Azure PowerShell 資訊，請參閱 [安裝並設定 Azure PowerShell](/powershell/azure/overview)。  

Azure PowerShell 0.9.8：  

         # Log in to your Azure account
        Add-AzureAccount

        # Select the Azure subscription you want to use to create the resource group if you have more than one subscription on your account.
        Select-AzureSubscription -SubscriptionName <subscription name>

        # If Stream Analytics has not been registered to the subscription, remove remark symbol below (#) to run the Register-AzureProvider cmdlet to register the provider namespace.
        #Register-AzureProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'

        # Create an Azure resource group
        New-AzureResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>

Azure PowerShell 1.0：  

         # Log in to your Azure account
        Connect-AzureRmAccount

        # Select the Azure subscription you want to use to create the resource group.
        Get-AzureRmSubscription –SubscriptionName "your sub" | Select-AzureRmSubscription

        # If Stream Analytics has not been registered to the subscription, remove remark symbol below (#) to run the Register-AzureProvider cmdlet to register the provider namespace.
        #Register-AzureRmResourceProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'

        # Create an Azure resource group
        New-AzureRMResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>



> [!NOTE]
> 以程式控制方式建立的串流分析工作預設不會啟用監視功能。  您可以在 Azure 入口網站中瀏覽到該工作的 [監視] 頁面，然後按一下 [啟用] 按鈕來手動啟用監視，或是按照 [Azure 串流分析 - 以程式設計方式監視串流分析工作](stream-analytics-monitor-jobs.md)中的步驟進行。
> 
> 

## <a name="azure-powershell-cmdlets-for-stream-analytics"></a>用於串流分析的 Azure PowerShell Cmdlet
下表列出可用來監視和管理 Azure 串流分析工作的 Azure PowerShell Cmdlet。 請注意，Azure PowerShell 有不同的版本。 
**下列範例中所列出的第一個命令是針對 Azure PowerShell 0.9.8，第二個命令是針對 Azure PowerShell 1.0。** Azure PowerShell 1.0 命令永遠會在命令中使用 "AzureRM"。

### <a name="get-azurestreamanalyticsjob--get-azurermstreamanalyticsjob"></a>Get-AzureStreamAnalyticsJob | Get-AzureRMStreamAnalyticsJob
列出 Azure 訂用帳戶或指定的資源群組中定義的所有「串流分析」工作，或取得與資源群組內特定工作相關的工作資訊。

**範例 1**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsJob

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsJob

此 PowerShell 命令會傳回 Azure 訂用帳戶中所有「串流分析」工作的相關資訊。

**範例 2**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 

此 PowerShell 命令會傳回資源群組 StreamAnalytics-Default-Central-US 中所有串流分析工作的相關資訊。

**範例 3**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob

此 PowerShell 命令會傳回 StreamAnalytics-Default-Central-US 資源群組中 StreamingJob 串流分析工作的相關資訊。

### <a name="get-azurestreamanalyticsinput--get-azurermstreamanalyticsinput"></a>Get-AzureStreamAnalyticsInput | Get-AzureRMStreamAnalyticsInput
列出指定串流分析工作中定義的所有輸入，或取得特定輸入的相關資訊。

**範例 1**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

此 PowerShell 命令會傳回在 StreamingJob 工作中定義的所有輸入的相關資訊。

**範例 2**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name EntryStream

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name EntryStream

此 PowerShell 命令會傳回在 StreamingJob 工作中定義，名為 EntryStream 之輸入的相關資訊。

### <a name="get-azurestreamanalyticsoutput--get-azurermstreamanalyticsoutput"></a>Get-AzureStreamAnalyticsOutput | Get-AzureRMStreamAnalyticsOutput
列出指定串流分析工作中定義的所有輸出，或取得特定輸出的相關資訊。

**範例 1**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

此 PowerShell 命令會傳回在 StreamingJob 工作中定義之輸出的相關資訊。

**範例 2**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name Output

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name Output

此 PowerShell 命令會傳回在 StreamingJob 工作中定義，名為 Output 之輸出的相關資訊。

### <a name="get-azurestreamanalyticsquota--get-azurermstreamanalyticsquota"></a>Get-AzureStreamAnalyticsQuota | Get-AzureRMStreamAnalyticsQuota
取得指定區域之串流處理單位配額的相關資訊。

**範例 1**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsQuota –Location "Central US" 

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsQuota –Location "Central US" 

此 PowerShell 命令會傳回美國中部區域之串流單位的配額和使用量的相關資訊。

### <a name="get-azurestreamanalyticstransformation--getazurermstreamanalyticstransformation"></a>Get-AzureStreamAnalyticsTransformation | GetAzureRMStreamAnalyticsTransformation
取得在串流分析工作中定義之特定轉換的相關資訊。

**範例 1**

Azure PowerShell 0.9.8：  

    Get-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name StreamingJob

Azure PowerShell 1.0：  

    Get-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name StreamingJob

此 PowerShell 命令會傳回 StreamingJob 工作中名為 StreamingJob 之轉換的相關資訊。

### <a name="new-azurestreamanalyticsinput--new-azurermstreamanalyticsinput"></a>New-AzureStreamAnalyticsInput | New-AzureRMStreamAnalyticsInput
在串流分析工作內建立新的輸入，或更新現有的指定輸入。

您可以在 .json 檔案中或命令列上指定輸入的名稱。 若兩者皆指定，命令列上的名稱必須與檔案中的名稱相同。

如果您指定已存在的輸入，而且沒有指定 –Force 參數，Cmdlet 會詢問是否要取代現有的輸入。

如果您指定 –Force 參數，並指定現有的輸入名稱，則不經過確認就會取代輸入。

如需 JSON 檔案結構與內容的詳細資訊，請參閱[串流分析管理 REST API 參考程式庫][stream.analytics.rest.api.reference]的[建立輸入 (Azure 串流分析)][msdn-rest-api-create-stream-analytics-input] 一節。

**範例 1**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" 

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" 

此 PowerShell 命令會透過 Input.json 檔案建立新的輸入。 如果輸入定義檔中已經定義指定名稱的現有輸入，Cmdlet 會詢問是否要取代它。

**範例 2**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream

此 PowerShell 命令會在名為 EntryStream 的工作中建立新的輸入。 如果已經定義此名稱的現有輸入，Cmdlet 會詢問是否要取代它。

**範例 3**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream -Force

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream -Force

此 PowerShell 命令會使用檔案的定義取代名為 EntryStream 的現有輸入來源的定義。

### <a name="new-azurestreamanalyticsjob--new-azurermstreamanalyticsjob"></a>New-AzureStreamAnalyticsJob | New-AzureRMStreamAnalyticsJob
在 Microsoft Azure 中建立新的串流分析工作，或更新現有指定工作的定義。

您可以在 .json 檔案中或命令列上指定工作的名稱。 若兩者皆指定，命令列上的名稱必須與檔案中的名稱相同。

如果您指定已存在的工作名稱，而且沒有指定 –Force 參數，Cmdlet 會詢問是否要取代現有的工作。

如果您指定 –Force 參數，並指定現有的工作名稱，則不經過確認就會取代工作定義。

如需 JSON 檔案結構與內容的詳細資訊，請參閱[串流分析管理 REST API 參考程式庫][stream.analytics.rest.api.reference]的[建立串流分析作業][msdn-rest-api-create-stream-analytics-job]一節。

**範例 1**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" 

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" 

此 PowerShell 命令會透過 JobDefinition.json 中的定義建立新的工作。 如果輸出定義檔中已經定義了指定名稱的現有輸出，Cmdlet 會詢問是否要取代它。

**範例 2**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" –Name StreamingJob -Force

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" –Name StreamingJob -Force

此 PowerShell 命令會取代 StreamingJob 的工作定義。

### <a name="new-azurestreamanalyticsoutput--new-azurermstreamanalyticsoutput"></a>New-AzureStreamAnalyticsOutput | New-AzureRMStreamAnalyticsOutput
在串流分析工作內建立新的輸出，或更新現有的輸出。  

您可以在 .json 檔案中或命令列上指定輸出的名稱。 若兩者皆指定，命令列上的名稱必須與檔案中的名稱相同。

如果您指定已存在的輸出，而且沒有指定 –Force 參數，Cmdlet 會詢問是否要取代現有的輸出。

如果您指定 –Force 參數，並指定現有的輸出名稱，則不經過確認就會取代輸出。

如需 JSON 檔案結構與內容的詳細資訊，請參閱[流分析管理 REST API 參考程式庫][stream.analytics.rest.api.reference]的[建立輸出 (Azure 串流分析)][msdn-rest-api-create-stream-analytics-output] 一節。

**範例 1**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output

此 PowerShell 命令會在 StreamingJob 工作中建立名為「output」的新輸出。 如果已經定義此名稱的現有輸出，Cmdlet 會詢問是否要取代它。

**範例 2**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output -Force

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output -Force

此 PowerShell 命令會取代 StreamingJob 工作中「output」的定義。

### <a name="new-azurestreamanalyticstransformation--new-azurermstreamanalyticstransformation"></a>New-AzureStreamAnalyticsTransformation | New-AzureRMStreamAnalyticsTransformation
在串流分析工作內建立新的轉換，或更新現有的轉換。

您可以在 .json 檔案中或命令列上指定轉換的名稱。 若兩者皆指定，命令列上的名稱必須與檔案中的名稱相同。

如果您指定已存在的轉換，而且沒有指定 –Force 參數，Cmdlet 會詢問是否要取代現有的轉換。

如果您指定 –Force 參數，並指定現有的轉換名稱，則不經過確認就會取代轉換。

如需 JSON 檔案結構與內容的詳細資訊，請參閱[串流分析管理 REST API 參考程式庫][stream.analytics.rest.api.reference]的[建立轉換 (Azure 串流分析)][msdn-rest-api-create-stream-analytics-transformation] 一節。

**範例 1**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform

此 PowerShell 命令會在 StreamingJob 工作中建立名為 StreamingJobTransform 的新轉換。 如果已經使用此名稱定義現有的轉換，Cmdlet 會詢問是否要取代它。

**範例 2**

Azure PowerShell 0.9.8：  

    New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform -Force

Azure PowerShell 1.0：  

    New-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform -Force

 此 PowerShell 命令會取代 StreamingJob 工作中 StreamingJobTransform 的定義。

### <a name="remove-azurestreamanalyticsinput--remove-azurermstreamanalyticsinput"></a>Remove-AzureStreamAnalyticsInput | Remove-AzureRMStreamAnalyticsInput
以非同步方式從 Microsoft Azure 中的 Stream Analytics 作業刪除特定的輸入。  
如果您指定 –Force 參數，則不經過確認就會刪除輸入。

**範例 1**

Azure PowerShell 0.9.8：  

    Remove-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EventStream

Azure PowerShell 1.0：  

    Remove-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EventStream

此 PowerShell 命令會移除 StreamingJob 工作中的 EventStream 輸入。  

### <a name="remove-azurestreamanalyticsjob--remove-azurermstreamanalyticsjob"></a>Remove-AzureStreamAnalyticsJob | Remove-AzureRMStreamAnalyticsJob
以非同步方式從 Microsoft Azure 中刪除特定的 Stream Analytics 作業。  
如果您指定 –Force 參數，則不經過確認就會刪除工作。

**範例 1**

Azure PowerShell 0.9.8：  

    Remove-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

Azure PowerShell 1.0：  

    Remove-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

此 PowerShell 命令會移除 StreamingJob 工作。  

### <a name="remove-azurestreamanalyticsoutput--remove-azurermstreamanalyticsoutput"></a>Remove-AzureStreamAnalyticsOutput | Remove-AzureRMStreamAnalyticsOutput
以非同步方式從 Microsoft Azure 中的 Stream Analytics 作業刪除特定的輸出。  
如果您指定 –Force 參數，則不經過確認就會刪除輸出。

**範例 1**

Azure PowerShell 0.9.8：  

    Remove-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

Azure PowerShell 1.0：  

    Remove-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

此 PowerShell 命令會移除 StreamingJob 工作中的 Output 輸出。  

### <a name="start-azurestreamanalyticsjob--start-azurermstreamanalyticsjob"></a>Start-AzureStreamAnalyticsJob | Start-AzureRMStreamAnalyticsJob
以非同步方式部署和啟動 Microsoft Azure 中的 Stream Analytics 作業。

**範例 1**

Azure PowerShell 0.9.8：  

    Start-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z

Azure PowerShell 1.0：  

    Start-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z

此 PowerShell 命令會啟動自訂輸出開始時間設為 2012 年 12 月 12 日 12:12:12 UTC 的 StreamingJob 工作。

### <a name="stop-azurestreamanalyticsjob--stop-azurermstreamanalyticsjob"></a>Stop-AzureStreamAnalyticsJob | Stop-AzureRMStreamAnalyticsJob
以非同步方式停止在 Microsoft Azure 中執行 Stream Analytics 作業，並取消配置正在使用的資源。 透過 Azure 入口網站和管理 API 仍可使用訂用帳戶內的該工作定義與中繼資料，也因此可以編輯和重新啟動工作。 您將不需要對處於已停止狀態的工作支付費用。

**範例 1**

Azure PowerShell 0.9.8：  

    Stop-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

Azure PowerShell 1.0：  

    Stop-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

此 PowerShell 命令會停止 StreamingJob 工作。  

### <a name="test-azurestreamanalyticsinput--test-azurermstreamanalyticsinput"></a>Test-AzureStreamAnalyticsInput | Test-AzureRMStreamAnalyticsInput
測試 Stream Analytics 連線到指定輸入的能力。

**範例 1**

Azure PowerShell 0.9.8：  

    Test-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EntryStream

Azure PowerShell 1.0：  

    Test-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EntryStream

此 PowerShell 命令會測試 StreamingJob 中 EntryStream 輸入的連接狀態。  

### <a name="test-azurestreamanalyticsoutput--test-azurermstreamanalyticsoutput"></a>Test-AzureStreamAnalyticsOutput | Test-AzureRMStreamAnalyticsOutput
測試 Stream Analytics 連線到指定輸出的能力。

**範例 1**

Azure PowerShell 0.9.8：  

    Test-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

Azure PowerShell 1.0：  

    Test-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

此 PowerShell 命令會測試 StreamingJob 中 Output 輸出的連接狀態。  

## <a name="get-support"></a>取得支援
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。 

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

[msdn-switch-azuremode]: http://msdn.microsoft.com/library/dn722470.aspx
[powershell-install]: http://azure.microsoft.com/documentation/articles/powershell-install-configure/
[msdn-rest-api-create-stream-analytics-job]: https://msdn.microsoft.com/library/dn834994.aspx
[msdn-rest-api-create-stream-analytics-input]: https://msdn.microsoft.com/library/dn835010.aspx
[msdn-rest-api-create-stream-analytics-output]: https://msdn.microsoft.com/library/dn835015.aspx
[msdn-rest-api-create-stream-analytics-transformation]: https://msdn.microsoft.com/library/dn835007.aspx

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301

