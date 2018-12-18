---
title: Azure Functions 的應用程式設定參考
description: Azure Functions 應用程式設定或環境變數的參考文件。
services: functions
author: ggailey777
manager: jeconnoc
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/22/2018
ms.author: glenga
ms.openlocfilehash: 46c1cb0a0cb3104e3705e4a7d4ef0dd894a7c2d7
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42819041"
---
# <a name="app-settings-reference-for-azure-functions"></a>Azure Functions 的應用程式設定參考

函式應用程式中應用程式設定所包含的全域設定選項會影響該函式應用程式的所有函式。 當您在本機執行時，這些設定是在環境變數中。 本文列出函式應用程式中可用的應用程式設定。

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md]

在 [host.json](functions-host-json.md) 檔案和 [local.settings.json](functions-run-local.md#local-settings-file) 檔案中，還有其他全域設定選項。

## <a name="appinsightsinstrumentationkey"></a>APPINSIGHTS_INSTRUMENTATIONKEY

使用 Application Insights 時的 Application Insights 檢測金鑰。 請參閱[監視 Azure Functions](functions-monitoring.md)。

|Key|範例值|
|---|------------|
|APPINSIGHTS_INSTRUMENTATIONKEY|5dbdd5e9-af77-484b-9032-64f83bb83bb|

## <a name="azurewebjobsdashboard"></a>AzureWebJobsDashboard

選擇性儲存體帳戶連接字串，用於儲存記錄並將它們顯示在入口網站的 [監視器] 索引標籤中。 儲存體帳戶必須是一般用途的帳戶，支援 Blob、佇列和資料表。 請參閱[儲存體帳戶](functions-infrastructure-as-code.md#storage-account)和[儲存體帳戶需求](functions-create-function-app-portal.md#storage-account-requirements)。

|Key|範例值|
|---|------------|
|AzureWebJobsDashboard|DefaultEndpointsProtocol=https;AccountName=[name];AccountKey=[key]|

## <a name="azurewebjobsdisablehomepage"></a>AzureWebJobsDisableHomepage

`true` 表示停用針對函式應用程式根 URL 所顯示的預設登陸頁面。 預設值為 `false`。

|Key|範例值|
|---|------------|
|AzureWebJobsDisableHomepage|true|

省略這個應用程式設定或將其設為 `false` 時，會顯示與下列範例類似的頁面，以回應 URL `<functionappname>.azurewebsites.net`。

![函式應用程式登陸頁面](media/functions-app-settings/function-app-landing-page.png)

## <a name="azurewebjobsdotnetreleasecompilation"></a>AzureWebJobsDotNetReleaseCompilation

`true` 表示在編譯 .NET 程式碼時使用「釋放」模式；`false` 表示使用「偵錯」模式。 預設值為 `true`。

|Key|範例值|
|---|------------|
|AzureWebJobsDotNetReleaseCompilation|true|

## <a name="azurewebjobsfeatureflags"></a>AzureWebJobsFeatureFlags

要啟用的搶鮮版 (Beta) 功能清單 (以逗號分隔)。 這些旗標所啟用的搶鮮版 (Beta) 功能還不適合在生產環境內使用，但在上線之前可以針對實驗使用予以啟用。

|Key|範例值|
|---|------------|
|AzureWebJobsFeatureFlags|feature1,feature2|

## <a name="azurewebjobsscriptroot"></a>AzureWebJobsScriptRoot

*host.json* 檔案和函式資料夾所在根目錄的路徑。 在函式應用程式中，預設值是 `%HOME%\site\wwwroot`。

|Key|範例值|
|---|------------|
|AzureWebJobsScriptRoot|%HOME%\site\wwwroot|

## <a name="azurewebjobssecretstoragetype"></a>AzureWebJobsSecretStorageType

指定要用於金鑰儲存的存放庫或提供者。 目前支援的存放庫是 Blob ("Blob") 和檔案系統 ("disabled")。 預設值是檔案系統 ("disabled")。

|Key|範例值|
|---|------------|
|AzureWebJobsSecretStorageType|disabled|

## <a name="azurewebjobsstorage"></a>AzureWebJobsStorage

Azure Functions 執行階段會將此儲存體帳戶連接字串用於所有函式 (由 HTTP 觸發的函式除外)。 儲存體帳戶必須是一般用途的帳戶，支援 Blob、佇列和資料表。 請參閱[儲存體帳戶](functions-infrastructure-as-code.md#storage-account)和[儲存體帳戶需求](functions-create-function-app-portal.md#storage-account-requirements)。

|Key|範例值|
|---|------------|
|AzureWebJobsStorage|DefaultEndpointsProtocol=https;AccountName=[name];AccountKey=[key]|

## <a name="azurewebjobstypescriptpath"></a>AzureWebJobs_TypeScriptPath

用於 TypeScript 之編譯器的路徑。 可讓您覆寫預設值 (需要的話)。

|Key|範例值|
|---|------------|
|AzureWebJobs_TypeScriptPath|%HOME%\typescript|

## <a name="functionappeditmode"></a>FUNCTION\_APP\_EDIT\_MODE

有效值為 "readwrite" 和 "readonly"。

|Key|範例值|
|---|------------|
|FUNCTION\_APP\_EDIT\_MODE|readonly|

## <a name="functionsextensionversion"></a>FUNCTIONS\_EXTENSION\_VERSION

要在此函式應用程式中使用的 Azure Functions 執行階段版本。 含主要版本的波狀符號表示使用該主要版本的最新版本 (例如，"~1")。 有相同主要版本的新版本可用時，會將它們自動安裝在函式應用程式中。 若要將應用程式釘選至特定版本，請使用完整版本號碼 (例如，"1.0.12345")。 預設值為 "~1"。

|Key|範例值|
|---|------------|
|FUNCTIONS\_EXTENSION\_VERSION|~1|

## <a name="websitecontentazurefileconnectionstring"></a>WEBSITE_CONTENTAZUREFILECONNECTIONSTRING

僅限取用量方案。 函式應用程式碼和設定儲存所在之儲存體帳戶的連接字串。 請參閱[建立函式應用程式](functions-infrastructure-as-code.md#create-a-function-app)。

|Key|範例值|
|---|------------|
|WEBSITE_CONTENTAZUREFILECONNECTIONSTRING|DefaultEndpointsProtocol=https;AccountName=[name];AccountKey=[key]|

## <a name="websitecontentshare"></a>WEBSITE\_CONTENTSHARE

僅限取用量方案。 函式應用程式碼和設定的檔案路徑。 Used with WEBSITE_CONTENTAZUREFILECONNECTIONSTRING. 預設值是開頭為函式應用程式名稱的唯一字串。 請參閱[建立函式應用程式](functions-infrastructure-as-code.md#create-a-function-app)。

|Key|範例值|
|---|------------|
|WEBSITE_CONTENTSHARE|functionapp091999e2|

## <a name="websitemaxdynamicapplicationscaleout"></a>WEBSITE\_MAX\_DYNAMIC\_APPLICATION\_SCALE\_OUT

函式應用程式可相應放大的執行個體數目上限。 預設值是無限制。

> [!NOTE]
> 此設定是針對預覽功能。

|Key|範例值|
|---|------------|
|WEBSITE\_MAX\_DYNAMIC\_APPLICATION\_SCALE\_OUT|10|

## <a name="websitenodedefaultversion"></a>WEBSITE\_NODE\_DEFAULT_VERSION

預設值為 "6.5.0"。

|Key|範例值|
|---|------------|
|WEBSITE\_NODE\_DEFAULT_VERSION|6.5.0|

## <a name="websiterunfromzip"></a>WEBSITE\_RUN\_FROM\_ZIP

可讓函式應用程式從掛接的套件檔案執行。

> [!NOTE]
> 此設定是針對預覽功能。

|Key|範例值|
|---|------------|
|WEBSITE\_RUN\_FROM\_ZIP|1|

有效值為 URL (可解析為部署套件檔案的位置) 或 `1`。 設定為 `1` 時，套件必須位於 `d:\home\data\SitePackages` 資料夾。 搭配使用 ZIP 部署與這項設定時，系統會將套件自動上傳到這個位置。  如需詳細資訊，請參閱[從套件檔案執行函式](run-functions-from-deployment-package.md)。

## <a name="next-steps"></a>後續步驟

[了解如何更新應用程式設定](functions-how-to-use-azure-function-app-settings.md#manage-app-service-settings)

[請參閱 host.json 檔案中的全域設定](functions-host-json.md)

[查看 App Service 應用程式的其他應用程式設定](https://github.com/projectkudu/kudu/wiki/Configurable-settings)
