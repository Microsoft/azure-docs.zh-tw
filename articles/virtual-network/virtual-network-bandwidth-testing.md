---
title: 測試 Azure VM 網路輸送量 | Microsoft Docs
description: 了解如何測試 Azure 虛擬機器網路輸送量。
services: virtual-network
documentationcenter: na
author: steveesp
manager: Gerald DeGrace
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/21/2017
ms.author: steveesp
ms.openlocfilehash: d65b86cc63a4fd39824a6421afd5ce9abb7fd270
ms.sourcegitcommit: ded74961ef7d1df2ef8ffbcd13eeea0f4aaa3219
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/29/2018
ms.locfileid: "28200974"
---
# <a name="bandwidththroughput-testing-ntttcp"></a>頻寬/輸送量測試 (NTTTCP)

在 Azure 中測試網路輸送量效能時，最好是使用工具來針對網路進行測試，而儘量不使用其他可能影響效能的資源。 建議使用 NTTTCP。

請將工具複製到兩個大小相同的 Azure VM。 一個 VM 作為「傳送端」，另一個作為「接收端」。

#### <a name="deploying-vms-for-testing"></a>部署用於測試的 VM
基於這項測試的目的，兩個 VM 應該位於相同的「雲端服務」或相同的「可用性設定組」中，以便我們使用其內部 IP 並將「負載平衡器」從測試中排除。 您可以使用 VIP 進行測試，但這類型的測試不在本文件的涵蓋範圍內。
 
記下「傳送端」的 IP 位址。 讓我將該 IP 稱為 "a.b.c.r"

記下該 VM 上的核心數目。 讓我們將此稱為 "\#num\_cores"
 
在傳送端 VM 和接收端 VM 上執行長達 300 秒 (或 5 分鐘) 的 NTTTCP 測試。

提示：第一次設定此測試時，您可以嘗試較短的測試期間以更快獲得回饋。 在工具如預期般運作之後，請將測試期間延長到 300 秒以獲得最精確的結果。

> [!NOTE]
> 傳送端**和**接收端必須指定**相同的**測試持續時間參數 (-t)。

測試單一 TCP 串流 10 秒：

接收端參數：ntttcp -r -t 10 -P 1

傳送端參數：ntttcp -s10.27.33.7 -t 10 -n 1 -P 1

> [!NOTE]
> 上述範例應該僅用來確認您的組態。 本文稍後會提供有效的測試範例。

## <a name="testing-vms-running-windows"></a>測試執行 Windows 的 VM：

#### <a name="get-ntttcp-onto-the-vms"></a>請將 NTTTCP 放到 VM 上。

下載最新版本：<https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769>

或搜尋它 (如果已移動)：<https://www.bing.com/search?q=ntttcp+download>\< -- 應該是第一個點選項目

請考慮將 NTTTCP 放在個別的資料夾，例如 c:\\tools

#### <a name="allow-ntttcp-through-the-windows-firewall"></a>允許 NTTTCP 通過 Windows 防火牆
在「接收端」的 Windows 防火牆上建立一個「允許」規則，以允許 NTTTCP 流量到達。 最簡單的方式是依名稱允許整個 NTTTCP 程式，而不是允許特定的輸入 TCP 連接埠。

請依照下列方式允許 NTTTCP 通過 Windows 防火牆：

netsh advfirewall firewall add rule program=\<PATH\>\\ntttcp.exe name="ntttcp" protocol=any dir=in action=allow enable=yes profile=ANY

例如，如果您將 ntttcp.exe 複製到 "c:\\tools" 資料夾，則命令會是這樣： 

netsh advfirewall firewall add rule program=c:\\tools\\ntttcp.exe name="ntttcp" protocol=any dir=in action=allow enable=yes profile=ANY

#### <a name="running-ntttcp-tests"></a>執行 NTTTCP 測試

啟動「接收端」上的 NTTTCP (**從 CMD 執行**，不是從 PowerShell)：

ntttcp -r –m [2\*\#num\_cores],\*,a.b.c.r -t 300

如果 VM 有四個核心且 IP 位址為 10.0.0.4，就會看起來像這樣：

ntttcp -r –m 8,\*,10.0.0.4 -t 300


啟動「傳送端」上的 NTTTCP (**從 CMD 執行**，不是從 PowerShell)：

ntttcp -s –m 8,\*,10.0.0.4 -t 300 

等候結果。


## <a name="testing-vms-running-linux"></a>測試執行 Linux 的 VM：

請使用 nttcp-for-linux。 可從 <https://github.com/Microsoft/ntttcp-for-linux> 取得

在 Linux VM (「傳送端」和「接收端」兩者) 上，執行下列命令以在 VM 上準備 ntttcp-for-linux：

CentOS - 安裝 Git：
``` bash
  yum install gcc -y  
  yum install git -y
```
Ubuntu - 安裝 Git：
``` bash
 apt-get -y install build-essential  
 apt-get -y install git
```
在兩部機器上都建立並安裝：
``` bash
 git clone https://github.com/Microsoft/ntttcp-for-linux
 cd ntttcp-for-linux/src
 make && make install
```

與 Windows 範例中相同，我們假設 Linux「接收端」的 IP 為 10.0.0.4

啟動「接收端」上的 NTTTCP-for-Linux：

``` bash
ntttcp -r -t 300
```

然後在「傳送端」上執行：

``` bash
ntttcp -s10.0.0.4 -t 300
```
 
如果未提供時間參數，則測試時間長度預設為 60 秒

## <a name="testing-between-vms-running-windows-and-linux"></a>在執行 Windows 和 LINUX 的 VM 之間進行測試：

在此案例中，我們應該啟用非同步模式，才能執行測試。 若要執行上述動作，請針對 Linux 使用 **-N 旗標**；針對 Windows 使用 **-ns 旗標**。

#### <a name="from-linux-to-windows"></a>從 Linux 到 Windows：

接收者 <Windows>：

``` bash
ntttcp -r -m <2 x nr cores>,*,<Windows server IP>
```

傳送者 <Linux>：

``` bash
ntttcp -s -m <2 x nr cores>,*,<Windows server IP> -N -t 300
```

#### <a name="from-windows-to-linux"></a>從 Windows 到 Linux：

接收者 <Linux>：

``` bash
ntttcp -r -m <2 x nr cores>,*,<Linux server IP>
```

傳送者 <Windows>：

``` bash
ntttcp -s -m <2 x nr cores>,*,<Linux  server IP> -ns -t 300
```
## <a name="testing-cloud-service-instances"></a>測試雲端服務執行個體：
您必須將以下區段新增到 ServiceDefinition.csdef 中
```xml
<Endpoints>
  <InternalEndpoint name="Endpoint3" protocol="any" />
</Endpoints> 
```

## <a name="next-steps"></a>後續步驟
* 視結果而定，可能有將您案例的[網路輸送量機器最佳化](virtual-network-optimize-network-bandwidth.md)的空間。
* 了解[頻寬配置給虛擬機器] (virtual-machine-network-throughput.md)的方式
* 深入了解 [Azure 虛擬網路常見問題集 (FAQ)](virtual-networks-faq.md)
