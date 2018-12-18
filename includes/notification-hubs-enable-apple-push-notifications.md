---
title: 包含檔案
description: 包含檔案
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 08/28/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: bf73f64cc1e1789b770d7fae5f62e0ec27425263
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42915158"
---
## <a name="generate-the-certificate-signing-request-file"></a>產生憑證簽署要求檔案

Apple Push Notification Service (APNS) 使用憑證來驗證您的通知服務。 遵循下列指示來建立必要的推播憑證，以便傳送和接收通知。 如需這些概念的詳細資訊，請參閱 [Apple Push Notification Service](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html) 官方文件。

產生憑證簽署要求 (CSR) 檔案，這將由 Apple 用來產生簽署的推播憑證。

1. 在您的 Mac 上，執行**金鑰鏈存取**工具。 它可從 Launch Pad 上的 [公用程式] 資料夾或 [其他] 資料夾開啟。
2. 按一下 [金鑰鏈存取]，並展開 [憑證助理]，然後按一下 [從憑證授權單位要求憑證...]。

    ![使用金鑰鏈存取要求新憑證](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-request-cert-from-ca.png)
3. 選取您的 [使用者電子郵件地址] 和 [一般名稱]，確定已勾選 [儲存至磁碟]，然後按一下 [繼續]。 請將 [CA Email Address]  欄位留空，因為它不是必要資訊。

    ![必要的憑證資訊](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-csr-info.png)

4. 在 [另存新檔] 中輸入憑證簽署要求 (CSR) 檔案的名稱，並且在 [位置] 中選取位置，然後按一下 [儲存]。

    ![選擇憑證的檔案名稱](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-save-csr.png)

    這個動作會在選取的位置中儲存 CSR 檔案，預設的位置是在桌面上。 請記住為檔案選擇的位置。

接著，您會向 Apple 註冊您的應用程式，並啟用推播通知，然後上傳這個匯出的 CSR 建立推播憑證。

## <a name="register-your-app-for-push-notifications"></a>針對推播通知註冊應用程式

若要將推播通知傳送至 iOS 應用程式，您必須向 Apple 註冊您的應用程式，並註冊進行推播通知。  

1. 如果您尚未註冊應用程式，請瀏覽至 Apple 開發人員中心的 [iOS 佈建入口網站](http://go.microsoft.com/fwlink/p/?LinkId=272456)，然後使用您的 Apple ID 登入，並按一下 [識別碼]，接著按一下 [應用程式識別碼]，最後按一下 **+** 符號註冊新的應用程式。

    ![iOS 佈建入口網站應用程式識別碼頁面](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-ios-appids.png)

2. 將新應用程式的下列三個欄位予以更新，然後按一下 [繼續] ：

    * **名稱**：在 [應用程式識別碼描述] 區段的 [名稱] 欄位中輸入應用程式的描述名稱。
    * **套件組合識別碼**：在 [明確的應用程式識別碼] 區段之下，以[應用程式分發指南](https://help.apple.com/xcode/mac/current/#/dev91fe7130a) (英文) 中所提的 `<Organization Identifier>.<Product Name>` 形式輸入 [套件組合識別碼]。 您使用的 [組織識別碼] 和 [產品名稱] 必須符合當您建立 XCode 專案時使用的組織識別碼和產品名稱。 在下列的快照中，*NotificationHubs* 做為組織識別碼，*GetStarted* 做為產品名稱。 確定此值符合您在 XCode 專案中使用的值，讓您使用正確的發行設定檔搭配 XCode。
    * **推播通知**：勾選 [應用程式服務] 區段中的 [推播通知] 選項。

    ![註冊新應用程式識別碼的表單](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-new-appid-info.png)

    此動作會產生您的應用程式識別碼，並要求您確定此資訊。 按一下 [註冊]  確認新的應用程式識別碼。

    按一下 [註冊] 之後，您會看見 [註冊完成] 畫面，如下圖所示。 按一下 [完成] 。

    ![應用程式識別碼註冊完成並顯示權利](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-appid-registration-complete.png)

3. 在開發人員中心中，於 [App IDs] 下找出您建立的 App ID，並在該列上按一下。

    ![應用程式識別碼清單](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-ios-appids2.png)

    按一下應用程式識別碼，即可顯示應用程式詳細資料。 按一下底部的 [編輯]  按鈕。

    ![編輯應用程式識別碼頁面](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-edit-appid.png)

4. 捲動到畫面底部，然後按一下 [Development Push SSL Certificate] 區段下方的 [建立憑證...] 按鈕。

    ![為應用程式識別碼建立憑證的按鈕](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-appid-create-cert.png)

    您會看到**新增 iOS 憑證**助理。

    > [!NOTE]
    > 本教學課程使用開發憑證。 註冊生產憑證時，將使用同一個程序。 只要確定在傳送通知時使用相同的憑證類型即可。

5. 按一下 [選擇檔案]，瀏覽到您在第一個工作中所建立之 CSR 檔案的儲存位置，然後按一下 [產生]。

    ![產生的憑證 CSR 上傳頁面](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-appid-cert-choose-csr.png)

6. 在入口網站建立憑證之後，依序按 [下載] 按鈕和 [完成]。

    ![產生的憑證 CSR 下載頁面](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-appid-download-cert.png)

    這會下載憑證並儲存到您電腦中的 [下載] 資料夾。

    ![在 [下載] 資料夾中找出憑證檔案](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-cert-downloaded.png)

    > [!NOTE]
    > 依預設，下載的檔案 (開發憑證) 的名稱會是 **aps_development.cer**。

7. 按兩下下載的推播憑證 **aps_development.cer**。

    此動作會將新的憑證安裝在金鑰鏈中，如下圖所示：

    ![顯示新憑證的金鑰鏈存取憑證清單](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-cert-in-keychain.png)

    > [!NOTE]
    > 憑證中的名稱可能會不同，不過字首會加上 **Apple Development iOS Push Services** 前置詞。

8. 在 Keychain Access 中，以滑鼠右鍵按一下您在 [憑證]  類別中建立的新推播憑證。 按一下 [匯出]、為檔案命名、選取 [.p12] 格式，然後按一下 [儲存]。

    ![以 p12 格式匯出憑證](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-export-cert-p12.png)

    記下匯出的 .p12 憑證的檔案名稱和位置。 這用於啟用 APNS 驗證功能。

    > [!NOTE]
    > 本教學課程會建立 QuickStart.p12 檔案。 Your file name and location might be different.

## <a name="create-a-provisioning-profile-for-the-app"></a>建立應用程式的佈建設定檔

1. 返回 [iOS 佈建入口網站](http://go.microsoft.com/fwlink/p/?LinkId=272456)，並依序選取 [佈建設定檔] 和 [全部]，然後按一下 **+** (加號) 按鈕以建立新的設定檔。 您會看到 [新增 iOS 佈建設定檔] 精靈：

    ![佈建設定檔清單](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-new-provisioning-profile.png)

2. 選取 [Development] 下的 [iOS App Development] 作為佈建設定檔類型，然後按一下 [Continue]。

3. 接著，從 [應用程式識別碼] 下拉式清單選取您建立的應用程式識別碼，然後按一下 [繼續]

    ![選取應用程式識別碼](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-select-appid-for-provisioning.png)

4. 在 [選取憑證] 畫面中，選取用於程式碼簽署的一般開發憑證，然後按一下 [繼續]。 此憑證不是您建立的推播憑證。

    ![選取憑證](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-provisioning-select-cert.png)

5. 接著，選取要用來測試的 [裝置]，然後按一下 [繼續]

    ![選取裝置](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-provisioning-select-devices.png)

6. 最後，在 [設定檔名稱] 中為設定檔挑選名稱，按一下 [產生]。

    ![選擇佈建設定檔名稱](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-provisioning-name-profile.png)

7. 當新的佈建設定檔建立好時，按一下它來下載並安裝在 Xcode 開發電腦上。 然後按一下 [完成]。

    ![下載佈建設定檔](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-provisioning-profile-ready.png)
