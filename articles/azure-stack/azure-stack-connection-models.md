---
title: Azure Stack 整合式系統連線模型 | Microsoft Docs
description: 決定多節點 Azure Stack 的部署規劃決策。
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
ms.reviewer: wfayed
ms.openlocfilehash: 7509d00815f56dc46bd276ffc67c4c607c54070a
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49338891"
---
# <a name="azure-stack-integrated-systems-connection-models"></a>Azure Stack 整合式系統連線模型
如果您對 Azure Stack 整合式系統有興趣，您將需要了解[數個資料中心整合考量](azure-stack-datacenter-integration.md)，如此 Azure Stack 部署才能判斷如何將系統融入您的資料中心。 此外，您必須確切地決定如何將 Azure Stack 整合到您的混合式雲端環境中。 本文提供這些主要決策 (包括 Azure 連線、身分識別儲存和計費模型決策) 的概觀。

如果您決定購買整合系統，原始設備製造商 (OEM) 硬體廠商可引導您更深入完成大部分規劃程序。 它們也會執行實際部署。

## <a name="choose-an-azure-stack-deployment-connection-model"></a>選擇 Azure Stack 部署連線模型
您可以選擇部署 Azure Stack 以連線網際網路 (和 Azure)，或者中斷連線。 若要發揮 Azure Stack 的最大功效，包括 Azure Stack 與 Azure 之間的混合式情節，您可能希望讓部署連線至 Azure。 此選擇會定義您身分識別儲存可用的選項 (Azure Active Directory 或 Active Directory 同盟服務) 和計費模型 (以使用時付費為基礎的計費或以容量為基礎的的計費)，如下圖和下表摘要說明： 

![Azure Stack 部署和計費案例](media/azure-stack-connection-models/azure-stack-scenarios.png)  
  
> [!IMPORTANT]
> 這是重要決策點！ 選擇 Active Directory 同盟服務 (AD FS) 或 Azure Active Directory (Azure AD) 是您必須在部署期間進行的一次性決策。 若未重新部署整個系統，之後便無法變更此選擇。  


|選項|已連線至 Azure|已中斷與 Azure 的連線|
|-----|-----|-----|
|Azure AD|![支援](media/azure-stack-connection-models/check.png)| |
|AD FS|![支援](media/azure-stack-connection-models/check.png)|![支援](media/azure-stack-connection-models/check.png)|
|以耗用量為基礎的計費|![支援](media/azure-stack-connection-models/check.png)| |
|以容量為基礎的計費|![支援](media/azure-stack-connection-models/check.png)|![支援](media/azure-stack-connection-models/check.png)|
|將更新套件直接下載到 Azure Stack|![支援](media/azure-stack-connection-models/check.png)|  |

在您決定要用於 Azure Stack 部署的 Azure 連線模型之後，必須針對身分識別儲存和計費方法進行其他連線相關決策。 

## <a name="next-steps"></a>後續步驟

[Azure 連線的 Azure Stack 部署決策](azure-stack-connected-deployment.md)

[Azure 中斷連線的 Azure Stack 部署決策](azure-stack-disconnected-deployment.md)
