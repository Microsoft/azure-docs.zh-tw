---
title: 使用 Power BI 工作區集合驗證和授權 | Microsoft Docs
description: 使用 Power BI 工作區集合驗證和授權。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: 1c1369ea-7dfd-4b6e-978b-8f78908fd6f6
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: 54eefc9c488718dae25e35ef0c5f0f5c50dea16c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041736"
---
# <a name="authenticating-and-authorizing-with-power-bi-workspace-collections"></a>使用 Power BI 工作區集合驗證和授權

Power BI 工作區集合服務會使用**金鑰**和**應用程式權杖**進行驗證和授權，而不使用明確的使用者驗證。 在此模型中，您的應用程式會管理使用者的驗證與授權。 然後，您的應用程式可以在必要時建立及傳送應用程式權杖，以告知我們的服務轉譯要求的報告。 雖然您仍然可以使用 Azure Active Directory 來進行使用者驗證與授權，但這項設計不需要您的應用程式這樣做。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

## <a name="two-ways-to-authenticate"></a>兩種驗證方式

**金鑰** - 您可以針對所有 Power BI 工作區集合 REST API 呼叫使用金鑰。 金鑰可以在 **Microsoft Azure 入口網站** 中找到，方法是選取 [所有設定]，然後按一下 [存取金鑰]。 一律將您的金鑰視為密碼。 這些金鑰具有在特定工作區集合上呼叫任何 REST API 的權限。

若要在 REST 呼叫上使用金鑰，請新增下列授權標頭︰

    Authorization: AppKey {your key}

**應用程式權杖** - 應用程式權杖用於所有內嵌的要求。 它們設計為在用戶端執行。權杖限制為單一報告，且最佳做法是設定到期時間。

應用程式權杖是 JWT (JSON Web 權杖)，由您的其中一個金鑰簽署。

您的應用程式權杖可包含下列宣告：

| 宣告 | 說明 |    
| --- | --- |
| **ver** |應用程式權杖的版本。 目前版本為 0.2.0。 |
| **aud** |權杖的預定接收者。 對於 Power BI 工作區集合，使用：https:\//analysis.windows.net/powerbi/api。 |
| **iss** |字串，表示已發出權杖的應用程式。 |
| **type** |正在建立的應用程式權杖類型。 目前唯一支援的類型為 **內嵌**。 |
| **wcn** |為其發出權杖的工作區集合名稱。 |
| **wid** |為其發出權杖的工作區識別碼。 |
| **rid** |為其發出權杖的報告識別碼。 |
| **username** (選擇性) |與 RLS 搭配使用，使用者名稱是字串，可以在套用 RLS 規則時協助識別使用者。 |
| **角色** (選擇性) |字串，包含套用資料列層級安全性規則時要選取的角色。 如果傳遞多個角色，應該將它們傳遞為字串陣列。 |
| **scp** (選擇性) |字串，包含權限範圍。 如果傳遞多個角色，應該將它們傳遞為字串陣列。 |
| **exp** (選擇性) |指出權杖到期的時間。 值應該傳遞為 Unix 時間戳記。 |
| **nbf** (選擇性) |指出權杖開始生效的時間。 值應該傳遞為 Unix 時間戳記。 |

範例應用程式權杖看起來如下：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ2ZXIiOiIwLjIuMCIsInR5cGUiOiJlbWJlZCIsIndjbiI6Ikd1eUluQUN1YmUiLCJ3aWQiOiJkNGZlMWViMS0yNzEwLTRhNDctODQ3Yy0xNzZhOTU0NWRhZDgiLCJyaWQiOiIyNWMwZDQwYi1kZTY1LTQxZDItOTMyYy0wZjE2ODc2ZTNiOWQiLCJzY3AiOiJSZXBvcnQuUmVhZCIsImlzcyI6IlBvd2VyQklTREsiLCJhdWQiOiJodHRwczovL2FuYWx5c2lzLndpbmRvd3MubmV0L3Bvd2VyYmkvYXBpIiwiZXhwIjoxNDg4NTAyNDM2LCJuYmYiOjE0ODg0OTg4MzZ9.v1znUaXMrD1AdMz6YjywhJQGY7MWjdCR3SmUSwWwIiI
```

解碼後，它看起來如下：

```
Header

{
    typ: "JWT",
    alg: "HS256:
}

Body

{
  "ver": "0.2.0",
  "wcn": "SupportDemo",
  "wid": "ca675b19-6c3c-4003-8808-1c7ddc6bd809",
  "rid": "96241f0f-abae-4ea9-a065-93b428eddb17",
  "iss": "PowerBISDK",
  "aud": "https://analysis.windows.net/powerbi/api",
  "exp": 1360047056,
  "nbf": 1360043456
}

```

SDK 中有方法可簡化應用程式權杖的建立。 例如，對於 .NET，您可以看看 [Microsoft.PowerBI.Security.PowerBIToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken) 類別和 [CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_) 方法。

對於 .NET SDK，您可以參考[範圍](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.scopes)。

## <a name="scopes"></a>範圍

在使用內嵌權杖時，您可能會想限制您授與了存取權之資源的使用量。 為此，您可以產生具有範圍權限的權杖。

以下是 Power BI 工作區集合的可用範圍。

|影響範圍|說明|
|---|---|
|Dataset.Read|提供讀取指定資料集的權限。|
|Dataset.Write|提供寫入指定資料集的權限。|
|Dataset.ReadWrite|提供讀取和寫入指定資料集的權限。|
|Report.Read|提供檢視指定報告的權限。|
|Report.ReadWrite|提供檢視和編輯指定報告的權限。|
|Workspace.Report.Create|提供在指定工作區內建立新報告的權限。|
|Workspace.Report.Copy|提供在指定工作區內複製現有報告的權限。|

您可以如下所示，在範圍之間使用空格來提供多個範圍。

```
string scopes = "Dataset.Read Workspace.Report.Create";
```

**必要宣告 - 範圍**

scp: {scopesClaim} scopesClaim 可以是字串或字串陣列，會記錄工作區資源 (報告、資料集等) 的允許權限

已定義範圍的解碼權杖看起來如下：

```
Header

{
    typ: "JWT",
    alg: "HS256:
}

Body

{
  "ver": "0.2.0",
  "wcn": "SupportDemo",
  "wid": "ca675b19-6c3c-4003-8808-1c7ddc6bd809",
  "rid": "96241f0f-abae-4ea9-a065-93b428eddb17",
  "scp": "Report.Read",
  "iss": "PowerBISDK",
  "aud": "https://analysis.windows.net/powerbi/api",
  "exp": 1360047056,
  "nbf": 1360043456
}

```

### <a name="operations-and-scopes"></a>作業和範圍

|作業|目標資源|權杖權限|
|---|---|---|
|根據資料集建立新的報告 (在記憶體內部)。|Dataset|Dataset.Read|
|根據資料集建立新的報告並儲存報告 (在記憶體內部)。|Dataset|* Dataset.Read<br>* Workspace.Report.Create|
|檢視和瀏覽/編輯現有報告 (在記憶體內部)。 Report.Read 暗指 Dataset.Read。 Report.Read 不允許儲存編輯。|報告|Report.Read|
|編輯並儲存現有報告。|報告|Report.ReadWrite|
|儲存報告複本 (另存新檔)。|報告|* Report.Read<br>* Workspace.Report.Copy|

## <a name="heres-how-the-flow-works"></a>以下是流程的運作方式
1. 將 API 金鑰複製到您的應用程式。 您可以在 **Azure 入口網站**取得金鑰。
   
    ![可以在 Azure 入口網站的哪裡找到 API 金鑰](media/get-started-sample/azure-portal.png)
1. 權杖判斷提示宣告，並有到期時間。
   
    ![應用程式權杖流程 - 權杖判斷提示宣告](media/get-started-sample/token-2.png)
1. 以 API 存取金鑰簽署權杖。
   
    ![應用程式權杖流程 - 簽署權杖](media/get-started-sample/token-3.png)
1. 使用者要求檢視報告。
   
    ![應用程式權杖流程 - 使用者要求檢視報告](media/get-started-sample/token-4.png)
1. 以 API 存取金鑰驗證權杖。
   
   ![應用程式權杖流程 - 驗證權杖](media/get-started-sample/token-5.png)
1. Power BI 工作區集合將報告傳送給使用者。
   
   ![應用程式權杖流程 - 服務將報告傳送給使用者](media/get-started-sample/token-6.png)

當 **Power BI 工作區集合**將報告傳送給使用者之後，使用者就可以在您自訂的應用程式中檢視報告。 例如，如果您匯入了 [分析銷售資料 PBIX 範例](http://download.microsoft.com/download/1/4/E/14EDED28-6C58-4055-A65C-23B4DA81C4DE/Analyzing_Sales_Data.pbix)，範例 Web 應用程式看起來就會像這樣︰

![應用程式內嵌的報告範例](media/get-started-sample/sample-web-app.png)

## <a name="see-also"></a>另請參閱

[CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_)  
[開始使用 Microsoft Power BI 工作區集合範例](get-started-sample.md)  
[常見的 Microsoft Power BI 工作區集合案例](scenarios.md)  
[開始使用 Microsoft Power BI 工作區集合](get-started.md)  
[PowerBI-CSharp Git 存放庫](https://github.com/Microsoft/PowerBI-CSharp)

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)