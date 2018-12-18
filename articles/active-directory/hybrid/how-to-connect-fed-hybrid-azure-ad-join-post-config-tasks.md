---
title: Azure AD Connect：加入混合式 Azure AD 後置設定工作 |Microsoft Docs
description: 此文件詳盡說明要完成加入混合式 Azure AD 所需的後置設定工作
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: billmath
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/10/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 2306c03480956a63859355428cbffe0706f43cbc
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306637"
---
# <a name="post-configuration-tasks-for-hybrid-azure-ad-join"></a>加入混合式 Azure AD 的後置設定工作

在您執行 Azure AD Connect 為組織進行加入混合式 Azure AD 的設定後，您必須再執行若干步驟，才能完成該設定。  請執行您的裝置適用的步驟即可。

## <a name="1-configure-controlled-rollout-optional"></a>1.設定受控制的導入 (選擇性)
所有已加入網域且執行 Windows 10 和 Windows Server 2016 的裝置，都會在所有設定步驟皆完成後自動向 Azure AD 註冊。 如果您偏好的受控制的導入，而不是此自動註冊，您可以使用群組原則選擇性地啟用或停用自動導入。  此群組原則應在開始進行其他設定步驟之前設定：Azure AD
* 在您的 Active Directory 中建立群組原則物件。
* 為其命名 (例如，加入混合式 Azure AD)。
* 編輯並移至：[電腦設定] > [原則] > [系統管理範本] > [Windows 元件] > [裝置註冊]。

>[!NOTE]
>2012R2 的原則設定位於 [電腦設定] > [原則] > [系統管理範本] > [Windows 元件] > [加入工作場所] > [自動將用戶端電腦加入工作場所]

* 停用此設定：將加入網域的電腦註冊為裝置。
* 套用並按一下 [確定]。
* 將 GPO 連結至您選擇的位置 (組織單位、安全性群組，或所有裝置的網域)。

## <a name="2-configure-network-with-device-registration-endpoints"></a>2.以裝置註冊端點設定網路
請確定下列 URL 可從組織網路內的電腦存取，以註冊至 Azure AD：

* https://enterpriseregistration.windows.net
* https://login.microsoftonline.com
* https://device.login.microsoftonline.com 

## <a name="3-implement-wpad-for-windows-10-devices"></a>3.實作適用於 Windows 10 裝置的 WPAD
如果您的組織可透過輸出 Proxy 存取網際網路，請實作 Web Proxy 自動探索 (WPAD)，以便將 Windows 10 電腦註冊至 Azure AD。

## <a name="4-configure-the-scp-in-any-forests-that-were-not-configured-by-azure-ad-connect"></a>4.在未經 Azure AD Connect 設定的任何樹系中設定 SCP 

服務連接點 (SCP) 會包含您的裝置將用於自動註冊的 Azure AD 租用戶資訊。  請執行您從 Azure AD Connect 下載的 PowerShell 指令碼 ConfigureSCP.ps1。

## <a name="5-configure-any-federation-service-that-was-not-configured-by-azure-ad-connect"></a>5.設定任何未經 Azure AD Connect 設定的同盟服務

如果您的組織使用同盟服務登入 Azure AD，則您的 Azure AD 信賴憑證者信任中的宣告規則必須允許裝置驗證。 如果您使用與 AD FS 的同盟，請移至 [AD FS 說明](https://aka.ms/aadrptclaimrules)以產生宣告規則。 如果您使用非 Microsoft 同盟解決方案，請連絡該提供者以取得指引。  

>[!NOTE]
>如果您使用舊版的 Windows 裝置，則此服務必須能夠在接收對 Azure AD 信任的要求時發出 authenticationmethod 和 wiaormultiauthn 宣告。 在 AD FS 中，您應新增可傳遞驗證方法的發行轉換規則。

## <a name="6-enable-azure-ad-seamless-sso-for-windows-down-level-devices"></a>6.為舊版的 Windows 裝置啟用 Azure AD 無縫式 SSO

如果您的組織使用密碼雜湊同步處理或傳遞驗證來登入 Azure AD，請啟用該登入方法的 Azure AD 無縫式 SSO，以驗證舊版的 Windows 裝置： https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso。 

## <a name="7-set-azure-ad-policy-for-windows-down-level-devices"></a>7.為舊版的 Windows 裝置設定 Azure AD 原則

若要註冊舊版 Windows 裝置，您必須確定 Azure AD 原則允許使用者註冊裝置。 

* 在 Azure 入口網站中登入您的帳戶。
* 移至：[Azure Active Directory] > [裝置] > [裝置設定]
* 將 [使用者可以向 Azure AD 註冊其裝置] 設定為 [全部]。
* 按一下 [Save] \(儲存)。

## <a name="8-add-azure-ad-endpoint-to-windows-down-level-devices"></a>8.為舊版的 Windows 裝置新增 Azure AD 端點

將 Azure AD 裝置驗證端點新增至您舊版 Windows 裝置上的近端內部網路區域，以避免在驗證裝置時出現憑證提示： https://device.login.microsoftonline.com 

如果您使用[無縫式 SSO](https://aka.ms/hybrid/sso)，也請在該區域上啟用 [允許透過指令碼進行狀態列更新]，並新增下列端點： https://autologon.microsoftazuread-sso.com 

## <a name="9-install-microsoft-workplace-join-on-windows-down-level-devices"></a>9.在舊版的 Windows 裝置上安裝 Microsoft Workplace Join

此安裝程式會在裝置系統上建立執行於使用者內容中的排程工作。 此工作會在使用者登入 Windows 時觸發。 此工作會在使用整合式 Windows 驗證進行驗證之後，使用使用者認證以無訊息方式將裝置加入 Azure AD。 下載中心位於 https://www.microsoft.com/download/details.aspx?id=53554。 

## <a name="10-configure-group-policy-to-allow-device-registration"></a>10.設定允許裝置註冊的群組原則

* 在您的 Active Directory 中建立群組原則物件 (若尚未建立)。
* 為其命名 (例如，加入混合式 Azure AD)。
* 編輯並移至：[電腦設定] > [原則] > [系統管理範本] > [Windows 元件] > [裝置註冊]
* 啟用：將加入網域的電腦註冊為裝置
* 套用並按一下 [確定]。
* 將 GPO 連結至您選擇的位置 (組織單位、安全性群組，或所有裝置的網域)。

>[!NOTE]
>2012R2 的原則設定位於 [電腦設定] > [原則] > [系統管理範本] > [Windows 元件] > [加入工作場所] > [自動將用戶端電腦加入工作場所]

## <a name="next-steps"></a>後續步驟
[設定裝置回寫](how-to-connect-device-writeback.md)
