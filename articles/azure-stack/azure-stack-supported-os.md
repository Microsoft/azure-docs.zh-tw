---
title: 適用於 Azure Stack 的支援客體作業系統 | Microsoft Docs
description: 可在 Azure Stack 上使用這些客體作業系統。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: be4d9b3ea7e5715d7c3a4df11b7e8bab4d1d4ca5
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405592"
---
# <a name="guest-operating-systems-supported-on-azure-stack"></a>Azure Stack 上的支援客體作業系統

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

## <a name="windows"></a>Windows

Azure Stack 支援下表所列的 Windows 客體作業系統：

| 作業系統 | 說明 | Marketplace 中有提供 |
| --- | --- | --- | --- | --- | --- |
| Windows Server 1709 版 | 64 位元 | 包含容器的 Core |
| Windows Server 2016 | 64 位元 |  Datacenter、Datacenter Core、包含容器的 Datacenter |
| Windows Server 2012 R2 | 64 位元 |  資料中心 |
| Windows Server 2012 | 64 位元 |  資料中心 |
| Windows Server 2008 R2 SP1 | 64 位元 |  資料中心 |
| Windows Server 2008 SP2 | 64 位元 |  自備映像 |
| Windows 10 (請參閱附註 1) | 64 位元、Pro 和 Enterprise | 自備映像 |

> [!NOTE]
> 若要在 Azure Stack 上部署 Windows 10 用戶端作業系統，您必須具備 [Windows 的每位使用者授權](https://www.microsoft.com/en-us/Licensing/product-licensing/windows10.aspx)，或者透過合格多租用戶主機服務提供者 ([QMTH](https://www.microsoft.com/en-us/CloudandHosting/licensing_sca.aspx)) 購買。

Marketplace 映像適用於隨用隨付或 BYOL (EA/SPLA) 授權。 不支援在單一 Azure Stack 執行個體上同時使用兩者。 在部署期間，Azure Stack 會將適當版本的客體代理程式插入映像中。

Marketplace 中有 Datacenter 版本可供下載；客戶可以自備包括其他版本在內的伺服器映像。 Marketplace 中沒有可用的 Windows 用戶端映像。

## <a name="linux"></a>Linux

列為 Marketplace 中可用的 Linux 發行版本包含必要的 Windows Azure Linux 代理程式 (WALA)。 如果您自備映像到 Azure Stack，請遵循[將 Linux 映像新增到 Azure Stack](azure-stack-linux.md) 中的方針。

> [!NOTE]
> 請使用最新公開的 WALA 版本來建置自訂映像。 2.2.18 以前的版本可能無法在 Azure Stack 上正常運作。
>
> Azure Stack 目前不支援 [cloud-init](https://cloud-init.io/)。

| 配送映像 | 說明 | 發行者 | Marketplace |
| --- | --- | --- | --- | --- | --- |
| CentOS 型 6.9 | 64 位元 | Rogue Wave | 是 |
| CentOS 型 7.4 | 64 位元 | Rogue Wave | 是 |
| ClearLinux | 64 位元 | ClearLinux.org | 是 |
| 容器 Linux |  64 位元 | CoreOS | Stable |
| Debian 8 "Jessie" | 64 位元 | credativ |  是 |
| Debian 9 "Stretch" | 64 位元 | credativ | 是 |
| Red Hat Enterprise Linux 7.x | 64 位元 | Red Hat |自備映像 |
| SLES 11SP4 | 64 位元 | SUSE | 是 |
| SLES 12SP3 | 64 位元 | SUSE | 是 |
| Ubuntu 14.04-LTS | 64 位元 | Canonical | 是 |
| Ubuntu 16.04-LTS | 64 位元 | Canonical | 是 |
| Ubuntu 18.04-LTS | 64 位元 | Canonical | 是 |

如需 Red Hat Enterprise Linux 支援資訊，請參閱 [Red Hat and Azure Stack: Frequently Asked Questions](https://access.redhat.com/articles/3413531) (Red Hat 和 Azure Stack：常見問題集)。

## <a name="next-steps"></a>後續步驟

如需關於 Azure Stack Marketplace 的詳細資訊，請參閱下列文章：

[下載 Marketplace 項目](azure-stack-download-azure-marketplace-item.md)  
[建立及發佈 Marketplace 項目](azure-stack-create-and-publish-marketplace-item.md)