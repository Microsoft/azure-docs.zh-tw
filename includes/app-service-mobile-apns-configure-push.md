---
author: conceptdev
ms.author: crdun
ms.service: app-service-mobile
ms.topic: include
ms.date: 08/23/2018
ms.openlocfilehash: 25bb5cfb0073f7533faddec43b38fb5031b82a43
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42809647"
---
1. 在您的 Mac 上啟動 **Keychain Access**。 在左側導覽列的 [類別] 下，開啟 [我的憑證]。 尋找您在上一節中下載的 SSL 憑證，然後公開其內容。 僅選取憑證 (不選取私密金鑰)。 然後[將它匯出](https://support.apple.com/kb/PH20122?locale=en_US)。
2. 在 [Azure 入口網站](https://portal.azure.com/)中，選取 [瀏覽全部] > [應用程式服務]。 然後選取您的 Mobile Apps 後端。 
3. 在 [設定] 下，選取 [App Service 推播]。 然後選取您的通知中樞名稱。 
4. 移至 [Apple 推播通知服務] > [上傳憑證]。 上傳 .p12 檔案，選取正確的**模式** (根據您稍早的用戶端 SSL 憑證為生產或沙箱)。 儲存任何變更。

您的服務現在已設定成在 iOS 上使用推播通知。

[1]: ./media/app-service-mobile-apns-configure-push/mobile-push-notification-hub.png
