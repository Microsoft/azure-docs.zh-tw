---
title: 開始使用 Microsoft Authenticator 應用程式 - Azure Active Directory | Microsoft Docs
description: 了解如何升級至最新版的 Microsoft Authenticator。
services: active-directory
author: eross-msft
manager: mtillman
ms.assetid: 3065a1ee-f253-41f0-a68d-2bd84af5ffba
ms.service: active-directoary
ms.workload: identity
ms.component: user-help
ms.topic: conceptual
ms.date: 07/30/2018
ms.author: lizross
ms.reviewer: librown
ms.openlocfilehash: c8e14091d7a3fb5b925735824b1dd3ce26f034e8
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42144034"
---
# <a name="get-started-with-the-microsoft-authenticator-app"></a>開始使用 Microsoft 驗證器應用程式

Microsoft Authenticator 應用程式可藉由為您的公司或學校帳戶 (例如 alain@contoso.com) 或是個人 Microsoft 帳戶 (例如 alain@outlook.com) 提供一層額外的安全性，協助防止未經授權存取帳戶，以及停止詐騙交易。

使用應用程式進行雙步驟驗證時，它可以下列兩種方式之一來運作：

- **通知。** 應用程式會將通知傳送到您的裝置。 請確定通知正確，然後選取 [驗證]。 如果您無法辨識通知，請選取 [拒絕]。

- **驗證碼。** 輸入您的使用者名稱和密碼之後，您可以開啟應用程式，然後將 [帳戶] 畫面上提供的驗證碼複製到登入畫面。 驗證碼會以第二種形式的驗證來運作。

## <a name="opt-in-for-two-step-verification"></a>選擇雙步驟驗證

您的組織會決定您是否要搭配公司或學校帳戶來使用雙步驟驗證。 您的系統管理員將會讓您知道需要設定並使用哪些驗證方法。 如需詳細資訊，請參閱 [Azure Multi-Factor Authentication 對我有何意義](multi-factor-authentication-end-user.md)。

針對您的個人 Microsoft 帳戶，您可以為自己設定雙步驟驗證。 如需詳細資訊和指示，請參閱[關於雙步驟驗證](https://support.microsoft.com/help/12408/microsoft-account-about-two-step-verification)。

您也可以搭配非 Microsoft 帳戶使用 Microsoft Authenticator 應用程式。 那些帳戶可能不是將此功能稱為雙步驟驗證，但您應該能夠在安全性或登入設定內找到它。 如需如何設定這些非 Microsoft 帳戶的詳細資訊，請參閱 [Microsoft 客戶支援影片](https://www.youtube.com/playlist?list=PLyhj1WZ29G65QdD9NxTOAm8HwOS-OBUrX) \(英文\)。

## <a name="install-the-app"></a>安裝應用程式

Microsoft Authenticator 應用程式適用於 [Android](https://go.microsoft.com/fwlink/?linkid=866594)、[iOS](https://go.microsoft.com/fwlink/?linkid=866594) 和 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)。 若要取得最佳體驗，您應該讓應用程式在要求您時收到通知。 

## <a name="add-accounts-to-the-app"></a>將帳戶新增至應用程式

您可以將公司或學校帳戶或是個人帳戶新增至 Microsoft Authenticator 應用程式。 

### <a name="add-a-personal-microsoft-account"></a>新增個人的 Microsoft 帳戶

針對個人的 Microsoft 帳戶 (您用來登入 Outlook.com、Xbox、Skype 等的帳戶)，您只需要登入在 Microsoft Authenticator應用程式中的帳戶。

### <a name="add-a-work-or-school-account"></a>新增公司或學校帳戶

1. 如果可能，請前往另一部電腦或裝置上的[其他安全性驗證](http://aka.ms/mfasetup)畫面。 如需如何進入此畫面的詳細資訊，請參閱[變更安全性設定](multi-factor-authentication-end-user-manage-settings.md#where-to-find-the-settings-page)或連絡您的系統管理員。

    >[!Note]
    >如果您的系統管理員已開啟安全性資訊預覽體驗，您就可以遵循[將安全性資訊設定為使用驗證器應用程式](security-info-setup-auth-app.md)一節中的指示。

2. 核取 [驗證器應用程式] 旁的方塊，然後選取 [設定]。

    ![安全性驗證設定畫面上的設定按鈕](./media/microsoft-authenticator-app-how-to/auth-app-configure.png)

    [設定行動裝置應用程式] 畫面隨即出現，並提供 QR 代碼，讓您可使用驗證器應用程式進行掃描。

    ![提供 QR 代碼的畫面](./media/microsoft-authenticator-app-how-to/auth-app-barcode.png)

3. 開啟 Microsoft 驗證器應用程式。 在 [帳戶] 畫面上，選取 [新增帳戶]，然後選取 [公司或學校帳戶]。

4. 使用裝置的相機來掃描 QR 代碼，然後選取 [完成]  以關閉 QR 代碼畫面。

    >[!Note]
    >如果相機無法正常運作，您可以[手動輸入 QR 代碼和 URL](#add-an-account-to-the-app-manually)。

    應用程式的 [帳戶] 畫面會顯示您的帳戶名稱和六位數的驗證碼。 為了提高安全性，驗證碼每隔 30 秒就會變更，讓您無法使用同一個驗證碼兩次。  

    ![帳戶畫面](./media/microsoft-authenticator-app-how-to/auth-app-accounts.png)

### <a name="add-an-account-to-the-app-manually"></a>手動新增帳戶至應用程式

1. 移至 [其他安全性驗證] 畫面。 如需如何進入此畫面的詳細資訊，請參閱 [變更安全性設定](multi-factor-authentication-end-user-manage-settings.md#where-to-find-the-settings-page)。

2. 核取 [驗證器應用程式] 旁的方塊，然後選取 [設定]。

    [設定行動裝置應用程式] 畫面隨即出現。

3. 從 [設定行動裝置應用程式] 畫面複製代碼和 URL 資訊，讓您可以將它們手動輸入到 QR 掃描器。

4. 開啟 Microsoft 驗證器應用程式。 在 [帳戶] 畫面上，選取 [新增帳戶]，然後選取 [公司或學校帳戶]。

5. 在 QR 掃描器畫面上，選取 [手動輸入代碼]。

    ![掃描 QR 代碼的畫面](./media/microsoft-authenticator-app-how-to/auth-app-manual-code.png)
   
6. 從含有 QR 代碼的畫面中將代碼和 URL 輸入到 [新增帳戶] 畫面，然後選取 [完成]。

    ![輸入代碼和 URL 的畫面](./media/microsoft-authenticator-app-how-to/auth-app-code-url.png)

    應用程式的 [帳戶] 畫面會顯示您的帳戶名稱和六位數的驗證碼。 為了提高安全性，驗證碼每隔 30 秒就會變更，讓您無法使用同一個驗證碼兩次。

### <a name="using-your-devices-fingerprint-or-facial-recognition-capabilities"></a>使用裝置的指紋或臉部辨識功能

您的組織可能需要 PIN，才能完成您的身分識別驗證。 您可以設定 Microsoft Authenticator 應用程式來使用裝置的指紋或臉部辨識功能，而非 PIN。 您可以藉由選取選項來使用裝置的生物識別功能 (而非 PIN) 作為身分識別，在首次使用驗證器應用程式來驗證帳戶時設定此動作。

## <a name="use-the-app-when-you-sign-in"></a>當您登入時使用應用程式

將帳戶新增到應用程式之後，您可以使用此應用程式來登入帳戶。

如果您選擇在應用程式中使用驗證碼，您將會開始在 [帳戶] 頁面上看到它們。 驗證碼每隔 30 秒就會變更，所以當您需要驗證碼時，永遠都可取得新的驗證碼。 但在您登入並接獲提示輸入驗證碼之前，不必理會它們。

## <a name="next-steps"></a>後續步驟

- 如果您有更多關於應用程式的一般問題，請參閱 [Microsoft Authenticator 常見問題集](microsoft-authenticator-app-faq.md)

- 如果您需要雙步驟驗證的詳細資訊，請參閱[對我的帳戶進行雙步驟驗證設定](multi-factor-authentication-end-user-first-time.md)

- 如果您想要關於安全性資訊的詳細資訊，請參閱[管理安全性資訊](security-info-manage-settings.md)
