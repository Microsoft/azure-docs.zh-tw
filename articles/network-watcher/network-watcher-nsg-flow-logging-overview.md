---
title: Azure 網路監看員的網路安全性群組流量記錄簡介 | Microsoft Docs
description: 本文說明如何使用 Azure 網路監看員的 NSG 流量記錄功能。
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 47d91341-16f1-45ac-85a5-e5a640f5d59e
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: ae4edb82fa5e192a30d297dae82199bb7efca0c2
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145153"
---
# <a name="introduction-to-flow-logging-for-network-security-groups"></a>網路安全性群組流量記錄簡介

網路安全性群組 (NSG) 流量記錄是網路監看員的一項功能，可讓您檢視透過 NSG 輸入和輸出 IP 流量的相關資訊。 流量記錄是以 json 格式撰寫，會顯示每一規則的輸出和輸入流量、套用流量的網路介面 (NIC)、關於流量的 5 個 Tuple 資訊 (來源/目的地 IP、來源/目的地連接埠、通訊協定)，以及流量是被允許或拒絕。

![流量記錄概觀](./media/network-watcher-nsg-flow-logging-overview/figure1.png)

雖然流量記錄是以 NSG 為目標，但其顯示方式與其他記錄不同。 流量記錄只會儲存在儲存體帳戶內，並且會採用以下範例所示的記錄路徑：

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```

在其他記錄上看到的保留原則也同樣適用於流量記錄。 您可以設定 1 天到 2147483647 天的記錄保留原則。 如果未設定保留原則，則會永遠保留記錄檔。

## <a name="log-file"></a>記錄檔

流量記錄包含下列屬性：

* **time** - 事件的記錄時間
* **systemId** - 網路安全性群組資源識別碼。
* **類別** - 事件的類別。 類別一律為 **NetworkSecurityGroupFlowEvent**
* **resourceid** - NSG 的資源識別碼
* **operationName** - 一律是 NetworkSecurityGroupFlowEvents
* **properties** - 流量屬性的集合
    * **Version** - 流量記錄事件結構描述的版本號碼
    * **flows** - 流量的集合。 這個屬性有多個適用於不同規則的項目
        * **rule** - 做為流量列出依據的規則
            * **flows** - 流量的集合
                * **mac** - 流量收集所在 VM 之 NIC 的 MAC 位址
                * **flowTuples** - 包含多個流量 tuple 屬性的逗號分隔格式字串
                    * **時間戳記** - 這個值是流量發生時的時間戳記，格式為 UNIX EPOCH
                    * **來源 IP** - 來源 IP
                    * **目的地 IP** - 目的地 IP
                    * **來源連接埠** - 來源連接埠
                    * **目的地連接埠** - 目的地連接埠
                    * **通訊協定** - 流量的通訊協定。 有效值為 **T** (若為 TCP) 和 **U** (若為 UDP)
                    * **流量流動** - 流量的流動方向。 有效值為 **I** (若為輸入) 和 **O** (若為輸出)。
                    * **流量** - 流量受到允許或拒絕。 有效值為 **A** (若允許) 和 **D** (若拒絕)。

之後的文字是流量記錄範例。 如您所見，有多筆記錄遵循上一節所述的屬性清單。

> [!NOTE]
> \*flowTuples 屬性中的值是逗號分隔清單。
 
```json
{
    "records":
    [
        
        {
             "time": "2017-02-16T22:00:32.8950000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282421,42.119.146.95,10.1.0.4,51529,5358,T,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282370,163.28.66.17,10.1.0.4,61771,3389,T,I,A","1487282393,5.39.218.34,10.1.0.4,58596,3389,T,I,A","1487282393,91.224.160.154,10.1.0.4,61540,3389,T,I,A","1487282423,13.76.89.229,10.1.0.4,53163,3389,T,I,A"]}]}]}
        }
        ,
        {
             "time": "2017-02-16T22:01:32.8960000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282481,195.78.210.194,10.1.0.4,53,1732,U,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282435,61.129.251.68,10.1.0.4,57776,3389,T,I,A","1487282454,84.25.174.170,10.1.0.4,59085,3389,T,I,A","1487282477,77.68.9.50,10.1.0.4,65078,3389,T,I,A"]}]}]}
        }
        ,
        {
             "time": "2017-02-16T22:02:32.9040000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282492,175.182.69.29,10.1.0.4,28918,5358,T,I,D","1487282505,71.6.216.55,10.1.0.4,8080,8080,T,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282512,91.224.160.154,10.1.0.4,59046,3389,T,I,A"]}]}]}
        }
        ,
        ...
```

## <a name="next-steps"></a>後續步驟

- 若要了解如何啟用流量記錄，請參閱[啟用 NSG 流量記錄](network-watcher-nsg-flow-logging-portal.md)。
- 若要深入了解 NSG 記錄，請參閱[網路安全性群組 (NSG) 的記錄檔分析](../virtual-network/virtual-network-nsg-manage-log.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。
- 若要判斷是否允許或拒絕流量進出 VM，請參閱[診斷 VM 網路流量篩選問題](diagnose-vm-network-traffic-filtering-problem.md)
