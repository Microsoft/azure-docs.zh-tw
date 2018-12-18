---
title: 包含檔案
description: 包含檔案
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 05/17/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: d9c8a0e6a3bd6d79a11ee0d0dab0500a209e5571
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38942051"
---
您可以透過在子網路或 VM 網路介面上建立網路篩選，對 Azure 中的虛擬機器 (VM) 開啟連接埠或建立端點。 您可將控制輸入和輸出流量的這些篩選器放在可接收流量的資源所附加的網路安全性群組上。

讓我們使用連接埠 80 上的 Web 流量的常見範例。 一旦您的 VM 設定為在標準 TCP 連接埠 80 上為 Web 要求提供服務 (請記得啟動適當的服務，並且在 VM 上開啟任何作業系統防火牆規則)，您可以︰

1. 建立網路安全性群組。
2. 建立輸入規則允許具有下列項目的流量︰
   * 目的地連接埠範圍為 "80"
   * 來源連接埠範圍為 "*" (允許任何來源連接埠)
   * 優先順序值小於 65,500 (比預設全面涵蓋拒絕輸入規則的優先順序高)
3. 讓網路安全性群組與 VM 網路介面或子網路產生關聯。

您可以建立複雜的網路組態，以使用網路安全性群組和規則保護您的環境。 我們的範例僅使用一個或兩個規則，允許 HTTP 流量或遠端管理。 如需詳細資訊，請參閱下面的[「相關資訊」](#more-information-on-network-security-groups)一節，或[什麼是網路安全性群組？](../articles/virtual-network/security-overview.md)

