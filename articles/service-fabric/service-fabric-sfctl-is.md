---
title: Azure Service Fabric CLI - sfctl is | Microsoft Docs
description: 描述 Service Fabric CLI sfctl is 命令。
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
ms.openlocfilehash: 53e49099fd3486d51f021528c9354cf32f4952d2
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39492784"
---
# <a name="sfctl-is"></a>sfctl is
查詢命令，並將其傳送至基礎結構服務。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| 命令 | 叫用指定基礎結構服務執行個體上的系統管理命令。 |
| query | 叫用指定基礎結構服務執行個體上的唯讀查詢。 |

## <a name="sfctl-is-command"></a>sfctl is command
叫用指定基礎結構服務執行個體上的系統管理命令。

對於設定一或多個基礎結構服務執行個體的叢集，此 API 提供將基礎結構特有的命令傳送至基礎結構服務特定執行個體的方法。 可用的命令和其對應的回應格式會根據叢集執行所在的基礎結構而有所不同。 這個 API 支援 Service Fabric 平台；這不表示直接從您的程式碼使用。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --command [必要] | 要叫用的命令文字。 命令的內容與基礎結構有關。 |
| --service-id | 基礎結構服務的識別。 <br><br> 這是不含 'fabric' URI 配置的基礎結構服務完整名稱。 只有當叢集中有多個基礎結構服務執行個體正在執行時，才需要提供此參數。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-is-query"></a>sfctl is query
叫用指定基礎結構服務執行個體上的唯讀查詢。

對於設定一或多個基礎結構服務執行個體的叢集，此 API 提供將基礎結構特有的查詢傳送至基礎結構服務特定執行個體的方法。 可用的命令和其對應的回應格式會根據叢集執行所在的基礎結構而有所不同。 這個 API 支援 Service Fabric 平台；這不表示直接從您的程式碼使用。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --command [必要] | 要叫用的命令文字。 命令的內容與基礎結構有關。 |
| --service-id | 基礎結構服務的識別。 <br><br> 這是不含 'fabric\:' URI 配置的基礎結構服務完整名稱。 只有當叢集中有多個基礎結構服務執行個體正在執行時，才需要提供此參數。 |
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