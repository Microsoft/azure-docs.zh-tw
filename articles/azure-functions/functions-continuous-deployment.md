---
title: Azure Functions 的持續部署 | Microsoft Docs
description: 使用 Azure App Service 的持續部署工具來發佈 Azure Functions。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.assetid: 361daf37-598c-4703-8d78-c77dbef91643
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/25/2016
ms.author: glenga
ms.openlocfilehash: 7529d20535eedab92d164df5a0435efeda83fca2
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44301543"
---
# <a name="continuous-deployment-for-azure-functions"></a>Azure Functions 的持續部署
Azure Functions 可讓您使用 App Service 持續整合來輕鬆部署您的函數應用程式。 Functions 可與 BitBucket、Dropbox、GitHub 和 Azure DevOps 整合。 這可讓使用這其中一項整合式服務進行函數程式碼更新的工作流程觸發以 Azure 為目的地的部署。 如果您不熟悉 Azure Functions，請從 [Azure Functions 概觀](functions-overview.md)來開始。

持續部署對於整合了多個經常參與的專案而言是一個絕佳選項。 它也可讓您維護函式程式碼的原始檔控制。 目前支援的部署來源如下：

* [Bitbucket](https://bitbucket.org/)
* [Dropbox](https://www.dropbox.com/)
* 外部存放庫 (Git 或 Mercurial)
* [Git 本機存放庫](../app-service/app-service-deploy-local-git.md)
* [GitHub](https://github.com)
* [OneDrive](https://onedrive.live.com/)
* [Azure DevOps Services](https://www.visualstudio.com/team-services/)

設定部署時，是依每一函數應用程式進行設定。 當持續部署啟用後，在入口網站中對函式程式碼的存取權會設定為「唯讀」 。

## <a name="continuous-deployment-requirements"></a>持續部署需求

您必須先設定部署來源，並將函數程式碼放入部署來源中，才能設定持續部署。 在指定的函數應用程式部署中，每個函式會存在於具名子目錄中，而其目錄名稱則為函式的名稱。  

[!INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

若要能夠從 Azure DevOps 部署，您必須先將 Azure DevOps 組織連結到 Azure 訂用帳戶。 如需詳細資訊，請參閱[設定適用於 Azure DevOps 組織的帳單](https://docs.microsoft.com/azure/devops/organizations/billing/set-up-billing-for-your-organization-vs?view=vsts#set-up-billing-via-the-azure-portal)。

## <a name="set-up-continuous-deployment"></a>設定連續部署
您可以使用此程序來為現有的函數應用程式設定持續部署。 下列步驟示範與 GitHub 存放庫的整合，但類似的步驟也適用於 Azure DevOps 或其他部署服務。

1. 在 [Azure 入口網站](https://portal.azure.com)中您的函數應用程式中，按一下 [平台功能] 和 [部署選項]。 
   
    ![設定持續部署](./media/functions-continuous-deployment/setup-deployment.png)
 
2. 然後，在 [部署] 刀鋒視窗中，按一下 [設定]。
 
    ![設定持續部署](./media/functions-continuous-deployment/setup-deployment-1.png)
   
2. 在 [部署來源] 刀鋒視窗中，按一下 [選擇來源]、填入所選部署來源的資訊，然後按一下 [確定]。
   
    ![選擇部署來源](./media/functions-continuous-deployment/choose-deployment-source.png)

設定完持續部署之後，部署來源中的所有檔案變更都會複製到函數應用程式，並觸發完整的網站部署。 當來源中的檔案更新時，便會重新部署網站。

## <a name="deployment-options"></a>部署選項

以下是一些典型的部署案例︰

- [建立預備部署](#staging)
- [將現有函式移至持續部署](#existing)

<a name="staging"></a>
### <a name="create-a-staging-deployment"></a>建立預備部署

函式應用程式尚未支援部署位置。 不過，您仍可使用持續整合來管理個別的預備部署和生產部署。

設定及使用預備部署的程序通常應該會像下面這樣︰

1. 在訂用帳戶中建立兩個函式應用程式，一個用於生產程式碼，一個用於預備。 

2. 建立部署來源 (如果還未擁有)。 此範例使用 [GitHub]。

3. 針對生產函數應用程式，完成上述在**設定持續部署**中的步驟，並將部署分支設定為 GitHub 存放庫的主要分支。
   
    ![選擇部署分支](./media/functions-continuous-deployment/choose-deployment-branch.png)

4. 針對預備函數應用程式重複此步驟，但改為在 GitHub 存放庫中選擇預備分支。 如果部署來源不支援分支功能，請使用不同的資料夾。
    
5. 對預備分支或資料夾中的程式碼進行更新，然後確認預備部署中已反映這些變更。

6. 經過測試之後，將預備分支的變更合併到主要分支。 這項合併會觸發以函數應用程式為目的地的部署。 如果部署來源不支援分支，請以預備資料夾中的檔案覆寫生產資料夾中的檔案。

<a name="existing"></a>
### <a name="move-existing-functions-to-continuous-deployment"></a>將現有函式移至持續部署
當您擁有在入口網站中建立並維護的現有函式時，您必須先使用 FTP 或本機 Git 存放庫下載現有的函式程式碼檔案，才能如上所述設定持續部署。 您可以在函式應用程式的 App Service 設定中進行此操作。 下載了檔案之後，可以將其上傳至所選的持續部署來源。

> [!NOTE]
> 在設定持續整合之後，您就再也無法於 Functions 入口網站編輯原始程式檔。

- [作法：設定部署認證](#credentials)
- [作法︰使用 FTP 下載檔案](#downftp)
- [操作說明︰使用本機 Git 存放庫來下載檔案](#downgit)

<a name="credentials"></a>
#### <a name="how-to-configure-deployment-credentials"></a>作法：設定部署認證
在使用 FTP 或 本機 Git 存放庫從函數應用程式下載檔案之前，您必須先設定認證以存取網站。 認證是在函式應用程式層級進行設定。 請使用下列步驟，在 Azure 入口網站中設定部署認證：

1. 在 [Azure 入口網站](https://portal.azure.com)中您的函數應用程式中，按一下 [平台功能] 和 [部署認證]。
   
    ![設定本機部署認證](./media/functions-continuous-deployment/setup-deployment-credentials.png)

2. 輸入使用者名稱和密碼，然後按一下 [儲存] 。 您現在可以使用這些認證從 FTP 或內建的 Git 儲存機制存取函式應用程式。

<a name="downftp"></a>
#### <a name="how-to-download-files-using-ftp"></a>作法︰使用 FTP 下載檔案

1. 在 [Azure 入口網站](https://portal.azure.com)中您的函數應用程式中，按一下 [平台功能] 和 [屬性]，然後複製 [FTP/部署使用者]、[FTP 主機名稱] 及 [FTPS 主機名稱] 的值。  

    [FTP/部署使用者] 中必須輸入入口網站中所顯示的值 (包括應用程式名稱)，以針對 FTP 伺服器提供適當的內容。
   
    ![取得部署資訊](./media/functions-continuous-deployment/get-deployment-credentials.png)

2. 從您的 FTP 用戶端，使用您所蒐集的連接資訊連接到您的應用程式，並下載函式的原始程式檔。

<a name="downgit"></a>
#### <a name="how-to-download-files-using-a-local-git-repository"></a>操作說明：使用本機 Git 存放庫來下載檔案

1. 在 [Azure 入口網站](https://portal.azure.com)中您的函數應用程式中，按一下 [平台功能] 和 [部署選項]。 
   
    ![設定持續部署](./media/functions-continuous-deployment/setup-deployment.png)
 
2. 然後，在 [部署] 刀鋒視窗中，按一下 [設定]。
 
    ![設定持續部署](./media/functions-continuous-deployment/setup-deployment-1.png)
   
2. 在 [部署來源] 刀鋒視窗中，按一下 [本機 Git 存放庫]，然後按一下 [確定]。

3. 在 [平台功能] 中，按一下 [屬性]，然後記下 Git URL 的值。 
   
    ![設定持續部署](./media/functions-continuous-deployment/get-local-git-deployment-url.png)

4. 使用 Git 感知命令列提示字元或您慣用的 Git 工具，來複製本機電腦上的存放庫。 Git 複製命令看起來如下︰
   
        git clone https://username@my-function-app.scm.azurewebsites.net:443/my-function-app.git

5. 將函式應用程式的檔案擷取到本機電腦上的複製，如下列範例所示︰
   
        git pull origin master
   
    如果系統要求，請提供[已設定的部署認證](#credentials)。  

[GitHub]: https://github.com/

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Azure Functions 的最佳作法](functions-best-practices.md)
