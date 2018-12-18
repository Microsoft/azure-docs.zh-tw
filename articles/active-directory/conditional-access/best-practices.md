---
title: Azure Active Directory 中條件式存取的最佳做法 | Microsoft Docs
description: 了解您在設定條件式存取原則時應該知道的事件及應避免的動作。
services: active-directory
keywords: 應用程式的條件式存取, Azure AD 條件式存取, 安全存取公司資源, 條件式存取原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 8c1d978f-e80b-420e-853a-8bbddc4bcdad
ms.service: active-directory
ms.component: conditional-access
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/23/2018
ms.author: markvi
ms.reviewer: calebb
ms.openlocfilehash: 4e9f5a9318db813b1a0f16d3599f74fd98e53ffc
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818052"
---
# <a name="best-practices-for-conditional-access-in-azure-active-directory"></a>Azure Active Directory 中條件式存取的最佳做法

透過 [Azure Active Directory (Azure AD) 條件式存取](../active-directory-conditional-access-azure-portal.md)，您可以控制授權使用者如何存取您的雲端應用程式。 本文提供以下相關資訊：

- 您應該知道的事情 
- 您在設定條件式存取原則時應該應避免的動作。 

本文假設您熟悉[什麼是Azure Active Directory 中的條件式存取](../active-directory-conditional-access-azure-portal.md)中所述的概念與術語



## <a name="whats-required-to-make-a-policy-work"></a>讓原則運作的必要條件？

當您建立新的原則時，未選取任何使用者、群組、應用程式或存取控制項。

![雲端應用程式](./media/best-practices/02.png)


若要讓您的原則運作，您必須設定：


|何事           | 方式                                  | 理由|
|:--            | :--                                  | :-- |
|**雲端應用程式** |您需要選取一或多個應用程式。  | 條件式存取原則的目標是要讓您控制授權的使用者如何存取您的應用程式。|
| **使用者和群組** | 您需要選取至少一個已獲授權存取您選取的雲端應用程式的使用者或群組。 | 永遠不會觸發未指派使用者和群組的條件式存取原則。 |
| **存取控制** | 您必須選取至少一個存取控制。 | 如果滿足您的條件，您的原則處理器需要知道該怎麼辦。|




## <a name="what-you-should-know"></a>您應該知道的事情

### <a name="how-are-assignments-evaluated"></a>如何評估指派？

所有指派邏輯上都會使用 **AND** 運算子。 如果您已設定多個指派，必須滿足所有的指派才能觸發原則。  

如果您需要設定一個位置條件，以套用至從您的組織網路外部進行的所有連線︰

- 包含**所有位置**
- 排除**所有信任的 IP**


### <a name="what-to-do-if-you-are-locked-out-of-the-azure-ad-admin-portal"></a>如果被鎖定在 Azure AD 管理入口網站之外該怎麼辦？

如果您因為條件式存取原則中不正確的設定而被 Azure AD 入口網站鎖定在外：

- 確認您的組織中有其他系統管理員尚未被鎖定。 具有 Azure 入口網站存取權的系統管理員可以停用會影響您的登入的原則。 

- 如果您的組織中沒有任何系統管理員可以更新原則，您需要提交支援要求。 Microsoft 支援服務可以檢閱和更新導致您無法存取的條件式存取原則。


### <a name="what-happens-if-you-have-policies-in-the-azure-classic-portal-and-azure-portal-configured"></a>如果您已在 Azure 傳統入口網站和 Azure 入口網站中設定一些原則，則會發生什麼情況？  

Azure Active Directory 會強制執行這兩個原則，而且只有在符合所有需求時，使用者才可取得存取權。

### <a name="what-happens-if-you-have-policies-in-the-intune-silverlight-portal-and-the-azure-portal"></a>如果您已在 Intune Silverlight 入口網站和 Azure 入口網站中設定一些原則，則會發生什麼情況？

Azure Active Directory 會強制執行這兩個原則，而且只有在符合所有需求時，使用者才可取得存取權。

### <a name="what-happens-if-i-have-multiple-policies-for-the-same-user-configured"></a>如果我已針對同一個使用者設定多個原則，則會發生什麼情況？  

針對每次登入，Azure Active Directory 會評估所有的原則，並確保在授與使用者存取權之前已符合所有的需求。 封鎖存取的優先順序高過所有其他組態設定。 


### <a name="does-conditional-access-work-with-exchange-activesync"></a>條件式存取是否適用於 Exchange ActiveSync？

是，您可以在條件式存取原則中使用 Exchange ActiveSync。






## <a name="what-you-should-avoid-doing"></a>您應該避免做的事

條件式存取架構可為您提供絕佳的組態彈性。 不過，絕佳的彈性也意謂著您應該先仔細地檢閱每個設定原則，然後才發行它，以避免產生不想要的結果。 在此情況下，您應該特別注意影響整個集合的指派，例如 **all users / groups / cloud apps**。

在您的環境中，應該避免使用下列組態：


**針對所有使用者、所有雲端應用程式：**

- **封鎖存取** - 此組態會封鎖您整個組織，這絕對不是一個好方法。

- **需要符合規範的裝置** - 針對尚未註冊其裝置的使用者，此原則會封鎖所有存取，包括對 Intune 入口網站的存取。 如果您是沒有已註冊裝置的系統管理員，此原則會阻擋您回到 Azure 入口網站來變更此原則。

- **需要加入網域** - 如果您還沒有已加入網域的裝置，此原則也可能封鎖您組織中所有使用者的存取。


**針對所有使用者、所有雲端應用程式、所有裝置平台：**

- **封鎖存取** - 此組態會封鎖您整個組織，這絕對不是一個好方法。


## <a name="how-should-you-deploy-a-new-policy"></a>您應該如何部署新的原則？

第一步，您應該使用[該怎麼辦工具](what-if-tool.md)評估您的原則。

當您準備好要將新的原則部署到您的環境時，應該分三個階段執行：

1. 將原則套用至較少的使用者，確認它如預期般運作。 

2.  當您將原則擴大到更多使用者的原則時，繼續從原則中排除所有的系統管理員。 這可確保如果需要變更，系統管理員仍然可以存取和更新原則。

3. 只在真的必要時，才將原則套用到所有使用者。 

最佳做法是建立這一個使用者帳戶：

- 專門用來管理原則 
- 從您的原則中排除


## <a name="policy-migration"></a>原則移轉

請考慮移轉尚未在 Azure 入口網站中建立的原則，因為：

- 您現在可以處理之前無法處理的案例。

- 您可以合併它們以減少必須管理的原則數目。   

- 您可以在一個集中位置管理您的所有條件存取原則。

- Azure 傳統入口網站將被淘汰。   


如需詳細資訊，請參閱[在 Azure 入口網站中移轉傳統原則](policy-migration.md)。


## <a name="next-steps"></a>後續步驟

如果您想要知道如何設定條件式存取原則，請參閱[利用 Azure Active Directory 條件式存取來取得特定應用程式的 MFA](app-based-mfa.md)。
