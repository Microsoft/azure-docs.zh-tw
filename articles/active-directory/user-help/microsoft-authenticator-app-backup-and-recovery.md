---
title: 使用 Microsoft Authenticator 應用程式進行備份與復原 - Azure Active Directory | Microsoft Docs
description: 瞭解如何使用 Microsoft Authenticator 應用程式來備份和復原您的帳戶認證。
services: active-directory
author: eross-msft
manager: mtillman
ms.component: user-help
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/28/2018
ms.author: lizross
ms.reviewer: olhaun
ms.openlocfilehash: b019523227bcd45e502ec84055cfb09025918331
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406119"
---
# <a name="backup-and-recover-account-credentials-with-the-microsoft-authenticator-app"></a>使用 Microsoft Authenticator 應用程式保來備份和復原帳戶認證

**適用範圍：**

- iOS 裝置

Microsoft Authenticator 應用程式會將您的帳戶認證和相關應用程式設定 (例如您的帳戶順序) 備份到雲端。 備份之後，您也可以使用此應用程式在新裝置上復原您的資訊，這有可能避免遭到鎖定或者必須重新建立帳戶。

>[!IMPORTANT]
> 對於每個備份儲存位置，您需要一個個人 Microsoft 帳戶和一個 iCloud 帳戶。 但是在該儲存位置內，您可以備份數個帳戶。 例如，您可以有個人帳戶、學校帳戶，以及 Facebook、Google 等的第三方帳戶。<br><br>系統只會儲存您的個人帳戶認證和協力廠商帳戶認證，包括您的使用者名稱與證明身分識別所需的帳戶驗證碼。 我們不會儲存任何與您帳戶相關聯的其他資訊，包括電子郵件或檔案。 我們也不會以任何方式或與其他產品或服務關聯或共用您的帳戶。 最後，您的 IT 管理員將不會取得有關上述任何帳戶的任何資訊。

## <a name="back-up-your-account-credentials"></a>備份您的帳戶認證
您必須先具備下列兩者，才能備份您的認證：

- 個人 [Microsoft 帳戶](https://account.microsoft.com/account)作為復原帳戶。

- [iCloud 帳戶](https://www.icloud.com/)作為實際儲存位置。 

要求您同時登入這兩個帳戶，可為您的備份資訊提供更強大的安全性。

**開啟雲端備份**
-   在您的 iOS 裝置上，選取 [設定]、[備份]，然後開啟 [自動備份]。

    您的帳戶認證會備份到您的 iCloud 帳戶。

    ![iOS 設定畫面，其中顯示自動備份設定的位置](./media/microsoft-authenticator-app-backup-and-recovery/backup-and-recovery-turn-on.png)

## <a name="recover-your-account-credentials-on-your-new-device"></a>在您的新裝置上復原您的帳戶認證
您可以從您的 iCloud 帳戶復原帳戶認證，方法是使用您在備份您的資訊時的相同 Microsoft 復原帳戶。

### <a name="to-recover-your-information"></a>復原您的資訊
1.  在您的 iOS 裝置上開啟 Microsoft Authenticator 應用程式，然後選取畫面底部的 [開始復原]。

    ![Microsoft Authenticator 應用程式，其中顯示可按一下 [開始復原] 的位置](./media/microsoft-authenticator-app-backup-and-recovery/backup-and-recovery-begin-recovery.png)

2.  使用您在備份過程中所使用的相同個人 Microsoft 帳戶，登入您的復原帳戶。

    您的帳戶認證會復原到新的裝置。

完成復原之後，您可能會注意到舊電話與新電話上 Microsoft Authenticator 應用程式中的個人 Microsoft 帳戶驗證碼不同。 驗證碼不同，因為每個裝置都有自己的唯一認證，但是在使用相關聯的電話進行登入時兩者均有效且有作用。

## <a name="recover-additional-accounts-requiring-more-verification"></a>復原其他需要更多驗證的帳戶
如果您使用推播通知搭配您的個人、公司或學校帳戶，您會在畫面上看到一則警示，指出您必須先提供額外的驗證，才可以復原您的資訊。 由於推播通知需要使用已繫結至特定裝置且永遠不會透過網路傳送的認證，所以您必須先證明您的身分識別，才能在裝置上建立認證。

對於個人 Microsoft 帳戶，您可輸入您的密碼以及替代電子郵件或電話號碼來證明您的身分識別。 針對公司或學校帳戶，您必須掃描帳戶提供者提供給您的 QR 代碼。

### <a name="to-provide-additional-verification-for-personal-accounts"></a>提供額外的個人帳戶驗證
1.  在 Microsoft Authenticator 應用程式的 [帳戶] 畫面中，選取您要復原的帳戶旁的下拉式箭號。

    ![Microsoft Authenticator 應用程式，其中顯示可用的帳戶及其相關聯的下拉式箭號](./media/microsoft-authenticator-app-backup-and-recovery/backup-and-recovery-arrow.png)

2.  選取 [登入進行復原]，輸入您的密碼，然後確認您的電子郵件地址或電話號碼作為額外的驗證。

    ![Microsoft Authenticator 應用程式，可讓您輸入您的登入資訊](./media/microsoft-authenticator-app-backup-and-recovery/backup-and-recovery-sign-in.png)

### <a name="to-provide-additional-verification-for-work-or-school-accounts"></a>提供額外的公司或學校帳戶驗證
1.  在 Microsoft Authenticator 應用程式的 [帳戶] 畫面中，選取您要復原的帳戶旁的下拉式箭號。

    ![Microsoft Authenticator 應用程式，其中顯示可用的帳戶及其相關聯的下拉式箭號](./media/microsoft-authenticator-app-backup-and-recovery/backup-and-recovery-additonal-accts.png)

2.  選取 [掃描 QR 代碼進行復原]，然後掃描 QR 代碼。

    ![Microsoft Authenticator 應用程式，可讓您掃描您的 QR 代碼](./media/microsoft-authenticator-app-backup-and-recovery/backup-and-recovery-scan-qr-code.png)

    >[!NOTE]
    >如需如何取得 QR 代碼的詳細資訊，請參閱[開始使用 Microsoft Authenticator 應用程式](https://docs.microsoft.com/azure/active-directory/user-help/microsoft-authenticator-app-how-to#add-accounts-to-the-app)或[將安全性資訊設定為使用驗證器應用程式](https://docs.microsoft.com/azure/active-directory/user-help/security-info-setup-auth-app#to-use-the-microsoft-authenticator-app) (取決於系統管理員是否已開啟安全性資訊)。

## <a name="troubleshooting-backup-and-recovery-problems"></a>針對備份和復原問題進行疑難排解
您的備份無法使用的可能原因如下：

-   **變更作業系統。** 您的備份會儲存在手機作業系統所提供的雲端儲存選項，這表示如果您在 Android 與 iOS 之間切換，便無法使用備份。 在此情況下，您必須在此應用程式內手動重新建立您的帳戶。

-   **網路或密碼問題。** 確定您已連線到網路，並已使用您在前一支 iPhone 上使用的相同 AppleID 登入 iCloud 帳戶。

-   **意外刪除。** 您可能已從先前的裝置或在管理雲端儲存體帳戶時刪除備份帳戶。 在此情況下，您必須在此應用程式內手動重新建立您的帳戶。

-   **現有的 Microsoft Authenticator 帳戶。** 如果您已經在 Microsoft Authenticator 應用程式中設定帳戶，則應用程式將無法復原您已備份的帳戶。 防止復原有助於確保過時資訊不會覆寫您的帳戶詳細資料。 在此情況下，您必須先從在 Microsoft Authenticator 應用程式中設定的現有帳戶移除任何現有的帳戶資訊，才可以復原您的備份。

## <a name="next-steps"></a>後續步驟
既然您已將帳戶認證備份和復原至新裝置，即可繼續使用 Microsoft Authenticator 應用程式來驗證您的身分識別。

## <a name="related-topics"></a>相關主題
- [開始使用 Microsoft Authenticator 應用程式](microsoft-authenticator-app-how-to.md)  

- [Microsoft Authenticator 應用程式常見問題集](microsoft-authenticator-app-faq.md)

- [Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/)
