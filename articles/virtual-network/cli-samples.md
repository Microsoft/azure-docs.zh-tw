---
title: 虛擬網路的 Azure CLI 範例 | Microsoft Docs
description: 虛擬網路的 Azure CLI 範例。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: jdial
ms.openlocfilehash: baa4dbc8f95e068eb1a939fdee53fb2a4ee8117f
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
---
# <a name="azure-cli-samples-for-virtual-network"></a>虛擬網路的 Azure CLI 範例

下表包含使用 Azure CLI 命令所建置的 Bash 指令碼連結：

| | |
|----|----|
| [建立多層式應用程式的虛擬網路](./scripts/virtual-network-cli-sample-multi-tier-application.md) | 建立具有前端和後端子網路的虛擬網路。 傳送到前端子網路的流量會限制為 HTTP 和 SSH，而傳送到後端子網路的流量則限制為 MySQL 且連接埠為 3306。 |
| [對等互連兩個虛擬網路](./scripts/virtual-network-cli-sample-peer-two-virtual-networks.md) | 在相同區域中建立和連線兩個虛擬網路。 |
| [透過網路虛擬設備路由傳送流量](./scripts/virtual-network-cli-sample-route-traffic-through-nva.md) | 建立具有前端和後端子網路的虛擬網路，以及可在兩個子網路間路由傳送流量的 VM。 |
| [篩選輸入和輸出 VM 網路流量](./scripts/virtual-network-cli-sample-filter-network-traffic.md) | 建立具有前端和後端子網路的虛擬網路。 傳輸至前端子網路的輸入網路流量限制為 HTTP、HTTPS 和 SSH。 不允許從後端子網路傳輸至網際網路的輸出流量。 |