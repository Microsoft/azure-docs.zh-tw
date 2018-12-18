---
title: Azure Web 應用程式的部署常見問題集 | Microsoft Docs
description: 對於 Azure App Service 的 Web 應用程式功能之中的部署相關的常見問題集獲得解答。
services: app-service\web
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: ab8750e5824cf9f7635d11a6b2be332b2f9a761c
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302583"
---
# <a name="deployment-faqs-for-web-apps-in-azure"></a>Azure 中 Web 應用程式的部署常見問題集

對於 [Azure App Service 的 Web 應用程式功能](https://azure.microsoft.com/services/app-service/web/)的部署問題，本文提供常見問題集的解答。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="i-am-just-getting-started-with-app-service-web-apps-how-do-i-publish-my-code"></a>我剛開始使用 App Service Web 應用程式。 如何發佈程式碼？

以下是發佈 Web 應用程式程式碼的一些選項：

*   使用 Visual Studio 進行部署。 如果您有 Visual Studio 解決方案，以滑鼠右鍵按一下 Web 應用程式專案，然後選取 [發佈]。
*   使用 FTP 用戶端進行部署。 在 Azure 入口網站中，下載您想要部署程式碼之 Web 應用程式的發行設定檔。 然後，使用相同的發行設定檔 FTP 認證將檔案上傳至 \site\wwwroot。

如需詳細資訊，請參閱[將應用程式部署到 App Service](app-service-deploy-local-git.md)。

## <a name="i-see-an-error-message-when-i-try-to-deploy-from-visual-studio-how-do-i-resolve-this"></a>當我嘗試從 Visual Studio 部署時，我看到一則錯誤訊息。 如何解決這個問題？

如果您看到下列訊息，您可能是使用舊版的 SDK：「在資源群組 'YourResourceGroup' 中部署資源 'YourResourceName' 期間發生錯誤：MissingRegistrationForLocation：訂用帳戶未在位置「美國中部」註冊資源類型「組件」。 請重新註冊此提供者以獲得此位置的存取權」。 

若要解決這個錯誤，請升級為[最新的 SDK](https://azure.microsoft.com/downloads/)。 如果您看到此訊息，而且您有最新的 SDK，請提交支援要求。

## <a name="how-do-i-deploy-an-aspnet-application-from-visual-studio-to-app-service"></a>如何將 ASP.NET 應用程式從 Visual Studio 部署到 App Service？
<a id="deployasp"></a>

本教學課程[五分鐘內在 Azure 中建立第一個 ASP.NET Web 應用程式](app-service-web-get-started-dotnet.md)示範如何使用 Visual Studio 2017，將 ASP.NET Web 應用程式部署到 App Service 中的 Web 應用程式。

## <a name="what-are-the-different-types-of-deployment-credentials"></a>不同類型的部署認證有哪些？

App Service 支援兩種認證類型，用於本機 Git 部署和 FTP/S 部署。 如需如何設定部署認證的詳細資訊，請參閱[設定 App Service 的部署認證](app-service-deployment-credentials.md)。

## <a name="what-is-the-file-or-directory-structure-of-my-app-service-web-app"></a>我的 App Service Web 應用程式的檔案或目錄結構是什麼？

如需您的 App Service 應用程式的檔案結構詳細資訊，請參閱 [Azure 中的檔案結構](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure)。

## <a name="how-do-i-resolve-ftp-error-550---there-is-not-enough-space-on-the-disk-when-i-try-to-ftp-my-files"></a>當我嘗試 FTP 我的檔案時，如何解決「FTP 錯誤 550 - 磁碟空間不足」？

如果您看到此訊息，可能是您即將用盡 Web 應用程式之服務方案中的磁碟配額。 您可能需要根據您的磁碟空間需求，相應增加至較高服務層。 如需定價方案和資源限制的詳細資訊，請參閱 [App Service 定價](https://azure.microsoft.com/pricing/details/app-service/)。

## <a name="how-do-i-set-up-continuous-deployment-for-my-app-service-web-app"></a>如何為 App Service Web 應用程式設定持續部署？

您可以從數個資源設定持續部署，包括 Azure DevOps、OneDrive、GitHub、Bitbucket、Dropbox 和其他 Git 存放庫。 這些選項可在入口網站中使用。 [持續部署至 App Service](app-service-continuous-deployment.md) 是很有幫助的教學課程，說明如何設定持續部署。

## <a name="how-do-i-troubleshoot-issues-with-continuous-deployment-from-github-and-bitbucket"></a>如何針對從 GitHub 和 Bitbucket 持續部署的問題進行疑難排解？

如需調查從 GitHub 或 Bitbucket 持續部署之問題的協助，請參閱[調查持續部署](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)。

## <a name="i-cant-ftp-to-my-site-and-publish-my-code-how-do-i-resolve-this"></a>我無法 FTP 至我的網站及發佈我的程式碼。 如何解決這個問題？

若要解決 FTP 問題：

1. 請確認您輸入正確的主機名稱和認證。 如需不同類型認證和使用方式的詳細資訊，請參閱[部署認證](https://github.com/projectkudu/kudu/wiki/Deployment-credentials)。
2. 請確認 FTP 連接埠未遭防火牆封鎖。 連接埠應該具有以下設定：
    * FTP 控制連線連接埠︰21
    * FTP 資料連線連接埠︰989、10001-10300

## <a name="how-do-i-publish-my-code-to-app-service"></a>如何將我的程式碼發佈至 App Service？

「Azure 快速入門」的設計目的是協助您使用部署堆疊和您選擇的方法來部署您的應用程式。 若要使用快速入門，請在 Azure 入口網站中移至 [設定] >  [應用程式部署]。

## <a name="why-does-my-app-sometimes-restart-after-deployment-to-app-service"></a>為什麼我的應用程式有時候會在部署至 App Service 之後重新啟動？

若要深入了解可能會造成重新啟動之應用程式部署的情況，請參閱[部署與執行階段問題](https://github.com/projectkudu/kudu/wiki/Deployment-vs-runtime-issues#deployments-and-web-app-restarts")。 如本文章所述，App Service 會將檔案部署到 wwwroot 資料夾。 它永遠不會直接重新啟動您的應用程式。

## <a name="how-do-i-integrate-azure-devops-code-with-app-service"></a>如何整合 Azure DevOps 程式碼與 App Service？

您有兩個選項可以使用 Azure DevOps 進行持續部署：

*   使用 Git 專案。 藉由使用該存放庫的部署選項以透過 App Service 連線。
*   使用 Team Foundation 版本控制 (TFVC) 專案。 藉由使用 App Service 的組建代理程式來部署。

這兩個選項的持續程式碼部署取決於現有的開發人員工作流程和簽入程序。 如需詳細資訊，請參閱這些文章： 

*   [實作您的應用程式到 Azure 網站的持續部署](https://www.visualstudio.com/docs/release/examples/azure/azure-web-apps-from-build-and-release-hubs)
*   [設定 Azure DevOps 組織，使其可部署至 Web 應用程式](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App)

## <a name="how-do-i-use-ftp-or-ftps-to-deploy-my-app-to-app-service"></a>如何使用 FTP 或 FTPS 將我的應用程式部署到 App Service？

如需使用 FTP 或 FTPS 將 Web 應用程式部署到 App Service 的詳細資訊，請參閱[使用 FTP/S 將您的應用程式部署到 App Service](app-service-deploy-ftp.md)。
