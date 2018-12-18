---
title: 為 Power BI 工作區集合中的報告切換檢視和編輯模式 | Microsoft Docs
description: 了解如何為 Power BI 工作區集合內的報告切換檢視和編輯模式。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: 19a576440742684849ffc74092162be7008621ba
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045013"
---
# <a name="toggle-between-view-and-edit-mode-for-reports-in-power-bi-workspace-collections"></a>為 Power BI 工作區集合中的報告切換檢視和編輯模式

了解如何為 Power BI 工作區集合內的報告切換檢視和編輯模式。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

## <a name="creating-an-access-token"></a>建立存取權杖

您必須建立可讓您檢視和編輯報告的存取權杖。 若要編輯和儲存報告，您需要 **Report.ReadWrite** 權杖權限。 如需詳細資訊，請參閱[在 Power BI 工作區集合中驗證和授權](app-token-flow.md)。

> [!NOTE]
> 這可讓您對現有報告進行編輯並儲存變更。 如果您也需要可支援**另存新檔**的功能，就需要提供額外的權限。 如需詳細資訊，請參閱[範圍](app-token-flow.md#scopes)。

```
using Microsoft.PowerBI.Security;

// rlsUsername and roles are optional
string scopes = "Report.ReadWrite";
PowerBIToken embedToken = PowerBIToken.CreateReportEmbedTokenForCreation(workspaceCollectionName, workspaceId, datasetId, null, null, scopes);

var token = embedToken.Generate("{access key}");
```

## <a name="embed-configuration"></a>內嵌組態

您必須提供權限和 viewMode，才能在編輯模式中看到 [儲存] 按鈕。 如需詳細資訊，請參閱[內嵌組態詳細資料](https://github.com/Microsoft/PowerBI-JavaScript/wiki/Embed-Configuration-Details)。

例如，在 JavaScript 中：

```
   <div id="reportContainer"></div>

    // Get models. Models, it contains enums that can be used.
    var models = window['powerbi-client'].models;

    // Embed configuration used to describe the what and how to embed.
    // This object is used when calling powerbi.embed.
    // This also includes settings and options such as filters.
    // You can find more information at https://github.com/Microsoft/PowerBI-JavaScript/wiki/Embed-Configuration-Details.
    var config= {
        type: 'report',
        accessToken: 'eyJ0eXAiO...Qron7qYpY9MI',
        embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed',
        id:  '5dac7a4a-4452-46b3-99f6-a25915e0fe55',
        permissions: models.Permissions.ReadWrite /*both save & save as buttons will be visible*/,
        viewMode: models.ViewMode.View,
        settings: {
            filterPaneEnabled: true,
            navContentPaneEnabled: true
        }
    };

    // Get a reference to the embedded report HTML element
    var reportContainer = $('#reportContainer')[0];

    // Embed the report and display it within the div container.
    var report = powerbi.embed(reportContainer, config);
```

這表示以 **viewMode** 作為基礎內嵌檢視模式報告已設為 **models.ViewMode.View**。

## <a name="view-mode"></a>檢視模式

如果您處於編輯模式，您可以使用下列 JavaScript 來切換為檢視模式。

```
// Get a reference to the embedded report HTML element
var reportContainer = $('#reportContainer')[0];

// Get a reference to the embedded report.
report = powerbi.get(reportContainer);

// Switch to view mode.
report.switchMode("view");

```

## <a name="edit-mode"></a>編輯模式

如果您處於檢視模式，您可以使用下列 JavaScript 來切換為編輯模式。

```
// Get a reference to the embedded report HTML element
var reportContainer = $('#reportContainer')[0];

// Get a reference to the embedded report.
report = powerbi.get(reportContainer);

// Switch to edit mode.
report.switchMode("edit");

```

## <a name="see-also"></a>另請參閱

[開始使用範例](get-started-sample.md)  
[內嵌報告](embed-report.md)  
[在 Power BI 工作區集合中驗證和授權](app-token-flow.md)  
[CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_)  
[JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[PowerBI-CSharp Git 存放庫](https://github.com/Microsoft/PowerBI-CSharp)  
[PowerBI-Node Git存放庫](https://github.com/Microsoft/PowerBI-Node)  

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)
