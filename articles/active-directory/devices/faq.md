---
title: Azure Active Directory 裝置管理常見問題集 | Microsoft Docs
description: Azure Active Directory 裝置管理常見問題集。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: cdc25576-37f2-4afb-a786-f59ba4c284c2
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.author: markvi
ms.reviewer: jairoc
ms.openlocfilehash: fd2505d6262948e193531d91222cdec319f33100
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39597148"
---
# <a name="azure-active-directory-device-management-faq"></a>Azure Active Directory 裝置管理常見問題集

**問：可以註冊 Android 或 iOS BYOD 裝置嗎？**

**答：** 可以，但只能使用 Azure 裝置註冊服務且僅限混合式客戶使用。 不支援使用 AD FS 中的內部部署裝置註冊服務。

**問：我要如何註冊 macOS 裝置？**

**答：** 若要註冊 macOS 裝置：

1.  [建立裝置相容性原則](https://docs.microsoft.com/intune/compliance-policy-create-mac-os)
2.  [定義 macOS 裝置的條件式存取原則](../active-directory-conditional-access-azure-portal.md) 

**備註：**

- 條件式存取原則包含的使用者需要[支援的 macOS 版 Office](../conditional-access/technical-reference.md#client-apps-condition) 才能存取資源。 

- 在首次存取嘗試期間，系統會提示使用者使用公司入口網站註冊裝置。

---

**問：我最近註冊了裝置。為什麼在 Azure 入口網站中我的使用者資訊底下看不到該裝置？**

**答：** 已加入混合式 Azure AD 的 Windows 10 裝置不會顯示在 [使用者裝置] 底下。
您需要在 Azure 入口網站中使用 [所有裝置] 檢視。 您也可以使用 PowerShell [Get-MsolDevice](/powershell/module/msonline/get-msoldevice?view=azureadps-1.0) Cmdlet。

只有下列裝置會在 [使用者裝置] 底下列出：

- 所有未加入混合式 Azure AD 的個人裝置。 
- 所有非 Windows 10/Windows Server 2016 裝置。
- 所有非 Windows 裝置 

--- 

**問：我要如何知道用戶端的裝置註冊狀態為何？**

**答：** 您可以使用 Azure 入口網站，移至 [所有裝置]，然後使用裝置識別碼來搜尋裝置。 檢查加入類型資料行中的值。

如果您想要從已註冊的裝置檢查本機裝置註冊狀態：

- 針對 Windows 10 和 Windows Server 2016 或更新的裝置，請執行 dsregcmd.exe /status。
- 舊版作業系統版本請執行 "%programFiles%\Microsoft Workplace Join\autoworkplace.exe"。

---

**問：我已在 Azure 入口網站中刪除或使用 Windows PowerShell 刪除，但為什麼裝置上的本機狀態指出其仍處於註冊狀態？**

**答：** 原先的設計就是如此。 該裝置將無法存取雲端中的資源。 

如果您想要重新註冊，就必須對該裝置採取手動動作。 

若要清除加入狀態，針對已加入內部部署 AD 網域的 Windows 10 與 Windows Server 2016，做法是：

1.  以系統管理員身分開啟命令提示字元。

2.  輸入 `dsregcmd.exe /debug /leave`

3.  登出再登入，以觸發向 Azure AD 重新註冊裝置的排定工作。 

針對已加入內部部署 AD 網域的舊版 Windows 作業系統版本，做法是：

1.  以系統管理員身分開啟命令提示字元。
2.  輸入 `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe /l"`。
3.  輸入 `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe /j"`。

---
**問：如何在本機裝置上退出已加入 Azure AD 的裝置？**

**答：** 
- 針對已加入 Azure AD 的混合式裝置，請務必關閉自動註冊，這樣的話，已排定的工作就不會再次註冊裝置。 接著，以系統管理員身分開啟命令提示字元，然後輸入 `dsregcmd.exe /debug /leave`。 或者，可跨多個裝置以指令碼方式執行此命令，以進行大量退出。

- 針對純粹已加入 Azure AD 的裝置，確定您具有離線的本機系統管理員帳戶 (若無帳戶則請建立)，因為您將無法使用任何 Azure AD 使用者認證進行登入。 接下來，移至 [設定] > [帳戶] > [存取公司或學校資源]。 選取您的帳戶，然後按一下 [中斷連線]。 遵循提示，然後在系統提示您時提供本機系統管理員認證。 重新啟動裝置以完成退出程序。

---

**問：我的使用者無法從已加入 Azure AD 的裝置搜尋印表機。如何從已加入 Azure AD 的裝置啟用列印功能？**

**答：** 關於如何為已加入 Azure AD 的裝置部署印表機，請參閱[混合式雲端列印](https://docs.microsoft.com/en-us/windows-server/administration/hybrid-cloud-print/hybrid-cloud-print-deploy) (英文)。 您需要內部部署 Windows Server 才能部署混合式雲端列印。 目前尚無法使用雲端式列印服務。 

---

**問：如何連線到遠端已加入 Azure AD 的裝置？**
**答：** 如需詳細資訊，請參閱這篇文章 https://docs.microsoft.com/en-us/windows/client-management/connect-to-remote-aadj-pc。

---

**問：為什麼我在 Azure 入口網站中看到重複的裝置項目？**

**答：**

-   就 Windows 10 和 Windows Server 2016 而言，如果重複嘗試將同一個裝置取消加入再重新加入，就可能出現重複的項目。 

-   如果您使用了 [新增工作或學校帳戶]，則每個使用 [新增工作或學校帳戶] 的 Windows 使用者都會以相同的裝置名稱建立一個新的裝置記錄。

-   已使用自動註冊加入內部部署 AD 網域的舊版 Windows 作業系統版本，將會針對登入裝置的每個網域使用者，以相同的裝置名稱建立一個新的裝置記錄。 

-   已清除、重新安裝，然後以相同名稱重新加入之已加入 Azure AD 的電腦將顯示為另一個具有相同裝置名稱的記錄。

---

**問：為什麼使用者仍然能夠從我已在 Azure 入口網站中停用的裝置存取資源？**

**答：** 套用撤銷最多可能需要一小時的時間才能完成。

>[!Note] 
>針對已註冊的裝置，建議您將裝置清除，以確保使用者無法存取該資源。 如需詳細資訊，請參閱[註冊裝置以在 Intune 中管理](https://docs.microsoft.com/intune/deploy-use/enroll-devices-in-microsoft-intune)。 


---

**問：為什麼我的使用者會看到「您無法從這裡前往該處」？**

**答：** 如果您已設定某些條件式存取規則來要求特定的裝置狀態，而該裝置未符合這些準則，使用者就會遭封鎖並看到此訊息。 請重新評估條件式存取原則規則，確保該裝置能夠符合準則，以避免出現此訊息。

---

**問：為什麼我的使用者並未在已加入 Azure AD 的裝置上看見 MFA 提示？**

**答：** 如果使用者是以多重要素驗證向 Azure AD 加入或註冊裝置，裝置本身即會成為該特定使用者受信任的第二個因素。 此後，只要同一位使用者登入裝置並存取應用程式，Azure AD 會將該裝置視為第二個因素，並且讓該使用者順暢地存取應用程式，而完全不會出現其他 MFA 提示。 此行為不適用於登入該裝置的任何其他使用者，因此，存取該裝置的其他所有使用者在存取需要 MFA 的應用程式之前，仍會收到需進行 MFA 的提示。

---

**問：我在 Azure 入口網站中的 [使用者資訊] 底下看到裝置記錄，而且可以看到狀態為已在裝置上註冊。我是否已針對使用條件式存取進行正確設定？**

**答：** deviceID 反映出來的裝置加入狀態必須與 Azure AD 上的狀態相符，並滿足所有條件式存取的評估準則。 如需詳細資訊，請參閱[透過條件式存取要求必須從受控裝置存取雲端應用程式](../conditional-access/require-managed-devices.md)。

---

**問：為什麼針對剛加入 Azure AD 的裝置，我收到「使用者名稱或密碼不正確」訊息？**

**答：** 此情況的常見原因如下：

- 您的使用者認證已經無效。

- 您的電腦無法與 Azure Active Directory 通訊。 請檢查是否有任何網路連線問題。

- 同盟登入會要求您的同盟伺服器必須支援 WS-Trust 作用中端點。 

- 您已啟用傳遞驗證，而且使用者必須在登入時變更暫時密碼。

---

**問：當我嘗試將電腦加入 Azure AD 時，為什麼會看到「糟糕，發生錯誤!」？**

**答：** 這是使用 Intune 來設定 Azure Active Directory 註冊的結果。 請確定嘗試加入 Azure AD 的使用者已獲派正確的 Intune 授權。 如需詳細資訊，請參閱[設定 Windows 裝置管理](https://docs.microsoft.com/intune/deploy-use/set-up-windows-device-management-with-microsoft-intune#azure-active-directory-enrollment)。  

---

**問：為什麼當我嘗試將電腦加入時，雖然沒有收到任何錯誤資訊，但卻發生失敗？**

**答：** 可能是因為使用者使用本機內建的系統管理員帳戶來登入裝置。 在使用 Azure Active Directory Join 來完成設定之前，請先建立一個不同的本機帳戶。 


---

**問：哪裡可以找到有關自動裝置註冊的疑難排解資訊？**

**答：** 如需疑難排解資訊，請參閱：

- [針對已加入 Azure AD 網域之電腦的自動註冊進行疑難排解 – Windows 10 和 Windows Server 2016](troubleshoot-hybrid-join-windows-current.md)

- [針對已加入 Azure AD 網域之 Windows 下層用戶端電腦的自動註冊進行疑難排解](troubleshoot-hybrid-join-windows-legacy.md)
 

---

