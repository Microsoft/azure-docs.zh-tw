---
title: 原則範本範例 | Microsoft Docs
description: 虛擬網路的 Azure 原則範本範例。
services: virtual-network
documentationcenter: ''
author: jimdial
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: ''
ms.date: 05/02/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: 2b8766a5353b015030872176e9032034afb7cb9d
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32779531"
---
# <a name="azure-policy-sample-templates-for-virtual-network"></a>虛擬網路的 Azure 原則範例範本

下表包含 [Azure 原則](../azure-policy/azure-policy-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)範本範例的連結。 這些範例位於 [Azure 原則範例存放庫](https://github.com/Azure/azure-policy)。

| | |
|---|---|
|**網路**||
| [對每個 NIC 使用 NSG X](../azure-policy/scripts/nsg-on-nic.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求搭配每個虛擬網路介面使用特定的網路安全性群組。 您需指定要使用之網路安全性群組的識別碼。 |
| [對每個子網路使用 NSG X](../azure-policy/scripts/nsg-on-subnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求搭配每個虛擬子網路使用特定的網路安全性群組。 您需指定要使用之網路安全性群組的識別碼。 |
| [沒有路由表](../azure-policy/scripts/no-user-def-route-table.md?toc=%2fazure%2fvirtual-network%2ftoc.json)  |禁止使用路由表來部署虛擬網路。 |
| [針對 VM 網路介面使用已核准的子網路](../azure-policy/scripts/use-approved-subnet-vm-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求網路介面使用已核准的子網路。 您需指定已核准之子網路的識別碼。 |
| [針對 VM 網路介面使用已核准的 vNet](../azure-policy/scripts/use-approved-vnet-vm-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求網路介面使用已核准的虛擬網路。 您需指定已核准之虛擬網路的識別碼。 |
|**監視**||
| [稽核診斷設定](../azure-policy/scripts/audit-diag-setting.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 稽核是否未針對指定的資源類型啟用診斷設定。 您需指定資源類型陣列，以檢查是否已啟用診斷設定。 |
|**名稱和文字慣例**||
| [允許多個名稱模式](../azure-policy/scripts/allow-multiple-name-patterns.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 允許多個名稱模式的其中一個用於資源。 |
| [需要類似模式](../azure-policy/scripts/enforce-like-pattern.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 請確定資源名稱符合某個模式的「類似」條件。 |
| [需要比對模式](../azure-policy/scripts/enforce-match-pattern.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 請確定資源名稱符合指定命名模式。 |
| [需要標籤比對模式](../azure-policy/scripts/enforce-tag-match-pattern.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 請確認標籤值符合文字模式。 |
|**標記**||
| [計費標籤原則計畫](../azure-policy/scripts/billing-tags-policy-init.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求針對成本中心和產品名稱使用指定的標籤值。 使用內建的原則來套用及強制執行必要的標籤。 您需為標籤指定必要的值。  |
| [對資源群組強制執行標籤及其值](../azure-policy/scripts/enforce-tag-rg.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求對資源群組使用標籤和值。 您需指定必要的標籤名稱和值。  |
| [強制執行標籤及其值](../azure-policy/scripts/enforce-tag-val.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求使用指定的標籤名稱和值。 您需指定要強制執行的標籤名稱和值。  |
| [套用標籤及其預設值](../azure-policy/scripts/apply-tag-def-val.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 附加指定的標籤名稱和值 (如果未提供該標籤)。 您需指定要套用的標籤名稱和值。  |
|**一般**||
| [允許的位置](../azure-policy/scripts/allowed-locs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 要求將所有資源都部署到已核准的位置。 您需指定已核准的位置陣列。  |
| [允許的資源類型](../azure-policy/scripts/allowed-res-types.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 確保只部署已核准的資源類型。 您需指定允許的資源類型陣列。  |
| [不允許的資源類型](../azure-policy/scripts/not-allowed-res-type.md?toc=%2fazure%2fvirtual-network%2ftoc.json) | 禁止部署指定的資源類型。 您需指定要封鎖的資源類型陣列。  |