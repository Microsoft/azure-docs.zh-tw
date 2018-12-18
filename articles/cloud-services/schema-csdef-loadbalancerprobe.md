---
title: Azure 雲端服務定義LoadBalancerProbe 結構描述 | Microsoft Docs
ms.custom: ''
ms.date: 04/14/2015
services: cloud-services
ms.reviewer: ''
ms.service: cloud-services
ms.suite: ''
ms.tgt_pltfrm: ''
ms.topic: reference
ms.assetid: 113374a8-8072-4994-9d99-de391a91e6ea
caps.latest.revision: 14
author: jpconnock
ms.author: jeconnoc
manager: timlt
ms.openlocfilehash: f7b0ba3b4797149798037dee0188850eff6baf1d
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39003283"
---
# <a name="azure-cloud-services-definition-loadbalancerprobe-schema"></a>Azure 雲端服務定義 LoadBalancerProbe 結構描述
負載平衡器探查是客戶定義的健康情況探查，可探查 UDP 端點和角色執行個體中的端點。 `LoadBalancerProbe` 不是獨立元素；它會與服務定義檔中的 Web 角色或背景工作角色結合。 `LoadBalancerProbe` 可由多個角色使用。

服務定義檔的預設副檔名為 .csdef。

## <a name="the-function-of-a-load-balancer-probe"></a>負載平衡器探查的函式
Azure Load Balancer 會負責將連入流量路由至角色執行個體。 負載平衡器會判斷哪些執行個體可以藉由定期探查每個執行個體來接收流量，以判斷該執行個體的健康情況。 負載平衡器每分鐘會探查每個執行個體多次。 有兩個不同的選項可以將執行個體健康情況提供給負載平衡器 – 預設負載平衡器探查，或自訂負載平衡器探查，藉由定義 .csdef 檔案中的 LoadBalancerProbe 來實作。

只有在執行個體處於 [就緒] 狀態 (也就是不在其他狀態中，例如 [忙碌]、[回收中] 或 [停止中]) 時，預設負載平衡器探查才會利用虛擬機器內的客體代理程式，它會接聽並以「HTTP 200 確定」作為回應。 如果客體代理程式無法以「HTTP 200 確定」回應，則 Azure Load Balancer 會將執行個體標示為沒有回應，並停止傳送流量到該執行個體。 Azure Load Balancer 會繼續 ping 執行個體，如果客體代理程式以 HTTP 200 回應，Azure Load Balancer 會再次將流量傳送至該執行個體。 使用 Web 角色時，您的網站程式碼通常會在不受 Azure 網狀架構或客體代理程式監視的 w3wp.exe 中執行，這表示 w3wp.exe 中的失敗 (例如 HTTP 500 回應) 不會向客體代理程式回報，而且負載平衡器不知道要將該執行個體退出循環。

自訂負載平衡器探查會覆寫預設客體代理程式探查，可讓您建立自己的自訂邏輯來判斷角色執行個體的健全狀況。 負載平衡器會定期探查您的端點 (預設為每 15 秒)，如果執行個體在逾時期間 (預設為 31 秒) 內以 TCP ACK 或 HTTP 200 回應，則會將執行個體列入循環。 這對於實作您自己的邏輯以從負載平衡器循環移除執行個體很有用，例如，如果執行個體 CPU 超過 90%，則傳回非 200 狀態。 針對使用 w3wp.exe 的 Web 角色，這也表示您能夠自動監視您的網站，因為網站程式碼中的錯誤會將非 200 狀態傳回給負載平衡器探查。 如果您未在 .csdef 檔案中定義 LoadBalancerProbe，則會使用預設負載平衡器行為 (如先前所述)。

如果您使用自訂負載平衡器探查，請確保您的邏輯會將 RoleEnvironment.OnStop 方法列入考量。 使用預設負載平衡器探查時，在呼叫 OnStop 之前，執行個體會被退出循環，但是在 OnStop 事件期間，自訂負載平衡器探查可以繼續傳回「200 確定」。 如果您使用 OnStop 事件來清除快取、停止服務，或者進行會影響服務執行階段行為的變更，則您需要確保自訂負載平衡器探查邏輯會從循環中移除執行個體。

## <a name="basic-service-definition-schema-for-a-load-balancer-probe"></a>負載平衡器探查的基本服務定義結構描述
 包含負載平衡器探查之服務定義檔的基本格式如下所示。

```xml
<ServiceDefinition …>
   <LoadBalancerProbes>
      <LoadBalancerProbe name="<load-balancer-probe-name>" protocol="[http|tcp]" path="<uri-for-checking-health-status-of-vm>" port="<port-number>" intervalInSeconds="<interval-in-seconds>" timeoutInSeconds="<timeout-in-seconds>"/>
   </LoadBalancerProbes>
</ServiceDefinition>
```

## <a name="schema-elements"></a>結構描述元素
服務定義檔的 `LoadBalancerProbes` 元素包含下列元素：

- [LoadBalancerProbes 元素](#LoadBalancerProbes)
- [LoadBalancerProbe 元素](#LoadBalancerProbe)

##  <a name="LoadBalancerProbes"></a> LoadBalancerProbes 元素
`LoadBalancerProbes` 元素會說明負載平衡器探查的集合。 此元素是 [LoadBalancerProbe 元素](#LoadBalancerProbe)的父代元素。 

##  <a name="LoadBalancerProbe"></a> LoadBalancerProbe 元素
`LoadBalancerProbe` 元素會定義模型的健康情況探查。 您可以定義多個負載平衡器探查。 

下表說明 `LoadBalancerProbe` 元素的屬性：

|屬性|類型|說明|
| ------------------- | -------- | -----------------|
| `name`              | `string` | 必要。 負載平衡器探查的名稱。 名稱必須是唯一的。|
| `protocol`          | `string` | 必要。 指定端點的通訊協定。 可能的值為 `http` 或 `tcp`。 如果已指定 `tcp`，則需要接收到的 ACK，探查才會成功。 如果已指定 `http`，則需要來自指定 URI 的「200 確定」回應，探查才會成功。|
| `path`              | `string` | 用來從 VM 要求健康情況狀態的 URI。 如果將 `protocol` 設為 `http`，則需要 `path`。 否則不允許。<br /><br /> 沒有預設值。|
| `port`              | `integer` | 選用。 用來與探查通訊的連接埠。 這對於任何端點都是選擇性的，因為接著會針對探查使用相同的連接埠。 您也可以針對其探查設定不同的通訊埠。 可能的值範圍從 1 到 65535 (含)。<br /><br /> 預設值是由端點設定。|
| `intervalInSeconds` | `integer` | 選用。 探查端點健康情況狀態的頻率間隔 (以秒為單位)。 一般而言，間隔略小於配置逾時期間 (以秒為單位) 的一半，可允許在執行個體退出循環之前進行兩個完整探查。<br /><br /> 預設值是 15，最小值是 5。|
| `timeoutInSeconds`  | `integer` | 選用。 逾時期間 (以秒為單位) 會套用至若無回應將會導致停止更多流量傳遞至端點的探查。 此值可以讓端點退出循環的速度比在 Azure 中使用的一般時間 (此為預設值) 更快或更慢。<br /><br /> 預設值是 31，最小值是 11。|

## <a name="see-also"></a>另請參閱
[雲端服務 (傳統) 定義結構描述](schema-csdef-file.md)