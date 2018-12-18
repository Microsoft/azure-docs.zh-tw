---
title: 如何 - 為來自不受信任網路的存取嘗試，設定 Azure Active Directory 條件式存取原則 | Microsoft Docs
description: 了解如何為來自不受信任網路的存取嘗試，設定 Azure Active Directory (Azure AD) 中的條件式存取原則。
services: active-directory
keywords: 應用程式的條件式存取, Azure AD 條件式存取, 安全存取公司資源, 條件式存取原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.component: conditional-access
ms.assetid: 8c1d978f-e80b-420e-853a-8bbddc4bcdad
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/23/2018
ms.author: markvi
ms.reviewer: calebb
ms.openlocfilehash: 5ddde65b2a68e71d86af6ce3dcd2847736cf5823
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627180"
---
# <a name="how-to-configure-conditional-access-policies-for-access-attempts-from-untrusted-networks"></a>如何：為來自不受信任網路的存取嘗試，設定條件式存取原則   

在行動第一、雲端第一的世界中，Azure Active Directory (Azure AD) 可讓您從任何地方單一登入至裝置、應用程式和服務。 因此，您的使用者可以從組織的網路以及任何不被信任的網際網路位置，存取雲端應用程式。 透過 [Azure Active Directory (Azure AD) 條件式存取](../active-directory-conditional-access-azure-portal.md)，您可以控制授權使用者如何存取您的雲端應用程式。 此脈絡中一個常見需求就是要控制從不被信任網路所發起的存取嘗試。 本文提供當您設定條件式存取原則來處理這項規定時所需的資訊。 

## <a name="prerequisites"></a>必要條件

本文假設您已熟悉以下各項： 

- Azure AD 條件式存取基本概念 
- 在 Azure 入口網站中設定條件式存取原則

請參閱：

- [什麼是 Azure Active Directory 中的條件式存取](../active-directory-conditional-access-azure-portal.md) - 條件式存取的概觀 

- [快速入門：利用 Azure Active Directory 條件式存取來取得特定應用程式的 MFA](app-based-mfa.md) - 取得設定條件式存取原則的某些經驗。 


## <a name="scenario-description"></a>案例描述

若要熟悉安全性與生產力的平衡，您只需要使用密碼來驗證使用者就足夠了。 不過，當有人從自不被信任的網路位置發出存取嘗試時，非合法使用者登入網路的情況會越加嚴重。 若要解決這個問題，您可以封鎖來自不被信任網路的存取嘗試。 或者，您也可以要求進行多重要素驗證 (MFA)，保證確實是其他合法的帳戶擁有者想登入網路。 

利用 Azure AD 條件式存取，您可以透過單一原則來處理此需求並： 

- 將存取權授與所選的雲端應用程式

- 將存取權授與所選使用者和群組  

- 從以下位置發出存取嘗試時 

- 要求多重要素驗證： 

    - 不被信任的位置


## <a name="considerations"></a>考量

此案例的挑戰是要將「有人嘗試從不被信任的位置進行存取時」變成條件式存取。 在條件式存取原則中，您可以設定[位置條件](location-condition.md)，即可解決各種與網路位置有關的案例。 位置條件可讓您選取具名位置，它代表 IP 位址範圍、國家和地區的邏輯群組。  

一般而言，您的組織擁有一或多個位址範圍，例如 199.30.16.0 - 199.30.16.24。
若要設定具名位置，請：

- 指定這個範圍 (199.30.16.0/24) 

- 指派一個描述性名稱，例如**公司網路** 


若不想嘗試定義所有不被信任的位置，您可以：

- 包含 

    ![條件式存取](./media/untrusted-networks/02.png)

- 排除所有信任的位置 

    ![條件式存取](./media/untrusted-networks/01.png)



## <a name="implementation"></a>實作

您可以參考本文描述方法，立即為不被信任的位置設定一個條件式存取原則。 您應該一律先測試原則，然後再套用至生產環境，以確認原則如預期般運作。 在理想情況下，您應該盡可能在測試租用戶中進行初始測試。 如需詳細資訊，請參閱[如何部署新的原則](best-practices.md#how-should-you-deploy-a-new-policy)。 



## <a name="next-steps"></a>後續步驟

如果想進一步深入了解條件式存取，請參閱[什麼是 Azure Active Directory 條件式存取？](../active-directory-conditional-access-azure-portal.md)