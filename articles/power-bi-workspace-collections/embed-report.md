---
title: 在 Azure Power BI 工作區集合中內嵌報表 | Microsoft Docs
description: 了解如何將 Power BI 工作區集合中的報表內嵌到您的應用程式。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: ''
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: 94476486ed87662f3d6b989b8d5360dd792f8824
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041175"
---
# <a name="embed-a-report-in-power-bi-workspace-collections"></a>在 Power BI 工作區集合中內嵌報表

了解如何將 Power BI 工作區集合中的報表內嵌到您的應用程式。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

我們將探討如何將報告實際內嵌到您的應用程式。 本文假設您在工作區集合的某個工作區內已存有報告。 如果您尚未完成該步驟，請參閱[開始使用 Power BI 工作區集合](get-started.md)。

您可以使用 .NET (C#) 或 Node.js SDK 以及 JavaScript，搭配 Power BI 工作區集合輕鬆建置應用程式。

## <a name="using-the-access-keys-to-use-rest-apis"></a>使用存取金鑰來使用 REST API

若要呼叫 REST API，您可以傳遞給定工作區集合的存取金鑰 (可從 Azure 入口網站取得)。 如需詳細資訊，請參閱[開始使用 Power BI 工作區集合](get-started.md)。

## <a name="get-a-report-id"></a>取得報表識別碼

每個存取權杖都會以報告為基礎。 您必須針對想要內嵌的報告取得給定的報告識別碼。 這可以根據[取得報告](https://msdn.microsoft.com/library/azure/mt711510.aspx) REST API 的呼叫來完成。 這會傳回報告識別碼和內嵌 URL。 這可以使用 Power BI .NET SDK 或直接呼叫 REST API 來完成。

### <a name="using-the-power-bi-net-sdk"></a>使用 Power BI .NET SDK

使用 .NET SDK 時，您必須建立以取自 Azure 入口網站的存取金鑰為基礎的權杖認證。 這需要您安裝 [Power BI API NuGet 套件](https://www.nuget.org/profiles/powerbi)。

**NuGet 套件安裝**

```
Install-Package Microsoft.PowerBI.Api
```

**C# 程式碼**

```
using Microsoft.PowerBI.Api.V1;
using Microsoft.Rest;

var credentials = new TokenCredentials("{access key}", "AppKey");
var client = new PowerBIClient(credentials);
client.BaseUri = new Uri(https://api.powerbi.com);

var reports = (IList<Report>)client.Reports.GetReports(workspaceCollectionName, workspaceId).Value;

// Select the report that you want to work with from the collection of reports.
```

### <a name="calling-the-rest-api-directly"></a>直接呼叫 REST API

```
System.Net.WebRequest request = System.Net.WebRequest.Create("https://api.powerbi.com/v1.0/collections/{collectionName}/workspaces/{workspaceId}/Reports") as System.Net.HttpWebRequest;

request.Method = "GET";
request.ContentLength = 0;
request.Headers.Add("Authorization", String.Format("AppKey {0}", accessToken.Value));

using (var response = request.GetResponse() as System.Net.HttpWebResponse)
{
    using (var reader = new System.IO.StreamReader(response.GetResponseStream()))
    {
        // Work with JSON response to get the report you want to work with.
    }

}
```

## <a name="create-an-access-token"></a>建立存取權杖

Power BI 工作區集合會使用內嵌權杖，其為 HMAC 簽署的 JSON Web 權杖。 權杖會使用來自 Power BI 工作區集合的存取金鑰進行簽署。 內嵌權杖依預設可用來提供要內嵌至應用程式之報告的唯讀存取權。 內嵌權杖會針對特定報告來核發，而且應該與內嵌 URL 相關聯。

存取權杖應該建立在伺服器上，因為會使用存取金鑰來簽署/加密權杖。 如需如何建立存取權杖的相關資訊，請參閱[使用 Power BI 工作區集合驗證和授權](app-token-flow.md)。 您也可以檢閱 [CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_) 方法。 以下是使用 .NET SDK for Power BI 時此方法所呈現樣貌的範例。

您會使用先前擷取到的報表識別碼。 建立內嵌權杖後，您接著將使用存取金鑰來產生可從 JavaScript 觀點使用的權杖。 要使用 PowerBIToken 類別，您必須安裝 [Power BI 核心 NuGut 套件](https://www.nuget.org/packages/Microsoft.PowerBI.Core/)。

**NuGet 套件安裝**

```
Install-Package Microsoft.PowerBI.Core
```

**C# 程式碼**

```
using Microsoft.PowerBI.Security;

// rlsUsername, roles and scopes are optional.
embedToken = PowerBIToken.CreateReportEmbedToken(workspaceCollectionName, workspaceId, reportId, rlsUsername, roles, scopes);

var token = embedToken.Generate("{access key}");
```

### <a name="adding-permission-scopes-to-embed-tokens"></a>在內嵌權杖新增權限範圍

在使用內嵌權杖時，您可能會想限制您授與了存取權之資源的使用量。 為此，您可以產生具有範圍權限的權杖。 如需詳細資訊，請參閱[範圍](app-token-flow.md#scopes)

## <a name="embed-using-javascript"></a>使用 JavaScript 進行內嵌

當您擁有存取權杖和報表識別碼之後，我們就能使用 JavaScript 來內嵌報表。 這需要您安裝 NuGet [Power BI JavaScript 封裝](https://www.nuget.org/packages/Microsoft.PowerBI.JavaScript/) \(英文\)。 embedUrl 將只是 https://embedded.powerbi.com/appTokenReportEmbed。

> [!NOTE]
> 您可以使用 [JavaScript 報告內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)來測試功能。 它也會提供可用之不同作業的程式碼範例。

**NuGet 套件安裝**

```
Install-Package Microsoft.PowerBI.JavaScript
```

**JavaScript 程式碼**

```
<script src="/scripts/powerbi.js"></script>
<div id="reportContainer"></div>

var embedConfiguration = {
    type: 'report',
    accessToken: 'eyJ0eXAiO...Qron7qYpY9MI',
    id: '5dac7a4a-4452-46b3-99f6-a25915e0fe55',
    embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed'
};

var $reportContainer = $('#reportContainer');
var report = powerbi.embed($reportContainer.get(0), embedConfiguration);
```

### <a name="set-the-size-of-embedded-elements"></a>設定內嵌元素大小

報告會根據其容器的大小自動內嵌。 若要覆寫內嵌項目的預設大小，只需新增適用於寬度和高度的 CSS 類別屬性或內嵌樣式即可。

## <a name="see-also"></a>另請參閱

[開始使用範例](get-started-sample.md)  
[在 Power BI 工作區集合中驗證和授權](app-token-flow.md)  
[CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_)  
[JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[Power BI JavaScript 套件](https://www.nuget.org/packages/Microsoft.PowerBI.JavaScript/)  
[Power BI API NuGet 套件](https://www.nuget.org/profiles/powerbi)
[Power BI 核心 NuGut 套件](https://www.nuget.org/packages/Microsoft.PowerBI.Core/)  
[PowerBI-CSharp Git 存放庫](https://github.com/Microsoft/PowerBI-CSharp)  
[PowerBI-Node Git存放庫](https://github.com/Microsoft/PowerBI-Node)  

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)
