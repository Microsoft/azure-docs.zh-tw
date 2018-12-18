---
title: Azure Stack 開發套件架構 | Microsoft Docs
description: 說明 Azure Stack 開發套件 (ASDK) 架構。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 21c54e2e996bb987f7a27ac3e6333df6f74d6f4b
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49338619"
---
# <a name="microsoft-azure-stack-development-kit-architecture"></a>Microsoft Azure Stack 開發套件架構
Azure Stack 開發套件 (ASDK) 是 Azure Stack 的單一節點部署。 所有元件都安裝在於單一主機電腦上執行的虛擬機器中。 

## <a name="logical-architecture-diagram"></a>邏輯架構圖
下圖說明 ASDK 及其元件的邏輯架構。

![ASDK 架構](media/asdk-architecture/image1.png)

## <a name="virtual-machine-roles"></a>虛擬機器角色
ASDK 會使用開發套件主機電腦上所裝載的下列 VM 來提供服務：

| 名稱 | 說明 |
| ----- | ----- |
| **AzS-ACS01** | Azure Stack 儲存體服務。|
| **AzS-ADFS01** | Active Directory 同盟服務 (AD FS)。  |
| **AzS-BGPNAT01** | 邊緣路由器，並提供適用於 Azure Stack 的 NAT 和 VPN 功能。 |
| **AzS-CA01** | 適用於 Azure Stack 角色服務的憑證授權單位服務。|
| **AzS-DC01** | 適用於 Microsoft Azure Stack 的 Active Directory、DNS 及 DHCP 服務。|
| **AzS-ERCS01** | 緊急修復主控台 VM。 |
| **AzS-GWY01** | 邊緣閘道服務，例如適用於租用戶網路的 VPN 站對站連線。|
| **AzS-NC01** | 管理 Azure Stack 網路服務的網路控制器。  |
| **AzS-SLB01** | Azure Stack 中適用於租用戶和 Azure Stack 基礎結構服務的負載平衡多工器服務。  |
| **AzS-SQL01** | 適用於 Azure Stack 基礎結構角色的內部資料存放區。  |
| **AzS-WAS01** | Azure Stack 管理入口網站和 Azure Resource Manager 服務。|
| **AzS-WASP01**| Azure Stack 使用者 (租用戶) 入口網站和 Azure Resource Manager 服務。|
| **AzS-XRP01** | 適用於 Microsoft Azure Stack 的基礎結構管理控制器，包括運算、網路和儲存體資源提供者。|


## <a name="next-steps"></a>後續步驟
[了解基本 ASDK 管理工作](asdk-admin-basics.md)
