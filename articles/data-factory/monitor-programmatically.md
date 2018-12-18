---
title: 以程式設計方式監視 Azure Data Factory | Microsoft Docs
description: 了解如何使用不同的軟體開發套件 (SDK) 來監視資料處理站中的管線。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: douglasl
ms.openlocfilehash: 343af57cc8f3e63965dc1fe1827b2945009ea8bf
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045777"
---
# <a name="programmatically-monitor-an-azure-data-factory"></a>以程式設計方式監視 Azure Data Factory
本文說明如何使用不同的軟體開發套件 (SDK) 來監視資料處理站中的管線。 

## <a name="data-range"></a>資料範圍

Data Factory 只會儲存管線執行資料 45 天。 當您以程式設計方式查詢有關 Data Factory 管線執行的資料時 (例如，使用 PowerShell 命令 `Get-AzureRmDataFactoryV2PipelineRun`)，可選的 `LastUpdatedAfter` 和 `LastUpdatedBefore` 參數沒有最大日期。 但如果您查詢過去一年的資料，例如，查詢不會傳回錯誤，但僅傳回最近 45 天的管線執行資料。

如果要將管線執行資料保存超過 45 天，請使用 [Azure 監視器](monitor-using-azure-monitor.md)設定您自己的診斷記錄。

## <a name="net"></a>.NET
如需有關使用 .NET SDK 來建立和監視管線的完整逐步解說，請參閱[使用 .NET 建立資料處理站和管線](quickstart-create-data-factory-dot-net.md)。

1. 新增下列程式碼來持續檢查管線執行狀態，直到它完成資料複製為止。

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, runResponse.RunId);
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

2. 新增下列程式碼來擷取複製活動執行詳細資料，例如所讀取/寫入的資料大小。

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");
   
    List<ActivityRun> activityRuns = client.ActivityRuns.ListByPipelineRun(
    resourceGroup, dataFactoryName, runResponse.RunId, DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)).ToList(); 
    if (pipelineRun.Status == "Succeeded")
        Console.WriteLine(activityRuns.First().Output);
    else
        Console.WriteLine(activityRuns.First().Error);
    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

如需有關 .NET SDK 的完整文件，請參閱 [Data Factory .NET SDK 參考](/dotnet/api/microsoft.azure.management.datafactory?view=azure-dotnet)。

## <a name="python"></a>Python
如需有關使用 Python SDK 來建立和監視管線的完整逐步解說，請參閱[使用 Python 建立資料處理站和管線](quickstart-create-data-factory-python.md)。

若要監視管線執行，請新增下列程式碼：

```python
#Monitor the pipeline run
time.sleep(30)
pipeline_run = adf_client.pipeline_runs.get(rg_name, df_name, run_response.run_id)
print("\n\tPipeline run status: {}".format(pipeline_run.status))
activity_runs_paged = list(adf_client.activity_runs.list_by_pipeline_run(rg_name, df_name, pipeline_run.run_id, datetime.now() - timedelta(1),  datetime.now() + timedelta(1)))
print_activity_run_details(activity_runs_paged[0])
```

如需有關 Python SDK 的完整文件，請參閱 [Data Factory Python SDK 參考](/python/api/overview/azure/datafactory?view=azure-python)。

## <a name="rest-api"></a>REST API
如需有關使用 REST API 來建立和監視管線的完整逐步解說，請參閱[使用 REST API 建立資料處理站和管線](quickstart-create-data-factory-rest-api.md)。
 
1. 執行下列程式碼以持續檢查管線執行狀態，直到完成複製資料為止。

    ```powershell
    $request = "https://management.azure.com/subscriptions/${subsId}/resourceGroups/${resourceGroup}/providers/Microsoft.DataFactory/factories/${dataFactoryName}/pipelineruns/${runId}?api-version=${apiVersion}"
    while ($True) {
        $response = Invoke-RestMethod -Method GET -Uri $request -Header $authHeader
        Write-Host  "Pipeline run status: " $response.Status -foregroundcolor "Yellow"

        if ($response.Status -eq "InProgress") {
            Start-Sleep -Seconds 15
        }
        else {
            $response | ConvertTo-Json
            break
        }
    }
    ```
2. 執行下列指令碼來取出複製活動執行詳細資料，例如，讀取/寫入資料的大小。

    ```PowerShell
    $request = "https://management.azure.com/subscriptions/${subsId}/resourceGroups/${resourceGroup}/providers/Microsoft.DataFactory/factories/${dataFactoryName}/pipelineruns/${runId}/activityruns?api-version=${apiVersion}&startTime="+(Get-Date).ToString('yyyy-MM-dd')+"&endTime="+(Get-Date).AddDays(1).ToString('yyyy-MM-dd')+"&pipelineName=Adfv2QuickStartPipeline"
    $response = Invoke-RestMethod -Method GET -Uri $request -Header $authHeader
    $response | ConvertTo-Json
    ```

如需有關 REST API 的完整文件，請參閱 [Data Factory REST API 參考](/rest/api/datafactory/)。

## <a name="powershell"></a>PowerShell
如需有關使用 PowerShell 來建立和監視管線的完整逐步解說，請參閱[使用 PowerShell 建立資料處理站和管線](quickstart-create-data-factory-powershell.md)。

1. 執行下列程式碼以持續檢查管線執行狀態，直到完成複製資料為止。

    ```powershell
    while ($True) {
        $run = Get-AzureRmDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

        if ($run) {
            if ($run.Status -ne 'InProgress') {
                Write-Host "Pipeline run finished. The status is: " $run.Status -foregroundcolor "Yellow"
                $run
                break
            }
            Write-Host  "Pipeline is running...status: InProgress" -foregroundcolor "Yellow"
        }

        Start-Sleep -Seconds 30
    }
    ```
2. 執行下列指令碼來取出複製活動執行詳細資料，例如，讀取/寫入資料的大小。

    ```powershell
    Write-Host "Activity run details:" -foregroundcolor "Yellow"
    $result = Get-AzureRmDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $result
    
    Write-Host "Activity 'Output' section:" -foregroundcolor "Yellow"
    $result.Output -join "`r`n"
    
    Write-Host "\nActivity 'Error' section:" -foregroundcolor "Yellow"
    $result.Error -join "`r`n"
    ```

如需有關 PowerShell Cmdlet 的完整文件，請參閱 [Data Factory PowerShell Cmdlet 參考](/powershell/module/azurerm.datafactoryv2/?view=azurermps-4.4.1)。

## <a name="next-steps"></a>後續步驟
請參閱[使用 Azure 監視器來監視管線](monitor-using-azure-monitor.md)文章，以了解如何使用 Azure 監視器來監視 Data Factory 管線。 

