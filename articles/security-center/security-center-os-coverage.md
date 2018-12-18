---
title: Azure 資訊安全中心支援的平台 | Microsoft Docs
description: 本文件提供的清單列出 Azure 資訊安全中心支援的 Windows 與 Linux 作業系統。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 70c076ef-3ad4-4000-a0c1-0ac0c9796ff1
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/26/2018
ms.author: terrylan
ms.openlocfilehash: b82b152ae4f44e475b5c3e9bda9d604f405503e0
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123726"
---
# <a name="supported-platforms-in-azure-security-center"></a>Azure 資訊安全中心支援的平台
針對使用傳統與 Resource Manager 部署模型建立的虛擬機器 (VM) 與電腦，提供安全性狀態監視和建議。

> [!NOTE]
> 深入了解 Azure 資源的[傳統和 Resource Manager 部署模型](../azure-classic-rm.md)。
>
>

## <a name="supported-platforms-for-windows-computers-and-vms"></a>Windows 電腦和 VM 支援的平台
支援的 Windows 作業系統：

* Windows Server 2008
* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016


## <a name="supported-platforms-for-linux-computers-and-vms"></a>Linux 電腦和 VM 支援的平台
支援的 Linux 作業系統：

* Ubuntu 版本 12.04 LTS、14.04 LTS、16.04 LTS
* Debian 版本 6、7、8、9
* CentOS 版本 5、6、7
* Red Hat Enterprise Linux (RHEL) 版本 5、6、7
* SUSE Linux Enterprise Server (SLES) 版本 11、12
* Oracle Linux 版本 5、6、7
* Amazon Linux 2012.09 到 2017
* 只有 x86_64 平台 (64 位元) 支援 Openssl 1.1.0

> [!NOTE]
> 尚未提供 Linux 作業系統的虛擬機器行為分析。
>
>

## <a name="vms-and-cloud-services"></a>VM 和雲端服務
也支援雲端服務中執行的 VM。 只監視生產位置中執行的雲端服務 Web 角色和背景工作角色。 若要深入了解雲端服務，請參閱[雲端服務概觀](../cloud-services/cloud-services-choose-me.md)。

## <a name="next-steps"></a>後續步驟

- [Azure 資訊安全中心規劃和操作指南](security-center-planning-and-operations-guide.md) - 了解如何規劃及了解採用 Azure 資訊安全中心的設計考量
- [Azure 資訊安全中心不同類型的安全性警示](security-center-alerts-type.md#virtual-machine-behavioral-analysis) - 深入了解資訊安全中心中的虛擬機器行為分析和損毀傾印記憶體分析
- [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
- [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) - 尋找有關 Azure 安全性與合規性的部落格文章
