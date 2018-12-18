---
title: 什麼是 Azure Stack？ | Microsoft Docs
description: Azure Stack 可讓您在資料中心內執行 Azure 服務。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: d9e6aee1-4cba-4df5-b5a3-6f38da9627a3
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: unknown
ms.custom: mvc
ms.openlocfilehash: b2b81938eafe104369e52e72f9958e2adf2cca6f
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49345215"
---
# <a name="what-is-azure-stack"></a>什麼是 Azure Stack？

Microsoft Azure Stack 是一個混合式雲端平台，可讓您在資料中心提供 Azure 服務。 這個平台是設計來支援不斷成長的商務需求。 Azure Stack 可為現代化應用程式 (例如，邊緣和中斷連線的環境) 實現新案例，也可以符合特定的安全性和合規性需求。

Azure Stack 是透過兩種部署選項來提供的，用以滿足您的需求。

## <a name="azure-stack-integrated-systems"></a>Azure Stack 整合系統
Azure Stack 整合系統是透過 Microsoft 與[硬體合作夥伴](https://azure.microsoft.com/overview/azure-stack/integrated-systems/)的合作來提供的，可建立既提供雲端步調的創新又兼顧計算管理簡易性的解決方案。 由於是以整合的軟硬體系統形式來提供 Azure Stack，因此您可以獲得所需的彈性和控制力，以及從雲端創新的能力。 Azure Stack 整合系統的大小範圍為 4 到 12 個節點，並且由硬體合作夥伴與 Microsoft 共同支援。  請使用 Azure Stack 整合系統，來為生產環境工作負載建立新案例和部署新解決方案。

## <a name="azure-stack-development-kit"></a>Azure Stack 開發套件

Microsoft [Azure Stack 開發套件 (ASDK)](.\asdk\asdk-what-is.md) 是單一節點的 Azure Stack 部署，您可以用來評估和瞭解 Azure Stack。  您也可以使用 ASDK 作為開發人員環境，來使用與 Azure 一致的 API 和工具建置應用程式。

>[!Note]
>ASDK 的目的並非要在生產環境中使用。

ASDK 有下列限制：

* ASDK 會與單一 Azure Active Directory (Azure AD) 或 Active Directory 同盟服務 (AD FS) 識別提供者相關聯。 您可以在此目錄中建立多位使用者，並對每位使用者指派訂用帳戶。
* Azure Stack 元件會部署在一部主機電腦上，因此租用戶資源可用的實體資源有限。 此設定不適合進行規模或效能評估。
* 因為單一主機和 NIC 部署需求，所以網路功能案例會受到限制。

## <a name="next-steps"></a>後續步驟

[重要功能與概念](azure-stack-key-features.md)

[Azure Stack：Azure 的擴充 (pdf)](https://azure.microsoft.com/resources/azure-stack-an-extension-of-azure/)
