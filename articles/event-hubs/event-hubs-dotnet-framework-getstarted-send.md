---
title: 使用 .NET Framework 將事件傳送至 Azure 事件中樞 | Microsoft Docs
description: 使用 .NET Framework 開始將事件傳送至事件中樞
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: c4974bd3-2a79-48a1-aa3b-8ee2d6655b28
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/03/2018
ms.author: shvija
ms.openlocfilehash: 7b5a4298ee4c67f0300bd4aabb7fc6373d8edba0
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005285"
---
# <a name="send-events-to-azure-event-hubs-using-the-net-framework"></a>使用 .NET Framework 將事件傳送至 Azure 事件中樞

「事件中樞」是一種服務，可處理來自連接裝置和應用程式的大量事件資料 (遙測)。 收集資料至「事件中樞」之後，可以使用存放裝置叢集來儲存資料，或使用即時分析提供者進行轉換。 此大規模事件收集和處理功能是新型應用程式架構 (包括物聯網 (IoT)) 的重要元件。

本教學課程示範如何使用 [Azure 入口網站](https://portal.azure.com)來建立事件中樞。 它也會示範如何使用在 C# 中使用 .NET Framework 撰寫的主控台應用程式，將事件傳送至事件中樞。 若要使用 .NET Framework 接收事件，請參閱[使用 .NET Framework 接收事件](event-hubs-dotnet-framework-getstarted-receive-eph.md)一文，或按一下左側目錄中適當的接收語言。

若要完成本教學課程，您需要下列必要條件：

* [Microsoft Visual Studio 2017 或更高版本](http://visualstudio.com)。
* 使用中的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/free/)。

## <a name="create-an-event-hubs-namespace-and-an-event-hub"></a>建立事件中樞命名空間和事件中樞

第一個步驟是使用 [Azure 入口網站](https://portal.azure.com)來建立「事件中樞」類型的命名空間，然後取得您應用程式與「事件中樞」進行通訊所需的管理認證。 若要建立命名空間和「事件中樞」，請依照[這篇文章](event-hubs-create.md)中的程序操作，然後繼續進行本教學課程中的下列步驟。

## <a name="create-a-sender-console-application"></a>建立傳送者主控台應用程式

在本節中，您將撰寫一個 Windows 主控台應用程式，以將事件傳送至事件中心。

1. 在 Visual Studio 中，使用 **主控台應用程式** 專案範本建立新的「Visual C# 桌面應用程式」專案。 將專案命名為 **Sender**。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-send/create-sender-csharp1.png)
2. 在 [方案總管] 中，以滑鼠右鍵按一下 [寄件者] 專案，然後按一下 [管理方案的 NuGet 封裝]。 
3. 按一下 [瀏覽] 索引標籤，然後搜尋 `WindowsAzure.ServiceBus`。 按一下 [安裝] 並接受使用條款。 
   
    ![](./media/event-hubs-dotnet-framework-getstarted-send/create-sender-csharp2.png)
   
    Visual Studio 會下載、安裝並新增 [Azure 服務匯流排程式庫 NuGet 封裝](https://www.nuget.org/packages/WindowsAzure.ServiceBus)的參考。
4. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：
   
  ```csharp
  using System.Threading;
  using Microsoft.ServiceBus.Messaging;
  ```
5. 將下列欄位新增至 **Program** 類別，並以您在上一節中建立的事件中樞名稱替代預留位置值，以及您先前儲存的命名空間層級連接字串。
   
  ```csharp
  static string eventHubName = "Your Event Hub name";
  static string connectionString = "namespace connection string";
  ```
6. 將下列方法加入至 **Program** 類別：
   
  ```csharp
  static void SendingRandomMessages()
  {
      var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
      while (true)
      {
          try
          {
              var message = Guid.NewGuid().ToString();
              Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, message);
              eventHubClient.Send(new EventData(Encoding.UTF8.GetBytes(message)));
          }
          catch (Exception exception)
          {
              Console.ForegroundColor = ConsoleColor.Red;
              Console.WriteLine("{0} > Exception: {1}", DateTime.Now, exception.Message);
              Console.ResetColor();
          }
   
          Thread.Sleep(200);
      }
  }
  ```
   
  這個方法會以 200 毫秒的延遲時間持續將事件傳送至事件中心。
7. 最後，將下列幾行新增至 **Main** 方法：
   
  ```csharp
  Console.WriteLine("Press Ctrl-C to stop the sender process");
  Console.WriteLine("Press Enter to start now");
  Console.ReadLine();
  SendingRandomMessages();
  ```
8. 執行程式，並確定沒有任何錯誤。
  
恭喜！ 您現在已將傳送訊息到事件中樞。

## <a name="next-steps"></a>後續步驟

您現在已建置一個可建立事件中樞及傳送資料的工作應用程式，接下來可繼續進行下列案例：

* [使用事件處理器主機接收事件](event-hubs-dotnet-framework-getstarted-receive-eph.md)
* [事件處理器主機參考](/dotnet/api/microsoft.servicebus.messaging.eventprocessorhost)
* [事件中心概觀](event-hubs-what-is-event-hubs.md)

<!-- Images. -->
[19]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj1.png
[20]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj2.png
[21]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs1.png
[22]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs2.png

