---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: shandilvarun
ms.service: virtual-machines
ms.topic: include
ms.date: 08/09/2018
ms.author: vashan, cynthn, rajsqr
ms.custom: include file
ms.openlocfilehash: 603e7c3a0c30eb42cb75d6a6ff87a96d847b7c9f
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "40100729"
---
Azure 虛擬機器 (VM) 會經歷不同狀態，這些狀態可以分成「佈建」和「電源」狀態。 本文的目的是要說明這些狀態，並具體指出何時會向客戶收取執行個體使用量費用。 

## <a name="power-states"></a>電源狀態

電源狀態代表 VM 的最後一個已知狀態。

![VM 電源狀態圖](./media/virtual-machines-common-states-lifecycle/vm-power-states.png)

<br>
下表提供每個執行個體狀態的描述，並指出是否會收取其執行個體使用量費用。

<table>
<tr>
<th>
State
</th>
<th>
說明
</th>
<th>
執行個體使用量計費
</th>
</tr>
<tr>
<td>
<p><b>啟動中</b></p>
</td>
<td>
<p>VM 正在啟動。</p>
<code>"statuses": [<br>
   {<br>
      "code": "PowerState/starting",<br>
       "level": "Info",<br>
        "displayStatus": "VM starting"<br>
    }<br>
    ]</code><br>
</td>
<td>
<p><b>不會計費</b></p>
</td>
</tr>
<tr>
<td>
<p><b>執行中</b></p>
</td>
<td>
<p>VM 的正常工作狀態</p>
<code>"statuses": [<br>
 {<br>
 "code": "PowerState/running",<br>
 "level": "Info",<br>
 "displayStatus": "VM running"<br>
 }<br>
 ]</code><br>
</td>
<td>
<p><b>會計費</b></p>
</td>
</tr>
<tr>
<td>
<p><b>停止中</b></p>
</td>
<td>
<p>這是過渡狀態。 完成時，便會顯示為**已停止**。</p>
<code>"statuses": [<br>
 {<br>
 "code": "PowerState/stopping",<br>
 "level": "Info",<br>
 "displayStatus": "VM stopping"<br>
 }<br>
 ]</code><br>
</td>
<td>
<p><b>會計費</b></p>
</td>
</tr>
<tr>
<td>
<p><b>已停止</b></p>
</td>
<td>
<p>已從客體 OS 內或藉由使用 PowerOff API 關閉 VM。</p>
<p>硬體仍配置給 VM，且會留在主機上。 </p>
<code>"statuses": [<br>
 {<br>
 "code": "PowerState/stopped",<br>
 "level": "Info",<br>
 "displayStatus": "VM stopped"<br>
 }<br>
 ]</code><br>
</td>
<td>
<p><b>不會計費&#42;</b></p>
</td>
</tr>
<tr>
<td>
<p><b>正在解除配置</b></p>
</td>
<td>
<p>過渡狀態。 完成時，VM 會顯示為**已解除配置**。</p>
<code>"statuses": [<br>
 {<br>
 "code": "PowerState/deallocating",<br>
 "level": "Info",<br>
 "displayStatus": "VM deallocating"<br>
 }<br>
 ]</code><br>
</td>
<td>
<p><b>不會計費&#42;</b></p>
</td>
</tr>
<tr>
<td>
<p><b>已解除配置</b></p>
</td>
<td>
<p>VM 已成功停止，並從主機中移除。 </p>
<code>"statuses": [<br>
 {<br>
 "code": "PowerState/deallocated",<br>
 "level": "Info",<br>
 "displayStatus": "VM deallocated"<br>
 }<br>
 ]</code><br>
</td>
<td>
<p><b>不會計費</b></p>
</td>
</tr>
</tbody>
</table>


&#42;某些 Azure 資源 (例如，磁碟和網路) 會產生費用，而無論執行個體的狀態為何。 

## <a name="provisioning-states"></a>佈建狀態

佈建狀態是使用者在 VM 上所起始的控制平面作業狀態。 這些狀態與 VM 的電源狀態不同。

- **建立** – 建立 VM。

- **更新** – 更新現有 VM 的模型。 對 VM 所做的某些非模型變更 (例如，啟動/重新啟動) 也屬於更新。

- **刪除** – 刪除 VM。

- **解除配置** – 此狀態會停止 VM 並將其從主機中移除。 解除配置 VM 可視為更新，所以會顯示與更新相關的佈建狀態。



平台接受使用者所起始的動作後，會有以下過渡作業狀態：

<br>

<table>
<tbody>
<tr>
<td width="162">
<p><b>狀態</b></p>
</td>
<td width="366">
<p>說明</p>
</td>
</tr>
<tr>
<td width="162">
<p><b>建立中</b></p>
</td>
<td width="366">
<code>"statuses": [<br>
 {<br>
 "code": "ProvisioningState/creating",<br>
 "level": "Info",<br>
 "displayStatus": "Creating"<br>
 }</code><br>
</td>
</tr>
<tr>
<td width="162">
<p><b>更新中</b></p>
</td>
<td width="366">
<code>"statuses": [<br>
 {<br>
 "code": "ProvisioningState/updating",<br>
 "level": "Info",<br>
 "displayStatus": "Updating"<br>
 }<br>
 ]</code><br>
</td>
</tr>
<tr>
<td width="162">
<p><b>刪除中</b></p>
</td>
<td width="366">
<code>"statuses": [<br>
 {<br>
 "code": "ProvisioningState/deleting",<br>
 "level": "Info",<br>
 "displayStatus": "Deleting"<br>
 }<br>
 ]</code><br>
</td>
</tr>
<tr>
<td width="162">
<p><b>OS 佈建狀態</b></p>
</td>
<td width="366">
<p>如果使用 OS 映像而非特製化映像來建立 VM，則會觀察到下列子狀態：</p>
<p>1. <b>OSProvisioningInprogress</b> &ndash; VM 正在執行，且客體 OS 的安裝正在進行中。 <p /> 
<code> "statuses": [<br>
 {<br>
 "code": "ProvisioningState/creating/OSProvisioningInprogress",<br>
 "level": "Info",<br>
 "displayStatus": "OS Provisioning In progress"<br>
 }<br>
]</code><br>
<p>2. <b>OSProvisioningComplete</b> &ndash; 短期狀態。 除非需要安裝任何擴充功能，否則 VM 會快速過渡為**成功**狀態。 安裝擴充功能需要一些時間。 <br />
<code> "statuses": [<br>
 {<br>
 "code": "ProvisioningState/creating/OSProvisioningComplete",<br>
 "level": "Info",<br>
 "displayStatus": "OS Provisioning Complete"<br>
 }<br>
]</code><br>
<p><b>附註</b>：如果發生 OS 失敗或 OS 未及時安裝，則 OS 佈建會過渡為**失敗**狀態。 客戶必須就基礎結構上已部署的 VM 支付費用。</p>
</td>
</tr>
</table>


作業完成後，VM 會過渡為下列其中一個狀態：

- **成功** – 使用者起始的動作已完成。

    ```
 "statuses": [ 
 {
     "code": "ProvisioningState/succeeded",
     "level": "Info",
     "displayStatus": "Provisioning succeeded",
     "time": "time"
 }
 ]
    ```

 

- **失敗** – 代表失敗的作業。 請參閱錯誤碼，以取得詳細資訊和可能的解決方案。

    ```
 "statuses": [
    {
      "code": "ProvisioningState/failed/InternalOperationError",
      "level": "Error",
      "displayStatus": "Provisioning failed",
      "message": "Operation abandoned due to internal error. Please try again later.",
      "time": "time"
    }
    ]
    ```



## <a name="vm-instance-view"></a>VM 執行個體檢視

執行個體檢視 API 會提供 VM 的執行狀態資訊。 如需詳細資訊，請參閱[虛擬機器 - 執行個體檢視](https://docs.microsoft.com/rest/api/compute/virtualmachines/instanceview) API。

Azure 資源總管會提供一個簡單的 UI 供您檢視 VM 的執行中狀態：[資源總管] (https://resources.azure.com/)。

佈建狀態可於 VM 屬性和執行個體檢視中看到。 電源狀態可於 VM 的執行個體檢視中看到。 

