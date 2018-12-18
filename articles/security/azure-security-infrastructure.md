---
title: Azure 基礎結構安全性 | Microsoft Docs
description: 本文描述 Microsoft 如何保護 Azure 資料中心的安全性。
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/06/2018
ms.author: terrylan
ms.openlocfilehash: 9385c6ea8d2a04e9a8595a22e6e2ff7638394b41
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39172967"
---
# <a name="azure-infrastructure-security"></a>Azure 基礎結構安全性
Microsoft Azure 是在 Microsoft 管理並操作的資料中心執行。 這些分散各地的資料中心在安全性與可靠性方面皆符合主要的業界標準，例如 ISO/IEC 27001:2013 和 NIST SP 800-53。 這些資料中心是由 Microsoft 作業人員管理、監視及進行系統管理。 這些作業人員在全天候持續提供世界上最大型線上服務方面擁有多年的經驗。

本系列文章提供 Microsoft 為保護 Azure 基礎結構所做措施的相關資訊。 這些文章說明：

- [實體安全性](azure-physical-security.md)
- [可用性](azure-infrastructure-availability.md)
- [元件和界線](azure-infrastructure-components.md)
- [網路架構](azure-infrastructure-network.md)
- [生產網路](azure-production-network.md)
- [SQL Database](azure-infrastructure-sql.md)
- [作業](azure-infrastructure-operations.md)
- [監視](azure-infrastructure-monitoring.md)
- [完整性](azure-infrastructure-integrity.md)
- [資料保護](azure-protection-of-customer-data.md)

## <a name="shared-responsibility-model"></a>共同責任模式
了解您與 Microsoft 之間的責任劃分相當重要。 在內部部署環境中，您擁有整個堆疊，但是當您移到雲端時，部分責任就會轉移給 Microsoft。 下圖說明依據堆疊部署類型 (軟體即服務 [SaaS]、平台即服務 [PaaS]、基礎結構即服務 [IaaS] 與內部部署) 區分的責任區域。

![顯示責任的圖表][1]

下列項目一律由您負責 (不論部署類型)：

- 資料
- 端點
- 帳戶
- 存取管理

請務必了解您與 Microsoft 之間就 SaaS、PaaS 及 IaaS 部署方面的職責劃分。 如需詳細資訊，請參閱[雲端運算的共同責任](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91/file/153019/1/Shared%20responsibilities%20for%20cloud%20computing.pdf)。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為協助保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件和界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)

<!--Image references-->
[1]: ./media/azure-security-infrastructure/responsibility-zones.png
