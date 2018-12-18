---
title: 作法 - 透過 Azure Active Directory 條件式存取要求必須從受控裝置存取雲端應用程式 | Microsoft Docs
description: 了解如何設定 Azure Active Directory (Azure AD) 裝置型條件式存取原則，以要求必須從受控裝置存取雲端應用程式。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: a27862a6-d513-43ba-97c1-1c0d400bf243
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/14/2018
ms.author: markvi
ms.reviewer: jairoc
ms.openlocfilehash: b59e4898f85de7ad93d9172cdb3c551a17799194
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39630478"
---
# <a name="how-to-require-managed-devices-for-cloud-app-access-with-conditional-access"></a>作法：透過條件式存取要求必須從受控裝置存取雲端應用程式

在行動第一、雲端第一的世界中，Azure Active Directory (Azure AD) 可讓您從任何地方單一登入至應用程式和服務。 授權使用者可以從多種裝置 (包括行動裝置和個人裝置) 存取您的雲端應用程式。 不過，許多環境都至少會有些許應用程式只應由符合您的安全性與相容性標準的裝置存取。 這些裝置也稱為受控裝置。 

本文說明如何設定條件式存取原則，以要求只有受控裝置才能存取您環境中的特定雲端應用程式。 


## <a name="prerequisites"></a>必要條件

要求須由受控裝置存取雲端應用程式，會將 **Azure AD 條件式存取**和 **Azure AD 裝置管理**繫結在一起。 如果您還不熟悉上述其中一種領域，您應該先閱讀下列主題：

- **[Azure Active Directory 中的條件式存取](../active-directory-conditional-access-azure-portal.md)** - 本文提供條件式存取的概念性概觀和相關術語。

- **[Azure Active Directory 中的裝置管理簡介](../devices/overview.md)** - 本文概略說明在組織的控制下可用來連接裝置的各種選項。 


## <a name="scenario-description"></a>案例描述

要兼顧安全性與生產力並不容易。 在可存取雲端資源的裝置種類激增的情況下，使用者的生產力得以提升。 而另一方面，您應該不希望環境中的特定資源在保護層級不確定的情況下受到裝置存取。 對於受影響的資源，您應要求使用者只能使用受控裝置加以存取。 

利用 Azure AD 條件式存取，您可以透過單一原則來處理此需求並：

- 將存取權授與所選的雲端應用程式

- 將存取權授與所選使用者和群組

- 必須使用受控裝置


## <a name="managed-devices"></a>受控裝置  

簡單來說，受控裝置是受到*某種*組織性控制的裝置。 在 Azure AD 中，受控裝置的必要條件是已向 Azure AD 註冊。 註冊裝置時，系統會以裝置物件的形式為裝置建立身分識別。 Azure 使用此物件來追蹤裝置的狀態資訊。 身為 Azure AD 管理員的您已經可以使用此物件來切換 (啟用/停用) 裝置的狀態。
  
![裝置型條件](./media/require-managed-devices/32.png)

若要向 Azure AD 註冊裝置，您有三個選項：

- **[Azure AD 註冊裝置](../devices/overview.md#azure-ad-registered-devices)** - 向 Azure AD 註冊個人裝置

- **[已加入 Azure AD 的裝置](../devices/overview.md#azure-ad-joined-devices)** - 向 Azure AD 註冊未加入內部部署 AD 的組織 Windows 10 裝置。 

- **[已加入混合式 Azure AD 的裝置](../devices/overview.md#hybrid-azure-ad-joined-devices)** - 取得 Windows 10 裝置或支援的下層裝置，該裝置會加入已向 Azure AD 註冊的內部部署 AD。

若要成為受控裝置，已註冊的裝置必須是**已加入混合式 Azure AD 的裝置**，或是**標示為符合規範的裝置**。  

![裝置型條件](./media/require-managed-devices/47.png)

 
## <a name="require-hybrid-azure-ad-joined-devices"></a>需要已加入混合式 Azure AD 的裝置

在條件式存取原則中，您可以選取 [需要已加入混合式 Azure AD 的裝置]，表示只能使用受控裝置來存取所選取的雲端應用程式。 

![裝置型條件](./media/require-managed-devices/10.png)

此設定僅適用於要加入內部部署 AD 的 Windows 10 或下層裝置 (例如 Windows 7 或 Windows 8)。 您只可以使用「加入混合式 Azure AD」向 Azure AD 註冊這些裝置，而這是註冊 Windows 10 裝置的[自動化程序](../devices/hybrid-azuread-join-plan.md)。 

![裝置型條件](./media/require-managed-devices/45.png)

如何讓已加入混合式 Azure AD 的裝置成為受控裝置？  若為已加入內部部署 AD 的裝置，則會假設已使用管理解決方案 (例如 **System Center Configuration Manager (SCCM)** 或**群組原則 (GP)** 強制控制這些裝置，進而管理它們。 因為 Azure AD 沒有任何方法可判斷裝置是否已套用任何方法，所以需要已加入混合式 Azure AD 的裝置是需要受控裝置的相對較弱機制。 如果這類裝置也是已加入混合式 Azure AD 的裝置，則由系統管理員判斷已加入網域的內部部署裝置所套用的方法是否足以造就受控裝置。


## <a name="require-device-to-be-marked-as-compliant"></a>裝置需要標記為符合規範

「裝置需要標記為符合規範」選項是要求受控裝置的最強形式。

![裝置型條件](./media/require-managed-devices/11.png)

此選項要求向 Azure AD 註冊裝置，而且裝置要由下列項目標示為符合規範：
         
- Intune。
- 一個第三方行動裝置管理 (MDM) 系統，可透過 Azure AD 整合來管理 Windows 10 裝置。 不支援針對 Windows 10 以外的裝置 OS 類型使用的第三方 MDM 系統。
 
![裝置型條件](./media/require-managed-devices/46.png)



對於標示為符合規範的裝置，您可以假設： 

- 您的員工用來存取公司資料的行動裝置受到控管
- 您的員工使用的行動裝置應用程式受到控管
- 協助控制您的員工存取及共用公司資訊的方式，進而保護公司資訊
- 裝置與其應用程式都符合公司安全性需求的規範




## <a name="next-steps"></a>後續步驟

在您的環境中設定裝置型條件式存取原則之前，您應該閱讀 [Azure Active Directory 中條件式存取的最佳做法](best-practices.md)。

