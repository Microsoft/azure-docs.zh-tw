---
title: Azure App Service 中的 OS 和執行階段修補 | Microsoft Docs
description: 說明 Azure App Service 如何更新 OS 和執行階段，以及要如何取得更新公告。
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/02/2018
ms.author: cephalin
ms.openlocfilehash: 9855becd7c047788ed310dff4317a5df87cc9b61
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047623"
---
# <a name="os-and-runtime-patching-in-azure-app-service"></a>Azure App Service 中的 OS 和執行階段修補

本文說明如何取得有關 [App Service](app-service-web-overview.md) 中 OS 或軟體的特定版本資訊。 

App Service 是平台即服務，這表示 Azure 會為您管理 OS 和應用程式堆疊；您只需要管理您的應用程式和其資料。 您可以在 [Azure 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/)中獲得更多對於 OS 和應用程式堆疊的控制能力。 記住這一點可幫助身為 App Service 使用者的您了解更多資訊，例如：

-   OS 更新的套用方式和時機？
-   針對重大漏洞 (例如零時差) 修補 App Service 的方式？
-   哪些 OS 和執行階段版本在執行您的應用程式？

為求安全，我們不會將某些安全性資訊細節發佈出來。 不過，本文的目標是要讓程序盡量透明以減少疑慮，並說明要如何隨時得知最新的安全性相關公告或執行階段更新。

## <a name="how-and-when-are-os-updates-applied"></a>OS 更新的套用方式和時機？

Azure 會在兩個層級管理 OS 修補，分別是實體伺服器和執行 App Service 資源的客體虛擬機器 (VM)。 兩者皆會每月更新，以配合每月的 [Patch Tuesday](https://technet.microsoft.com/security/bulletins.aspx) 排程。 這些更新會以能夠保證 Azure 服務具有高可用性 SLA 的方式自動套用。 

如需更新套用方式的詳細資訊，請參閱[揭露 App Service OS 更新背後的魔法](https://blogs.msdn.microsoft.com/appserviceteam/2018/01/18/demystifying-the-magic-behind-app-service-os-updates/)。

## <a name="how-does-azure-deal-with-significant-vulnerabilities"></a>Azure 如何處理重大漏洞？

有嚴重漏洞需要立即修補時 (例如[零時差漏洞](https://wikipedia.org/wiki/Zero-day_(computing)))，我們會依個案逐一處理高優先順序的更新。

請造訪 [Azure 安全性部落格](https://azure.microsoft.com/blog/topics/security/)，以隨時了解最新的 Azure 重大安全性公告。 

## <a name="when-are-supported-language-runtimes-updated-added-or-deprecated"></a>何時會更新、新增或取代所支援的語言執行階段？

所支援的語言執行階段若有新的穩定版本 (主要、次要或修補)，將會定期新增至 App Service 執行個體。 有些更新會覆寫現有安裝，有些則會與現有版本並存安裝。 覆寫安裝的意思是，您的應用程式會自動在更新的執行階段上執行。 並存安裝的意思則是，您必須手動移轉應用程式，以便利用新的執行階段版本。 如需詳細資訊，請參閱下列其中一個小節。

執行階段更新和取代會在以下位置公告：

- https://azure.microsoft.com/updates/?product=app-service 
- https://github.com/Azure/app-service-announcements/issues

> [!NOTE] 
> 這裡的資訊適用於 App Service 應用程式內建的語言執行階段。 例如，您上傳至 App Service 的自訂執行階段會維持不變，除非您手動升級。
>
>

### <a name="new-patch-updates"></a>新的修補程式更新

.NET、PHP、Java SDK 或 Tomcat/Jetty 版本的修補程式更新，會透過以新版本覆寫現有安裝的方式自動套用。 Node.js 修補程式更新則會與現有版本並存安裝 (類似於下一節的主要和次要版本)。 新的 Python 修補程式版本可手動透過[網站延伸模組](https://www.siteextensions.net/packages?q=Tags%3A%22python%22)與內建的 Python 安裝並存安裝。

### <a name="new-major-and-minor-versions"></a>新的主要和次要版本

有新增的主要或次要版本時，便會與現有版本並存安裝。 您可以手動將應用程式升級至新的版本。 如果您是在設定檔中 (例如 `web.config` 和 `package.json`) 設定執行階段版本，則必須以相同方法升級。 如果您使用 App Service 設定來設定執行階段版本，則可以在 [Azure 入口網站](https://portal.azure.com)中或藉由在 [Cloud Shell](../cloud-shell/overview.md) 中執行 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) 命令來加以變更，如下列範例所示：

```azurecli-interactive
az webapp config set --net-framework-version v4.7 --resource-group <groupname> --name <appname>
az webapp config set --php-version 7.0 --resource-group <groupname> --name <appname>
az webapp config appsettings set --settings WEBSITE_NODE_DEFAULT_VERSION=8.9.3 --resource-group <groupname> --name <appname>
az webapp config set --python-version 3.4 --resource-group <groupname> --name <appname>
az webapp config set --java-version 1.8 --java-container Tomcat --java-container-version 9.0 --resource-group <groupname> --name <appname>
```

### <a name="deprecated-versions"></a>已取代的版本  

在要取代較舊的版本時，我們會公佈移除日期，讓您據此規劃執行階段版本升級。 

## <a name="how-can-i-query-os-and-runtime-update-status-on-my-instances"></a>如何在執行個體上查詢 OS 和執行階段更新狀態？  

您無法存取重要的 OS 資訊 (請參閱 [Azure App Service 上的作業系統功能](web-sites-available-operating-system-functionality.md))，不過 [Kudu 主控台](https://github.com/projectkudu/kudu/wiki/Kudu-console)可讓您針對 OS 版本和執行階段版本來查詢 App Service 執行個體。 

下表說明如何找到 Windows 的版本和執行應用程式之語言執行階段的版本：

| 資訊 | 所在位置 | 
|-|-|
| Windows 版本 | 請參閱 `https://<appname>.scm.azurewebsites.net/Env.cshtml` (在 [系統資訊] 底下) |
| .NET 版本 | 位於 `https://<appname>.scm.azurewebsites.net/DebugConsole`，在命令提示字元中執行下列命令： <br>`powershell -command "gci 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Net Framework Setup\NDP\CDF'"` |
| .NET Core 版本 | 位於 `https://<appname>.scm.azurewebsites.net/DebugConsole`，在命令提示字元中執行下列命令： <br> `dotnet --version` |
| PHP 版本 | 位於 `https://<appname>.scm.azurewebsites.net/DebugConsole`，在命令提示字元中執行下列命令： <br> `php --version` |
| 預設的 Node.js 版本 | 在 [Cloud Shell](../cloud-shell/overview.md) 中執行下列命令︰ <br> `az webapp config appsettings list --resource-group <groupname> --name <appname> --query "[?name=='WEBSITE_NODE_DEFAULT_VERSION']"` |
| Python 版本 | 位於 `https://<appname>.scm.azurewebsites.net/DebugConsole`，在命令提示字元中執行下列命令： <br> `python --version` |  

> [!NOTE]  
> 您無法存取登錄位置 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages` (這裡儲存了有關 ["KB" 修補程式](https://docs.microsoft.com/security-updates/SecurityBulletins/securitybulletins)的資訊)。
>
>

## <a name="more-resources"></a>其他資源

[信任中心：安全性](https://www.microsoft.com/en-us/trustcenter/security)  
[Azure App Service 上的 64 位元 ASP.NET Core](https://gist.github.com/glennc/e705cd85c9680d6a8f1bdb62099c7ac7)
