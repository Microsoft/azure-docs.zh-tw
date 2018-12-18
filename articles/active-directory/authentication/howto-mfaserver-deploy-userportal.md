---
title: Azure MFA Server 使用者入口網站 | Microsoft Docs
description: 開始使用 Azure MFA 和使用者入口網站。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 1bd9dd82fb72c3d6b0759139680647be764f273a
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39161548"
---
# <a name="user-portal-for-the-azure-multi-factor-authentication-server"></a>Azure Multi-Factor Authentication Server 的使用者入口網站

使用者入口網站是一個 IIS 網站，可讓使用者註冊 Azure Multi-Factor Authentication (MFA) 及維護自己的帳戶。 使用者可以變更電話號碼、變更 PIN，或選擇在下次登入時略過雙步驟驗證。

使用者可以使用一般的使用者名稱和密碼登入使用者入口網站，然後完成雙步驟驗證通話或回答安全性問題，以完成驗證。 如果允許使用者註冊，使用者可以在第一次登入使用者入口網站時設定電話號碼和 PIN。

您可以設定使用者入口網站管理員，並授與新增使用者及更新現有使用者的權限。

視您的環境而定，建議您在與 Azure Multi-factor Authentication Server 相同的伺服器，或在另一個網際網路對向的伺服器上部署使用者入口網站。

![MFA 使用者入口網站](./media/howto-mfaserver-deploy-userportal/portal.png)

> [!NOTE]
> 使用者入口網站只適用於 Multi-Factor Authentication Server。 如果您在雲端中使用 Multi-Factor Authentication，請讓使用者參閱[設定您的帳戶進行雙步驟驗證](../user-help/multi-factor-authentication-end-user-first-time.md)或[管理您的設定進行雙步驟驗證](../user-help/multi-factor-authentication-end-user-manage-settings.md)。

## <a name="install-the-web-service-sdk"></a>安裝 Web 服務 SDK

在任一案例中，如果 Azure Multi-Factor Authentication (MFA) Server 上尚**未**安裝 Azure Multi-Factor Authentication Web 服務 SDK，請完成後續步驟。

1. 開啟 Multi-Factor Authentication Server 主控台。
2. 移至 [Web 服務 SDK] 並選取 [安裝 Web 服務 SDK]。
3. 除非基於某種原因而需要變更預設值，否則使用預設值來完成安裝。
4. 將 SSL 憑證繫結至 IIS 中的網站。

如果您有關於在 IIS 伺服器上設定 SSL 憑證的問題，請參閱[如何在 IIS 上設定 SSL](https://docs.microsoft.com/iis/manage/configuring-security/how-to-set-up-ssl-on-iis) 一文。

Web 服務 SDK 必須受到 SSL 憑證保護。 自我簽署憑證適用於這項用途。 請將此憑證匯入到「使用者入口網站」Web 伺服器上 [本機電腦] 帳戶的 [受信任的根憑證授權單位] 存放區中，這樣它才會在起始 SSL 連線時信任該憑證。

![MFA Server 組態設定 Web 服務 SDK](./media/howto-mfaserver-deploy-userportal/sdk.png)

## <a name="deploy-the-user-portal-on-the-same-server-as-the-azure-multi-factor-authentication-server"></a>在與 Azure Multi-Factor Authentication Server 相同的伺服器上部署使用者入口網站

若要在與 Azure Multi-Factor Authentication Server **相同的伺服器**上安裝使用者入口網站，您必須滿足以下必要條件：

* IIS，包括 ASP.NET 及 IIS 6 metabase 相容性 (適用於 IIS 7 或更高版本)
* 具有電腦和網域系統管理員權限的帳戶 (如果適用)。 帳戶需有建立 Active Directory 安全性群組的權限。
* 使用 SSL 憑證保護使用者入口網站。
* 使用 SSL 憑證保護 Azure Multi-Factor Authentication Web 服務 SDK。

若要部署使用者入口網站，請依照下列步驟執行︰

1. 開啟 Azure Multi-Factor Authentication Server 主控台，按一下左側功能表中的 [使用者入口網站] 圖示，然後按一下 [安裝使用者入口網站]。
2. 除非基於某種原因而需要變更預設值，否則使用預設值來完成安裝。
3. 將 SSL 憑證繫結至 IIS 中的網站

   > [!NOTE]
   > 此 SSL 憑證通常是公開簽署的 SSL 憑證。

4. 從任何電腦開啟網頁瀏覽器，並巡覽至安裝使用者入口網站的 URL (例如：https://mfa.contoso.com/MultiFactorAuth))。 確定未出現任何憑證警告或錯誤。

![MFA Server 使用者入口網站安裝](./media/howto-mfaserver-deploy-userportal/install.png)

如果您有關於在 IIS 伺服器上設定 SSL 憑證的問題，請參閱[如何在 IIS 上設定 SSL](https://docs.microsoft.com/iis/manage/configuring-security/how-to-set-up-ssl-on-iis) 一文。

## <a name="deploy-the-user-portal-on-a-separate-server"></a>在個別的伺服器上部署使用者入口網站

如果 Azure Multi-Factor Authentication Server 執行所在的伺服器不是網際網路對向，您應該在**個別的網際網路對向伺服器**上安裝使用者入口網站。

如果您的組織使用 Microsoft Authenticator 應用程式作為其中一種驗證方法，並想要在自己的伺服器上部署使用者入口網站，請完成下列需求︰

* 使用 Azure Multi-Factor Authentication Server v6.0 或更高版本。
* 在執行 Microsoft Internet Information Services (IIS) 6.x 或更高版本的網際網路對向網頁伺服器上安裝使用者入口網站。
* 在使用 IIS 6.x 時，請確定 ASP.NET v2.0.50727 已安裝、註冊並設定為 [已允許]。
* 使用 IIS 7.x 或更高版本時，包括基本驗證、ASP.NET 和 IIS 6 Metabase 相容性。
* 使用 SSL 憑證保護使用者入口網站。
* 使用 SSL 憑證保護 Azure Multi-Factor Authentication Web 服務 SDK。
* 確保使用者入口網站可以透過 SSL 連線至 Azure Multi-Factor Authentication Web 服務 SDK。
* 確保使用者入口網站可以使用 "PhoneFactor Admins" 安全性群組中服務帳戶的認證來驗證 Azure Multi-Factor Authentication Web 服務 SDK。 如果 Azure Multi-Factor Authentication Server 在加入網域的伺服器上執行，此服務帳戶和群組應存在於 Active Directory 中。 如果伺服器未加入網域，此服務帳戶和群組會存在於 Azure Multi-Factor Authentication Server 本機。

若要將使用者入口網站安裝在 Azure Multi-Factor Authentication Server 以外的伺服器上，必須進行下列步驟：

1. **在 MFA Server 上**，瀏覽至安裝路徑 (例如：C:\Program Files\Multi-Factor Authentication Server)，然後將 **MultiFactorAuthenticationUserPortalSetup64** 檔案複製到您要安裝它的網際網路對向伺服器可以存取的位置。
2. **在網際網路對向網頁伺服器上**，以系統管理員身分執行 MultiFactorAuthenticationUserPortalSetup64 安裝檔案、視需要變更網站，以及將虛擬目錄變更為簡短名稱 (如果您想要)。
3. 將 SSL 憑證繫結至 IIS 中的網站。

   > [!NOTE]
   > 此 SSL 憑證通常是公開簽署的 SSL 憑證。

4. 瀏覽至 **C:\inetpub\wwwroot\MultiFactorAuth**
5. 在 [記事本] 中編輯 Web.Config 檔案

    * 找出 **"USE_WEB_SERVICE_SDK"** 機碼並將 **value="false"** 變更為 **value="true"**
    * 找出 **"WEB_SERVICE_SDK_AUTHENTICATION_USERNAME"** 機碼並將 **value=""** 變更為 **value="DOMAIN\User"**，其中的 DOMAIN\User 是屬於 "PhoneFactor Admins" 群組的服務帳戶。
    * 找出 **"WEB_SERVICE_SDK_AUTHENTICATION_PASSWORD"** 機碼並將 **value=""** 變更為 **value="Password"**，其中的 Password 是在前一行中輸入之服務帳戶的密碼。
    * 找出 **https://www.contoso.com/MultiFactorAuthWebServiceSdk/PfWsSdk.asmx** 值並將此預留位置 URL 變更為我們在步驟 2 中安裝的 Web 服務 SDK URL。
    * 儲存 Web.Config 檔案並關閉 [記事本]。

6. 從任何電腦開啟網頁瀏覽器，並巡覽至安裝使用者入口網站的 URL (例如：https://mfa.contoso.com/MultiFactorAuth))。 確定未出現任何憑證警告或錯誤。

如果您有關於在 IIS 伺服器上設定 SSL 憑證的問題，請參閱[如何在 IIS 上設定 SSL](https://docs.microsoft.com/iis/manage/configuring-security/how-to-set-up-ssl-on-iis) 一文。

## <a name="configure-user-portal-settings-in-the-azure-multi-factor-authentication-server"></a>在 Azure Multi-Factor Authentication Server 中進行使用者入口網站設定

既然使用者入口網站已安裝完成，您需要設定 Azure Multi-Factor Authentication Server，使其與入口網站搭配運作。

1. 在 Azure Multi-Factor Authentication Server 主控台中，按一下 [使用者入口網站] 圖示。 在 [設定] 索引標籤上，於 [使用者入口網站 URL] 文字方塊中輸入使用者入口網站的 URL。 如果電子郵件功能已啟用，將使用者匯入 Azure Multi-Factor Authentication Server 時，系統會將此 URL 包含在傳送給使用者的電子郵件中。
2. 選擇要在使用者入口網站中使用的設定。 例如，如果您允許使用者選擇其驗證方法，請確認 [允許使用者選取方法] 連同他們可以選擇的方法已核取。
3. 在 [系統管理員] 索引標籤上定義誰應該是系統管理員。您可以使用 [新增]/[編輯] 方塊中的核取方塊和下拉式清單，建立細微的系統管理權限。

選用組態：
- **安全性問題** - 定義您的環境核准的安全性問題及其顯示語言。
- **通過的工作階段** - 使用 MFA，設定使用者入口網站與表單架構網站的整合。
- **受信任的 IP** - 允許使用者從受信任的 IP 清單或範圍驗證時，略過 MFA。

![MFA Server 使用者入口網站組態](./media/howto-mfaserver-deploy-userportal/config.png)

Azure Multi-Factor Authentication 伺服器為使用者入口網站提供數個選項。 下表提供這些選項及其用途說明的清單。

| 使用者入口網站設定 | 說明 |
|:--- |:--- |
| 使用者入口網站 URL | 輸入裝載入口網站的 URL。 |
| 主要驗證 | 指定登入入口網站時所使用的驗證類型。 Windows、Radius 或 LDAP 驗證。 |
| 允許使用者登入 | 可讓使用者在使用者入口網站的登入頁面輸入使用者名稱和密碼。 如果未選取此選項，則方塊會呈現灰色。 |
| 允許使用者註冊 | 在設定畫面中提示使用者輸入其他資訊，例如電話號碼，以允許使用者註冊 Multi-Factor Authentication。 [提示輸入備用電話] 可讓使用者指定次要電話號碼。 [提示輸入協力廠商 OATH 權杖] 可讓使用者指定協力廠商 OATH 權杖。 |
| 允許使用者起始單次許可 | 允許使用者起始單次許可。 如果使用者設定此選項，則會在使用者下次登入時生效。 [提示輸入許可秒數] 提供方塊讓使用者變更預設的 300 秒。 否則 300 秒就很適合單次許可。 |
| 允許使用者選取方法 | 可讓使用者指定其主要連絡方法。 此方法可以是通話、簡訊、行動裝置應用程式或 OATH 權杖。 |
| 允許使用者選取語言 | 允許使用者變更用於通話、簡訊、行動應用程式或 OATH 權杖的語言。 |
| 允許使用者啟用行動應用程式 | 可讓使用者產生啟用代碼，以完成伺服器使用的行動應用程式啟動程序。  您也可以設定允許使用者在其中啟動應用程式的裝置數目 (1 到 10)。 |
| 使用遞補用的安全性問題 | 允許在雙步驟驗證失敗時使用安全性問題。 您可以指定必須正確回答的安全性問題數目。 |
| 允許使用者關聯協力廠商 OATH 權杖 | 可讓使用者指定協力廠商 OATH 權杖。 |
| 使用遞補用的 OATH 權杖 | 允許在雙步驟驗證失敗時使用 OATH 權杖。 您也可以指定工作階段逾時 (以分鐘為單位)。 |
| 啟用記錄 | 在使用者入口網站啟用記錄。 記錄檔位於：C:\Program Files\Multi-Factor Authentication Server\Logs。 |

這些設定啟用之後，當使用者登入使用者入口網站時，就能在入口網站中看到這些設定。

![使用者入口網站設定](./media/howto-mfaserver-deploy-userportal/portalsettings.png)

### <a name="self-service-user-enrollment"></a>自助式使用者註冊

如果要讓使用者登入並註冊，您必須選取 [設定] 索引標籤底下的 [允許使用者登入] 和 [允許使用者註冊] 選項。請記住，您選取的設定會影響使用者的登入體驗。

例如，當使用者首次登入使用者入口網站時，便會進入 [Azure Multi-Factor Authentication 使用者設定] 頁面。 根據您如何設定 Azure Multi-Factor Authentication，使用者也許能夠選取驗證方法。

如果他們選取 [語音通話] 驗證方法，或已預先設定為使用該方法，此頁面會提示使用者輸入主要電話號碼和分機 (如果有的話)。 他們也可以輸入備用電話號碼。

![註冊主要和備份電話號碼](./media/howto-mfaserver-deploy-userportal/backupphone.png)

如果使用者驗證時需要使用 PIN，此頁面會提示他們建立 PIN。 使用者輸入電話號碼和 PIN (如果需要的話) 之後，接著按一下 [立即撥打電話給我以進行驗證] 按鈕。 Azure Multi-Factor Authentication 會撥打到使用者的主要電話號碼執行通話驗證。 使用者必須接聽電話並輸入 PIN (如果需要的話)，然後按下 #，以進入自助式註冊程序的下一個步驟。

如果使用者選取 [簡訊] 驗證方法，或已預先設定為使用該方法，此頁面會提示使用者輸入行動電話號碼。 如果使用者驗證時需要使用 PIN，此頁面也會提示他們輸入 PIN。  使用者輸入電話號碼和 PIN (如果需要的話) 之後，接著按一下 [立即傳簡訊給我以進行驗證] 按鈕。 Azure Multi-Factor Authentication 會向使用者的行動電話發送簡訊驗證。 使用者會收到附有一次性密碼 (OTP) 的簡訊，然後以該 OTP 和其 PIN (如果適用) 回覆訊息。

![使用者入口網站簡訊](./media/howto-mfaserver-deploy-userportal/text.png)

如果使用者選取 [行動裝置應用程式] 驗證方法，此頁面會提示使用者在其裝置上安裝 Microsoft 驗證器應用程式，並產生啟用代碼。 使用者安裝應用程式之後，接著按一下 [產生啟用代碼] 按鈕。

> [!NOTE]
> 若要使用 Microsoft 驗證器應用程式，使用者必須啟用其裝置的推播通知。

此頁面接著會顯示啟用代碼和 URL，以及條碼圖片。 如果使用者驗證時需要使用 PIN，此頁面也會提示他們輸入 PIN。 使用者在 Microsoft 驗證器應用程式中輸入啟用代碼和 URL，或使用條碼掃描器來掃描條碼圖片，並按一下 [啟用] 按鈕。

啟用完成之後，使用者按一下 [立即驗證我的身分] 按鈕。 Azure Multi-Factor Authentication 會對使用者的行動裝置應用程式執行驗證。 使用者必須輸入 PIN (如果需要的話)，並在其行動應用程式中按下 [驗證] 按鈕，以進入自助式註冊程序的下一個步驟。

如果系統管理員已設定 Azure Multi-Factor Authentication Server 來收集安全性問題和答案，則使用者會進入 [安全性問題] 頁面。 使用者必須選取四個安全性問題，並回答所選取的問題。

![使用者入口網站安全性問題](./media/howto-mfaserver-deploy-userportal/secq.png)

使用者自助式註冊現已完成，使用者也已登入使用者入口網站。 只要系統管理員允許變更方法，使用者以後可隨時重新登入使用者入口網站來變更電話號碼、PIN、驗證方法和安全性問題。

## <a name="next-steps"></a>後續步驟

- [部署 Azure Multi-Factor Authentication Server 行動裝置應用程式 Web 服務](howto-mfaserver-deploy-mobileapp.md)