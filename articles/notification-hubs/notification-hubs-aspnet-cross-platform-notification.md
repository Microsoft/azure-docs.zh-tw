---
title: 使用 Azure 通知中樞向使用者傳送跨平台通知 (ASP.NET)
description: 了解如何使用通知中樞範本，在單一要求中傳送以所有平台為目標的跨平台通知。
services: notification-hubs
documentationcenter: ''
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 11d2131b-f683-47fd-a691-4cdfc696f62b
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: multiple
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: c9d1874fb611b349403736593fdc9eccc45d2d4d
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42141479"
---
# <a name="send-cross-platform-notifications-to-users-with-notification-hubs"></a>使用通知中心向使用者傳送跨平台通知
在之前的[使用通知中心來通知使用者]教學課程中，您已了解如何將通知推播至所有由特定經驗證使用者所註冊的裝置。 在該教學課程中，需要用多個要求來傳送通知給每個支援的用戶端平台。 Azure 通知中心可支援範本，讓您指定特定裝置接收通知的方式。 此方法使得傳送跨平台通知變得更簡單。 

本文示範如何運用範本，在單一要求中傳送以所有平台為目標的跨平台通知。 如需這些範本的詳細資訊，請參閱 [Azure 通知中樞概觀][Templates]。

> [!IMPORTANT]
> Windows Phone 8.1 及更早版本的專案不支援 Visual Studio 2017。 如需詳細資訊，請參閱 [Visual Studio 2017 平台目標及相容性](https://www.visualstudio.com/en-us/productinfo/vs2017-compatibility-vs)。

> [!NOTE]
> 透過通知中樞，裝置可註冊多個具有相同標籤的範本。 在此情況下，當傳入的訊息符合該標籤時，就會有多個通知傳遞至裝置 (每個通知各用於一個範本)。 透過此流程，您就能讓相同訊息顯示在多個視覺通知中，例如以徽章形式和 Windows 市集應用程式中的快顯通知形式。
> 
> 

若要使用範本傳送跨平台通知，請執行下列動作：

1. 在 Visual Studio 的 [方案總管] 中展開 [控制器]  資料夾，然後開啟 RegisterController.cs 檔案。

2. 在 **Put** 方法中找出建立新註冊的程式碼區塊，並將 `switch` 內容取代為下列程式碼：
   
        switch (deviceUpdate.Platform)
        {
            case "mpns":
                var toastTemplate = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                    "<wp:Notification xmlns:wp=\"WPNotification\">" +
                       "<wp:Toast>" +
                            "<wp:Text1>$(message)</wp:Text1>" +
                       "</wp:Toast> " +
                    "</wp:Notification>";
                registration = new MpnsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
                break;
            case "wns":
                toastTemplate = @"<toast><visual><binding template=""ToastText01""><text id=""1"">$(message)</text></binding></visual></toast>";
                registration = new WindowsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
                break;
            case "apns":
                var alertTemplate = "{\"aps\":{\"alert\":\"$(message)\"}}";
                registration = new AppleTemplateRegistrationDescription(deviceUpdate.Handle, alertTemplate);
                break;
            case "gcm":
                var messageTemplate = "{\"data\":{\"message\":\"$(message)\"}}";
                registration = new GcmTemplateRegistrationDescription(deviceUpdate.Handle, messageTemplate);
                break;
            default:
                throw new HttpResponseException(HttpStatusCode.BadRequest);
        }
   
    這段程式碼會呼叫平台特有方法來建立範本註冊，而非原生註冊。 因為範本註冊源自原生註冊，因此不需要修改現有註冊。

3. 在 [通知] 控制器中，將 **sendNotification** 方法取代為下列程式碼：
   
        public async Task<HttpResponseMessage> Post()
        {
            var user = HttpContext.Current.User.Identity.Name;
            var userTag = "username:" + user;
   
            var notification = new Dictionary<string, string> { { "message", "Hello, " + user } };
            await Notifications.Instance.Hub.SendTemplateNotificationAsync(notification, userTag);
   
            return Request.CreateResponse(HttpStatusCode.OK);
        }
   
    這段程式碼會同時將通知傳送至所有平台，完全不需要指定原生裝載。 通知中樞將以所提供的 *tag* 值 (指定於註冊的範本中) 建置並傳遞正確的裝載到每個裝置。

4. 重新發佈您的 WebApi 後端專案。

5. 重新執行用戶端應用程式，然後驗證註冊已成功。

6. (選用) 將此用戶端應用程式部署到第二個裝置，然後執行此應用程式。
    每個裝置上都會顯示通知。

## <a name="next-steps"></a>後續步驟
您已完成本教學課程，現在可參閱下列主題進一步了解通知中心和範本：

* [Use Notification Hubs to send breaking news]: Demonstrates another scenario for using templates.
* [Azure 通知中樞概觀][Templates]：包含範本的詳細資訊。

<!-- Anchors. -->

<!-- Images. -->




<!-- URLs. -->
[Push to users ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Push to users Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Visual Studio 2012 Express for Windows 8]: http://go.microsoft.com/fwlink/?LinkId=257546

[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Azure Notification Hubs]: http://go.microsoft.com/fwlink/p/?LinkId=314257
[使用通知中心來通知使用者]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Templates]: http://go.microsoft.com/fwlink/p/?LinkId=317339
[Notification Hub How to for Windows Store]: http://msdn.microsoft.com/library/windowsazure/jj927172.aspx
