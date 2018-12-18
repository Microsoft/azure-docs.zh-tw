---
title: Azure MFA Server 升級 | Microsoft Docs
description: 將 Azure Multi-Factor Authentication Server 升級為較新版本的步驟和指引。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 7e7952a327134197f1e8492931d7ada871789395
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42145755"
---
# <a name="upgrade-to-the-latest-azure-multi-factor-authentication-server"></a>升級為最新的 Azure Multi-Factor Authentication Server

本文將逐步引導您完成升級 Azure Multi-Factor Authentication (MFA) Server v6.0 或更新版本的程序。 如果您需要升級舊版的 PhoneFactor Agent，請參閱[將 PhoneFactor Agent 升級為 Azure Multi-Factor Authentication Server](howto-mfaserver-deploy-upgrade-pf.md)。

如果您要從 v6.x 或更舊版本升級為 v7.x 或更新版本，所有元件都會從 .NET 2.0 變更為 .NET 4.5。 所有元件也都需要 Microsoft Visual C++ 2015 可轉散發套件更新 1 或更新版本。 MFA Server 安裝程式會同時安裝這些元件的 x86 和 x64 版本 (如果尚未安裝)。 如果使用者入口網站和行動裝置應用程式 Web 服務在不同的伺服器上執行，則您需要先安裝這些套件，再升級這些元件。 您可以在 [Microsoft 下載中心](https://www.microsoft.com/download/)搜尋最新的 Microsoft Visual C++ 2015 可轉散發套件更新。 

## <a name="install-the-latest-version-of-azure-mfa-server"></a>安裝最新版的 Azure MFA Server

1. 使用[下載 zure Multi-Factor Authentication Server](howto-mfaserver-deploy.md#download-the-mfa-server) 中的指示，來取得最新版的 Azure MFA Server。
2. 在您的主要 MFA Server 上，針對位於 C:\Program Files\Multi-Factor Authentication Server\Data\PhoneFactor.pfdata (假設此為預設安裝位置) 的 MFA Server 資料檔案製作備份。
3. 如果您執行多部伺服器以取得高可用性，請變更用戶端系統來向 MFA Server 驗證，如此便能讓它們停止將流量傳送到正在升級的伺服器。 如果您使用負載平衡器，請從負載平衡器移除 MFA Server、進行升級，然後將伺服器加回到伺服器陣列。
4. 在每部 MFA Server 上執行新的安裝程式。 先升級次級伺服器，因為它們可以讀取由主要伺服器所複寫的舊資料檔。 

  您不需解除安裝目前的 MFA Server，就能執行安裝程式。 安裝程式會執行就地升級。 系統會從先前安裝的登錄中挑選安裝路徑，因此它會安裝在相同位置 (例如，C:\Program Files\Multi-Factor Authentication Server)。 
  
5. 如果系統提示您安裝 Microsoft Visual C++ 2015 可轉散發套件的更新套件，請接受提示。 隨即會同時安裝套件的 x86 和 x64 版本。
5. 如果您使用 Web 服務 SDK，系統會提示您安裝新的 Web 服務 SDK。 當您安裝新的 Web 服務 SDK 時，請確定虛擬目錄名稱符合先前安裝的虛擬目錄 (例如，MultiFactorAuthWebServiceSdk)。
6. 在所有次級伺服器上重複執行這些步驟。 將其中一部次級伺服器升階為新的主要伺服器，然後升級舊的主要伺服器。 

## <a name="upgrade-the-user-portal"></a>升級使用者入口網站

1. 製作 web.config 檔案的備份，此檔案位於使用者入口網站安裝位置的虛擬目錄中 (例如，C:\inetpub\wwwroot\MultiFactorAuth)。 如果對預設佈景主題做了任何變更，也請製作 App_Themes\Default 資料夾的備份。 最好是建立預設資料夾的複本，並建立新的佈景主題，再變更預設佈景主題。
2. 如果使用者入口網站與其他 MFA Server 元件在相同的伺服器上執行，MFA Server 安裝會提示您更新使用者入口網站。 接受提示並安裝使用者入口網站更新。 檢查虛擬目錄名稱是否符合先前安裝的虛擬目錄 (例如，MultiFactorAuth)。
3. 如果使用者入口網站位在它自己的伺服器上，請從其中一部 MFA Server 的安裝位置複製 MultiFactorAuthenticationUserPortalSetup64.msi 檔案，然後將它放入使用者入口網站 Web 伺服器中。 執行安裝程式。 

  如果發生錯誤並指出「需要 Microsoft Visual C++ 2015 可轉散發套件更新 1 或更新版本」，可從[Microsoft 下載中心](https://www.microsoft.com/download/)下載並安裝最新的更新套件。 同時安裝 x86 和 x64 版本。

4. 安裝更新的使用者入口網站軟體之後，將您在步驟 1 中製作的 web.config 備份與新的 web.config 檔案進行比較。 如果新的 web.config 中沒有任何新的屬性，請將您的備份 web.config 複製到虛擬目錄，以覆寫新檔案。 另一個選項是從備份檔案複製 appSettings 值和 Web 服務 SDK URL，然後貼入新的 web.config。

如果您在多部伺服器上具有使用者入口網站，請全部重複執行安裝。 

## <a name="upgrade-the-mobile-app-web-service"></a>升級行動裝置應用程式 Web 服務

> [!NOTE]
> 從 Azure MFA Server 8.0 至 8.0+ 以前的版本升級時，可以在升級之後解除安裝行動裝置應用程式 Web 服務

## <a name="upgrade-the-ad-fs-adapters"></a>升級 AD FS 配接器

### <a name="if-mfa-runs-on-different-servers-than-ad-fs"></a>如果 MFA 與 AD FS 不是在同一部伺服器上執行

只有當您將 Multi-Factor Authentication Server 與您的 AD FS 伺服器分開執行時，才適用這些指示。 如果這兩個服務在相同的伺服器上執行，請略過本節，並移至安裝步驟。 

1. 儲存已在 AD FS 中註冊的 MultiFactorAuthenticationAdfsAdapter.config 複本，或者使用下列 PowerShell 命令來將組態匯出：`Export-AdfsAuthenticationProviderConfigurationData -Name [adapter name] -FilePath [path to config file]`。 根據先前安裝的版本，配接器名稱可以是 "WindowsAzureMultiFactorAuthentication" 或 "AzureMfaServerAuthentication"。
2. 將下列檔案從 MFA Server 安裝位置複製到 AD FS 伺服器：

  - MultiFactorAuthenticationAdfsAdapterSetup64.msi
  - Register-MultiFactorAuthenticationAdfsAdapter.ps1
  - Unregister-MultiFactorAuthenticationAdfsAdapter.ps1
  - MultiFactorAuthenticationAdfsAdapter.config

3. 編輯 Register-MultiFactorAuthenticationAdfsAdapter.ps1 指令碼，作法是在 `Register-AdfsAuthenticationProvider` 命令的結尾加上 `-ConfigurationFilePath [path]`。 以 MultiFactorAuthenticationAdfsAdapter.config 檔案或上一步中匯出的組態檔之完整路徑取代 *[path]*。 

  在新的 MultiFactorAuthenticationAdfsAdapter.config 中檢查屬性，以查看它們是否相符舊的組態檔。 如果已在新版本中加入或移除任何屬性，可將屬性值從舊的組態檔複製到新的組態檔，或是修改舊的組態檔以使其相符。

### <a name="install-new-ad-fs-adapters"></a>安裝新的 AD FS 配接器

> [!IMPORTANT] 
> 您的使用者在本節的步驟 3-8 期間，將不需執行雙步驟驗證。 如果您在多個叢集中設定了 AD FS，您可以在伺服器陣列中個別移除、升級和還原每一個叢集，而不會影響其他叢集，以避免產生停機時間。

1. 從伺服器陣列中移除某些 AD FS 伺服器。 當其他伺服器仍在執行時，更新這些伺服器。
2. 在已從 AD FS 伺服器陣列中移除的每部伺服器上安裝新的 AD FS 配接器。 如果在每部 AD FS 伺服器上安裝 MFA Server，您可以透過 MFA Server 系統管理 UX 來更新。 否則，可透過執行 MultiFactorAuthenticationAdfsAdapterSetup64.msi 來更新。 

  如果發生錯誤並指出「需要 Microsoft Visual C++ 2015 可轉散發套件更新 1 或更新版本」，可從[Microsoft 下載中心](https://www.microsoft.com/download/)下載並安裝最新的更新套件。 同時安裝 x86 和 x64 版本。

3. 移至 [AD FS] > [驗證原則] > [編輯全域多重要素驗證原則]。 取消勾選 **[WindowsAzureMultiFactorAuthentication]** 或 **[AzureMFAServerAuthentication]** \(根據目前安裝的版本而定)。 

  完成此步驟之後，您必須先完成步驟 8，才能在此 AD FS 叢集中透過 MFA Server 進行雙步驟驗證。

4. 執行 Unregister-MultiFactorAuthenticationAdfsAdapter.ps1 PowerShell 指令碼來取消註冊舊版的 AD FS 配接器。 確認 *-Name* 參數 (可以是 “WindowsAzureMultiFactorAuthentication” 或 "AzureMFAServerAuthentication") 符合步驟 3 中顯示的名稱。 這適用於相同 AD FS 叢集中的所有伺服器，因為有一個中央組態。
5. 執行 Register-MultiFactorAuthenticationAdfsAdapter.ps1 PowerShell 指令碼來註冊新的 AD FS 配接器。 這適用於相同 AD FS 叢集中的所有伺服器，因為有一個中央組態。
6. 在已從 AD FS 伺服器陣列中移除的每部伺服器上重新啟動 AD FS 服務。
7. 將更新的伺服器加回 AD FS 伺服器陣列，並從該伺服器陣列中移除其他伺服器。
8. 移至 [AD FS] > [驗證原則] > [編輯全域多重要素驗證原則]。 勾選 [AzureMfaServerAuthentication]。
9. 重複執行步驟 2 來更新現在已從 AD FS 伺服器陣列中移除的伺服器，然後在這些伺服器上重新啟動 AD FS 服務。
10. 將這些伺服器加回 AD FS 伺服器陣列。

## <a name="next-steps"></a>後續步驟

- 取得[使用 Azure Multi-Factor Authentication 與協力廠商 VPN 的進階案例](howto-mfaserver-nps-vpn.md)的範例

- [將 MFA Server 與 Windows Server Active Directory 同步處理](howto-mfaserver-dir-ad.md)

- 為您的應用程式[設定 Windows 驗證](howto-mfaserver-windows.md)
