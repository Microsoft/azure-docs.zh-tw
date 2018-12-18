---
title: 在 Azure Data Factory 中建立輪轉視窗觸發程序 | Microsoft Docs
description: 了解如何在 Azure Data Factory 中建立依輪轉視窗執行管線的觸發程序。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/27/2018
ms.author: shlo
ms.openlocfilehash: c42d6235af8a5ab27fbd550b63c301fd9c6f15b1
ms.sourcegitcommit: 7ad9db3d5f5fd35cfaa9f0735e8c0187b9c32ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39325028"
---
# <a name="create-a-trigger-that-runs-a-pipeline-on-a-tumbling-window"></a>建立依輪轉視窗執行管線的觸發程序
本文章提供建立、啟動、監視輪轉視窗觸發程序的步驟。 如需觸發程序及支援類型的詳細資訊，請參閱[管線執行和觸發程序](concepts-pipeline-execution-triggers.md)。

輪轉視窗觸發程序是可從指定的開始時間定期引發，同時還能保留狀態的一種觸發程序。 輪轉視窗是一系列大小固定、非重疊的連續時間間隔。 輪轉視窗觸發程序與管線會有一對一的關係，並只能參考單一管線。

## <a name="data-factory-ui"></a>Data Factory UI

若要在 Azure 入口網站中建立輪轉視窗觸發程序，請選取 [觸發程序] > [輪轉視窗] > [下一步]，然後設定用來定義輪轉視窗的屬性。

![在 Azure 入口網站中建立輪轉視窗觸發程序](media/how-to-create-tumbling-window-trigger/create-tumbling-window-trigger.png)

## <a name="tumbling-window-trigger-type-properties"></a>輪轉視窗觸發程序類型屬性
輪轉視窗有下列觸發程序類型屬性：

```  
{
    "name": "MyTriggerName",
    "properties": {
        "type": "TumblingWindowTrigger",
        "runtimeState": "<<Started/Stopped/Disabled - readonly>>",
        "typeProperties": {
            "frequency": "<<Minute/Hour>>",
            "interval": <<int>>,
            "startTime": "<<datetime>>",
            "endTime: "<<datetime – optional>>"",
            "delay": "<<timespan – optional>>",
            “maxConcurrency”: <<int>> (required, max allowed: 50),
            "retryPolicy": {
                "count":  <<int - optional, default: 0>>,
                “intervalInSeconds”: <<int>>,
            }
        },
        "pipeline":
            {
                "pipelineReference": {
                    "type": "PipelineReference",
                    "referenceName": "MyPipelineName"
                },
                "parameters": {
                    "parameter1": {
                        "type": "Expression",
                        "value": "@{concat('output',formatDateTime(trigger().outputs.windowStartTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                    },
                    "parameter2": {
                        "type": "Expression",
                        "value": "@{concat('output',formatDateTime(trigger().outputs.windowEndTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                    },
                    "parameter3": "https://mydemo.azurewebsites.net/api/demoapi"
                }
            }
      }    
}
```  

下表提供與輪轉視窗觸發程序之週期和排程相關的主要 JSON 元素的概要概觀：

| JSON 元素 | 說明 | 類型 | 允許的值 | 必要 |
|:--- |:--- |:--- |:--- |:--- |
| **type** | 觸發程序的類型。 類型是固定的值 "TumblingWindowTrigger"。 | 字串 | "TumblingWindowTrigger" | 是 |
| **runtimeState** | 觸發程序執行時間的目前狀態。<br/>**附註**：這個元素是 \<readOnly>。 | 字串 | "Started"、"Stopped"、"Disabled" | 是 |
| **frequency** | 一個字串，代表觸發程序一再執行的頻率單位 (分鐘或小時)。 如果 **startTime** 日期值比 **frequency** 值更細微，計算視窗界限時，會將 **startTime** 日期納入計算。 例如，如果 **frequency** 值是小時，**startTime** 值是 2016-04-01T10:10:10Z，則第一個視窗是 (2017-09-01T10:10:10Z, 2017-09-01T11:10:10Z)。 | 字串 | "minute"、"hour"  | 是 |
| **interval** | 代表 **frequency** 值之間隔的整數值，用來決定觸發程序執行的頻率。 例如，如果 **interval** 為 3，而 **frequency** 為 "hour"，則觸發程序就會每隔 3 小時重複執行一次。 | 整數  | 正整數。 | 是 |
| **startTime**| 第一次出現，可以是過去。 第一個觸發程序間隔是 (**startTime**, **startTime** + **interval**)。 | Datetime | 日期時間值。 | 是 |
| **endTime**| 最後一次出現，可以是過去。 | Datetime | 日期時間值。 | 是 |
| **delay** | 視窗延遲開始資料處理所延遲的時間長度。 管線執行會在預期執行時間加上 **delay** 後開始。 **delay** 定義觸發程序要在超過到期時間多久之後，才觸發新的執行。 **delay** 不會改變視窗的 **startTime**。 例如，**delay** 值為 00:10:00 表示延遲 10 分鐘。 | Timespan  | 時間值，預設值是 00:00:00。 | 否 |
| **maxConcurrency** | 就緒視窗可引發的同時執行觸發程序數目。 例如，為昨天回填每小時執行，結果會有 24 個視窗。 如果 **maxConcurrency** = 10，只有前 10 個視窗 (00:00-01:00 - 00:09:00-10) 會引發觸發程序事件。 前 10 個觸發的管線執行完成之後，才會引發接下來 10 個視窗 (10:00-11:00 - 19:00 20:00) 的觸發程序執行。 繼續以本範例的 **maxConcurrency** = 10 說明，如果有 10 個就緒視窗，則總共會有 10 個管線執行。 如果只有 1 個就緒視窗，則只有 1 個管線執行。 | 整數  | 1 到 50 之間的整數。 | 是 |
| **retryPolicy: Count** | 到管線執行標示為 [失敗] 前的重試次數。  | 整數  | 整數，預設值為 0 (無重試)。 | 否 |
| **retryPolicy: intervalInSeconds** | 重試嘗試之間的延遲 (以秒指定) | 整數  | 秒數，預設值是 30。 | 否 |

### <a name="windowstart-and-windowend-system-variables"></a>WindowStart 和 WindowEnd 系統變數

您可以在您的 **pipeline** (管線) 定義中使用輪轉視窗觸發程序的 **WindowStart** 和 **WindowEnd** 系統變數 (也是查詢的一部分)。 將系統變數當作參數傳遞給 **trigger** (觸發程序) 定義中的管線。 以下範例示範如何將這些變數當作參數傳遞：

```  
{
    "name": "MyTriggerName",
    "properties": {
        "type": "TumblingWindowTrigger",
            ...
        "pipeline":
            {
                "pipelineReference": {
                    "type": "PipelineReference",
                    "referenceName": "MyPipelineName"
                },
                "parameters": {
                    "MyWindowStart": {
                        "type": "Expression",
                        "value": "@{concat('output',formatDateTime(trigger().outputs.windowStartTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                    },
                    "MyWindowEnd": {
                        "type": "Expression",
                        "value": "@{concat('output',formatDateTime(trigger().outputs.windowEndTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                    }
                }
            }
      }    
}
```  

若要在管線定義中使用 **WindowStart** 和 **WindowEnd** 系統變數值，分別使用您的 "MyWindowStart" 和 "MyWindowEnd" 參數。

### <a name="execution-order-of-windows-in-a-backfill-scenario"></a>回填案例的視窗執行順序
當有多個視窗可執行時 (特別是在回填案例中)，視窗的執行順序是確定的，從最舊到最新的間隔。 目前無法修改此行為。

### <a name="existing-triggerresource-elements"></a>現有 TriggerResource 元素
下列各點適用於現有的 **TriggerResource** 元素：

* 如果觸發程序的 **frequency** 元素 (或視窗大小) 值變更了，已經處理的視窗狀態「不會」重設。 觸發程序會繼續由它執行的最新視窗引發，並使用新的視窗大小。
* 如果觸發程序的 **endTime** 元素值變更了 (新增或更新)，已經處理的視窗狀態「不會」重設。 觸發程序會採用新的 **endTime** 值。 如果新的 **endTime** 值在已經執行的視窗之前，觸發程序會停止。 反之，觸發程序會在遇到新的 **endTime** 值時停止。

## <a name="sample-for-azure-powershell"></a>Azure PowerShell 的範例
本節說明如何使用 Azure PowerShell 來建立、啟動及監視觸發程序。

1. 在 C:\ADFv2QuickStartPSH\ 資料夾中，使用下列內容建立名為 **MyTrigger.json** 的 JSON 檔案：

   > [!IMPORTANT]
   > 在您儲存 JSON 檔案之前，請先將 **startTime** 元素的值設定為目前的 UTC 時間。 將 **endTime** 元素的值設定為目前 UTC 時間的一小時之後。

    ```json   
    {
      "name": "PerfTWTrigger",
      "properties": {
        "type": "TumblingWindowTrigger",
        "typeProperties": {
          "frequency": "Minute",
          "interval": "15",
          "startTime": "2017-09-08T05:30:00Z",
          "delay": "00:00:01",
          "retryPolicy": {
            "count": 2,
            "intervalInSeconds": 30
          },
          "maxConcurrency": 50
        },
        "pipeline": {
          "pipelineReference": {
            "type": "PipelineReference",
            "referenceName": "DynamicsToBlobPerfPipeline"
          },
          "parameters": {
            "windowStart": "@trigger().outputs.windowStartTime",
            "windowEnd": "@trigger().outputs.windowEndTime"
          }
        },
        "runtimeState": "Started"
      }
    }
    ```  

2. 使用 **Set-AzureRmDataFactoryV2Trigger** Cmdlet 來建立觸發程序：

    ```powershell
    Set-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger" -DefinitionFile "C:\ADFv2QuickStartPSH\MyTrigger.json"
    ```
    
3. 使用 **Get-AzureRmDataFactoryV2Trigger** Cmdlet 來確認觸發程序的狀態為 **Stopped** \(已停止\)：

    ```powershell
    Get-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

4. 使用 **Start-AzureRmDataFactoryV2Trigger** Cmdlet 來啟動觸發程序：

    ```powershell
    Start-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

5. 使用 **Get-AzureRmDataFactoryV2Trigger** Cmdlet 來確認觸發程序的狀態為 **Started** \(已啟動\)：

    ```powershell
    Get-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

6. 在 Azure PowerShell 中使用 **Get-AzureRmDataFactoryV2TriggerRun** Cmdlet 來取得觸發程序執行。 若要取得有關觸發程序執行的資訊，請定期執行以下命令。 更新 **TriggerRunStartedAfter** 和 **TriggerRunStartedBefore** 的值，以符合您的觸發程序定義中的值：

    ```powershell
    Get-AzureRmDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-08T00:00:00" -TriggerRunStartedBefore "2017-12-08T01:00:00"
    ```
    
若要在 Azure 入口網站中監視觸發程序執行和管線執行，請參閱[監視管線執行](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline)。

## <a name="next-steps"></a>後續步驟
如需有關觸發程序的詳細資訊，請參閱[管線執行和觸發程序](concepts-pipeline-execution-triggers.md#triggers)。
