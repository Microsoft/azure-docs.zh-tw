---
title: 包含檔案
description: 包含檔案
services: firewall
author: vhorne
ms.service: firewall
ms.topic: include
ms.date: 7/30/2018
ms.author: victorh
ms.custom: include file
ms.openlocfilehash: 0a4a66543b323b1dbe56158851dafcb1dfe695f0
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47186048"
---
| 資源 | 預設限制 |
| --- | --- |
| 已處理的資料量 |1000 TB/防火牆/月 <sup>1</sup> |
|規則|1 萬個應用程式規則、1 萬個網路規則|
|VNet 對等互連|針對中樞和輪輻實作，上限為 50 個輪輻 VNET。|
|全域對等互連|不支援。 您應該在每個區域都至少部署一個防火牆。|
|單一網路規則中的最大連接埠|15<br>連接埠範圍 (例如：2 - 10) 會計算為兩個。


<sup>1</sup> 如需增加這些限制，請連絡 Azure 支援中心。
