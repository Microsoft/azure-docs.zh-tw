---
title: 為 Azure Stack 上的應用程式服務設定部署來源 | Microsoft Docs
description: 服務管理員該如何為 Azure Stack 上的 App Service 設定部署來源 (Git、GitHub、BitBucket、DropBox 和 OneDrive)
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/08/2018
ms.author: sethm
ms.reviewer: anwestg
ms.openlocfilehash: fdb91f8989bced3d148c858f131e7d78f1d9f51c
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077132"
---
# <a name="configure-deployment-sources"></a>設定部署來源
*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*


Azure Stack 上的 App Service 支援從多個原始檔控制提供者進行隨選部署。 這項功能讓應用程式開發人員直接從其原始檔控制儲存機制進行部署。 如果使用者要設定 App Service 以連線至其儲存機制，雲端操作員必須先設定 Azure App Service on Azure Stack 與原始檔控制提供者之間的整合。  

除了本機 Git 外，也支援下列原始檔控制提供者：

* GitHub
* BitBucket
* OneDrive
* DropBox

## <a name="view-deployment-sources-in-app-service-administration"></a>檢視 App Service 管理中的部署來源

1. 以服務管理員身分登入 Azure Stack 管理入口網站 (https://adminportal.local.azurestack.external)。
2. 瀏覽至 [資源提供者]，然後選取 [App Service 資源提供者管理]。![App Service 資源提供者管理][1]
3. 按一下 [原始檔控制組態]。  您會在此看見所有已設定之部署來源的清單。
    ![App Service 資源提供者管理原始檔控制組態][2]

## <a name="configure-github"></a>設定 GitHub

您必須有 GitHub 帳戶才能完成此工作。 您可能會想要使用組織的帳戶，而非個人帳戶。

1. 登入 GitHub，瀏覽至 https://www.github.com/settings/developers，然後按一下 [註冊新的應用程式]。
    ![GitHub - 註冊新的應用程式][3]
2. 輸入 [應用程式名稱]，例如，Azure App Service on Azure Stack。
3. 輸入 [首頁 URL]。 首頁 URL 必須是 Azure Stack 入口網站位址。 例如： https://portal.local.azurestack.external。
4. 輸入 [應用程式描述]。
5. 輸入 [授權回呼 URL]。  在預設的 Azure Stack 部署中，URL 的形式為 https://portal.local.azurestack.external/TokenAuthorize，如果您在不同的網域下執行，請以您的網域替代 local.azurestack.external
6. 按一下 [註冊應用程式]。  您現在會看到列出應用程式之 [用戶端識別碼] 和 [用戶端密碼] 的頁面。
    ![GitHub - 已完成的應用程式註冊][5]
7.  在新的瀏覽器索引標籤或視窗中，以服務管理員身分登入 Azure Stack 管理入口網站 (https://adminportal.local.azurestack.external)。
8.  瀏覽至 [資源提供者]，然後選取 [App Service 資源提供者管理]。
9. 按一下 [原始檔控制組態]。
10. 複製 [用戶端識別碼] 和 [用戶端密碼] 並貼到對應的 GitHub 輸入方塊。
11. 按一下 [檔案] 。

## <a name="configure-bitbucket"></a>設定 BitBucket

您必須有 BitBucket 帳戶才能完成此工作。 您可能會想要使用組織的帳戶，而非個人帳戶。

1. 登入 BitBucket 並瀏覽至您的帳戶下的**整合**。
    ![BitBucket 儀表板 - 整合][7]
2. 按一下 [存取管理] 下的 [OAuth] 並 [新增取用者]。
    ![BitBucket 新增 OAuth 取用者][8]
3. 輸入取用者的 [名稱]，例如，Azure App Service on Azure Stack。
4. 輸入應用程式的 [描述]。
5. 輸入 [回呼 URL]。  在預設的 Azure Stack 部署中，回呼 URL 的形式為 https://portal.local.azurestack.external/TokenAuthorize，如果您在不同的網域下執行，請以您的網域替代 azurestack.local。  Url 必須按照此處列出的大小寫，BitBucket 整合才能成功。
6. 輸入 **URL** - 此 URL 應為 Azure Stack 入口網站 URL，例如 https://portal.local.azurestack.external。
7. 選取需要的 [權限]：
    - **存放庫**：*讀取*
    - **Webhook**：*讀取和寫入*
8. 按一下 [檔案] 。  您現在會看到這個新的應用程式，以及 **OAuth 取用者**底下的**金鑰**和**密碼**。
    ![BitBucket 應用程式清單][9]
9.  在新的瀏覽器索引標籤或視窗中，以服務管理員身分登入 Azure Stack 管理入口網站 (https://adminportal.local.azurestack.external)。
10.  瀏覽至 [資源提供者]，然後選取 [App Service 資源提供者管理]。
11. 按一下 [原始檔控制組態]。
12. 複製**金鑰**並貼到 BitBucket 的 [用戶端識別碼] 輸入方塊，以及複製**密碼**並貼到 BitBucket 的 [用戶端密碼] 輸入方塊。
13. 按一下 [檔案] 。


## <a name="configure-onedrive"></a>設定 OneDrive

您必須有連結到 OneDrive 帳戶的 Microsoft 帳戶，才能完成這項工作。  您可能會想要使用組織的帳戶，而非個人帳戶。

> [!NOTE]
> 目前不支援商務用 OneDrive 帳戶。

1. 瀏覽至 https://apps.dev.microsoft.com/?referrer=https%3A%2F%2Fdev.onedrive.com%2Fapp-registration.htm 並使用您的 Microsoft 帳戶登入。
2. 在 [我的應用程式] 下，按一下 [新增應用程式]。
![OneDrive 應用程式][10]
3. 為新的應用程式註冊輸入**名稱**，輸入 **Azure Stack 上的 App Service**，然後按一下 [建立應用程式]
4. 下一個畫面會列出新應用程式的屬性。 記錄 [應用程式識別碼]。
![OneDrive 應用程式屬性][11]
5. 在 [應用程式密碼] 下，按一下 [產生新密碼]。 請記下**產生的新密碼**。 這是您的應用程式密碼，並且當您在這個階段按一下 [確定] 之後，就無法擷取此密碼。
6. 在 [平台] 下，按一下 [新增平台]，然後選取 [網站]。
7. 輸入 [重新導向 URI]。  在預設的 Azure Stack 部署中，重新導向 URI 的形式為 https://portal.local.azurestack.external/TokenAuthorize，如果您在不同的網域下執行，請以您的網域替代 azurestack.local ![OneDrive 應用程式 - 新增 Web 平台][12]
8. 新增 [Microsoft Graph 權限] - [委派的權限]
    - **Files.ReadWrite.AppFolder**
    - **User.Read**  
      ![OneDrive 應用程式 - Graph 權限][13]
9. 按一下 [檔案] 。
10.  在新的瀏覽器索引標籤或視窗中，以服務管理員身分登入 Azure Stack 管理入口網站 (https://adminportal.local.azurestack.external)。
11.  瀏覽至 [資源提供者]，然後選取 [App Service 資源提供者管理]。
12. 按一下 [原始檔控制組態]。
13. 複製**應用程式識別碼**並貼到 OneDrive 的 [用戶端識別碼] 輸入方塊，以及複製**密碼**並貼到 OneDrive 的 [用戶端密碼] 輸入方塊。
14. 按一下 [檔案] 。

## <a name="configure-dropbox"></a>設定 DropBox

> [!NOTE]
> 您必須有 DropBox 帳戶才能完成這項工作。  您可能會想要使用組織的帳戶，而非個人帳戶。

1. 瀏覽至 https://www.dropbox.com/developers/apps 並使用您的 DropBox 帳戶登入。
2. 按一下 [ **建立應用程式**]。

    ![Dropbox 應用程式][14]

3. 選取 [DropBox API]。
4. 將存取層級設為 [應用程式資料夾]。
5. 輸入應用程式的**名稱**。
![Dropbox 應用程式註冊][15]
6. 按一下 [建立應用程式]。  您現在會看到列出應用程式設定的頁面，包括**應用程式金鑰**和**應用程式密碼**。
7. 確認 [應用程式資料夾名稱] 設為 **Azure App Service on Azure Stack**。
8. 設定 [OAuth 2 重新導向 URI]，然後按一下 [新增]。  在預設的 Azure Stack 部署中，重新導向 URI 的形式為 https://portal.local.azurestack.external/TokenAuthorize，如果您在不同的網域下執行，請以您的網域替代 azurestack.local。
![Dropbox 應用程式組態][16]
9.  在新的瀏覽器索引標籤或視窗中，以服務管理員身分登入 Azure Stack 管理入口網站 (https://adminportal.local.azurestack.external)。
10.  瀏覽至 [資源提供者]，然後選取 [App Service 資源提供者管理]。
11. 按一下 [原始檔控制組態]。
12. 複製**應用程式金鑰**並貼到 DropBox 的 [用戶端識別碼] 輸入方塊，以及複製**應用程式密碼**並貼到 DropBox 的 [用戶端密碼] 輸入方塊。
13. 按一下 [檔案] 。


<!--Image references-->
[1]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin.png
[2]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-source-control-configuration.png
[3]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-github-developer-applications.png
[4]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-github-register-a-new-oauth-application-populated.png
[5]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-github-register-a-new-oauth-application-complete.png
[6]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-roles-management-server-repair-all.png
[7]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-bitbucket-dashboard.png
[8]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-bitbucket-access-management-add-oauth-consumer.png
[9]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-bitbucket-access-management-add-oauth-consumer-complete.png
[10]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-applications.png
[11]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-application-registration.png
[12]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-application-platform.png
[13]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-application-graph-permissions.png
[14]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Dropbox-applications.png
[15]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Dropbox-application-registration.png
[16]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Dropbox-application-configuration.png

## <a name="next-steps"></a>後續步驟

使用者現在可以對[連續部署](https://docs.microsoft.com/azure/app-service-web/app-service-continuous-deployment)、[本機 Git 部署](https://docs.microsoft.com/azure/app-service-web/app-service-deploy-local-git)和[雲端資料夾同步處理](https://docs.microsoft.com/azure/app-service-web/app-service-deploy-content-sync)之類使用部署來源。
