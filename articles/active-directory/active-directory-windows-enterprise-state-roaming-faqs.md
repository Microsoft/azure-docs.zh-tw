---
title: 設定和資料漫遊常見問題集 | Microsoft Docs
description: 回答 IT 系統管理員可能會遇到的設定和應用程式資料同步處理的一些問題。
services: active-directory
keywords: 企業狀態漫遊設定, windows 雲端, 企業狀態漫遊常見問題集
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: curtand
ms.component: devices
ms.assetid: c0824f5c-129b-4240-969f-921f6a64eae7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2018
ms.author: markvi
ms.openlocfilehash: 39d3913013d9528405191c5f052a943f1869048a
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44026851"
---
# <a name="settings-and-data-roaming-faq"></a>設定和資料漫遊常見問題集
本文將回答 IT 系統管理員可能會遇到的設定和應用程式資料同步處理的一些問題。

## <a name="what-data-roams"></a>哪些資料會漫遊？
**Windows 設定**：Windows 作業系統內建的電腦設定。 一般而言，這些是將您電腦個人化的設定，這些設定包含下列廣泛的類別：

* *佈景主題*- 包括桌面主題和工作列設定之類的功能。
* *Internet Explorer 設定*- 包括最近開啟的索引標籤和我的最愛。
* *Edge 瀏覽器設定*- 例如我的最愛和閱讀清單。
* *密碼*- 包括網際網路密碼、Wi-Fi 設定檔等。
* *語言喜好設定*- 包括鍵盤配置、系統語言、日期和時間等的設定。
* *輕鬆存取功能*- 例如高對比佈景主題、朗讀程式及放大鏡。
* *其他 Windows 設定*，例如滑鼠設定。

**應用程式資料**：通用 Windows 應用程式可以將設定資料寫入漫遊資料夾中，而系統會對寫入這個資料夾中的任何資料自動進行同步處理。 由個別應用程式開發人員設計應用程式以充分利用這項功能。 如需有關如何開發使用漫遊功能之通用 Windows 應用程式的詳細資訊，請參閱 [AppData 儲存 API](https://msdn.microsoft.com/library/windows/apps/mt299098.aspx) 和 [Windows 8 AppData 漫遊開發人員部落格](http://blogs.msdn.com/b/windowsappdev/archive/2012/07/17/roaming-your-app-data.aspx)。

## <a name="what-account-is-used-for-settings-sync"></a>哪些帳戶用於設定同步處理？
在 Windows 8.1 中，設定同步處理一律使用取用者 Microsoft 帳戶。 企業使用者有能力可以將 Microsoft 帳戶連接至 Active Directory 網域帳戶，以存取設定同步處理。在 Windows 10 中，這項連接的 Microsoft 帳戶功能將由主要/次要帳戶架構取代。

主要帳戶的定義是用來登入 Windows 的帳戶。 這可以是 Microsoft 帳戶、Azure Active Directory (Azure AD) 帳戶、內部部署 Active Directory 帳戶或本機帳戶。 除了主要帳戶，Windows 10 使用者可以將一或多個次要雲端帳戶新增至他們的裝置。 次要帳戶通常是 Microsoft 帳戶、Azure AD 帳戶或一些像是 Gmail 或 Facebook 的其他帳戶。 這些次要帳戶提供其他服務 (例如單一登入和 Windows 市集) 的存取權，但是無法進行設定同步處理。

在 Windows 10 中，只有裝置的主要帳戶可用於設定同步處理 (請參閱[我該如何從 Windows 8 中的 Microsoft 帳戶設定同步處理升級至 Windows 10 中的 Azure AD 設定同步處理？](active-directory-windows-enterprise-state-roaming-faqs.md#how-do-i-upgrade-from-microsoft-account-settings-sync-in-windows-8-to-azure-ad-settings-sync-in-windows-10))。

裝置上不同使用者帳戶之間的資料永遠不會混合。 有兩個設定可用來同步處理設定：

* Windows 設定一律會使用主要帳戶進行漫遊。
* 應用程式資料將會利用用來取得應用程式的帳戶來標記。 只會同步處理以主要帳戶標記的應用程式。透過 Windows 市集或行動裝置管理 (MDM) 側載應用程式時，會決定應用程式擁有權標記。

如果無法識別應用程式的擁有者，它會以主要帳戶漫遊。 如果裝置是從 Windows 8 或 Windows 8.1 升級到 Windows 10，則所有應用程式都會被標記成由 Microsoft 帳戶取得。 這是因為大多數使用者都是透過 Windows 市集取得應用程式，而在 Windows 10 之前，並沒有為 Azure AD 帳戶提供 Windows 市集支援。 如果應用程式是透過離線授權安裝，則會使用裝置上的主要帳戶標記應用程式。

> [!NOTE]
> 企業擁有且連接到 Azure AD 的 Windows 10 裝置無法再將它們的 Microsoft 帳戶連接到網域帳戶。 將會從已加入所連接之 Active Directory 或 Azure AD 環境的 Windows 10 裝置中，移除可將 Microsoft 帳戶連接到網域帳戶並將所有使用者資料同步至 Microsoft 帳戶 (亦即透過已連接的 Microsoft 帳戶和 Active Directory 功能進行的 Microsoft 帳戶漫遊) 的能力。
>
>

## <a name="how-do-i-upgrade-from-microsoft-account-settings-sync-in-windows-8-to-azure-ad-settings-sync-in-windows-10"></a>我該如何從 Windows 8 中的 Microsoft 帳戶設定同步處理升級至 Windows 10 中的 Azure AD 設定同步處理？
如果您加入了執行 Windows 8.1 且具有已連接的 Microsoft 帳戶的 Active Directory 網域，您將會透過您的 Microsoft 帳戶來同步處理設定。 升級至 Windows 10 之後，只要您是已加入網域的使用者且 Active Directory 網域未與 Azure AD 連接，您就會繼續透過 Microsoft 帳戶來同步處理使用者設定。

如果內部部署 Active Directory 網域與 Azure AD 連接，您的裝置將會嘗試使用連接的 Azure AD 帳戶來同步處理設定。 如果 Azure AD 系統管理員未啟用「企業狀態漫遊」，連接的 Azure AD 帳戶將會停止同步處理設定。 如果您是 Windows 10 使用者且使用 Azure AD 身分識別來登入，則在您的系統管理員啟用透過 Azure AD 進行設定同步處理的功能之後，您將會立即開始同步處理 Windows 設定。

如果您將任何個人資料儲存在公司裝置上，您應該注意 Windows OS 和應用程式資料會開始同步至 Azure AD。 這有下列含意：

* 您的個人 Microsoft 帳戶設定將會與您工作或學校 Azure AD 帳戶上的設定差距拉大。 這是因為 Microsoft 帳戶和 Azure AD 設定同步處理現在使用不同的帳戶。
* 個人資料 (例如 Wi-Fi 密碼、Web 認證，以及先前透過已連接的 Microsoft 帳戶同步處理的 Internet Explorer 我的最愛) 將會透過 Azure AD 進行同步處理。

## <a name="how-do-microsoft-account-and-azure-ad-enterprise-state-roaming-interoperability-work"></a>Microsoft 帳戶和 Azure AD 企業狀態漫遊互通性如何運作？
在 Windows 10 的 2015 年 11 月版或更新版本中，一次只針對一個帳戶支援「企業狀態漫遊」。 如果您使用工作或學校 Azure AD 帳戶登入 Windows，則所有資料都將透過 Azure AD 進行同步處理。 如果您使用個人 Microsoft 帳戶登入 Windows，則所有資料都會透過 Microsoft 帳戶進行同步處理。 通用 AppData 只會使用裝置上的主要登入帳戶進行漫遊，並且只有當應用程式的授權是由主要帳戶所擁有時才會漫遊。 任何次要帳戶所擁有的應用程式的通用 AppData 都不會進行同步處理。

## <a name="do-settings-sync-for-azure-ad-accounts-from-multiple-tenants"></a>對來自多個租用戶的 Azure AD 帳戶進行設定同步處理？
當來自不同 Azure AD 租用戶的多個 Azure AD 帳戶位於相同的裝置時，您必須更新裝置的登錄，才能與每個 Azure AD 租用戶的 Azure Rights Management 服務進行通訊。  

1. 尋找每個 Azure AD 租用戶的 GUID。 開啟 Azure 入口網站，然後選取 Azure AD 租用戶。 租用戶的 GUID 是在所選租用戶的 [屬性] 頁面 (https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties) 上，其標示為 [目錄識別碼]。 
2. 取得 GUID 之後，您必須新增登錄機碼 **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\SettingSync\WinMSIPC\<租用戶識別碼 GUID>**。
   從「租用戶識別碼 GUID」機碼，建立一個名為 **AllowedRMSServerUrls** 的新多字串值 (REG-MULTI-SZ)。 針對其資料，指定裝置所存取之其他 Azure 租用戶的授權發佈點 URL。
3. 您可以藉由從 AADRM 模組執行 **Get-AadrmConfiguration** Cmdlet 來尋找授權發佈點 URL。 如果 **LicensingIntranetDistributionPointUrl** 與 **LicensingExtranetDistributionPointUrl** 的值不同，則請同時指定這兩個值。 如果值相同，則只要指定值一次。

## <a name="what-are-the-roaming-settings-options-for-existing-windows-desktop-applications"></a>適用於現有 Windows 傳統型應用程式的漫遊設定選項有哪些？
漫遊僅適用於通用 Windows 應用程式。 有兩個選項可用來在現有 Windows 傳統型應用程式上啟用漫遊：

* [傳統型應用程式橋接器](https://aka.ms/desktopbridge) 可協助您輕鬆地將現有的 Windows 傳統型應用程式帶到「通用 Windows 平台」。 從這裡，只需進行最低限度的程式碼變更，即可利用 Azure AD 的應用程式資料漫遊功能。 桌面橋接器會為應用程式提供應用程式身分識別，需要有此身分識別才能為現有的傳統型應用程式啟用應用程式資料漫遊。
* [User Experience Virtualization (UE-V)](https://technet.microsoft.com/library/dn458947.aspx) 可協助您為現有的 Windows 傳統型應用程式建立自訂設定範本，並為 Win32 應用程式啟用漫遊功能。 這個選項不需要應用程式開發人員變更應用程式的程式碼。 UE-V 僅限於已購買 Microsoft Desktop Optimization Pack 之客戶的內部部署 Active Directory 漫遊。

系統管理員可以設定讓 UE-V 漫遊 Windows 傳統型應用程式資料，方法是透過 [UE-V 群組原則](https://technet.microsoft.com/itpro/mdop/uev-v2/configuring-ue-v-2x-with-group-policy-objects-both-uevv2)變更 Windows OS 設定與通用應用程式資料的漫遊，包括：

* 「漫遊 Windows 設定」群組原則
* 「不要同步處理 Windows 應用程式」群組原則
* 應用程式區段中的 Internet Explorer 漫遊

在未來，Microsoft 可能會調查讓 UE-V 深度整合到 Windows 及透過 Azure AD 雲端擴充 UE-V 漫遊設定的方式。

## <a name="can-i-store-synced-settings-and-data-on-premises"></a>我可以將同步處理的設定和資料儲存在內部部署環境中嗎？
「企業狀態漫遊」會將所有同步處理的資料儲存在 Microsoft 雲端。 UE-V 提供一個內部部署漫遊解決方案。

## <a name="who-owns-the-data-thats-being-roamed"></a>誰擁有正在進行漫遊的資料？
企業擁有透過企業狀態漫遊來漫遊的資料。 資料會儲存在 Azure 資料中心。 在傳輸過程中和在雲端待用的所有使用者資料，都會使用來自 Azure 資訊保護的 Azure Rights Management 服務進行加密。 相較於以 Microsoft 帳戶為基礎的設定同步處理 (只會針對某些機密資料 (例如使用者認證) 在其離開裝置前進行加密)，這是一大改進。

Microsoft 致力於保護客戶資料。 企業使用者的設定資料在離開 Windows 10 裝置前會自動由 Azure Microsoft Azure AD Rights Management 服務加密，因此沒有任何其他使用者可以讀取此資料。 如果貴組織擁有 Azure Rights Management 服務的付費訂用帳戶，您可以使用其他保護功能，例如追蹤及撤銷文件、自動保護包含機密資訊的電子郵件，以及管理您自己的金鑰 (「自備金鑰」解決方案，也稱為 BYOK)。 如需有關這些功能以及此保護服務運作方式的詳細資訊，請參閱[什麼是 Azure Rights Management](/azure/information-protection/what-is-information-protection)。

## <a name="can-i-manage-sync-for-a-specific-app-or-setting"></a>我可以管理特定應用程式或設定的同步處理嗎？
在 Windows 10 中，沒有 MDM 或群組原則設定可以停用個別應用程式的漫遊。 租用戶系統管理員可以停用受控裝置上所有應用程式的 AppData 同步處理，但是在每個應用程式或應用程式內的層級並沒有更細微的控制。

## <a name="how-can-i-enable-or-disable-roaming"></a>如何啟用或停用漫遊功能？
在 [設定] 應用程式中，移至 [帳戶] > [同步您的設定]。 您可以從這個頁面看見正在使用哪一個帳戶漫遊設定，您可以啟用或停用要漫遊得個別群組設定。

## <a name="what-is-microsofts-recommendation-for-enabling-roaming-in-windows-10"></a>Microsoft 對於在 Windows 10 中啟用漫遊功能有什麼建議？
Microsoft 提供數個不同的設定漫遊解決方案可供使用，包括漫遊使用者設定檔、UE-V 和企業狀態漫遊。  Microsoft 承諾將在未來的 Windows 版本中，致力於發展企業狀態漫遊。 如果貴組織尚未準備就緒或不想要將資料移到雲端，則建議您使用 UE-V 做為主要的漫遊技術。 如果貴組織既需要適用於現有 Windows 傳統型應用程式的漫遊支援，又迫切想要移到雲端，建議您同時使用「企業狀態漫遊」和 UE-V。 雖然 UE-V 和「企業狀態漫遊」是非常類似的技術，但是它們並不互斥。 它們可彼此互補以協助確保貴組織提供您使用者所需的漫遊服務。  

使用企業狀態漫遊和 UE-V 時，適用下列規則：

* 企業狀態漫遊是裝置上主要的漫遊代理程式。 UE-V 是用來補充「Win32 間距」。
* 使用 UE-V 群組原則時，應該停用 Windows 設定和現代化 UWP 應用程式資料的 UE-V 漫遊。 「企業狀態漫遊」已經涵蓋這些漫遊。

## <a name="how-does-enterprise-state-roaming-support-virtual-desktop-infrastructure-vdi"></a>企業狀態漫遊如何支援虛擬桌面基礎結構 (VDI)？
Windows 10 用戶端 SKU 支援「企業狀態漫遊」，但伺服器 SKU 則不支援。 如果用戶端 VM 裝載於 Hypervisor 電腦上，而您是從遠端登入虛擬機器，您的資料就會漫遊。 如果有多位使用者共用相同的作業系統，而使用者是從遠端登入伺服器以獲得完整桌面體驗，漫遊可能會無法運作。 針對後者以工作階段為基礎的案例，並未正式支援。

## <a name="what-happens-when-my-organization-purchases-a-subscription-that-includes-azure-rights-management-after-using-roaming"></a>我的組織在使用漫遊之後，購買包含 Azure Rights Management 的訂用帳戶會發生什麼情況？
如果貴組織已經藉由有使用限制的 Azure Rights Management 免費訂用帳戶在 Windows 10 中使用漫遊，購買包含 Azure Rights Management 保護服務的[付費訂用帳戶](https://azure.microsoft.com/pricing/details/information-protection/)不會對漫遊功能造成任何影響，且您的 IT 系統管理員也不需要進行任何組態變更。

## <a name="known-issues"></a>已知問題
如需已知問題清單，請參閱[疑難排解](active-directory-windows-enterprise-state-roaming-troubleshooting.md)一節中的文件。 

## <a name="related-topics"></a>相關主題
* [企業狀態漫遊概觀](active-directory-windows-enterprise-state-roaming-overview.md)
* [在 Azure Active Directory 中啟用企業狀態漫遊](active-directory-windows-enterprise-state-roaming-enable.md)
* [設定同步處理的群組原則和 MDM 設定](active-directory-windows-enterprise-state-roaming-group-policy-settings.md)
* [Windows 10 漫遊設定參考](active-directory-windows-enterprise-state-roaming-windows-settings-reference.md)
* [疑難排解](active-directory-windows-enterprise-state-roaming-troubleshooting.md)
