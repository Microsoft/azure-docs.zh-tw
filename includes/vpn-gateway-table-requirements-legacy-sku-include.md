---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: d8091fdade9cd417af58755d8245c2fb091b86b3
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30197168"
---
下表列出 PolicyBased 和 RouteBased VPN 閘道的需求。 此資料表適用於資源管理員與傳統部署模型。 就傳統模型而言，PolicyBased VPN 閘道與「靜態」閘道相同，而 RouteBased 閘道則與「動態」閘道相同。

|  | **PolicyBased 基本 VPN 閘道** | **RouteBased 基本 VPN 閘道** | **RouteBased 標準 VPN 閘道** | **RouteBased 高效能 VPN 閘道** |
| --- | --- | --- | --- | --- |
| **站對站連線能力 (S2S)** |PolicyBased VPN 組態 |RouteBased VPN 組態 |RouteBased VPN 組態 |RouteBased VPN 組態 |
| **點對站連線 (P2S)** |不支援 |支援 (可與 S2S 並存) |支援 (可與 S2S 並存) |支援 (可與 S2S 並存) |
| **驗證方法** |預先共用金鑰 |S2S 連線的預先共用金鑰，P2S 連線的憑證 |S2S 連線的預先共用金鑰，P2S 連線的憑證 |S2S 連線的預先共用金鑰，P2S 連線的憑證 |
| **S2S 連接的數目上限** |1 |10 |10 |30 |
| **P2S 連接的數目上限** |不支援 |128 |128 |128 |
| **作用中路由支援 (BGP)** |不支援 |不支援 |支援 |支援 |
