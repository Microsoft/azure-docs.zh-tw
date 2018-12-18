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
ms.openlocfilehash: 5de227e5de5ef9b41f6e0f64db86b7195259f7d6
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "30196758"
---
* **PolicyBased︰** 原則式 VPN 先前在傳統部署模型內稱為靜態路由閘道。 原則式 VPN 會根據使用內部部署網路與 Azure VNet 之間的位址首碼組合所設定的 IPsec 原則，透過 IPsec 通道加密和導向封包。 原則 (或流量選取器) 通常定義為 VPN 裝置組態中的存取清單。 原則式 VPN 類型的值是 PolicyBased 。 使用 PolicyBased VPN，請記住下列限制︰
  
  * PolicyBased VPN「只有」  在「基本」閘道 SKU 上才能使用。 這個 VPN 類型與其他閘道 SKU 不相容。
  * 使用 PolicyBased VPN 時，您只能有 1 個通道。
  * 您只能將 PolicyBased VPN 用於 S2S 連線，而且僅限用於特定組態。 大多數「VPN 閘道」組態都需要一個 RouteBased VPN。
* **RouteBased︰** 路由式 VPN 先前在傳統部署模型內稱為動態路由閘道。 路由式 Vpn 會使用 IP 轉送或路由表中的「路由」，直接封包至其對應的通道介面。 然後，通道介面會加密或解密輸入和輸出通道的封包。 路由式 VPN 的原則或流量選取器會設定為任何對任何 (或萬用字元)。 路由式 VPN 類型的值是 RouteBased 。
