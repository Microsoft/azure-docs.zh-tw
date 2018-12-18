---
title: 如何使用 Azure 入口網站來管理裝置 | Microsoft Docs
description: 了解如何使用 Azure 入口網站來管理裝置。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/25/2018
ms.author: markvi
ms.reviewer: jairoc
ms.openlocfilehash: ff1d51021038909c132bef4cb680589b9951f218
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041420"
---
# <a name="how-to-manage-devices-using-the-azure-portal"></a>如何使用 Azure 入口網站來管理裝置


使用 Azure Active Directory (Azure AD) 中的裝置管理，您可以確保使用者會從符合安全性與合規性之標準的裝置來存取您的資源。 

本文：

- 假設您熟悉 [Azure Active Directory 中的裝置管理簡介](overview.md)

- 為您提供關於使用 Azure 入口網站管理裝置的資訊

## <a name="manage-devices"></a>管理裝置 

Azure 入口網站可提供您一個集中管理裝置的位置。 您可以使用[直接連結](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices)或依照下列手動步驟，來前往該位置：

1. 以系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。

2. 在左側導覽列上，按一下 [Active Directory]。

    ![設定裝置設定](./media/device-management-azure-portal/01.png)

3. 在 [管理] 區段中，按一下 [裝置]。

    ![設定裝置設定](./media/device-management-azure-portal/74.png)
 
[裝置] 頁面可讓您：

- 設定裝置管理設定

- 尋找裝置

- 執行裝置管理工作

- 檢閱裝置管理相關的稽核記錄  
  

## <a name="configure-device-settings"></a>設定裝置設定

若要使用 Azure 入口網站管理裝置，您的裝置必須已向 Azure AD [註冊或已加入](overview.md#getting-devices-under-the-control-of-azure-ad) Azure AD。 身為系統管理員，您可以透過設定裝置設定，來微調註冊和加入裝置的程序。 

![設定裝置設定](./media/device-management-azure-portal/22.png)

[裝置設定] 頁面可讓您設定：

![管理 Intune 裝置](./media/device-management-azure-portal/21.png)


- **使用者可以將裝置加入 Azure AD** - 這項設定可讓您選取哪些使用者能夠[將裝置加入](overview.md#azure-ad-joined-devices) Azure AD。 預設值是 [全部]。 此設定僅適用於 Windows 10 上的 Azure AD Join。

- **加入 Azure AD 的裝置上其他本機系統管理員** - 您可以選取哪些使用者會授與裝置的本機系統管理員權限。 新增至此處的使用者將會新增至 Azure AD 中的「裝置系統管理員」角色。 Azure AD 中的全域管理員和裝置擁有者預設會授與本機系統管理員權限。 此選項是可透過 Azure AD Premium 或 Enterprise Mobility Suite (EMS) 等產品使用的進階編輯功能。 

- **使用者可以向 Azure AD 註冊其裝置** - 您需要設定這項設定，才能向 Azure AD [註冊](overview.md#azure-ad-registered-devices)裝置。 如果您選取 [無]，當裝置未加入 Azure AD 或混合式 Azure AD 時，則不允許註冊這些裝置。 需要先註冊 (registration)，才可註冊 (enrollment) Microsoft Intune 或適用於 Office 365 的行動裝置管理 (MDM)。 如果您已設定任一服務，則會選取 [全部] 且無法使用 [無]。

- **需要 Multi-Factor Auth 才能加入裝置** - 您可以選擇使用者是否需要提供次要驗證因素，才能將其裝置[加入](overview.md#azure-ad-joined-devices) Azure AD。 預設值為 [ **否**]。 建議在註冊裝置時要求 Multi-Factor Authentication。 啟用此服務的 Multi-Factor Authentication 之前，您必須確定已為註冊其裝置的使用者設定 Multi-Factor Authentication。 如需不同 Azure Multi-Factor Authentication 服務的詳細資訊，請參閱[開始使用 Azure Multi-Factor Authentication](../authentication/concept-mfa-whichversion.md)。 此設定不會影響 Windows 10 或 Windows 7 的混合式聯結。 這僅適用於 Windows 10 上的 Azure AD Join 以及 Windows 10、iOS 和 Android 的 BYO 裝置註冊。 

- **裝置數目上限** - 這項設定可讓您選取使用者可在 Azure AD 中擁有的裝置數目上限。 如果使用者達到此配額，則在移除一或多個現有的裝置之前，將無法新增其他裝置。 今日已加入 Azure AD 或 Azure AD 已註冊的所有裝置都會計入裝置配額。 預設值為 **20**。

- **使用者可以在裝置間同步設定及應用程式資料** - 根據預設，這項設定會設定為 [無]。 選取特定使用者或群組，或是選取 [全部]，以便在使用者的 Windows 10 裝置間同步其設定及應用程式資料。 深入了解同步在 Windows 10 中的運作方式。
此選項是可透過 Azure AD Premium 或 Enterprise Mobility Suite (EMS) 等產品使用的進階功能。
 




## <a name="locate-devices"></a>尋找裝置

您有兩個選項可用來尋找已註冊及已加入的裝置：

- [裝置] 頁面之 [管理] 區段中的 [所有裝置]  

    ![所有裝置](./media/device-management-azure-portal/41.png)


- [使用者] 頁面之 [管理] 區段中的 [裝置]
 
    ![所有裝置](./media/device-management-azure-portal/43.png)



您可以透過這兩個選項移至檢視，該檢視：


- 可讓您使用顯示名稱作為篩選條件來搜尋裝置。

- 為您提供已註冊和已加入裝置的詳細概觀

- 可讓您執行一般裝置管理工作
   

![所有裝置](./media/device-management-azure-portal/51.png)

就某些 iOS 裝置而言，包含單引號的裝置名稱可能會使用看起來像單引號的不同字元。 因此，搜尋這類裝置有點棘手 - 如果您沒有看到正確的搜尋結果，請確定搜尋字串包含相符的單引號字元。

## <a name="device-management-tasks"></a>裝置管理工作

身為系統管理員，您可以管理已註冊或已加入的裝置。 本節為您提供一般裝置管理工作的相關資訊。


### <a name="manage-an-intune-device"></a>管理 Intune 裝置

如果您是 Intune 管理員，您可以管理標示為 **Microsoft Intune** 的裝置。 系統管理員可能會看到其他裝置 

![管理 Intune 裝置](./media/device-management-azure-portal/31.png)


### <a name="enable--disable-an-azure-ad-device"></a>啟用/停用 Azure AD 裝置

若要啟用/停用裝置，您有兩個選項：

- [所有裝置] 頁面上的 [工作] 功能表 ("...")

    ![管理 Intune 裝置](./media/device-management-azure-portal/71.png)

- [裝置] 頁面上的工具列

    ![管理 Intune 裝置](./media/device-management-azure-portal/32.png)


**備註：**

- 您必須是 Azure AD 中的全域管理員，才能啟用/停用裝置。 
- 停用裝置可防止裝置存取您的 Azure AD 資源。 



### <a name="delete-an-azure-ad-device"></a>刪除 Azure AD 裝置

若要刪除裝置，您有兩個選項：

- [所有裝置] 頁面上的 [工作] 功能表 ("...")

    ![管理 Intune 裝置](./media/device-management-azure-portal/72.png)

- [裝置] 頁面上的工具列

    ![刪除裝置](./media/device-management-azure-portal/34.png)


**備註：**

- 您必須是 Azure AD 中的全域管理員或 Intune 系統管理員，才能刪除裝置。

- 刪除裝置：
 
    - 防止裝置存取您的 Azure AD 資源。 

    - 移除所有附加至裝置的詳細資料，例如適用於 Windows 裝置的 BitLocker 金鑰。  

    - 代表無法復原的活動，除非必要，否則不建議使用。

如果裝置已由另一個管理授權單位 (例如，Microsoft Intune) 管理，請確定已抹除/淘汰裝置，再於 Azure AD 中刪除裝置。

 


### <a name="view-or-copy-device-id"></a>檢視或複製裝置識別碼

您可以使用裝置識別碼來確認裝置上的裝置識別碼詳細資料，或在疑難排解期間使用 PowerShell。 若要存取複製選項，請按一下裝置。

![檢視裝置識別碼](./media/device-management-azure-portal/35.png)
  

### <a name="view-or-copy-bitlocker-keys"></a>檢視或複製 BitLocker 金鑰

您可以檢視和複製 BitLocker 金鑰，以協助使用者復原他們所加密的磁碟機。 這些金鑰只適用於已加密並將其金鑰儲存在 Azure AD 中的 Windows 裝置。 您可以在存取裝置的詳細資料時複製這些金鑰。
 
![檢視 BitLocker 金鑰](./media/device-management-azure-portal/36.png)

若要檢視或複製 BitLocker 金鑰，您必須是裝置的擁有者，或是至少已獲指派下列其中一個角色的使用者：

- 全域管理員
- 技術服務管理員
- 安全性系統管理員
- 安全性讀取者
- Intune 服務管理員

> [!NOTE]
> 已聯結混合式 Azure AD 的 Windows 10 裝置沒有擁有者。 因此，如果您依擁有者尋找裝置卻未找到，請依裝置識別碼來搜尋。


## <a name="audit-logs"></a>稽核記錄


您可以透過活動記錄來取得裝置活動。 這包括由裝置註冊服務和使用者所觸發的活動：

- 建立裝置，以及在裝置上新增擁有者/使用者

- 變更裝置設定

- 刪除或更新裝置等裝置作業
 
稽核資料的進入點是 [裝置] 頁面之 [活動] 區段中的 [稽核記錄]。

![稽核記錄](./media/device-management-azure-portal/61.png)


稽核記錄的預設清單檢視顯示︰

- 發生時間與日期

- 目標

- 活動的啟動者/執行者 (對象)

- 活動 (項目)

![稽核記錄](./media/device-management-azure-portal/63.png)

您可以按一下工具列中的 [資料行] 來自訂清單檢視。
 
![稽核記錄](./media/device-management-azure-portal/64.png)


若要將報告的資料縮小至您適用的層級，您可以使用下列欄位篩選稽核資料︰

- 類別
- 活動資源類型
- 活動
- 日期範圍
- 目標
- 啟動者 (執行者)

除了篩選，您還可以搜尋特定項目。

![稽核記錄](./media/device-management-azure-portal/65.png)

## <a name="next-steps"></a>後續步驟

* [Azure Active Directory 中的裝置管理簡介](overview.md)



