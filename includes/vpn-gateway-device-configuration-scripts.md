---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/30/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 00889717e0c22477b9933725bccc7de05c82bc4f
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47454592"
---
| **廠商** | **裝置系列** | **韌體版本** |
| --- | --- | --- |
|Cisco | ISR| IOS 15.1 (預覽)|
|Cisco | ASA | 適用於版本低於 9.8 之 ASA 的 ASA (*) RouteBased (IKEv2 - 沒有 BGP) |
|Cisco | ASA | 適用於 9.8 版以上之 ASA 的 ASA RouteBased (IKEv2 - 沒有 BGP) |
|Juniper | SRX_GA | 12.x|
|Juniper | SSG_GA | ScreenOS 6.2.x|
|Juniper | JSeries_GA | JunOS 12.x|
|Ubiquiti| EdgeRouter| EdgeOS v1.10x RouteBased VTI|
|Ubiquiti| EdgeRouter| EdgeOS v1.10x RouteBased BGP|

> [!NOTE]
> (*) 必要：NarrowAzureTrafficSelectors (啟用 UsePolicyBasedTrafficSelectors 選項) 和 CustomAzurePolicies (IKE/IPsec)
>
