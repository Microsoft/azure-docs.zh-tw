---
title: 設定雙步驟驗證 - Azure Active Directory | Microsoft Docs
description: 當您的公司設定 Azure Multi-Factor Authentication 時，系統會提示您註冊雙步驟驗證。 了解如何進行設定。
services: active-directory
keywords: 如何使用 Azure Directory, 雲端中的 Active Directory, Active Directory 教學課程
author: eross-msft
manager: mtillman
ms.reviewer: richagi
ms.assetid: 46f83a6a-dbdd-4375-8dc4-e7ea77c16357
ms.workload: identity
ms.service: active-directory
ms.component: user-help
ms.topic: conceptual
ms.date: 05/15/2017
ms.author: lizross
ms.openlocfilehash: b6d548183cfa548f9ad35f06922752c929e318ac
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041950"
---
# <a name="set-up-my-account-for-two-step-verification"></a>對我的帳戶進行雙步驟驗證設定
雙步驟驗證是額外的安全性步驟，可協助保護您的帳戶使其他人更難以破解。 如果您正在閱讀這篇文章，可能會收到一封來自您工作或學校的系統管理員關於 Multi-Factor Authentication 的電子郵件。 或者也許您嘗試登入並收到訊息，要求您設定其他安全性驗證。 如果是這個情況，**則您在完成自動註冊程序之前將無法登入**。

本文將協助您設定您的**工作或學校帳戶**。 如果您想要對您自己的個人 Microsoft 帳戶啟用雙步驟驗證，請參閱 [雙步驟驗證](https://support.microsoft.com/help/12408/microsoft-account-about-two-step-verification)。

## <a name="set-up-your-account"></a>設定帳戶

當貴公司支援人員要求您開始使用雙步驟驗證時，您會看到一個畫面，指出 [系統管理員已要求您為這個帳戶設定額外的安全性驗證]：

![設定](./media/multi-factor-authentication-end-user-first-time/first.png)

若要開始使用，請選取 [立即設定]。

如果登入時未看到類似畫面，請遵循[管理您的雙步驟驗證設定](multi-factor-authentication-end-user-manage-settings.md#where-to-find-the-settings-page)中的指示，以尋找您可以在其中管理驗證選項的設定頁面。

## <a name="decide-how-you-want-to-verify-your-sign-ins"></a>決定您要如何驗證登入

註冊程序中的第一個問題是您要我們連絡您的方式。 看看資料表中的選項，並使用連結移至每個方法的設定步驟。

| 連絡方法 | 說明 |
| --- | --- |
| [行動應用程式](#use-a-mobile-app-as-the-contact-method) |- **收到驗證的通知。** 這個選項會將通知推送至您智慧型手機或平板電腦上的驗證器應用程式。 檢視通知，如果合法則選取應用程式中的 [驗證]。 您的工作或學校可能會要求您輸入 PIN 後才能進行驗證。<br>- **驗證碼。** 在此模式中，驗證器應用程式會產生每 30 秒更新一次的驗證碼。 在登入介面中輸入最新的驗證碼。<br>Microsoft Authenticator 應用程式適用於 [Android](https://go.microsoft.com/fwlink/?linkid=866594)、[iOS](https://go.microsoft.com/fwlink/?linkid=866594) 和 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)。 |
| [行動電話通話或文字](#use-your-mobile-phone-as-the-contact-method) |- **撥打電話**撥打自動語音電話給您所提供的電話號碼。 接聽電話並按電話鍵盤上的 # 進行驗證。<br>- **簡訊**傳送包含驗證碼的簡訊。 遵循文字中的提示，回覆簡訊或將所提供的驗證碼輸入登入介面。 |
| [辦公室電話通話](#use-your-office-phone-as-the-contact-method) |撥打自動語音電話給您所提供的電話號碼。 接聽電話並按電話鍵盤上的 # 進行驗證。 |

## <a name="use-a-mobile-app-as-the-contact-method"></a>使用行動應用程式做為連絡方式
使用此方法會要求您在手機或平板電腦上安裝驗證器應用程式。 本文中的步驟是根據 Microsoft Authenticator 應用程式，適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [iOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

>[!NOTE]
>您不必使用 Microsoft Authenticator 應用程式。 如果您已在使用另一個驗證器應用程式，則可繼續使用它。

1. 從下拉式清單中選取**行動應用程式**。
2. 選取 [接收驗證的通知] 或 [使用驗證碼]，然後選取 [設定]。

   ![[其他安全性驗證] 畫面](./media/multi-factor-authentication-end-user-first-time/mobileapp.png)

3. 在您的手機或平板電腦上，開啟應用程式並選取 **+** 以新增帳戶。 (在 Android 裝置上，選取三個點，然後**新增帳戶**。)
4. 指定您想要新增的是工作帳戶還是學校帳戶。 隨即開啟您手機上的 QR 代碼掃描器。 如果您的相機未正常運作，您可以選取手動輸入公司資訊。 如需詳細資訊，請參閱 [手動新增帳戶](#add-an-account-manually)。  
5. 掃描隨畫面顯示的 QR 代碼圖片來設定行動裝置應用程式。  按一下 [完成]  以關閉 QR 代碼畫面。  

   ![QR 代碼畫面](./media/multi-factor-authentication-end-user-first-time/scan2.png)

6. 在手機上完成啟動時，選取 [連絡我] 。  此步驟會將通知或驗證碼傳送到您的手機。 選取 [驗證] 。  
7. 如果貴公司需要 PIN 來核准登入驗證，請輸入它。

   ![輸入 PIN 的方塊](./media/multi-factor-authentication-end-user-first-time/scan3.png)

8. 輸入完 PIN 之後，請選取 [關閉] 。 此時，您的驗證應會成功。
9. 建議您輸入您的行動電話號碼，以防萬一您無法存取行動裝置應用程式。 請從下拉式清單中指定您的國家/地區，然後在國家/地區名稱旁邊的方塊中輸入您的行動電話號碼。 選取 [下一步] 。
10. 此時，系統會提示您設定非瀏覽器應用程式的應用程式密碼，例如 Outlook 2010 或更舊版本，或是 Apple 裝置上的原生電子郵件應用程式。 這是因為某些應用程式不支援雙步驟驗證。 如果您不使用這些應用程式，按一下 [完成]，並略過其餘的步驟。
11. 如果您正在使用這類應用程式，複製提供的應用程式密碼並將此密碼而非一般密碼貼到應用程式。 您可以對多個應用程式使用相同的應用程式密碼。 如需詳細資訊，[應用程式密碼協助]。
12. 按一下 [完成] 。

### <a name="add-an-account-manually"></a>手動新增帳戶
如果您想要以手動方式將帳戶新增至行動裝置應用程式，而不是使用 QR 讀取器，請遵循下列步驟。

1. 選取 [手動輸入帳戶]  按鈕。  
2. 輸入顯示條碼的同一頁所提供的代碼和 URL。 此資料會填入行動裝置應用程式上的 [代碼] 和 [URL] 方塊中。

    ![設定](./media/multi-factor-authentication-end-user-first-time/barcode2.png)
3. 當啟用完成時，請選取 [連絡我] 。 此步驟會將通知或驗證碼傳送到您的手機。 選取 [驗證] 。

## <a name="use-your-mobile-phone-as-the-contact-method"></a>使用行動電話做為連絡方式
1. 從下拉式清單中選取**驗證電話**。  

    ![設定](./media/multi-factor-authentication-end-user-first-time/phone.png)  
2. 從下拉式清單中選擇您的國家/地區，然後輸入您的行動電話號碼。
3. 選取您想要用於行動電話的方法 - 簡訊或電話。
4. 選取 [與我連絡] 以確認您的電話號碼。 根據您選取的模式，我們會傳送文字或撥打電話給您。 遵循螢幕上提供的指示，然後選取 [確認]。
5. 此時，系統會提示您設定非瀏覽器應用程式的應用程式密碼，例如 Outlook 2010 或更舊版本，或是 Apple 裝置上的原生電子郵件應用程式。 這是因為某些應用程式不支援雙步驟驗證。 如果您不使用這些應用程式，按一下 [完成]，並略過其餘的步驟。
6. 如果您正在使用這類應用程式，複製提供的應用程式密碼並將此密碼而非一般密碼貼到應用程式。 您可以對多個應用程式使用相同的應用程式密碼。 如需詳細資訊，[應用程式密碼協助]。
7. 按一下 [完成] 。

## <a name="use-your-office-phone-as-the-contact-method"></a>使用辦公室電話做為連絡方式
1. 從下拉式清單中選取**辦公室電話**  

    ![設定](./media/multi-factor-authentication-end-user-first-time/office.png)  
2. 會自動將您的公司連絡人資訊填入電話號碼方塊。 如果號碼錯誤或遺失，請要求系統管理員進行變更。
3. 選取 [與我連絡] 以確認您的電話號碼，我們會撥打您的號碼。 遵循螢幕上提供的指示，然後選取 [確認]。
4. 此時，系統會提示您設定非瀏覽器應用程式的應用程式密碼，例如 Outlook 2010 或更舊版本，或是 Apple 裝置上的原生電子郵件應用程式。 這是因為某些應用程式不支援雙步驟驗證。 如果您不使用這些應用程式，按一下 [完成]，並略過其餘的步驟。
5. 如果您正在使用這類應用程式，複製提供的應用程式密碼並將此密碼而非一般密碼貼到應用程式。 您可以對多個應用程式使用相同的應用程式密碼。 如需詳細資訊，請參閱 [什麼是應用程式密碼](multi-factor-authentication-end-user-app-passwords.md)。
6. 按一下 [完成] 。

## <a name="next-steps"></a>後續步驟
* 變更您慣用的選項並[管理您雙步驟驗證的設定](multi-factor-authentication-end-user-manage-settings.md)
* 針對不支援雙步驟驗證的原生裝置應用程式，設定[應用程式密碼](multi-factor-authentication-end-user-app-passwords.md)。
* 查看 [Microsoft 驗證器應用程式](microsoft-authenticator-app-how-to.md)，即使在您沒有行動服務時也能進行快速、安全的驗證。
