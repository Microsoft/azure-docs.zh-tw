---
title: Azure 中 Windows VM 的已排定事件
description: 針對您的 Windows 虛擬機器使用 Azure Metadata Service 的已排程事件。
author: EricRadzikowskiMSFT
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/01/2020
ms.author: ericrad
ms.reviwer: mimckitt
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e4b5248ecb47c9456836aa9c4d7ebb2ad122c1dd
ms.sourcegitcommit: c7153bb48ce003a158e83a1174e1ee7e4b1a5461
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2021
ms.locfileid: "98231866"
---
# <a name="azure-metadata-service-scheduled-events-for-windows-vms"></a>Azure 中繼資料服務：Windows VM 的已排定事件

已排定的事件是一項「Azure 中繼資料服務」，可讓您的應用程式有時間準備虛擬機器維護。 它提供即將進行維護事件 (例如重新啟動) 的相關資訊，讓您的應用程式可進行準備以及限制中斷。 它適用於所有 Azure 虛擬機器類型 (包括 Windows 和 Linux 上的 PaaS 和 IaaS)。 

如需 Linux 上已排定事件的資訊，請參閱 [Linux VM 的已排定事件](../linux/scheduled-events.md)。

> [!Note] 
> 已排定事件已在所有 Azure 區域中正式推出。 請參閱[版本和區域可用性](#version-and-region-availability)以取得最新的版本資訊。

## <a name="why-use-scheduled-events"></a>為什麼要使用已排定事件？

許多應用程式可受益於 VM 維護的準備時間。 這些時間可用來執行應用程式特定的工作，能改善可用性、可靠性與服務性，包括： 

- 檢查點和還原。
- 清空連線。
- 主要複本容錯移轉。
- 從負載平衡器移除集區。
- 事件記錄。
- 順利關機。

使用已排定事件，應用程式就可以探索維護所發生的時間，以及限制其影響的觸發工作。  

排程的事件會提供下列使用案例中的事件：

- [平台起始的維護](../maintenance-and-updates.md?bc=/azure/virtual-machines/windows/breadcrumb/toc.json&toc=/azure/virtual-machines/windows/toc.json) (例如，主機的 VM 重新開機、即時移轉或記憶體保留更新)
- 虛擬機器正在預期即將失敗的[效能下降主機硬體](https://azure.microsoft.com/blog/find-out-when-your-virtual-machine-hardware-is-degraded-with-scheduled-events)上執行
- 使用者起始的維護 (例如，使用者重新啟動或重新部署 VM)
- [現成 VM](../spot-vms.md) 和[現成擴展集](../../virtual-machine-scale-sets/use-spot.md)執行個體收回。

## <a name="the-basics"></a>基本概念  

  如果您是使用可由 VM 內存取的 REST 端點來執行 VM，中繼資料服務會公開這類相關資訊。 這項資訊是透過無法路由傳送的 IP 取得，因此不會在 VM 之外公開。

### <a name="scope"></a>影響範圍
排程的事件會傳送到：

- 獨立虛擬機器。
- 雲端服務中的所有 VM。
- 可用性設定組中的所有 VM。
- 可用性區域中的所有 Vm。
- 擴展集放置群組中的所有 VM。 

> [!NOTE]
> 所有虛擬機器的 Scheduled Events， (網狀架構控制器中的 Vm)  (FC) 租使用者會傳遞至 FC 租使用者中的所有 Vm。 FC 租使用者等同于獨立 VM、整個雲端服務、整個可用性設定組，以及 VM 擴展集的放置群組 (VMSS) ，不論可用性區域的使用情形為何。 

因此，請檢查事件中的 `Resources` 欄位以找出哪些 VM 受到影響。

### <a name="endpoint-discovery"></a>端點探索
針對已啟用 VNET 的 VM，可以從靜態非可路由 IP `169.254.169.254` 取得「中繼資料服務」。 最新版已排定事件的完整端點為： 

 > `http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01`

如果 VM 不是建立在「虛擬網路」內 (雲端服務和傳統 VM 的預設案例)，就需要額外的邏輯，才能探索要使用的 IP 位址。 請參閱此範例以了解如何[探索主機端點](https://github.com/azure-samples/virtual-machines-python-scheduled-events-discover-endpoint-for-non-vnet-vm) \(英文\)。

### <a name="version-and-region-availability"></a>版本和區域可用性
已排定事件服務已進行版本設定。 版本是必要項目；目前版本為 `2019-01-01`。

| 版本 | 版本類型 | 區域 | 版本資訊 | 
| - | - | - | - | 
| 2019-08-01 | 正式運作 | 全部 | <li> 新增 EventSource 的支援 |
| 2019-04-01 | 正式運作 | 全部 | <li> 新增事件描述的支援 |
| 2019-01-01 | 正式運作 | 全部 | <li> 已新增支援虛擬機器擴展集 EventType 'Terminate' |
| 2017-11-01 | 正式運作 | 全部 | <li> 已新增支援現成 VM 收回 EventType 'Preempt'<br> | 
| 2017-08-01 | 正式運作 | 全部 | <li> 已從 IaaS VM 的資源名稱中移除預留底線<br><li>強制所有要求的中繼資料標頭需求 | 
| 2017-03-01 | 預覽 | 全部 | <li>初始版本 |


> [!NOTE] 
> 先前排定事件的預覽版支援作為 API 版本的 {latest}。 此格式將不再受到支援且之後會遭到取代。

### <a name="enabling-and-disabling-scheduled-events"></a>啟用和停用 Scheduled Events
系統會在您第一次提出事件要求時，為您的服務啟用「已排定事件」。 您可能會在第一次呼叫中遇到長達兩分鐘的延遲回應。

如果長達 24 小時未提出要求，您的服務就會停用已排定事件。

### <a name="user-initiated-maintenance"></a>使用者起始的維護
使用者透過 Azure 入口網站、API、CLI 或 PowerShell 起始的 VM 維護，將會產生「已排定事件」。 這可讓您測試應用程式中的維護準備邏輯，讓應用程式可以為使用者起始的維護預作準備。

如果您重新啟動 VM，則會排定類型為 `Reboot` 的事件。 如果您重新部署 VM，則會排定類型為 `Redeploy` 的事件。

## <a name="use-the-api"></a>使用 API

### <a name="headers"></a>headers
查詢中繼資料服務時，您必須提供 `Metadata:true` 標頭以免不小心重新導向要求。 所有排程的事件都需要 `Metadata:true` 標頭。 要求中未包含標頭會導致中繼資料服務「不正確的要求」回應。

### <a name="query-for-events"></a>查詢事件
您只要進行下列呼叫，即可查詢已排定事件：

#### <a name="bash"></a>Bash
```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

回應包含排定的事件陣列。 空白陣列表示目前沒有任何排定的事件。
在有排定事件的情況下，回應會包含事件陣列。 
```
{
    "DocumentIncarnation": {IncarnationID},
    "Events": [
        {
            "EventId": {eventID},
            "EventType": "Reboot" | "Redeploy" | "Freeze" | "Preempt" | "Terminate",
            "ResourceType": "VirtualMachine",
            "Resources": [{resourceName}],
            "EventStatus": "Scheduled" | "Started",
            "NotBefore": {timeInUTC},       
            "Description": {eventDescription},
            "EventSource" : "Platform" | "User",
        }
    ]
}
```

### <a name="event-properties"></a>事件屬性
|屬性  |  描述 |
| - | - |
| EventId | 此事件的全域唯一識別碼。 <br><br> 範例： <br><ul><li>602d9444-d2cd-49c7-8624-8643e7171297  |
| EventType | 此事件造成的影響。 <br><br> 值： <br><ul><li> `Freeze`:虛擬機器已排定暫停幾秒鐘。 CPU 和網路連線可能會暫止，但不會影響記憶體或已開啟的檔案。<li>`Reboot`:虛擬機器已排定要重新開機 (非持續性記憶體都會遺失)。 <li>`Redeploy`:虛擬機器已排定要移至另一個節點 (暫時磁碟都會遺失)。 <li>`Preempt`:正在刪除現成虛擬機器 (暫時磁碟會遺失)。 <li> `Terminate`:虛擬機器已排定要刪除。 |
| ResourceType | 受此事件影響的資源類型。 <br><br> 值： <ul><li>`VirtualMachine`|
| 資源| 受此事件影響的資源清單。 其中最多只能包含來自一個[更新網域](../manage-availability.md)的機器，但不能包含更新網域中的所有機器。 <br><br> 範例： <br><ul><li> ["FrontEnd_IN_0", "BackEnd_IN_0"] |
| EventStatus | 此事件的狀態。 <br><br> 值： <ul><li>`Scheduled`:此事件已排定在 `NotBefore` 屬性所指定的時間之後啟動。<li>`Started`:已啟動事件。</ul> 未曾提供 `Completed` 或類似的狀態。 當事件完成時，不會再傳回事件。
| NotBefore| 自此之後可啟動此事件的時間。 <br><br> 範例： <br><ul><li> Mon, 19 Sep 2016 18:29:47 GMT  |
| Description | 此事件的描述。 <br><br> 範例： <br><ul><li> 主機伺服器正在進行維護。 |
| EventSource | 事件的起始端。 <br><br> 範例： <br><ul><li> `Platform`：這個事件是由平臺所起始。 <li>`User`：這個事件是由使用者所起始。 |

### <a name="event-scheduling"></a>事件排定
系統會根據事件類型，為每個事件在未來安排最少的時間量。 事件的 `NotBefore` 屬性會反映這個時間。 

|EventType  | 最短時間通知 |
| - | - |
| 凍結| 15 分鐘 |
| 重新啟動 | 15 分鐘 |
| 重新部署 | 10 分鐘 |
| Preempt | 30 秒 |
| Terminate | [可由使用者設定](../../virtual-machine-scale-sets/virtual-machine-scale-sets-terminate-notification.md#enable-terminate-notifications)：5 至 15 分鐘 |

> [!NOTE] 
> 在某些情況下，Azure 能夠預測主機因為硬體效能下降而失敗，並嘗試排定移轉來減輕服務中斷的影響。 受影響的虛擬機器會收到含有 `NotBefore` (通常是未來幾天) 的已排定事件。 實際時間依預測的失敗風險評量而有所不同。 Azure 會盡可能提前 7 天通知，但實際時間不一定，如果預期硬體很可能即將失敗，則時間可能更短。 若要將您服務的風險降至最低，以防萬一硬體在系統起始移轉之前失敗，我們建議您盡快重新部署虛擬機器。

### <a name="polling-frequency"></a>輪詢頻率

您可以視需要經常或不頻繁地輪詢端點是否有更新。 不過，要求之間的時間越長，您可能會失去回應即將發生之事件的時間。 大部分的事件都有5至15分鐘的事先通知，雖然在某些情況下，事先通知可能會比30秒少。 為了確保您有最多時間可以採取緩和動作，建議您每秒輪詢一次服務。

### <a name="start-an-event"></a>啟動事件 

在您得知即將發生的事件，並完成正常關機邏輯之後，即可使用 `EventId` 向中繼資料服務進行 `POST` 呼叫，以核准未處理的事件。 對 Azure 來說，此呼叫可以將通知時間縮到最短 (可能的話)。 

以下是 `POST` 要求本文中必須要有的 JSON 範例。 要求需包含 `StartRequests` 清單。 每個 `StartRequest` 都包含您需要加速之事件的 `EventId`：
```
{
    "StartRequests" : [
        {
            "EventId": {EventId}
        }
    ]
}
```

#### <a name="bash-sample"></a>Bash 範例
```
curl -H Metadata:true -X POST -d '{"StartRequests": [{"EventId": "f020ba2e-3bc0-4c40-a10b-86575a9eabd5"}]}' http://169.254.169.254/metadata/scheduledevents?api-version=2019-01-01
```

> [!NOTE] 
> 認可事件時，即會允許事件中所有 `Resources` 的事件繼續進行，而不僅是認可此事件 VM。 因此，您可以選擇一個領導者來協調認可；最簡單的方法是選擇 `Resources` 欄位的第一部機器。

## <a name="python-sample"></a>Python 範例 

下列範例會查詢中繼資料服務來找出已排定事件，並核准每個未處理的事件：

```python
#!/usr/bin/python

import json
import socket
import urllib2

metadata_url = "http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01"
this_host = socket.gethostname()


def get_scheduled_events():
    req = urllib2.Request(metadata_url)
    req.add_header('Metadata', 'true')
    resp = urllib2.urlopen(req)
    data = json.loads(resp.read())
    return data


def handle_scheduled_events(data):
    for evt in data['Events']:
        eventid = evt['EventId']
        status = evt['EventStatus']
        resources = evt['Resources']
        eventtype = evt['EventType']
        resourcetype = evt['ResourceType']
        notbefore = evt['NotBefore'].replace(" ", "_")
    description = evt['Description']
    eventSource = evt['EventSource']
        if this_host in resources:
            print("+ Scheduled Event. This host " + this_host +
                " is scheduled for " + eventtype + 
        " by " + eventSource + 
        " with description " + description +
        " not before " + notbefore)
            # Add logic for handling events here


def main():
    data = get_scheduled_events()
    handle_scheduled_events(data)


if __name__ == '__main__':
    main()
```

## <a name="next-steps"></a>後續步驟 
- 觀看 [Azure Friday 上的已排定事件](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance) \(英文\) 的示範。 
- 在 [Azure 執行個體中繼資料已排定事件 GitHub 存放庫](https://github.com/Azure-Samples/virtual-machines-scheduled-events-discover-endpoint-for-non-vnet-vm) \(英文\) 中檢閱排程的事件程式碼範例。
- 深入了解[執行個體中繼資料服務](instance-metadata-service.md)中提供的 API。
- 了解 [Azure 中 Windows 虛擬機器預定進行的維修](../maintenance-and-updates.md?bc=/azure/virtual-machines/windows/breadcrumb/toc.json&toc=/azure/virtual-machines/windows/toc.json)。
