---
title: 在 Azure Active Directory 中設定內部部署條件式存取 | Microsoft Docs
description: 在 Windows Server 2012 R2 中使用 Active Directory Federation Services (AD FS) 來啟用對內部部署應用程式之條件式存取的逐步指南。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.component: devices
ms.assetid: 6ae9df8b-31fe-4d72-9181-cf50cfebbf05
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2018
ms.author: markvi
ms.reviewer: jairoc
ms.custom: seohack1
ms.openlocfilehash: ccb3c6b1349576c0fa91066f4e98f0a53f5bef45
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319587"
---
# <a name="setting-up-on-premises-conditional-access-by-using-azure-active-directory-device-registration"></a>使用 Azure Active Directory 裝置註冊來設定內部部署條件式存取

當您要求使用者在工作場所將其個人裝置加入至 Azure Active Directory (Azure AD) 裝置註冊服務時，他們的裝置便可標示為組織已知的裝置。 以下是一個逐步指南，說明如何在 Windows Server 2012 R2 中使用 Active Directory Federation Services (AD FS)，來啟用對內部部署應用程式的條件式存取。

> [!NOTE]
> 使用在 Azure Active Directory 裝置註冊服務條件式存取原則中註冊的裝置時，需要有 Office 365 授權或 Azure AD Premium 授權。 這些包括內部部署資源中的 AD FS 所強制執行的原則。
> 
> 如需有關內部部署之條件式存取案例的詳細資訊，請參閱 [從任何裝置加入工作場所網路，以在所有公司應用程式進行 SSO 和無縫銜接的第二要素驗證](https://technet.microsoft.com/library/dn280945.aspx)。

凡是購買 Azure Active Directory Premium 授權的客戶皆可使用這些功能。

## <a name="supported-devices"></a>支援的裝置

* 已加入網域的 Windows 7 裝置
* 個人和已加入網域的 Windows 8.1 裝置
* iOS 6 和更新版本 (適用於 Safari 瀏覽器)
* Android 4.0 或更新版本、Samsung GS3 或更新的手機、Samsung Note 2 或更新的平板電腦

## <a name="scenario-prerequisites"></a>案例必要條件

* Office 365 或 Azure Active Directory Premium 的訂用帳戶
* Azure Active Directory 租用戶
* Windows Server Active Directory (Windows Server 2008 或更新版本)
* Windows Server 2012 R2 中更新的結構描述
* Azure Active Directory Premium 的授權
* Windows Server 2012 R2 Federation Services，針對 Azure AD 的 SSO 設定
* Windows Server 2012 R2 Web 應用程式 Proxy 
* Microsoft Azure Active Directory Connect (Azure AD Connect) [(下載 Azure AD Connect)](http://www.microsoft.com/download/details.aspx?id=47594)
* 已驗證的網域

## <a name="known-issues-in-this-release"></a>此版本已知的問題

* 裝置型條件式存取原則會要求將裝置物件從 Azure Active Directory 寫回至 Active Directory。 將裝置物件寫回至 Active Directory 可能需要長達三小時的時間。
* 進行用戶端憑證驗證時，iOS 7 裝置一律會提示使用者選取憑證。
* iOS 8.3 之前的某些 iOS 8 版本無法運作。

## <a name="scenario-assumptions"></a>案例假設

此案例假設您有一個由 Azure AD 租用戶與內部部署 Active Directory 所組成的混合式環境。 這些租用戶應該以 Azure AD Connect 連接、具有已驗證的網域，以及具有用來進行 SSO 的 AD FS。 請使用下列檢查清單來協助您根據需求設定環境。

## <a name="checklist-prerequisites-for-conditional-access-scenario"></a>檢查清單︰條件式存取案例的必要條件

將 Azure AD 租用戶與內部部署 Active Directory 執行個體連接。

## <a name="configure-azure-active-directory-device-registration-service"></a>設定 Azure Active Directory 裝置註冊服務

使用本指南來為您的組織部署及設定 Azure Active Directory 裝置註冊服務。

本指南假設您已設定 Windows Server Active Directory 並已訂閱 Microsoft Azure Active Directory。 請參閱稍早所述的必要條件。

若要使用您的 Azure Active Directory 租用戶來部署 Azure Active Directory 裝置註冊服務，請依序完成下列檢查清單中的工作。 當參考連結將您導向概念性主題時，請在之後返回此檢查清單，以便繼續執行剩餘的工作。 有些工作包含案例驗證步驟，可協助您確認是否已成功完成該步驟。

## <a name="part-1-enable-azure-active-directory-device-registration"></a>第 1 部分：啟用 Azure Active Directory 裝置註冊

請依照檢查清單中的步驟來啟用及設定 Azure Active Directory 裝置註冊服務。

| Task | 參考 | 
| --- | --- |
| 在 Azure Active Directory 租用戶中啟用裝置註冊，以允許裝置加入工作場所。 預設不會為服務啟用 Azure Multi-Factor Authentication。 不過，建議您在註冊裝置時使用 Multi-Factor Authentication。 在 Active Directory 註冊服務中啟用 Multi-Factor Authentication 之前，請確定已為 Multi-Factor Authentication 提供者設定 AD FS。 |[啟用 Azure Active Directory 裝置註冊](active-directory-device-registration-get-started.md)| 
|裝置透過尋找已知的 DNS 記錄來探索您的 Azure Active Directory 裝置註冊服務。 請設定您的公司 DNS，以便讓裝置探索您的 Azure Active Directory 裝置註冊服務。 |[設定 Azure Active Directory 裝置註冊探索](active-directory-device-registration-get-started.md)| 


## <a name="part-2-deploy-and-configure-windows-server-2012-r2-active-directory-federation-services-and-set-up-a-federation-relationship-with-azure-ad"></a>第 2 部分：部署和設定 Windows Server 2012 R2 Active Directory Federation Services，以及設定與 Azure AD 的同盟關係

| Task | 參考 |
| --- | --- |
| 使用 Windows Server 2012 R2 結構描述延伸模組來部署 Active Directory Domain Services。 您不需要將任何網域控制站升級到 Windows Server 2012 R2。 結構描述升級是唯一的要求。 |[升級您的 Active Directory Domain Services 結構描述](#upgrade-your-active-directory-domain-services-schema) |
| 裝置透過尋找已知的 DNS 記錄來探索您的 Azure Active Directory 裝置註冊服務。 請設定您的公司 DNS，以便讓裝置探索您的 Azure Active Directory 裝置註冊服務。 |[準備您的 Active Directory 支援裝置](#prepare-your-active-directory-to-support-devices) |

## <a name="part-3-enable-device-writeback-in-azure-ad"></a>第 3 部分：在 Azure AD 中啟用裝置回寫

| Task | 參考 |
| --- | --- |
| 完成「在 Azure AD Connect 中啟用裝置回寫」的第 2 部分。 完成之後，請返回本指南。 |[在 Azure AD Connect 中啟用裝置回寫](hybrid/how-to-connect-device-writeback.md) |

## <a name="optional-part-4-enable-multi-factor-authentication"></a>[選擇性] 第 4 部分：啟用 Multi-Factor Authentication

強烈建議您設定數個 Multi-Factor Authentication 選項其中之一。 如果您想要要求 Multi-Factor Authentication，請參閱[選擇您的 Multi-Factor Authentication 安全性解決方案](authentication/concept-mfa-whichversion.md)。 它包含每個解決方案的描述，以及可協助您設定您所選解決方案的連結。

## <a name="part-5-verification"></a>第 5 部分：驗證

部署程序現已完成，您可以試試一些案例。 請使用下列連結來進行服務實驗以熟悉其功能。

| Task | 參考 |
| --- | --- |
| 使用 Azure Active Directory 裝置註冊服務將一些裝置加入到您的工作場所。 您可以將 iOS、Windows 及 Android 裝置加入。 |[使用 Azure Active Directory 裝置註冊服務將裝置加入到您的工作場所](#join-devices-to-your-workplace-using-azure-active-directory-device-registration) |
| 使用系統管理員入口網站來檢視及啟用或停用已註冊的裝置。 在這項工作中，您將使用系統管理員入口網站來檢視一些已註冊的裝置。 |[Azure Active Directory 裝置註冊服務概觀](active-directory-device-registration-get-started.md) |
| 確認裝置物件已從 Azure Active Directory 寫回至 Windows Server Active Directory。 |[確認將已註冊的裝置寫回至 Active Directory](#verify-registered-devices-are-written-back-to-active-directory) |
| 現在使用者可以註冊其裝置，您可以在 AD FS 中建立僅允許已註冊裝置的應用程式存取原則。 在這項工作中，您需建立應用程式存取規則和自訂拒絕存取訊息。 |[建立應用程式存取原則和自訂的拒絕存取訊息](#create-an-application-access-policy-and-custom-access-denied-message) |

## <a name="integrate-azure-active-directory-with-on-premises-active-directory"></a>整合 Azure Active Directory 與內部部署 Active Directory

**請參閱：**

- [整合您的內部部署目錄與 Azure Active Directory](hybrid/whatis-hybrid-identity.md) - 可檢閱概念性資訊。

- [自訂 Azure AD Connect 安裝](hybrid/how-to-connect-install-custom.md) - 可獲得安裝指示。


## <a name="upgrade-your-active-directory-domain-services-schema"></a>升級您的 Active Directory 網域服務結構描述

> [!NOTE]
> 將 Active Directory 結構描述升級之後，即無法回復此程序。 建議您先在測試環境中執行升級。
> 

1. 使用同時具備企業系統管理員和結構描述系統管理員權限的帳戶來登入網域控制站。
1. 將 **[media]\support\adprep** 目錄和子目錄複製到其中一個 Active Directory 網域控制站 (其中 **[media]** 是 Windows Server 2012 R2 安裝媒體的路徑)。
1. 從命令提示字元，瀏覽至 **adprep** 目錄並執行 **adprep.exe /forestprep**。 請遵循畫面上的指示完成結構描述升級。

## <a name="prepare-your-active-directory-to-support-devices"></a>準備您的 Active Directory 以支援裝置

> [!NOTE]
> 這是您必須執行的一次性作業，以準備 Active Directory 樹系支援裝置。 若要完成此程序，您必須以企業系統管理員權限登入，且您的 Active Directory 樹系必須具有 Windows Server 2012 R2 結構描述。
> 


### <a name="prepare-your-active-directory-forest"></a>準備您的 Active Directory 樹系

1. 在您的同盟伺服器上，開啟 Windows PowerShell 命令視窗，然後輸入 **Initialize-ADDeviceRegistration**。 
1. 收到 **ServiceAccountName**提示時，請輸入您選取做為 AD FS 服務帳戶的服務帳戶名稱。 如果是 gMSA 帳戶，請使用 **domain\accountname$** 格式來輸入帳戶。 對於網域帳戶，請使用格式 **domain\accountname**。

### <a name="enable-device-authentication-in-ad-fs"></a>在 AD FS 中啟用裝置驗證

1. 在您的同盟伺服器上，開啟 AD FS 管理主控台並移至 [AD FS] > [驗證原則]。
1. 在 [動作] 窗格上，選取 [編輯全域主要驗證]。
1. 勾選 [啟用裝置驗證]，然後選取 [確定]。
1. AD FS 預設會定期從 Active Directory 移除未使用的裝置。 使用 Azure Active Directory 裝置註冊服務時，請停用此工作，以便在 Azure 中管理裝置。

### <a name="disable-unused-device-cleanup"></a>停用未使用的裝置清除

在您的同盟伺服器上，開啟 Windows PowerShell 命令視窗，然後輸入 **Set-AdfsDeviceRegistration -MaximumInactiveDays 0**。

### <a name="prepare-azure-ad-connect-for-device-writeback"></a>準備 Azure AD Connect 以便裝置回寫

完成第 1 部分：準備 Azure AD Connect。

## <a name="join-devices-to-your-workplace-by-using-azure-active-directory-device-registration-service"></a>使用 Azure Active Directory 裝置註冊服務將裝置加入到您的工作場所

### <a name="join-an-ios-device-by-using-azure-active-directory-device-registration"></a>使用 Azure Active Directory 裝置註冊來加入 iOS 裝置

Azure Active Directory 裝置註冊針對 iOS 裝置會使用「空中下載設定檔」註冊程序。 當使用者使用 Safari 來連線到設定檔註冊 URL 時，此程序就會開始。 URL 格式如下所示：

`https://enterpriseregistration.windows.net/enrollmentserver/otaprofile/<yourdomainname>`

在此案例中，`yourdomainname` 是您使用 Azure Active Directory 來設定的網域名稱。 例如，如果您的網域名稱是 contoso.com，URL 就會如下所示：

`https://enterpriseregistration.windows.net/enrollmentserver/otaprofile/contoso.com`

有許多不同的方式可讓 URL 與您的使用者進行通訊。 例如，其中一個建議方式是在 AD FS 的自訂應用程式拒絕存取訊息中發佈此 URL。 即將推出的[建立應用程式存取原則和自訂的拒絕存取訊息](#create-an-application-access-policy-and-custom-access-denied-message)一節中將會說明此訊息。

### <a name="join-a-windows-81-device-by-using-azure-active-directory-device-registration"></a>使用 Azure Active Directory 裝置註冊來加入 Windows 8.1 裝置

1. 在 Windows 8.1 裝置上，選取 [電腦設定] > [網路] > [工作場所]。
1. 以 UPN 格式輸入您的使用者名稱；例如 **dan@contoso.com**。
1. 選取 [加入] 。
1. 出現提示時，使用您的認證登入。 裝置會隨即加入。

### <a name="join-a-windows-7-device-by-using-azure-active-directory-device-registration"></a>使用 Azure Active Directory 裝置註冊來加入 Windows 7 裝置

若要註冊已加入網域的 Windows 7 裝置，您必須部署[裝置註冊軟體套件](https://www.microsoft.com/download/details.aspx?id=53554)。

如需如何使用套件的相關指示，請參閱[非 Windows 10 電腦的 Windows Installer 套件](devices/hybrid-azuread-join-control.md#control-windows-down-level-devices)。

## <a name="verify-that-registered-devices-are-written-back-to-active-directory"></a>確認將已註冊的裝置寫回至 Active Directory

您可以使用 LDP.exe 或「ADSI 編輯」來檢視及確認裝置物件已寫回至 Active Directory。 兩者都是透過 Active Directory 系統管理工具提供的工具。

從 Azure Active Directory 寫回的裝置物件預設會放在與 AD FS 伺服器陣列相同的網域中。

`CN=RegisteredDevices,defaultNamingContext`

## <a name="create-an-application-access-policy-and-custom-access-denied-message"></a>建立應用程式存取原則和自訂的拒絕存取訊息

請考慮下列案例：您在 AD FS 中建立應用程式信賴憑證者信任，並設定只允許已註冊裝置的發佈授權規則。 現在只允許已註冊的裝置存取應用程式。 

若要讓使用者輕鬆存取應用程式，您需設定自訂的拒絕存取訊息，其中包含如何加入其裝置的指示。 現在您的使用者已可順暢地註冊其裝置以存取應用程式。

下列步驟說明如何實作此案例。

> [!NOTE]
> 本節假設您已在 AD FS 中為您的應用程式設定信賴憑證者信任。
> 

1. 開啟 AD FS MMC 工具，然後選取 [AD FS] > [信任關係] > [信賴憑證者信任]。
1. 找出將套用這個新存取規則的應用程式。 在應用程式上按一下滑鼠右鍵，然後選取 [編輯宣告規則]。
1. 選取 [發佈授權規則] 索引標籤，然後選取 [新增規則]。
1. 從 [宣告規則] 範本下拉式清單中，選取 [根據傳入宣告來允許或拒絕使用者]。 然後，選取 [下一步]。
1. 在 [宣告規則名稱] 欄位中，輸入**允許從已註冊的裝置存取**。
1. 從 [傳入宣告類型] 下拉式清單中，選取 [為登錄的使用者]。
1. 在 [傳入宣告值] 欄位中，輸入 **true**。
1. 選取 [允許具有這個傳入宣告的使用者存取]  選項按鈕。
1. 選取 [完成]，然後選取 [套用]。
1. 移除所有比您所建立之規則更寬鬆的規則。 例如，移除預設的 [允許所有使用者存取] 規則。

您的應用程式現在已設定為只有在使用者來自他們已註冊並加入工作場所的裝置時才允許存取。 如需更進階的存取原則，請參閱[針對敏感性應用程式使用額外的 Multi-Factor Authentication 來管理風險](https://technet.microsoft.com/library/dn280949.aspx)。

接下來，您將為應用程式設定自訂的錯誤訊息。 錯誤訊息可讓使用者知道他們必須先將其裝置加入到工作場所，才能存取應用程式。 您可以使用自訂 HTML 和 PowerShell 來建立自訂的應用程式拒絕存取訊息。

在您的同盟伺服器上，開啟 PowerShell 命令視窗，然後輸入下列命令。 將部分的命令取代成系統專屬的項目：

`Set-AdfsRelyingPartyWebContent -Name "relying party trust name" -ErrorPageAuthorizationErrorMessage`

在您可以存取此應用程式之前，您必須註冊您的裝置。

**如果您使用 iOS 裝置，請選取此連結加入您的裝置**：

`https://enterpriseregistration.windows.net/enrollmentserver/otaprofile/yourdomain.com`

將這個 iOS 裝置加入至您的工作場所。

如果您使用 Windows 8.1 裝置，則可以選取 [電腦設定]> [網路] > [工作場所] 來加入您的裝置。

在上述命令中，「信賴憑證者信任名稱」是 AD FS 中您應用程式「信賴憑證者信任」物件的名稱。
而 **yourdomain.com** 則是您已使用 Azure Active Directory 來設定的網域名稱 (例如 contoso.com)。
請務必從您傳遞給 **Set-AdfsRelyingPartyWebContent** Cmdlet 的 HTML 內容中移除任何分行符號 (如果有的話)。

現在，當使用者從沒有向 Azure Active Directory 裝置註冊服務註冊的裝置存取您的應用程式時，會看到錯誤。
