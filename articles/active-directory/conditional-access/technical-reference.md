---
title: Azure Active Directory 條件式存取設定參考 | Microsoft Docs
description: 取得 Azure Active Directory 條件式存取原則中所支援設定的概觀。
services: active-directory.
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: 56a5bade-7dcc-4dcf-8092-a7d4bf5df3c1
ms.service: active-directory
ms.component: conditional-access
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/11/2018
ms.author: markvi
ms.reviewer: spunukol
ms.openlocfilehash: e44b68a2ed2786b449e2680ca4a8ca531058b85c
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432788"
---
# <a name="azure-active-directory-conditional-access-settings-reference"></a>Azure Active Directory 條件式存取設定參考

您可以使用 [Azure Active Directory (Azure AD) 條件式存取](../active-directory-conditional-access-azure-portal.md)來控制已獲授權使用者存取您資源的方式。   

本文會針對條件式存取原則中的下列設定選項，為您提供支援資訊： 

- 雲端應用程式指派

- 裝置平台條件 

- 雲端應用程式條件

- 已核准的用戶端應用程式需求


如果這不是您要尋找的資訊，請在本文結尾留言。

## <a name="cloud-apps-assignments"></a>雲端應用程式指派

使用條件式存取原則，您就能控制使用者存取您[雲端應用程式](conditions.md#cloud-apps)的方式。 當您設定條件式存取原則時，必須選取至少一個雲端應用程式。 

![選取您原則適用的雲端應用程式](./media/technical-reference/09.png)


### <a name="microsoft-cloud-applications"></a>Microsoft 雲端應用程式

您可以將條件式存取原則指派給下列 Microsoft 的雲端應用程式：

- Azure 資訊保護 - [進一步了解](/azure/information-protection/faqs#i-see-azure-information-protection-is-listed-as-an-available-cloud-app-for-conditional-accesshow-does-this-work)

- Azure RemoteApp

- Microsoft Dynamics 365

- Microsoft Office 365 Yammer

- Microsoft Office 365 Exchange Online

- Microsoft Office 365 SharePoint Online (包括商務用 OneDrive 和 Project Online)

- Microsoft Power BI 

- Azure DevOps

- Microsoft Teams


### <a name="other-applications"></a>其他應用程式 

除了 Microsoft 雲端應用程式之外，您還可以將條件式存取原則指派給下列類型的雲端應用程式：

- Azure AD 連線的應用程式

- 預先整合的同盟軟體即服務 (SaaS) 應用程式

- 使用密碼單一登入 (SSO) 的應用程式

- 企業營運應用程式

- 使用 Azure AD 應用程式 Proxy 的應用程式


## <a name="device-platform-condition"></a>裝置平台條件

在條件式存取原則中，您可以設定裝置平台條件，將原則繫結到用戶端上的作業系統。 Azure AD 條件式存取支援下列裝置平台：

- Android

- iOS

- Windows Phone

- Windows

- macOS


![將存取原則繫結至用戶端作業系統](./media/technical-reference/41.png)





## <a name="client-apps-condition"></a>用戶端應用程式條件 

在條件式存取原則中，您可以設定[用戶端應用程式](conditions.md#client-apps)條件，將原則繫結至已啟動存取嘗試的用戶端應用程式。 當有人從下列類型的用戶端應用程式嘗試存取時，將用戶端應用程式條件設定為授與或封鎖存取：

- [瀏覽器]
- 行動裝置應用程式和桌面應用程式

![控制對用戶端應用程式的存取](./media/technical-reference/03.png)

### <a name="supported-browsers"></a>支援的瀏覽器 

在您的條件式存取原則中，您可以選取 [瀏覽器] 作為用戶端應用程式。

![控制對支援之瀏覽器的存取](./media/technical-reference/05.png)

此設定適用於所有瀏覽器。 不過，為了滿足像是符合裝置需求規範的裝置原則，因此支援下列作業系統和瀏覽器：


| 作業系統                     | 瀏覽器                            | 支援     |
| :--                    | :--                                 | :-:         |
| Windows 10             | Internet Explorer、Edge、Chrome     | ![勾選][1] |
| Windows 8 / 8.1        | Internet Explorer、Chrome           | ![勾選][1] |
| Windows 7              | Internet Explorer、Chrome           | ![勾選][1] |
| iOS                    | Safari、Intune Managed Browser      | ![勾選][1] |
| Android                | Chrome、Intune Managed Browser      | ![勾選][1] |
| Windows Phone          | Internet Explorer、Edge             | ![勾選][1] |
| Windows Server 2016    | Internet Explorer、Edge             | ![勾選][1] |
| Windows Server 2016    | Chrome                              | 敬請期待 |
| Windows Server 2012 R2 | Internet Explorer、Chrome           | ![勾選][1] |
| Windows Server 2008 R2 | Internet Explorer、Chrome           | ![勾選][1] |
| macOS                  | Chrome、Safari                      | ![勾選][1] |



#### <a name="chrome-support"></a>Chrome 支援

如需 **Windows 10 Creators Update (1703 版)** 或更新版本中的 Chrome 支援，請安裝[這個擴充功能](https://chrome.google.com/webstore/detail/windows-10-accounts/ppnbnpeolgkicgegkbkbjmhlideopiji)。

若要自動將此擴充功能部署到 Chrome 瀏覽器，請建立下列登錄機碼：

|    |    |
|--- | ---|
|Path | HKEY_LOCAL_MACHINE\Software\Policies\Google\Chrome\ExtensionInstallForcelist |
|名稱 | 1 |
|類型 | REG_SZ (字串) |
|資料 | ppnbnpeolgkicgegkbkbjmhlideopiji; https://clients2.google.com/service/update2/crx

如需 **Windows 8.1 和 7** 中的 Chrome 支援，請建立下列登錄機碼：

|    |    |
|--- | ---|
|Path | HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome\AutoSelectCertificateForUrls |
|名稱 | 1 |
|類型 | REG_SZ (字串) |
|資料 | {"pattern":"https://device.login.microsoftonline.com","filter":{"ISSUER":{"CN":"MS-Organization-Access"}}}|

這些瀏覽器支援裝置驗證，因此可以根據原則來識別和驗證裝置。 如果瀏覽器在私用模式中執行，裝置檢查將會失敗。 


### <a name="supported-mobile-applications-and-desktop-clients"></a>支援的行動裝置應用程式和桌面用戶端

在您的條件式存取原則中，您可以選取 [行動裝置應用程式和桌面用戶端] 作為用戶端應用程式。


![控制對支援之行動裝置應用程式或桌面用戶端的存取](./media/technical-reference/06.png)


這項設定會影響從下列行動裝置應用程式和桌面用戶端進行的存取嘗試： 


|用戶端應用程式|目標服務|平台|
|---|---|---|
|Azure 遠端應用程式|Azure 遠端應用程式服務|Windows 10、Windows 8.1、Windows 7、iOS、Android 和 Mac OS X|
|Dynamics CRM 應用程式|Dynamics CRM|Windows 10、Windows 8.1、iOS 和 Android|
|[電子郵件]/[行事曆]/[人員] 應用程式、Outlook 2016、Outlook 2013 (使用新式驗證)|Office 365 Exchange Online|Windows 10|
|應用程式的 MFA 和位置原則。 不支援裝置型原則。 |任何 My Apps 應用程式服務|Android 和 iOS|
|Microsoft Teams Services - 這會控制支援 Microsoft Teams 及其所有用戶端應用程式的所有服務 - Windows 桌面、iOS、Android、WP 和 Web 用戶端|Microsoft Teams|Windows 10、Windows 8.1、Windows 7、iOS、Android 及 macOS |
|Office 2016 應用程式、Office 2013 (具備新式驗證)、OneDrive 同步處理用戶端 (請參閱[附註](https://support.office.com/en-US/article/Azure-Active-Directory-conditional-access-with-the-OneDrive-sync-client-on-Windows-028d73d7-4b86-4ee0-8fb7-9a209434b04e))|Office 365 SharePoint Online|Windows 8.1、Windows 7|
|Office 2016 應用程式、通用 Office 應用程式、Office 2013 (具備新式驗證)、OneDrive 同步處理用戶端 (請參閱[附註](https://support.office.com/en-US/article/Azure-Active-Directory-conditional-access-with-the-OneDrive-sync-client-on-Windows-028d73d7-4b86-4ee0-8fb7-9a209434b04e))、預計未來提供的 Office Groups 支援、預計未來提供的 SharePoint 應用程式支援|Office 365 SharePoint Online|Windows 10|
|macOS 版 Office 2016 (僅限 Word、Excel、PowerPoint、OneNote)。 未來規劃支援商務用 OneDrive|Office 365 SharePoint Online|Mac OS X|
|Office 行動應用程式|Office 365 SharePoint Online|Android、iOS|
|Office Yammer 應用程式|Office 365 Yammer|Windows 10、iOS、Android|
|Outlook 2016 (macOS 版 Office)|Office 365 Exchange Online|Mac OS X|
|Outlook 2016、Outlook 2013 (已啟用新式驗證)、商務用 Skype (採用新式驗證)|Office 365 Exchange Online|Windows 8.1、Windows 7|
|Outlook 行動應用程式|Office 365 Exchange Online|Android、iOS|
|PowerBI 應用程式|PowerBI service|Windows 10、Windows 8.1、Windows 7、Android 和 iOS|
|商務用 Skype|Office 365 Exchange Online|Android、iOS |
|Azure DevOps 應用程式|Azure DevOps|Windows 10、Windows 8.1、Windows 7、iOS 和 Android|



## <a name="approved-client-app-requirement"></a>核准的用戶端應用程式需求 

在條件式存取原則中，您可以要求從已核准用戶端應用程式對所選取雲端應用程式進行存取嘗試。 

![控制對經過核准之用戶端應用程式的存取](./media/technical-reference/21.png)

此設定適用於下列用戶端應用程式：


- Microsoft Intune 受控瀏覽器
- Microsoft PowerBI
- Microsoft Invoicing
- Microsoft Launcher
- Microsoft Azure 資訊保護
- Microsoft Excel
- Microsoft Kaizala 
- Microsoft OneDrive
- Microsoft OneNote
- Microsoft Outlook
- Microsoft Planner
- Microsoft PowerPoint
- Microsoft SharePoint
- Microsoft 商務用 Skype
- Microsoft StaffHub
- Microsoft Teams
- Microsoft Visio
- Microsoft Word
- Microsoft To-Do
- Microsoft Stream
- Microsoft Edge



**備註**

- 經過核准的用戶端應用程式支援 Intune 行動應用程式管理功能。

- **需要經過核准的用戶端應用程式**需求：

    - 僅支援[裝置平台條件](#device-platforms-condition)適用的 iOS 和 Android。


## <a name="next-steps"></a>後續步驟

- 如需條件式存取的概觀，請參閱[什麼是 Azure Active Directory 中的條件式存取？](../active-directory-conditional-access-azure-portal.md)
- 如果您已準備好設定您環境的條件式存取原則，請參閱 [Azure Active Directory 中條件式存取的建議做法](best-practices.md)。



<!--Image references-->
[1]: ./media/technical-reference/01.png


