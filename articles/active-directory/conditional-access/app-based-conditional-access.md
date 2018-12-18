---
title: Azure Active Directory 應用程式型條件式存取 | Microsoft Docs
description: 了解 Azure Active Directory 應用程式型條件式存取有何作用。
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
ms.date: 06/13/2018
ms.author: markvi
ms.reviewer: spunukol
ms.openlocfilehash: f34fc4c41094292db9bed1294ee7b26ec04c96c6
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39630597"
---
# <a name="azure-active-directory-app-based-conditional-access"></a>Azure Active Directory 應用程式型條件式存取  

您的員工使用行動裝置來處理個人和工作事務。 在維護員工的生產力時，您也希望能預防資料遺失。 有了 Azure Active Directory (Azure AD) 應用程式型條件式存取，您可以將雲端應用程式的存取限制為能保護公司資料的用戶端應用程式。  

本主題說明如何設定 Azure AD 應用程式型條件式存取。

## <a name="overview"></a>概觀

透過 [Azure AD 應用程式型條件式存取](overview.md)，您可以微調授權使用者如何存取您的資源。 例如，您可以將雲端應用程式的存取限制為受信任的裝置。

您可以使用 [Intune 應用程式保護原則](https://docs.microsoft.com/intune/app-protection-policy)來協助保護公司資料。 Intune 應用程式保護原則不需要行動裝置管理 (MDM) 解決方案，因此不論您是否在裝置管理解決方案中註冊裝置，都可以保護公司的資料。

Azure Active Directory 應用程式型條件式存取，可讓您將雲端應用程式的存取限制為支援 Intune 應用程式保護原則的用戶端應用程式。 例如，您可以限制唯有 Outlook 應用程式能存取 Exchange Online。

在條件式存取術語中，這些用戶端應用程式稱為**核准的用戶端應用程式**。  


![條件式存取](./media/app-based-conditional-access/05.png)


如需核准的用戶端應用程式清單，請參閱[核准的用戶端應用程式需求](technical-reference.md#approved-client-app-requirement)。


您可以將應用程式型條件式存取與其他原則 (如[裝置型條件式存取原則](require-managed-devices.md)) 結合，讓保護個人和公司裝置資料的方式更具彈性。

 


## <a name="before-you-begin"></a>開始之前

本主題假設您已熟悉：

- [核准的用戶端應用程式需求](technical-reference.md#approved-client-app-requirement)技術參考。


- [Azure Active Directory 中的條件式存取](overview.md)的基本概念。

- 如何[設定條件式存取原則](app-based-mfa.md)。

- [條件式存取原則的移轉](best-practices.md#policy-migration)。
 

## <a name="prerequisites"></a>必要條件

若要建立應用程式型條件式存取原則，您必須擁有 Enterprise Mobility + Security 或 Azure Active Directory Premium 訂用帳戶，而且使用者必須獲得 EMS 或 Azure AD 的授權。 


## <a name="exchange-online-policy"></a>Exchange Online 原則 

本案例包含存取 Exchange Online 的應用程式型條件式存取原則。


### <a name="scenario-playbook"></a>情節腳本

此情節假設使用者：

- 將使用 iOS 或 Android 原生郵件應用程式的電子郵件設定為連線到 Exchange

- 收到一封電子郵件，郵件顯示僅能使用 Outlook 應用程式存取

- 透過連結下載應用程式

- 開啟 Outlook 應用程式並使用 Azure AD 認證登入

- 系統提示安裝 Authenticator (iOS) 或公司入口網站 (Android) 以繼續進行

- 安裝應用程式，而且可以返回 Outlook 應用程式繼續進行

- 系統提示註冊裝置

- 能夠存取電子郵件

任何 Intune 應用程式防護原則都會在存取公司資料時啟動，而且可能會提示使用者重新啟動應用程式、使用其他 PIN 碼等 (若有針對應用程式和平台設定)。

### <a name="configuration"></a>組態 

**步驟 1 - 設定 Exchange Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/01.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。

3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 做為雲端應用程式。

    ![條件式存取](./media/app-based-conditional-access/07.png)

4. **條件：**[條件] 需要設定 [裝置平台] 和 [用戶端應用程式]：

    a. [裝置平台] 請選取 [Android] 和 [iOS]。

    ![條件式存取](./media/app-based-conditional-access/03.png)

    b. [用戶端應用程式] 需選取 [行動裝置應用程式和桌面應用程式]。

    ![條件式存取](./media/app-based-conditional-access/04.png)

5. [存取控制項] 必須選取 [需要核准的用戶端應用程式 \(預覽\)\]。

    ![條件式存取](./media/app-based-conditional-access/05.png)
 

**步驟 2 - 使用 Active Sync (EAS) 設定 Exchange Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/06.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。


3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 做為雲端應用程式。

    ![條件式存取](./media/app-based-conditional-access/07.png)

4. **條件：**[條件] 需要設定 [用戶端應用程式]。 

    a. [用戶端應用程式] 請選取 [Exchange Active Sync]。

    ![條件式存取](./media/app-based-conditional-access/08.png)

    b. [存取控制項] 必須選取 [需要核准的用戶端應用程式 \(預覽\)\]。

    ![條件式存取](./media/app-based-conditional-access/05.png)


**步驟 3 - 設定 iOS 和 Android 用戶端應用程式的 Intune 應用程式防護原則**


![條件式存取](./media/app-based-conditional-access/09.png)

如需詳細資訊，請參閱[使用 Microsoft Intune 保護應用程式和資料](https://docs.microsoft.com/intune-classic/deploy-use/protect-apps-and-data-with-microsoft-intune)。


## <a name="exchange-online-and-sharepoint-online-policy"></a>Exchange Online 和 SharePoint Online 原則

以下情節為使用受核准的應用程式以採取行動裝置應用程式管理原則的條件式存取來存取 Exchange Online 和 SharePoint Online。

### <a name="scenario-playbook"></a>情節腳本

此情節假設使用者：

- 嘗試使用 SharePoint 應用程式連接還有檢視其公司網站

- 嘗試使用與 Outlook 應用程式認證相同的認證登入

- 不需要重新註冊，即可取得資源的存取權


### <a name="configuration"></a>組態

**步驟 1 - 設定 Exchange Online 和 SharePoint Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/71.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。


3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 和 [Office 365 SharePoint Online] 做為雲端應用程式。 

    ![條件式存取](./media/app-based-conditional-access/02.png)

4. **條件：**[條件] 需要設定 [裝置平台] 和 [用戶端應用程式]：

    a. [裝置平台] 請選取 [Android] 和 [iOS]。

    ![條件式存取](./media/app-based-conditional-access/03.png)

    b. [用戶端應用程式] 需選取 [行動裝置應用程式和桌面應用程式]。

    ![條件式存取](./media/app-based-conditional-access/04.png)

5. [存取控制項] 必須選取 [需要核准的用戶端應用程式 \(預覽\)\]。

    ![條件式存取](./media/app-based-conditional-access/05.png)




**步驟 2 - 使用 Active Sync (EAS) 設定 Exchange Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/06.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。

3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 做為雲端應用程式。 線上 

    ![條件式存取](./media/app-based-conditional-access/07.png)

4. **條件：**[條件] 需要設定 [用戶端應用程式]：

    a. [用戶端應用程式] 請選取 [Exchange Active Sync]。

    ![條件式存取](./media/app-based-conditional-access/08.png)

    b. [存取控制項] 必須選取 [需要核准的用戶端應用程式 \(預覽\)\]。

    ![條件式存取](./media/app-based-conditional-access/05.png)




**步驟 3 - 設定 iOS 和 Android 用戶端應用程式的 Intune 應用程式防護原則**


![條件式存取](./media/app-based-conditional-access/09.png)

如需詳細資訊，請參閱[使用 Microsoft Intune 保護應用程式和資料](https://docs.microsoft.com/intune-classic/deploy-use/protect-apps-and-data-with-microsoft-intune)。


## <a name="app-based-or-compliant-device-policy-for-exchange-online-and-sharepoint-online"></a>適用於 Exchange Online 和 SharePoint Online 的應用程式型或相容裝置原則

本案例包含存取 Exchange Online 的應用程式型或相容裝置條件式存取原則。


### <a name="scenario-playbook"></a>情節腳本

此情節假設：
 
- 某些使用者已申請 (無論是否使用公司裝置)

- 未使用應用程式保護的應用程式向 Azure AD 申請和註冊的使用者需要註冊裝置，才能存取資源

- 已使用應用程式保護的應用程式完成申請的使用者不需要重新註冊裝置


### <a name="configuration"></a>組態

**步驟 1 - 設定 Exchange Online 和 SharePoint Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/62.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。

3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 和 [Office 365 SharePoint Online] 做為雲端應用程式。 

     ![條件式存取](./media/app-based-conditional-access/02.png)

4. **條件：**[條件] 需要設定 [裝置平台] 和 [用戶端應用程式]。 
 
    a. [裝置平台] 請選取 [Android] 和 [iOS]。

    ![條件式存取](./media/app-based-conditional-access/03.png)

    b. [用戶端應用程式] 需選取 [行動裝置應用程式和桌面應用程式]。

    ![條件式存取](./media/app-based-conditional-access/04.png)

5. [存取控制項] 需要選取下列項目：

    - [裝置需要標記為合規]

    - [需要經過核准的用戶端應用程式 \(預覽\)\]

    - [需要其中一個選取的控制項]   
 
    ![條件式存取](./media/app-based-conditional-access/11.png)



**步驟 2 - 使用 Active Sync (EAS) 設定 Exchange Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/61.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。

3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 做為雲端應用程式。 

    ![條件式存取](./media/app-based-conditional-access/07.png)

4. **條件：**[條件] 需要設定 [用戶端應用程式]。 

    選取 [Exchange Active Sync] 做為 **用戶端應用程式*。

    ![條件式存取](./media/app-based-conditional-access/08.png)

5. [存取控制項] 必須選取 [需要核准的用戶端應用程式 \(預覽\)\]。
 
    ![條件式存取](./media/app-based-conditional-access/11.png)




**步驟 3 - 設定 iOS 和 Android 用戶端應用程式的 Intune 應用程式防護原則**


![條件式存取](./media/app-based-conditional-access/09.png)

如需詳細資訊，請參閱[使用 Microsoft Intune 保護應用程式和資料](https://docs.microsoft.com/intune-classic/deploy-use/protect-apps-and-data-with-microsoft-intune)。





## <a name="app-based-and-compliant-device-policy-for-exchange-online-and-sharepoint-online"></a>適用於 Exchange Online 和 SharePoint Online 的應用程式型和相容裝置原則

本案例包含存取 Exchange Online 的應用程式型和相容裝置條件式存取原則。


### <a name="scenario-playbook"></a>情節腳本

此情節假設使用者：
 
-   將使用 iOS 或 Android 原生郵件應用程式的電子郵件設定為連線到 Exchange
-   接收指出存取權要求註冊您裝置的電子郵件
-   下載公司入口網站並登入公司入口網站
-   檢查電子郵件，並且被要求使用 Outlook 應用程式
-   下載 Outlook 應用程式
-   開啟 Outlook 應用程式，並輸入申請時所用的認證
-   使用者即可存取電子郵件

任何 Intune 應用程式防護原則都會在存取公司資料時啟動，而且可能會提示使用者重新啟動應用程式、使用其他 PIN 碼等 (若有針對應用程式和平台設定)。


### <a name="configuration"></a>組態

**步驟 1 - 設定 Exchange Online 和 SharePoint Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/62.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。

3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 和 [Office 365 SharePoint Online] 做為雲端應用程式。 

     ![條件式存取](./media/app-based-conditional-access/02.png)

4. **條件：**[條件] 需要設定 [裝置平台] 和 [用戶端應用程式]。 
 
    a. [裝置平台] 請選取 [Android] 和 [iOS]。

    ![條件式存取](./media/app-based-conditional-access/03.png)

    b. [用戶端應用程式] 需選取 [行動裝置應用程式和桌面應用程式]。

    ![條件式存取](./media/app-based-conditional-access/04.png)

5. [存取控制項] 需要選取下列項目：

    - [裝置需要標記為合規]

    - [需要經過核准的用戶端應用程式 \(預覽\)\]

    - [需要所有選取的控制項]   
 
    ![條件式存取](./media/app-based-conditional-access/13.png)



**步驟 2 - 使用 Active Sync (EAS) 設定 Exchange Online 的 Azure AD 條件式存取原則**

對於此步驟中的條件式存取原則，您需要設定下列元件：

![條件式存取](./media/app-based-conditional-access/61.png)

1. 條件式存取原則的**名稱**。

2. **使用者和群組**：每個條件式存取原則必須至少選取一個使用者或群組。

3. **雲端應用程式：** 您必須選取 [Office 365 Exchange Online] 做為雲端應用程式。 

    ![條件式存取](./media/app-based-conditional-access/07.png)

4. **條件：**[條件] 需要設定 [用戶端應用程式]。 

    [用戶端應用程式] 請選取 [Exchange Active Sync]。

    ![條件式存取](./media/app-based-conditional-access/08.png)

5. [存取控制項] 需要選取下列項目：

    - [裝置需要標記為合規]

    - [需要經過核准的用戶端應用程式 \(預覽\)\]

    - [需要所有選取的控制項]   
 
    ![條件式存取](./media/app-based-conditional-access/64.png)




**步驟 3 - 設定 iOS 和 Android 用戶端應用程式的 Intune 應用程式防護原則**


![條件式存取](./media/app-based-conditional-access/09.png)

如需詳細資訊，請參閱[使用 Microsoft Intune 保護應用程式和資料](https://docs.microsoft.com/intune-classic/deploy-use/protect-apps-and-data-with-microsoft-intune)。






## <a name="next-steps"></a>後續步驟

如果您想要知道如何設定條件式存取原則，請參閱[利用 Azure Active Directory 條件式存取來取得特定應用程式的 MFA](app-based-mfa.md)。

如果您已準備好設定您環境的條件式存取原則，請參閱 [Azure Active Directory 中條件式存取的最佳做法](best-practices.md)。 
