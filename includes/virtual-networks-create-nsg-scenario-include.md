---
title: 包含檔案
description: 包含檔案
services: virtual-network
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 04/13/2018
ms.author: genli
ms.custom: include file
ms.openlocfilehash: 873549442284ede2e9f020bd90879f721b9c1a18
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38760349"
---
## <a name="scenario"></a>案例
為了更清楚說明如何建立 NSG，此文件會使用下列案例：

![VNet 案例](./media/virtual-networks-create-nsg-scenario-include/figure1.png)

在這個案例中，您會在 **TestVNet** 虛擬網路的每個子網路中建立 NSG ，如下所述： 

* **NSG-FrontEnd**。 前端 NSG 會套用到「前端」子網路，且包含兩個規則：    
  * **rdp-rule**。 允許 RDP 流量流向「前端」子網路。
  * **web-rule**。 允許 HTTP 流量流向「前端」子網路。
* **NSG-BackEnd**。 後端 NSG 會套用到「後端」子網路，且包含兩個規則：    
  * **sql-rule**。 僅允許 SQL 流量來自「前端」子網路。
  * **web-rule**。 拒絕來自「後端」子網路的所有網際網路繫結流量。

這些規則的組合會建立類似 DMZ 的案例，其後端子網路只能接收來自前端子網路 SQL 的傳入流量，且無網際網路的存取權；而前端子網路則可與網際網路通訊，並接收傳入的 HTTP 要求。

