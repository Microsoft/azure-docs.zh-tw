---
title: 包含檔案
description: 包含檔案
services: service-bus-relay
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 08/16/2018
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: 8d73a22473ffff358c7424249c7581f6af740718
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452799"
---
### <a name="create-a-console-application"></a>建立主控台應用程式

如果您已在建立轉送時停用 [Requires Client Authorization] \(需要用戶端授權\) 選項，則可以使用任何瀏覽器將要求傳送至混合式連線 URL。 若要存取受保護的端點，您需要建立權杖，並在 `ServiceBusAuthorization` 標頭中傳遞權杖，如下所示。

在 Visual Studio 中建立新的**主控台應用程式 (.NET Framework)** 專案。

### <a name="add-the-relay-nuget-package"></a>新增轉送 NuGet 封裝

1. 以滑鼠右鍵按一下新建立的專案，然後選取 [管理 NuGet 套件]。
2. 選取 [包括搶鮮版] 選項。 
3. 選取 [瀏覽]，然後搜尋 **Microsoft.Azure.Relay**。 在搜尋結果中，選取 [Microsoft Azure 轉送]。
4. 針對版本，選取 [2.0.0-preview1-20180523]。 
5. 選取 [安裝] 以完成安裝。 關閉對話方塊。

### <a name="write-code-to-send-requests"></a>撰寫程式碼來傳送要求

1. 在 Program.cs 檔案的頂端，將現有 `using` 陳述式取代為下列 `using` 陳述式：
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Net.Http;
    using Microsoft.Azure.Relay;
    ```
2. 將常數新增至 `Program` 類別以取得混合式連線詳細資料。 將方括號中的預留位置取代為您在建立混合式連線時所取得的值。 務必使用完整命名空間名稱。
   
    ```csharp
    private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
    private const string ConnectionName = "{HybridConnectionName}";
    private const string KeyName = "{SASKeyName}";
    private const string Key = "{SASKey}";
    ```
3. 將下列方法新增至 `Program` 類別：
   
    ```csharp
    private static async Task RunAsync()
    {
        var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                KeyName, Key);
        var uri = new Uri(string.Format("https://{0}/{1}", RelayNamespace, ConnectionName));
        var token = (await tokenProvider.GetTokenAsync(uri.AbsoluteUri, TimeSpan.FromHours(1))).TokenString;
        var client = new HttpClient();
        var request = new HttpRequestMessage()
        {
            RequestUri = uri,
            Method = HttpMethod.Get,
        };
        request.Headers.Add("ServiceBusAuthorization", token);
        var response = await client.SendAsync(request);
        Console.WriteLine(await response.Content.ReadAsStringAsync());        Console.ReadLine();
    }
    ```
4. 將下列程式碼行新增至 `Program` 類別中的 `Main` 方法。
   
    ```csharp
    RunAsync().GetAwaiter().GetResult();
    ```
   
    Program.cs 看起來應該像這樣：
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.Relay;
   
    namespace Client
    {
        class Program
        {
            private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
            private const string ConnectionName = "{HybridConnectionName}";
            private const string KeyName = "{SASKeyName}";
            private const string Key = "{SASKey}";
   
            static void Main(string[] args)
            {
                RunAsync().GetAwaiter().GetResult();
            }
   
            private static async Task RunAsync()
            {
               var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                KeyName, Key);
                var uri = new Uri(string.Format("https://{0}/{1}", RelayNamespace, ConnectionName));
                var token = (await tokenProvider.GetTokenAsync(uri.AbsoluteUri, TimeSpan.FromHours(1))).TokenString;
                var client = new HttpClient();
                var request = new HttpRequestMessage()
                {
                    RequestUri = uri,
                    Method = HttpMethod.Get,
                };
                request.Headers.Add("ServiceBusAuthorization", token);
                var response = await client.SendAsync(request);
                Console.WriteLine(await response.Content.ReadAsStringAsync());
            }
        }
    }
    ```

