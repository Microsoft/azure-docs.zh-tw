---
title: Azure Service Fabric CLI- sfctl chaos schedule | Microsoft Docs
description: 描述 Service Fabric CLI sfctl chaos schedule 命令。
services: service-fabric
documentationcenter: na
author: Christina-Kang
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: cli
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 07/31/2018
ms.author: bikang
ms.openlocfilehash: 176b04b9bb16b5f183298c75f16bceb5e885e293
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39492235"
---
# <a name="sfctl-chaos-schedule"></a>sfctl chaos schedule
取得並設定 chaos 排程。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| get | 取得定義 Chaos 執行時間及執行方式的 Chaos 排程。 |
| set | 設定 Chaos 所要使用的 Chaos 排程。 |

## <a name="sfctl-chaos-schedule-get"></a>sfctl chaos schedule get
取得定義 Chaos 執行時間及執行方式的 Chaos 排程。

取得使用中的 Chaos 排程版本，以及定義 Chaos 執行時間及執行方式的 Chaos 排程。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-chaos-schedule-set"></a>sfctl chaos schedule set
設定 Chaos 所要使用的 Chaos 排程。

設定 Chaos 目前使用中的 Chaos 排程。 Chaos 會根據 Chaos 排程自動排定執行。 所提供的輸入排程中版本必須與伺服器上的 Chaos 排程版本相符。 如果所提供的版本與伺服器上的版本不符，就不會更新 Chaos 排程。 如果所提供的版本與伺服器上的版本相符，則會更新 Chaos 排程，而伺服器上的 Chaos 排程版本會增加 1，並在達到 2,147,483,647 之後歸 0。 如果 Chaos 在進行此呼叫時正在執行，此呼叫將會失敗。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --chaos-parameters-dictionary | 針對字串名稱對應於作業所要使用的 ChaosParameters，代表對應關係的 JSON 編碼清單。 |
| --expiry-date-utc | 何時該停止使用排程來排定 Chaos 的日期和時間。  預設值\: 9999-12-31T23\:59\:59.999Z。 |
| --jobs | 代表何時該執行 Chaos 及要搭配哪些參數來執行 Chaos 的 ChaosScheduleJobs JSON 編碼清單。 |
| --start-date-utc | 何時該開始使用排程來排定 Chaos 的日期和時間。  預設值\: 1601-01-01T00\:00\:00.000Z。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |
| --version | 排程的版本號碼。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

### <a name="examples"></a>範例

下列命令會將排程設定為 (假設目前的排程版本為 0) 在 2016-01-01 開始並在 2038-01-01 到期，一週 7 天、每天 24 小時執行 Chaos。 Chaos 將會排定在該時間於叢集上執行。

    sfctl chaos schedule set --version 0 --start-date-utc "2016-01-01T00:00:00.000Z" --expiry-date-utc "2038-01-01T00:00:00.000Z"
    --chaos-parameters-dictionary 
    [  
    {  
        "Key":"adhoc",
        "Value":{  
            "MaxConcurrentFaults":3,
            "EnableMoveReplicaFaults":true,
            "ChaosTargetFilter":{  
                "NodeTypeInclusionList":[  
                "N0010Ref",
                "N0020Ref",
                "N0030Ref",
                "N0040Ref",
                "N0050Ref"
                ]
            },
            "MaxClusterStabilizationTimeoutInSeconds":60,
            "WaitTimeBetweenIterationsInSeconds":15,
            "WaitTimeBetweenFaultsInSeconds":30,
            "TimeToRunInSeconds":"600",
            "Context":{  
                "Map":{  
                "test":"value"
                }
            },
            "ClusterHealthPolicy":{  
                "MaxPercentUnhealthyNodes":0,
                "ConsiderWarningAsError":true,
                "MaxPercentUnhealthyApplications":0
            }
        }
    }
    ]
    --jobs 
    [  
    {  
        "ChaosParameters":"adhoc",
        "Days":{  
            "Sunday":true,
            "Monday":true,
            "Tuesday":true,
            "Wednesday":true,
            "Thursday":true,
            "Friday":true,
            "Saturday":true
        },
        "Times":[  
            {  
                "StartTime":{  
                "Hour":0,
                "Minute":0
                },
                "EndTime":{  
                "Hour":23,
                "Minute":59
                }
            }
        ]
    }
    ]

## <a name="next-steps"></a>後續步驟
- [設定](service-fabric-cli.md) Service Fabric CLI。
- 了解如何使用[範例指令碼](/azure/service-fabric/scripts/sfctl-upgrade-application)來使用 Service Fabric CLI。