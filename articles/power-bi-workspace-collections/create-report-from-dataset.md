---
title: 在 Power BI 工作區集合中從資料集建立新的報告 | Microsoft Docs
description: 您現在可以在自己的應用程式中，從資料集建立 Power BI 工作區集合報告。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: ''
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: dbfe8c98dce1879a2e0956fafba32ab6e608bfdb
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046649"
---
# <a name="create-a-new-report-from-a-dataset-in-power-bi-workspace-collections"></a>在 Power BI 工作區集合中，從資料集建立新的報告

您現在可以在自己的應用程式中，從資料集建立 Power BI 工作區集合報告。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

其驗證方法類似內嵌報告所用的方法。 會以資料集特有的存取權杖為基礎。 用於 PowerBI.com 的權杖是由 Azure Active Directory (AAD) 所發行。 Power BI 工作區集合權杖是由您自己的應用程式所發行。

在建立 Embedded 報告時，權杖會針對特定資料集加以核發。 權杖應該與相同元素上的內嵌 URL 相關聯，以確保每個項目都有獨一無二的權杖。 若要建立 Embedded 報告，則必須在存取權杖中提供 Dataset.Read 和 Workspace.Report.Create 範圍。

## <a name="create-access-token-needed-to-create-new-report"></a>建立所需的存取權杖來建立新的報告

Power BI 工作區集合會使用內嵌權杖，其為 HMAC 簽署的 JSON Web 權杖。 權杖會使用來自 Power BI 工作區集合的存取金鑰進行簽署。 內嵌權杖依預設可用來提供要內嵌至應用程式之報告的唯讀存取權。 內嵌權杖會針對特定報告來核發，而且應該與內嵌 URL 相關聯。

存取權杖應該建立在伺服器上，因為會使用存取金鑰來簽署/加密權杖。 如需如何建立存取權杖的相關資訊，請參閱[使用 Power BI 工作區集合驗證和授權](app-token-flow.md)。 您也可以檢閱 [CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_) 方法。 以下是使用 .NET SDK for Power BI 時此方法所呈現樣貌的範例。

在此範例中，我們擁有用來建立新報告所需要的資料集識別碼。 我們也需要新增 Dataset.Read 和 Workspace.Report.Create 的範圍。

要使用 PowerBIToken 類別，您必須安裝 [Power BI 核心 NuGut 套件](https://www.nuget.org/packages/Microsoft.PowerBI.Core/)。

**NuGet 套件安裝**

```
Install-Package Microsoft.PowerBI.Core
```

**C# 程式碼**

```
using Microsoft.PowerBI.Security;

// rlsUsername and roles are optional
string scopes = "Dataset.Read Workspace.Report.Create";
PowerBIToken embedToken = PowerBIToken.CreateReportEmbedTokenForCreation(workspaceCollectionName, workspaceId, datasetId, null, null, scopes);

var token = embedToken.Generate("{access key}");
```

## <a name="create-a-new-blank-report"></a>建立新的空白報告

若要建立新報告，請提供建立組態。 這應該包括存取權杖、embedURL 和我們想要用來建立報告的 datasetID。 這需要您安裝 NuGet [Power BI JavaScript 套件](https://www.nuget.org/packages/Microsoft.PowerBI.JavaScript/)。 embedUrl 將只是 https://embedded.powerbi.com/appTokenReportEmbed。

> [!NOTE]
> 您可以使用 [JavaScript 報告內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)來測試功能。 它也會提供可用之不同作業的程式碼範例。

**NuGet 套件安裝**

```
Install-Package Microsoft.PowerBI.JavaScript
```

**JavaScript 程式碼**

```
<div id="reportContainer"></div>
  
var embedCreateConfiguration = {
        accessToken: 'eyJ0eXAiO...Qron7qYpY9MI',
        embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed',
        datasetId: '5dac7a4a-4452-46b3-99f6-a25915e0fe55',
    };
    
    // Grab the reference to the div HTML element that will host the report
    var reportContainer = $('#reportContainer')[0];

    // Create report
    var report = powerbi.createReport(reportContainer, embedCreateConfiguration);
```

呼叫 powerbi.createReport() 會讓編輯模式的空白畫布出現在 div 元素內。

![新增空白報告](media/create-report-from-dataset/create-new-report.png)

## <a name="save-new-reports"></a>儲存新報告

在您呼叫**另存新檔**作業後，才會建立報告。 這可從 [檔案] 功能表或從 JavaScript 來進行。

```
 // Get a reference to the embedded report.
    report = powerbi.get(reportContainer);
    
    var saveAsParameters = {
        name: "newReport"
    };

    // SaveAs report
    report.saveAs(saveAsParameters);
```

> [!IMPORTANT]
> 在呼叫**另存新檔**後，才會建立新報告。 儲存之後，畫布仍會顯示編輯模式的資料集，而非報告。 您必須像處理任何其他報告一樣，重新載入新的報告。

![檔案功能表 - 另存新檔](media/create-report-from-dataset/save-new-report.png)

## <a name="load-the-new-report"></a>載入新報告

若要與新報告互動，您必須利用和應用程式內嵌一般報告相同的方式，將新報告內嵌，也就是說，必須特別為新報告核發新權杖，然後呼叫內嵌方法。

```
<div id="reportContainer"></div>
  
var embedConfiguration = {
        accessToken: 'eyJ0eXAiO...Qron7qYpY9MJ',
        embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed',
        reportId: '5dac7a4a-4452-46b3-99f6-a25915e0fe54',
    };
    
    // Grab the reference to the div HTML element that will host the report
    var reportContainer = $('#reportContainer')[0];

    // Embed report
    var report = powerbi.embed(reportContainer, embedConfiguration);
```

## <a name="automate-save-and-load-of-a-new-report-using-the-saved-event"></a>使用「已儲存」的事件自動儲存和載入新報告

若要自動進行「另存新檔」的程序，然後載入新報告，您可以利用「已儲存」的事件。 這個事件的引發條件為：儲存作業完成，並傳回包含新 reportId、報告名稱、舊 reportId (如果有的話) 的 Json 物件，而且如果該作業是另存新檔或儲存時。

```
{
  "reportObjectId": "5dac7a4a-4452-46b3-99f6-a25915e0fe54",
  "reportName": "newReport",
  "saveAs": true,
  "originalReportObjectId": null
}
```

若要自動進行程序，您可以接聽「已儲存」的事件、取得新 reportId、建立新權杖，然後將新報告內嵌在其中。

```
<div id="reportContainer"></div>
  
var embedCreateConfiguration = {
        accessToken: 'eyJ0eXAiO...Qron7qYpY9MI',
        embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed',
        datasetId: '5dac7a4a-4452-46b3-99f6-a25915e0fe55',
    };
    
    // Grab the reference to the div HTML element that will host the report
    var reportContainer = $('#reportContainer')[0];

    // Create report
    var report = powerbi.createReport(reportContainer, embedCreateConfiguration);


   var saveAsParameters = {
        name: "newReport"
    };

    // SaveAs report
    report.saveAs(saveAsParameters);

    // report.on will add an event handler which prints to Log window.
    report.on("saved", function(event) {
        
         // get new Token
         var newReportId =  event.detail.reportObjectId;

        // create new Token. This is a function that the application should provide
        var newToken = createAccessToken(newReportId,scopes /*provide the wanted scopes*/);
        
        
    var embedConfiguration = {
        accessToken: newToken ,
        embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed',
        reportId: newReportId,
    };

    // Embed report
    var report = powerbi.embed(reportContainer, embedConfiguration);
       
   // report.off removes a given event handler if it exists.
   report.off("saved");
    });
```

## <a name="see-also"></a>另請參閱

[開始使用範例](get-started-sample.md)  
[儲存報告](save-reports.md)  
[內嵌報告](embed-report.md)  
[在 Power BI 工作區集合中驗證和授權](app-token-flow.md)  
[Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)  
[JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[Power BI 核心 NuGut 套件](https://www.nuget.org/packages/Microsoft.PowerBI.Core/)  
[Power BI JavaScript 套件](https://www.nuget.org/packages/Microsoft.PowerBI.JavaScript/)  

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)