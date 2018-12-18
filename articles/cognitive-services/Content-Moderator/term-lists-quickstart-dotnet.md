---
title: 在 C# 中根據自訂字詞清單檢查文字 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 如何使用 Content Moderator SDK for C# 以自訂字詞清單仲裁文字。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: quickstart
ms.date: 10/10/2018
ms.author: sajagtap
ms.openlocfilehash: af65fffaf864fd4d5b7123b4c6b66644ef6624e7
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49310859"
---
# <a name="quickstart-check-text-against-a-custom-term-list-in-c"></a>快速入門：在 C# 中根據自訂字詞清單檢查文字

Azure Content Moderator 中的預設全域字詞清單已可滿足大部分內容審核需求。 不過，您可能會需要審查屬於您組織的特定字詞。 例如，您可能要標記競爭對手名稱以供進一步檢閱。 

您可以使用 [Content Moderator SDK for .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 建立自訂字詞清單，以搭配文字仲裁 API 使用。

本文提供資訊和範例程式碼，可協助您開始使用 Content Moderator SDK for .NET 來執行下列操作：
- 建立清單。
- 將字詞新增至清單。
- 針對清單中的字詞審查字詞。
- 刪除清單中的字詞。
- 刪除清單。
- 編輯清單資訊。
- 重新整理索引，以便將清單變更包含在新的掃描中。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

## <a name="sign-up-for-content-moderator-services"></a>註冊 Content Moderator 服務

您必須有訂用帳戶金鑰，才能透過 REST API 或 SDK 使用 Content Moderator 服務。 請在 [Azure 入口網站](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator)中訂閱 Content Moderator 服務，以取得此項目。

## <a name="create-your-visual-studio-project"></a>建立 Visual Studio 專案

1. 將一個新的 [主控台應用程式 (.NET Framework)] 專案新增到您的解決方案。

1. 將專案命名為 **TermLists**。 選取此專案作為解決方案的單一啟始專案。

### <a name="install-required-packages"></a>安裝必要的套件

安裝下列適用於 TermLists 專案的 NuGet 套件：

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Microsoft.Rest.ClientRuntime.Azure
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>更新程式的 using 陳述式

新增下列 `using` 陳述式。

```csharp
using Microsoft.Azure.CognitiveServices.ContentModerator;
using Microsoft.CognitiveServices.ContentModerator;
using Microsoft.CognitiveServices.ContentModerator.Models;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
```

### <a name="create-the-content-moderator-client"></a>建立 Content Moderator 用戶端

新增下列程式碼，為您的訂用帳戶建立 Content Moderator 用戶端。

> [!IMPORTANT]
> 以您的區域識別碼和訂用帳戶訂用帳戶的值更新 **AzureRegion** 和 **CMSubscriptionKey** 欄位。

```csharp
/// <summary>
/// Wraps the creation and configuration of a Content Moderator client.
/// </summary>
/// <remarks>This class library contains insecure code. If you adapt this 
/// code for use in production, use a secure method of storing and using
/// your Content Moderator subscription key.</remarks>
public static class Clients
{
    /// <summary>
    /// The region/location for your Content Moderator account, 
    /// for example, westus.
    /// </summary>
    private static readonly string AzureRegion = "YOUR API REGION";

    /// <summary>
    /// The base URL fragment for Content Moderator calls.
    /// </summary>
    private static readonly string AzureBaseURL =
        $"https://{AzureRegion}.api.cognitive.microsoft.com";

    /// <summary>
    /// Your Content Moderator subscription key.
    /// </summary>
    private static readonly string CMSubscriptionKey = "YOUR API KEY";

    /// <summary>
    /// Returns a new Content Moderator client for your subscription.
    /// </summary>
    /// <returns>The new client.</returns>
    /// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
    /// When you have finished using the client,
    /// you should dispose of it either directly or indirectly. </remarks>
    public static ContentModeratorClient NewClient()
    {
        // Create and initialize an instance of the Content Moderator API wrapper.
        ContentModeratorClient client = new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey));

        client.Endpoint = AzureBaseURL;
        return client;
    }
}
```

### <a name="add-private-properties"></a>新增私有屬性

將下列私有屬性新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// The language of the terms in the term lists.
/// </summary>
private const string lang = "eng";

/// <summary>
/// The minimum amount of time, in milliseconds, to wait between calls
/// to the Content Moderator APIs.
/// </summary>
private const int throttleRate = 3000;

/// <summary>
/// The number of minutes to delay after updating the search index before
/// performing image match operations against the list.
/// </summary>
private const double latencyDelay = 0.5;
```

## <a name="create-a-term-list"></a>建立字詞清單

使用 **ContentModeratorClient.ListManagementTermLists.Create** 建立字詞清單。 要**建立**的第一個參數是包含 MIME 類型的字串，這應該是 "application/json"。 如需詳細資訊，請參閱 [API 參考](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f) \(英文\)。 第二個參數是**主體**物件，其中包含新字詞清單的名稱和描述。

> [!NOTE]
> 上限是 **5 個字詞清單**，其中每個清單**不可超過 10,000 個字詞**。

將下列方法定義新增至 TermLists 命名空間、Program 類別。

> [!NOTE]
> 您的 Content Moderator 服務金鑰會有每秒要求數目 (RPS) 的速率限制。如果超出此限制，SDK 就會擲回錯誤碼為 429 的例外狀況。 免費層金鑰有一個 RPS 速率限制。

```csharp
/// <summary>
/// Creates a new term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <returns>The term list ID.</returns>
static string CreateTermList (ContentModeratorClient client)
{
    Console.WriteLine("Creating term list.");

    Body body = new Body("Term list name", "Term list description");
    TermList list = client.ListManagementTermLists.Create("application/json", body);
    if (false == list.Id.HasValue)
    {
        throw new Exception("TermList.Id value missing.");
    }
    else
    {
        string list_id = list.Id.Value.ToString();
        Console.WriteLine("Term list created. ID: {0}.", list_id);
        Thread.Sleep(throttleRate);
        return list_id;
    }
}
```

## <a name="update-term-list-name-and-description"></a>更新字詞清單的名稱和描述

使用 **ContentModeratorClient.ListManagementTermLists.Update** 更新字詞清單資訊。 第一個要**更新**的參數是字詞清單識別碼。 第二個參數是 MIME 類型，這應該是 "application/json"。 如需詳細資訊，請參閱 [API 參考](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f685) \(英文\)。 第三個參數是**主體**物件，其中包含新的名稱和描述。

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Update the information for the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to update.</param>
/// <param name="name">The new name for the term list.</param>
/// <param name="description">The new description for the term list.</param>
static void UpdateTermList (ContentModeratorClient client, string list_id, string name = null, string description = null)
{
    Console.WriteLine("Updating information for term list with ID {0}.", list_id);
    Body body = new Body(name, description);
    client.ListManagementTermLists.Update(list_id, "application/json", body);
    Thread.Sleep(throttleRate);
}
```

## <a name="add-a-term-to-a-term-list"></a>將字詞新增至字詞清單

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Add a term to the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to update.</param>
/// <param name="term">The term to add to the term list.</param>
static void AddTerm (ContentModeratorClient client, string list_id, string term)
{
    Console.WriteLine("Adding term \"{0}\" to term list with ID {1}.", term, list_id);
    client.ListManagementTerm.AddTerm(list_id, term, lang);
    Thread.Sleep(throttleRate);
}
```

## <a name="get-all-terms-in-a-term-list"></a>取得字詞清單中所有字詞

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Get all terms in the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to get all terms.</param>
static void GetAllTerms(ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Getting terms in term list with ID {0}.", list_id);
    Terms terms = client.ListManagementTerm.GetAllTerms(list_id, lang);
    TermsData data = terms.Data;
    foreach (TermsInList term in data.Terms)
    {
        Console.WriteLine(term.Term);
    }
    Thread.Sleep(throttleRate);
}
```

## <a name="add-code-to-refresh-the-search-index"></a>新增程式碼以重新整理搜尋索引

變更字詞清單之後，重新整理其搜尋索引即可將變更納入其中，下次您就可使用此字詞清單來審查文字。 這就像桌面搜尋引擎 (如果已啟用) 或 Web 搜尋引擎持續重新整理其索引以包含新檔案或頁面一樣。

使用 **ContentModeratorClient.ListManagementTermLists.RefreshIndexMethod** 重新整理字詞清單搜尋索引。

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Refresh the search index for the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to refresh.</param>
static void RefreshSearchIndex (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Refreshing search index for term list with ID {0}.", list_id);
    client.ListManagementTermLists.RefreshIndexMethod(list_id, lang);
    Thread.Sleep((int)(latencyDelay * 60 * 1000));
}
```

## <a name="screen-text-using-a-term-list"></a>使用字詞清單審查文字

透過 **ContentModeratorClient.TextModeration.ScreenText** 使用字詞清單來審查文字時可利用下列參數。

- 字詞清單中的字詞語言。
- MIME 類型，其可以是 "text/html"、"text/xml"、"text/markdown" 或 "text/plain"。
- 要審查的文字。
- 布林值。 將此欄位設為 **true**，可在審查文字前自動校正文字。
- 布林值。 將此欄位設為 **true**，可偵測文字中的個人識別資訊 (PII)。
- 字詞清單識別碼。

如需詳細資訊，請參閱 [API 參考](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f) \(英文\)。

**ScreenText** 會傳回 **Screen** 物件，其中具有 **Terms** 屬性，可列出 Content Moderator 在審查中偵測到的所有字詞。 請注意，如果 Content Moderator 在審查期間未偵測到任何字詞，**Terms** 屬性的值為 **null**。

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Screen the indicated text for terms in the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to use to screen the text.</param>
/// <param name="text">The text to screen.</param>
static void ScreenText (ContentModeratorClient client, string list_id, string text)
{
    Console.WriteLine("Screening text: \"{0}\" using term list with ID {1}.", text, list_id);
    Screen screen = client.TextModeration.ScreenText(lang, "text/plain", text, false, false, list_id);
    if (null == screen.Terms)
    {
        Console.WriteLine("No terms from the term list were detected in the text.");
    }
    else
    {
        foreach (DetectedTerms term in screen.Terms)
        {
            Console.WriteLine(String.Format("Found term: \"{0}\" from list ID {1} at index {2}.", term.Term, term.ListId, term.Index));
        }
    }
    read.Sleep(throttleRate);
}
```

## <a name="delete-terms-and-lists"></a>刪除字詞和清單

刪除字詞或清單相當簡單。 您可以使用 SDK 來執行下列工作：

- 刪除字詞。 (**ContentModeratorClient.ListManagementTerm.DeleteTerm**)
- 刪除清單中的所有字詞，但不刪除清單。 (**ContentModeratorClient.ListManagementTerm.DeleteAllTerms**)
- 刪除清單及其所有內容。 (**ContentModeratorClient.ListManagementTermLists.Delete**)

### <a name="delete-a-term"></a>刪除字詞

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Delete a term from the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to delete the term.</param>
/// <param name="term">The term to delete.</param>
static void DeleteTerm (ContentModeratorClient client, string list_id, string term)
{
    Console.WriteLine("Removed term \"{0}\" from term list with ID {1}.", term, list_id);
    client.ListManagementTerm.DeleteTerm(list_id, term, lang);
    Thread.Sleep(throttleRate);
}
```

### <a name="delete-all-terms-in-a-term-list"></a>刪除字詞清單中所有字詞

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Delete all terms from the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to delete all terms.</param>
static void DeleteAllTerms (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Removing all terms from term list with ID {0}.", list_id);
    client.ListManagementTerm.DeleteAllTerms(list_id, lang);
    Thread.Sleep(throttleRate);
}
```

### <a name="delete-a-term-list"></a>刪除字詞清單

將下列方法定義新增至 TermLists 命名空間、Program 類別。

```csharp
/// <summary>
/// Delete the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to delete.</param>
static void DeleteTermList (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Deleting term list with ID {0}.", list_id);
    client.ListManagementTermLists.Delete(list_id);
    Thread.Sleep(throttleRate);
}
```

## <a name="putting-it-all-together"></a>總整理

將 **Main** 方法定義新增至 **TermLists** 命名空間、**Program** 類別。 最後，關閉 **Program** 類別和 **TermLists** 命名空間。

```csharp
static void Main(string[] args)
{
    using (var client = Clients.NewClient())
    {
        string list_id = CreateTermList(client);

        UpdateTermList(client, list_id, "name", "description");
        AddTerm(client, list_id, "term1");
        AddTerm(client, list_id, "term2");

        GetAllTerms(client, list_id);

        // Always remember to refresh the search index of your list
        RefreshSearchIndex(client, list_id);

        string text = "This text contains the terms \"term1\" and \"term2\".";
        ScreenText(client, list_id, text);

        DeleteTerm(client, list_id, "term1");

        // Always remember to refresh the search index of your list
        RefreshSearchIndex(client, list_id);

        text = "This text contains the terms \"term1\" and \"term2\".";
        ScreenText(client, list_id, text);

        DeleteAllTerms(client, list_id);
        DeleteTermList(client, list_id);

        Console.WriteLine("Press ENTER to close the application.");
        Console.ReadLine();
    }
}
```

## <a name="run-the-application-to-see-the-output"></a>執行應用程式以查看此輸出

下列幾行將是您的輸出，但是資料可能會有所差異。

```
Creating term list.
Term list created. ID: 252.
Updating information for term list with ID 252.

Adding term "term1" to term list with ID 252.
Adding term "term2" to term list with ID 252.

Getting terms in term list with ID 252.
term1
term2

Refreshing search index for term list with ID 252.

Screening text: "This text contains the terms "term1" and "term2"." using term list with ID 252.
Found term: "term1" from list ID 252 at index 32.
Found term: "term2" from list ID 252 at index 46.

Removed term "term1" from term list with ID 252.

Refreshing search index for term list with ID 252.

Screening text: "This text contains the terms "term1" and "term2"." using term list with ID 252.
Found term: "term2" from list ID 252 at index 46.

Removing all terms from term list with ID 252.
Deleting term list with ID 252.
Press ENTER to close the application.
```

## <a name="next-steps"></a>後續步驟

針對這個及其他適用於 .NET 的 Content Moderator 快速入門取得 [Content Moderator .NET SDK](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 和 [Visual Studio 解決方案](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)，並開始進行您的整合。
