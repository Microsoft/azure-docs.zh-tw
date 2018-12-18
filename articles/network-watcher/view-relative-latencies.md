---
title: 檢視特定位置中 Azure 區域的相對延遲 | Microsoft Docs
description: 了解如何檢視特定位置中 Azure 區域跨網際網路提供者的相對延遲。
services: network-watcher
documentationcenter: ''
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/14/2017
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: a6c2ffa619eeff8b455df8a8b2157525af12c640
ms.sourcegitcommit: 3cdc82a5561abe564c318bd12986df63fc980a5a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/05/2018
ms.locfileid: "27601016"
---
# <a name="view-relative-latency-to-azure-regions-from-specific-locations"></a>檢視特定位置中 Azure 區域的相對延遲

在本教學課程中，了解如何使用 Azure [網路監看員](network-watcher-monitoring-overview.md)服務來協助您以您的使用者人數統計為基礎，決定要在哪一個 Azure 區域部署您的應用程式或服務。 此外，還可以使用它來協助您評估對 Azure 的服務提供者連線。  
        
## <a name="create-a-network-watcher"></a>建立網路監看員

如果您在至少一個 Azure [區域](https://azure.microsoft.com/regions)中已有網路監看員，可以略過本節中的工作。 建立網路監看員的資源群組。 在此範例中，會在美國東部區域中建立資源群組，但您可以在任何 Azure 區域中建立資源群組。

```powershell
New-AzureRmResourceGroup -Name NetworkWatcherRG -Location eastus
```

建立網路監看員。 您必須在至少一個 Azure 區域中建立網路監看員。 在此範例中，會在美國東部 Azure 區域中建立網路監看員。

```powershell
New-AzureRmNetworkWatcher -Name NetworkWatcher_eastus -ResourceGroupName NetworkWatcherRG -Location eastus
```

## <a name="compare-relative-network-latencies-to-a-single-azure-region-from-a-specific-location"></a>比較相對的網路延遲與特定位置中的單一 Azure 區域

從特定位置到服務部署所在的 Azure 區域來評估服務提供者，或是針對使用者報告的問題 (例如「站台變慢」) 進行疑難排解。 例如，下列命令會傳回美國華盛頓州與美國西部 2 Azure 區域、在 2017 年 12 月 13 日至 15 日之間的平均相對網際網路服務提供者延遲：

```powershell
Get-AzureRmNetworkWatcherReachabilityReport `
  -NetworkWatcherName NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG `
  -Location "West US 2" `
  -Country "United States" `
  -State "washington" `
  -StartTime "2017-12-13" `
  -EndTime "2017-12-15"
```

> [!NOTE]
> 您在前一個命令中指定的區域不必與您擷取網路監看員時所指定的區域相同。 前一個命令只會要求您指定現有的網路監看員。 網路監看員可位於任何區域。 如果您指定 `-Country` 和 `-State` 的值，它們必須是有效的。 值會區分大小寫。 資料可供有限數目的國家/地區、州和城市使用。 在[檢視可用的國家/地區、州、城市及提供者](#view-available)中執行命令，以檢視可與前一個命令搭配使用的國家/地區、城市及狀態清單。 

> [!WARNING]
> 您必須針對 `-StartTime` 和 `-EndTime` 指定 2017 年 11 月 14 日之後的日期。 指定 2017 年 11 月 14 日之前的日期不會傳回任何資料。 

前一個命令的輸出如下：

```powershell
AggregationLevel   : State
ProviderLocation   : {
                       "Country": "United States",
                       "State": "washington"
                     }
ReachabilityReport : [
                       {
                         "Provider": "Qwest Communications Company, LLC - ASN 209",
                         "AzureLocation": "West US 2",
                         "Latencies": [
                           {
                             "TimeStamp": "2017-12-14T00:00:00Z",
                             "Score": 92
                           },
                           {
                             "TimeStamp": "2017-12-13T00:00:00Z",
                             "Score": 92
                           }
                         ]
                       },
                       {
                         "Provider": "Comcast Cable Communications, LLC - ASN 7922",
                         "AzureLocation": "West US 2",
                         "Latencies": [
                           {
                             "TimeStamp": "2017-12-14T00:00:00Z",
                             "Score": 96
                           },
                           {
                             "TimeStamp": "2017-12-13T00:00:00Z",
                             "Score": 96
                           }
                         ]
                       }
                     ]
```

在傳回的輸出中，[分數] 的值是跨區域和提供者時的相對延遲。 1 分是最差 (最高) 的延遲，而 100 分是最低的延遲。 相對延遲是以一日平均計算。 在上述範例中，兩天的延遲顯然都相同，且兩個提供者的延遲之間差異很小，此外，兩個提供者的延遲在 1-100 級別中顯然也都很低。 儘管這是預期行為，因為美國華盛頓州實際上是接近美國西部 2 Azure 區域，而有時結果並未如預期。 您指定的日期範圍越大，在一段時間可平均的延遲就更多。

## <a name="compare-relative-network-latencies-across-azure-regions-from-a-specific-location"></a>比較跨 Azure 區域與特定位置中的相對網路延遲

如果您不是使用 `-Location` 指定特定位置與特定 Azure 區域之間的相對延遲，而是需要從特定的實體位置決定對所有 Azure 區域的相對延遲，您也可以這麼做。 例如，如果您的主要使用者是位於華盛頓州的 Comcast 使用者，下列命令可協助您評估要在哪一個 Azure 區域中部署服務：

```powershell
Get-AzureRmNetworkWatcherReachabilityReport `
  -NetworkWatcherName NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG `
  -Provider "Comcast Cable Communications, LLC - ASN 7922" `
  -Country "United States" `
  -State "washington" `
  -StartTime "2017-12-13" `
  -EndTime "2017-12-15"
```

>[!NOTE]
不同於您指定單一位置時，如果您不指定位置，或是指定多個位置，例如「美國西部 2」、「美國西部」，就必須在執行命令時，指定網際網路服務提供者。 

## <a name="view-available"></a>檢視可用的國家/地區、州、城市及提供者

資料可供特定的網際網路服務提供者、國家/地區、州和城市使用。 若要檢視所有您可以檢視資料的可用網際網路服務提供者、國家/地區、州和城市的清單，請輸入下列命令：

```powershell
Get-AzureRmNetworkWatcherReachabilityProvidersList -NetworkWatcherName NetworkWatcher_eastus -ResourceGroupName NetworkWatcherRG
```

資料僅可供前一個命令所傳回的國家/地區、州和城市使用。 前一個命令會要求您指定現有的網路監看員。 範例已在名為 *NetworkWatcherRG*的資源群組中指定 NetworkWatcher_eastus 網路監看員，但是您可以指定任何現有的網路監看員。 如果您沒有現有的網路監看員，請建立一個網路監看員，方法是完成[建立網路監看員](#create-a-network-watcher)中的工作。 

在執行前一個命令後，您可以篩選傳回的輸出，方法是指定 [國家/地區]、[州] 和 [城市] 的有效值。  例如，若要檢視美國境內華盛頓州的西雅圖中可用的網際網路服務提供者清單，請輸入下列命令：

```powershell
Get-AzureRmNetworkWatcherReachabilityProvidersList `
  -NetworkWatcherName NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG `
  -City seattle `
  -Country "United States" `
  -State washington
```

> [!WARNING]
> 針對 [國家/地區] 指定的值必須是大寫和小寫。 針對 [州] 和 [城市] 指定的值必須是小寫。 值必須列在執行不包含 [國家/地區]、[州] 和 [城市] 值的命令之後所傳回的輸出中。 如果您指定了不正確的大小寫，或所指定的 [國家/地區]、[州] 或 [城市] 值不在執行不包含這些屬性的值之命令之後傳回的輸出中，傳回的輸出就是空白的。
