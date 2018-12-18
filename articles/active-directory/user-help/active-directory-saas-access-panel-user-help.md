---
title: 取得在 Azure Active Directory 中存取和使用 MyApps 入口網站的協助 | Microsoft Docs
description: 取得在存取面板中登入及執行常見工作的協助。
services: active-directory
author: eross-msft
manager: mtillman
ms.assetid: c67cd675-b567-41e1-8bc2-e06fe0b38d3b
ms.service: active-directory
ms.component: user-help
ms.workload: identity
ms.topic: conceptual
ms.date: 01/15/2018
ms.author: lizross
ms.reviewer: japere
ms.openlocfilehash: 5503681b051066f6601be97bda246128763e7bc7
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306405"
---
# <a name="troubleshoot-issues-with-accessing-and-using-the-myapps-portal"></a>針對存取和使用 MyApps 入口網站的問題進行疑難排解

如果您在登入或使用 MyApps 入口網站時遇到問題，請在連絡技術服務人員或系統管理員以取得協助之前，先嘗試下列疑難排解提示。

## <a name="i-am-having-trouble-signing-into-the-myapps-portal"></a>我無法登入 MyApps 入口網站

請嘗試下列一般提示：

- 首先，查看您所使用的 URL 是否正確：[https://myapps.microsoft.com](https://myapps.microsoft.com)。
- 嘗試將 URL 新增至瀏覽器的信任網站。
- 確定密碼正確，且尚未過期。 如需詳細資訊，請參閱[重設您的工作密碼或學校密碼](active-directory-passwords-update-your-own-password.md)。
- 檢查以確定驗證連絡資訊是最新的，並且不會封鎖存取。 如需詳細資訊，請參閱 [Azure Multi-Factor Authentication 對我有何意義？](https://docs.microsoft.com/azure/multi-factor-authentication/end-user/multi-factor-authentication-end-user)。
- 嘗試清除瀏覽器的 Cookie，然後嘗試再次登入。

如果您嘗試登入時仍遇到問題，請連絡系統管理員。

## <a name="i-seem-to-be-having-password-issues"></a>我似乎有密碼方面的問題

如果您忘記密碼、未曾收到 IT 人員給予的密碼、系統鎖定了您的帳戶或您想要變更密碼，請參閱[忘記 Azure AD 密碼的說明](active-directory-passwords-update-your-own-password.md)。

## <a name="i-need-to-register-for-password-reset"></a>我需要註冊密碼重設

您可以重設密碼或解除鎖定您的帳戶，而不需使用自助式密碼重設 (SSPR) 與某人對話。 在您可以使用這項功能之前，必須先註冊驗證方法，或確認系統管理員所需的預先定義驗證方法。 如需詳細資訊，請參閱[註冊自助式密碼重設](active-directory-passwords-reset-register.md)。

## <a name="i-am-having-trouble-installing-the-my-apps-secure-sign-in-extension"></a>我無法安裝 My Apps 安全登入延伸模組

MyApps 入口網站需要支援 JavaScript 且已啟用 CSS 的瀏覽器。 如果您使用密碼單一登入應用程式，就必須一併安裝隨附的延伸模組。 當您啟動已設定密碼單一登入的應用程式時，就會自動下載此擴充功能。

請檢查以確定您符合下列瀏覽器需求：

- **Edge**：在 Windows 10 Anniversary Edition 或更新版本上。
- **Chrome**：在 Windows 7 或更新版本，和在 MacOS X 或更新版本上。
- **Firefox 26.0 或更新版本**：在 Windows XP SP2 或更新版本，和在 Mac OS X 10.6 或更新版本上。
- **Internet Explorer 11**：在 Windows 7 或更新版本上 (部分支援)。

您也可以直接從下列網站下載擴充功能：

- [Chrome](https://go.microsoft.com/fwlink/?linkid=866367)
- [Edge](https://go.microsoft.com/fwlink/?linkid=845176)
- [Firefox](https://go.microsoft.com/fwlink/?linkid=866366)

如果您已安裝擴充功能，但仍遇到問題，請嘗試下列各項：

- 檢查您的瀏覽器擴充功能設定以確定擴充功能已啟用。
- 重新啟動瀏覽器，然後登入 MyApps 入口網站。
- 清除瀏覽器的 Cookie，然後登入 MyApps 入口網站。
- 如需存取診斷工具和設定 Internet Explorer 擴充功能的逐步指示，請參閱[針對 Internet Explorer 的存取面板擴充功能進行疑難排解](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ie-troubleshooting)。

## <a name="use-the-my-apps-secure-sign-in-extension"></a>使用 My Apps 安全登入擴充功能
* 如果您是使用 `https://myapps.microsoft.com` 以外的 My Apps URL，請執行下列作業來設定您的預設 URL：
   1. 在尚未登入擴充功能時，以滑鼠右鍵按一下擴充功能圖示。
   2. 在功能表上，選取 [My Apps URL]。
   3. 選取預設 URL。
   4. 選取擴充功能圖示。
   5. 若要登入擴充功能，請選取 [登入以開始使用]。

* 若要直接從瀏覽器登入應用程式，請執行下列作業：
   1. 安裝擴充功能後，請選取 [登入以開始使用] 來登入擴充功能。
   2. 使用登入 URL 來登入應用程式。  
       登入 URL 通常是顯示登入表單之應用程式的 URL。
      該擴充功能應該會變更狀態，並讓您知道有密碼可用。
   3. 若要登入，請選取擴充功能圖示。

* 若要從擴充功能啟動應用程式，請執行下列動作：
   1. 安裝擴充功能後，請選取 [登入以開始使用] 來登入擴充功能。
   2. 選取擴充功能圖示將其功能表開啟。
   3. 在 MyApps 入口網站中搜尋可用的應用程式。
   4. 在搜尋結果清單中，選取應用程式。  
       您所使用的最後三個應用程式會顯示在 [最近使用的] 捷徑清單中。

> [!NOTE]
> 這些選項僅適用於 Edge、Chrome 和 Firefox。

## <a name="how-do-i-add-a-new-app"></a>如何新增應用程式？

1.  在 [應用程式] 頁面上，選取 [新增應用程式]。
2.  搜尋您想要新增的應用程式，然後選取 [新增]。

   > [!NOTE]
   > * 只有當系統管理員已為您的帳戶啟用此選項時，您才可以存取此選項。
   > * 如果應用程式需要權限，您可能需要等待系統管理員核准。

## <a name="how-do-i-manage-my-group-memberships"></a>如何管理我的群組成員資格？

選取 [群組] 圖格，然後執行下列其中一個動作：
* 若要建立群組，請在 [我擁有的群組] 底下，選取 [建立群組]，然後依照指示進行。
* 若要加入群組，請在 [我所在的群組] 底下，選取 [加入群組]，然後依照指示進行。

   > [!NOTE]
   > * 只有當系統管理員已為您的帳戶啟用此選項時，您才可以存取此選項。
   > * 如果您是群組的成員，您可以檢視詳細資料及脫離群組。
   > * 如果您是群組的擁有者，則您可以檢視詳細資料、新增或移除成員，以及脫離群組。