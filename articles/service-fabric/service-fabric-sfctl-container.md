---
title: Azure Service Fabric CLI- sfctl container | Microsoft Docs
description: 描述 Service Fabric CLI sfctl container 命令。
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
ms.openlocfilehash: 27108d27ee27346e4cba44e6778faff56df70a36
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495123"
---
# <a name="sfctl-container"></a>sfctl container
在叢集節點上執行容器相關命令。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| invoke-api | 叫用容器 REST API。 |
| logs | 擷取容器記錄。 |

## <a name="sfctl-container-invoke-api"></a>sfctl container invoke-api
叫用容器 REST API。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id           [必要] | 應用程式識別碼。 |
| --code-package-instance-id [必要] | 程式碼套件執行個體識別碼，可由 'service code-package-list' 來擷取。 |
| --code-package-name        [必要] | 程式碼套件名稱。 |
| --container-api-uri-path   [必要] | 容器 REST API URI 路徑，請使用 '{id}' 來取代容器名稱/識別碼。 |
| --node-name [必要] | 節點的名稱。 |
| --service-manifest-name    [必要] | 服務資訊清單名稱。 |
| --container-api-body | 容器 REST API 的 HTTP 要求本文。 |
| --container-api-content-type | 容器 REST API 的內容類型，預設為 'application/json'。 |
| --container-api-http-verb | 容器 REST API 的 HTTP 動詞，預設為 GET。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-container-logs"></a>sfctl container logs
擷取容器記錄。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id           [必要] | 應用程式識別碼。 |
| --code-package-instance-id [必要] | 程式碼套件執行個體識別碼，可由 'service code-package-list' 來擷取。 |
| --code-package-name        [必要] | 程式碼套件名稱。 |
| --node-name [必要] | 節點的名稱。 |
| --service-manifest-name    [必要] | 服務資訊清單名稱。 |
| --tail | 只從記錄結尾傳回此數目的記錄行。 請以整數指定，或指定 all 來輸出所有記錄行。 預設為 'all'。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |


## <a name="next-steps"></a>後續步驟
- [設定](service-fabric-cli.md) Service Fabric CLI。
- 了解如何使用[範例指令碼](/azure/service-fabric/scripts/sfctl-upgrade-application)來使用 Service Fabric CLI。