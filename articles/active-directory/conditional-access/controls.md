---
title: 什麼是 Azure Active Directory 條件式存取中的存取控制？ | Microsoft Docs
description: 了解 Azure Active Directory 條件式存取中的存取控制如何運作。
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
ms.date: 08/28/2018
ms.author: markvi
ms.reviewer: calebb
ms.openlocfilehash: 050ccff8501a22526e9382a620258b0f846efe5c
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43128716"
---
# <a name="what-are-access-controls-in-azure-active-directory-conditional-access"></a>什麼是 Azure Active Directory 條件式存取中的存取控制？ 

透過 [Azure Active Directory (Azure AD) 條件式存取](../active-directory-conditional-access-azure-portal.md)，您可以控制授權使用者如何存取您的雲端應用程式。 在條件式存取原則中，您會定義對觸發原則之原因 ("when this happens") 做出的回應 ("do this")。 

![控制](./media/controls/10.png)


在條件式存取的情境中， 

- "**When this happens**" 稱為**條件**

- "**Then do this**" 稱為**存取控制**


條件陳述式與控制項的組合代表條件式存取原則。

![控制](./media/controls/61.png)

每種控制項都是必須由登入之使用者或系統履行的要求，或是規範使用者在登入後之行為的限制。 

控制項分為兩種： 

- **授與控制項** - 限制存取權限

- **工作階段控制項** - 限制工作階段內的存取權限

本主題說明 Azure AD 條件式存取中的各種控制項。 

## <a name="grant-controls"></a>授與控制

透過授與控制項，您可以藉由選取需要的控制項來全面封鎖存取權限，或允許在滿足其他要求的情況下授與存取權限。 若要實施多個控制項，您可以要求：

- 滿足所有選取的控制項 (*AND*) 
- 滿足一項選取的控制項 (*OR*)

![控制](./media/controls/17.png)



### <a name="multi-factor-authentication"></a>Multi-Factor Authentication

您可以使用這個控制項來要求使用者通過 Multi-Factor Authentication 後，才能存取指定的雲端應用程式。 這個控制項支援以下多重要素提供者： 

- Azure Multi-Factor Authentication 

- 結合 Active Directory Federation Services (AD FS) 的內部部署 Multi-Factor Authentication 提供者。
 
對於未獲授權但已取得有效使用者之主要認證存取權的使用者，使用 Multi-Factor Authentication 有助於防止其存取資源。



### <a name="compliant-device"></a>符合規範的裝置

您可以設定以裝置作為基礎的條件式存取原則。 裝置型條件式存取原則的目標是，只從[受控裝置](require-managed-devices.md)授與已設定資源的存取權。 定義受控裝置的必要選項之一，是要求符合規範裝置。 如果選取此選項，使用已向您的 Azure Active Directory [註冊](../devices/overview.md)且經 Intune (適用於任何裝置 OS) 或您的協力廠商 MDM 系統 (適用於 Windows 10 裝置) 標示為符合規範的裝置進行存取嘗試時，您的條件式存取原則就會對其授與存取權。 不支援針對 Windows 10 以外的裝置 OS 類型使用的第三方 MDM 系統。

如需詳細資訊，請參閱[設定 Azure Active Directory 裝置型條件式存取原則](require-managed-devices.md)。

### <a name="hybrid-azure-ad-joined-device"></a>已加入混合式 Azure AD 的裝置

設定裝置型條件式存取原則的另一個必要選項，是要求已加入混合式 Azure AD 的裝置。 這項需求是指加入內部部署 Active Directory 的 Windows 桌上型電腦、膝上型電腦和企業平板電腦。 如果選取此選項，當使用已加入您內部部署 Active Directory 和 Azure Active Directory 的裝置來嘗試存取時，您的條件式存取原則就會對其授與存取權。  

如需詳細資訊，請參閱[設定 Azure Active Directory 裝置型條件式存取原則](require-managed-devices.md)。





### <a name="approved-client-app"></a>已核准的用戶端應用程式

由於員工使用行動裝置來處理個人和工作事務，因此即使您不負責管理公司資料，可能也會想要在員工使用裝置存取公司資料時保護資料。
您可以使用 [Intune 應用程式保護原則](https://docs.microsoft.com/intune/app-protection-policy)來保護公司的資料，並且不受任何行動裝置管理 (MDM) 解決方案影響。


藉由核准的用戶端應用程式，您可以要求嘗試存取雲端應用程式的用戶端應用程式支援 [Intune 應用程式保護原則](https://docs.microsoft.com/intune/app-protection-policy)。 例如，您可以限制唯有 Outlook 應用程式能存取 Exchange Online。 要求通過核准之用戶端應用程式的條件式存取原則，也稱為[應用程式型條件式存取原則](app-based-conditional-access.md)。 如需支援的核准用戶端應用程式清單，請參閱[核准的用戶端應用程式需求](technical-reference.md#approved-client-app-requirement)。


### <a name="terms-of-use"></a>使用條款

您可以要求租用戶中的使用者先同意使用條款，再授與他們存取資源的權利。 身為系統管理員，您可以上傳 PDF 文件來設定及自訂使用條款。 使用者如果落在此控制項的控制範圍內，就必須同意使用條款才會獲得應用程式的存取權。 


### <a name="custom-controls"></a>自訂控制項 

您可以在條件式存取中建立自訂控制項，以將使用者重新導向至相容的服務，從而滿足 Azure Active Directory 之外的其他需求。 這可讓您使用某些外部的多重要素驗證和驗證提供者，來強制執行條件式存取規則或建置您自己的自訂服務。 為了滿足此控制項，系統會將使用者的瀏覽器重新導向至外部服務，執行任何必要的驗證或確認活動，再將瀏覽器重新導向回到 Azure Active Directory。 使用者如果成功通過驗證或確認，就會繼續進行條件式存取流程。 

## <a name="custom-controls"></a>自訂控制項

自訂控制項是 Azure Active Directory Premium P1 版本的一項功能。 使用自訂控制項時，系統會將使用者重新導向到相容的服務，以滿足 Azure Active Directory 之外的進一步需求。 為了滿足此控制項，系統會將使用者的瀏覽器重新導向至外部服務，執行任何必要的驗證或確認活動，再將瀏覽器重新導向回到 Azure Active Directory。 Azure Active Directory 會確認回應，而且使用者如果成功通過驗證或確認，就會繼續進行條件式存取流程。

這些控制項會允許使用某些外部或自訂的服務來作為條件式存取控制項，而且一般都能擴充條件式存取的功能。

目前提供相容服務的提供者包括：

- [Duo Security](https://duo.com/docs/azure-ca)

- [委託 Datacard](https://www.entrustdatacard.com/products/authentication/intellitrust)

- RSA

- [Trusona](https://www.trusona.com/docs/azure-ad-integration-guide)


如需這些服務的詳細資訊，請直接連絡相關提供者。

### <a name="creating-custom-controls"></a>建立自訂控制項

若要建立自訂控制項，請先連絡您想要使用的提供者。 每個非 Microsoft 的提供者在註冊、訂閱或成為服務一部分等方面，以及在指出您想要與條件式存取整合的方面，都有自己的處理程序和需求。 屆時，提供者會提供您 JSON 格式的資料區塊。 這項資料可讓提供者和條件式存取一起為您的租用戶工作，建立新的控制項，以及定義條件式存取要如何分辨使用者是否已成功執行提供者所提出的驗證。

複製 JSON 資料並貼到相關文字方塊中。 除非您明確地了解您要進行的變更，否則請勿變更 JSON。 進行變更可能會讓提供者與 Microsoft 之間的連線中斷，進而可能將您的帳戶鎖定，讓您和您的使用者無法使用。

用來建立自訂控制項的選項位於 [條件式存取] 頁面的 [管理] 區段。

![控制](./media/controls/82.png)

按一下 [新增自訂控制項] 隨即會開啟刀鋒視窗，裡面有文字方塊可供輸入控制項的 JSON 資料。  


![控制](./media/controls/81.png)


### <a name="deleting-custom-controls"></a>刪除自訂控制項

若要刪除自訂控制項，您必須先確定沒有任何條件式存取原則正在使用它。 完成之後：

1. 移至 [自訂控制項] 清單

2. 按一下 [...]  

3. 選取 [刪除] 。

### <a name="editing-custom-controls"></a>編輯自訂控制項

若要編輯自訂控制項，您必須先刪除目前的控制項，然後使用更新後的資訊建立新的控制項。




## <a name="session-controls"></a>工作階段控制項

工作階段控制項可讓您限制雲端應用程式內的體驗。 工作階段控制項是由雲端應用程式強制執行，並依賴 Azure AD 對應用程式提供有關工作階段的其他資訊。

![控制](./media/controls/31.png)

### <a name="use-app-enforced-restrictions"></a>使用應用程式強制執行限制

您可以使用這個控制項，要求 Azure AD 將裝置資訊傳遞至雲端應用程式。 這有助於雲端應用程式了解使用者是否來自符合規範的裝置或加入網域的裝置。 此控制項目前僅支援使用 SharePoint 作為雲端應用程式。 視裝置狀態而定，SharePoint 會使用裝置資訊來提供使用者有限或完整的經驗。
若要深入了解如何要求 SharePoint 的有限存取，請參閱[從非受控裝置控制存取](https://aka.ms/spolimitedaccessdocs)。



## <a name="next-steps"></a>後續步驟

- 如果您想要知道如何設定條件式存取原則，請參閱[利用 Azure Active Directory 條件式存取來取得特定應用程式的 MFA](app-based-mfa.md)。

- 如果您已準備好設定您環境的條件式存取原則，請參閱 [Azure Active Directory 中條件式存取的最佳做法](best-practices.md)。 
