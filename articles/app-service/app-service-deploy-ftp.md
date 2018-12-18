---
title: 使用 FTP/S 將您的應用程式部署至 Azure App Service | Microsoft Docs
description: 了解如何使用 FTP 或 FTPS 將您的應用程式部署至 Azure App Service。
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: ''
ms.assetid: ae78b410-1bc0-4d72-8fc4-ac69801247ae
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/05/2018
ms.author: cephalin;dariac
ms.openlocfilehash: 66d375022d200cc916c77c059fa64eb6dbbc17e2
ms.sourcegitcommit: cfff72e240193b5a802532de12651162c31778b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39308141"
---
# <a name="deploy-your-app-to-azure-app-service-using-ftps"></a>使用 FTP/S 將您的應用程式部署至 Azure App Service

這篇文章說明如何使用 FTP 或 FTPS 將您的 Web 應用程式、行動裝置應用程式後端或 API 應用程式部署到 [Azure App Service (英文)](http://go.microsoft.com/fwlink/?LinkId=529714)。

您應用程式的 FTP/S 端點已經啟動。 啟用 FTP/S 部署不需要任何組態。

## <a name="open-ftp-dashboard"></a>開啟 FTP 儀表板

在 [Azure 入口網站](https://portal.azure.com)中，開啟應用程式的[資源頁面](../azure-resource-manager/resource-group-portal.md#manage-resources)。

若要開啟 FTP 儀表板，請按一下 持續傳遞 (預覽) > FTP > 儀表板。

![開啟 FTP 儀表板](./media/app-service-deploy-ftp/open-dashboard.png)

## <a name="get-ftp-connection-information"></a>取得 FTP 連線資訊

在 FTP 儀表板中，按一下 [複製] 複製 FTPS 端點和應用程式認證。

![複製 FTP 資訊](./media/app-service-deploy-ftp/ftp-dashboard.png)

建議您使用**應用程式認證**來部署到您的應用程式，因為它對於每個應用程式都是唯一的。 不過，如果您按一下 [使用者認證]，則可以設定使用者層級的認證，以將其用於 FTP/S 登入到訂用帳戶中的所有 App Service 應用程式。

## <a name="deploy-files-to-azure"></a>將檔案部署至 Azure

1. 從您的 FTP 用戶端 (例如 [Visual Studio](https://www.visualstudio.com/vs/community/) 或 [FileZilla](https://filezilla-project.org/download.php?type=client))，使用您所蒐集的連線資訊來連線到您的應用程式。
3. 將您的檔案和其個別的目錄結構複製到 Azure 中的 [**/site/wwwroot** 目錄](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure) (或 WebJobs 的 **/site/wwwroot/App_Data/Jobs/** 目錄)。
4. 瀏覽至您的應用程式 URL，以確認應用程式運作正常。 

> [!NOTE] 
> 與 [Git 型部署](app-service-deploy-local-git.md)不同，FTP 部署不支援下列的部署自動化︰ 
>
> - 相依性還原 (例如 NuGet、NPM、PIP 和編輯器自動化)
> - .NET 二進位檔的編譯
> - web.config 的產生 (此處是 [Node.js 範例](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps))
> 
> 請在本機電腦上手動產生這些必要檔案，然後與您的應用程式一起部署這些檔案。
>
>

## <a name="enforce-ftps"></a>強制使用 FTPS

為了增強安全性，您應該只允許 FTP over SSL。 如果您不使用 FTP 部署，您也可以停用 FTP 和 FTPS。

在 [Azure 入口網站](https://portal.azure.com)中您應用程式的資源頁面中，選取左導覽列中的 [應用程式設定]。

若要停用未加密的 FTP，請選取 [僅限 FTPS]。 若要完全停用 FTP 和 FTPS，請選取 [停用]。 完成時，按一下 [儲存]。 如果使用 [僅限 FTPS]，您必須瀏覽至 Web 應用程式的 **SSL 設定** 刀鋒視窗，來強制使用 TLS 1.1 或更高版本。 TLS 1.0 不支援 [僅限 FTPS] 功能。

![停用 FTP/S](./media/app-service-deploy-ftp/disable-ftp.png)

## <a name="automate-with-scripts"></a>使用指令碼進行自動化

如果是使用 [Azure CLI](/cli/azure) 進行 FTP 部署，請參閱[建立 Web 應用程式並使用 FTP 部署檔案 (Azure CLI)](./scripts/app-service-cli-deploy-ftp.md)。

如果是使用 [Azure PowerShell](/cli/azure) 進行 FTP 部署，請參閱[使用 FTP 將檔案上傳至 Web 應用程式 (PowerShell)](./scripts/app-service-powershell-deploy-ftp.md)。

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="troubleshoot-ftp-deployment"></a>疑難排解 FTP 部署

- [如何疑難排解 FTP 部署？](#how-can-i-troubleshoot-ftp-deployment)
- [我無法連線到 FTP 及發佈我的程式碼。如何解決此問題？](#im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue)
- [如何透過被動模式連線到 Azure App Service 中的 FTP？](#how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode)

### <a name="how-can-i-troubleshoot-ftp-deployment"></a>如何疑難排解 FTP 部署？

疑難排解 FTP 部署的第一個步驟是隔離部署問題與執行階段應用程式問題。

部署問題通常會導致沒有任何檔案或錯誤檔案部署到您的應用程式。 您可以調查 FTP 部署或選取替代部署路徑 (例如原始檔控制)，即可進行疑難排解。

執行階段應用程式問題通常會導致正確的檔案集部署到您的應用程式，但是應用程式行為不正確。 您可以將焦點放在執行階段程式碼行為，並且調查特定失敗路徑，即可進行疑難排解。

若要判斷部署或執行階段問題，請參閱[部署與執行階段問題](https://github.com/projectkudu/kudu/wiki/Deployment-vs-runtime-issues)。

### <a name="im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue"></a>我無法連線到 FTP 及發佈我的程式碼。 如何解決此問題？
請檢查您已輸入正確的主機名稱和[認證](#step-1--set-deployment-credentials)。 此外，檢查防火牆並未封鎖您電腦上的下列 FTP 連接埠：

- FTP 控制連線連接埠︰21
- FTP 資料連線連接埠︰989、10001-10300
 
### <a name="how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode"></a>如何透過被動模式連線到 Azure App Service 中的 FTP？
Azure App Service 支援透過主動與被動模式進行連線。 建議使用被動模式，因為部署電腦通常位於防火牆背後 (或在作業系統中或是家用或公司網路的一部分)。 請參閱 [WinSCP 文件中的範例](https://winscp.net/docs/ui_login_connection)。 

## <a name="next-steps"></a>後續步驟

如需更多的進階部署案例，請嘗試[使用 Git 部署至 Azure ](app-service-deploy-local-git.md)。 Git 型部署至 Azure 可啟用版本控制、封裝還原、MSBuild 等等。

## <a name="more-resources"></a>其他資源

* [Azure App Service 部署認證](app-service-deploy-ftp.md)
