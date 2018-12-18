---
title: Azure App Service 部署認證 | Microsoft Docs
description: 了解如何使用 Azure App Service 部署認證。
services: app-service
documentationcenter: ''
author: dariagrigoriu
manager: erikre
editor: mollybos
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 01/05/2016
ms.author: dariagrigoriu
ms.openlocfilehash: a17260770f0b2e0a73585ce4108bd5625ac22229
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39436143"
---
# <a name="configure-deployment-credentials-for-azure-app-service"></a>設定 Azure App Service 的部署認證
[Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714) 支援兩種認證類，用於[本機 Git 部署](app-service-deploy-local-git.md)和 [FTP/S 部署](app-service-deploy-ftp.md)。 這些與您的 Azure Active Directory 認證不相同。

* **使用者層級認證**︰一組用於整個 Azure 帳戶的認證。 它可以用來將任何應用程式部署至 Azure 帳戶有權存取的任何訂用帳戶的 App Service 中。 有些預設認證集是在 [App Service]  >  [&lt;app_name>]  >  [部署認證] 中設定。 也有些預設集合是呈現在入口網站 GUI 中 (例如應用程式的[資源頁面](../azure-resource-manager/resource-group-portal.md#manage-resources)的 [概觀] 和 [屬性])。

    > [!NOTE]
    > 透過角色型存取控制 (RBAC) 或共同管理員權限委派 Azure 資源的存取權時，收到應用程式存取權的每個 Azure 使用者都可以使用他/她的個人使用者層級認證，直到存取權被撤銷為止。 這些部署認證不得與其他 Azure 使用者共用。
    >
    >

* **應用程式層級認證**︰一組用於單個 應用程式的認證。 它只可以用來部署該應用程式。 每個應用程式的認證會在應用程式建立時自動產生，並且可以在應用程式的發行設定檔中找到。 您無法以手動方式設定此認證，但您隨時可以為應用程式重設認證。

    > [!NOTE]
    > 若要透過角色型存取控制 (RBAC) 給予某人這些認證的存取權，您需要讓他們成為 Web 應用程式的參與者或更高級的角色。 不允許發佈讀取器，因此無法存取這些認證。
    >
    >

## <a name="userscope"></a>設定及重設使用者層級的認證

您可以在任何應用程式的[資源頁面](../azure-resource-manager/resource-group-portal.md#manage-resources)中設定使用者層級認證。 無論在哪一個應用程式中設定這些認證，它都會套用至所有應用程式和您的 Azure 帳戶中的所有訂用帳戶。 

設定使用者層級認證：

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [App Service] > &lt;any_app>  >  [部署認證]。

    > [!NOTE]
    > 在入口網站中，您必須至少有一個應用程式，才能存取 [部署認證] 頁面。 不過，若使用 [Azure CLI](/cli/azure/webapp/deployment/user?view=azure-cli-latest#az-webapp-deployment-user-set)，沒有現有應用程式也可以設定使用者層級認證。

2. 設定使用者名稱和密碼，然後按一下 [儲存]。

    ![](./media/app-service-deployment-credentials/deployment_credentials_configure.png)

一旦設定好您的部署認證，就可以在應用程式的 [概觀] 中找到「Git」部署使用者名稱，

![](./media/app-service-deployment-credentials/deployment_credentials_overview.png)

以及在應用程式的 [屬性] 中找到 「FTP」部署使用者名稱。

![](./media/app-service-deployment-credentials/deployment_credentials_properties.png)

> [!NOTE]
> Azure 不會顯示您的使用者層級部署密碼。 如果您忘記密碼，將無法取得密碼。 不過，您可以依照本節的步驟來重設您的認證。
>
>  

## <a name="appscope"></a>設定及重設應用程式層級的認證
App Service 中每個應用程式的認證會儲存在 XML 發佈設定檔中。

若要取得應用程式層級的認證：

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [App Service] >&lt;any_app>  >  [概觀]。

2. 按一下 [...更多]  >  [取得發行設定檔]，便會開始下載 .PublishSettings 檔案。

    ![](./media/app-service-deployment-credentials/publish_profile_get.png)

3. 開啟 .PublishSettings 檔案，找到包含 `publishMethod="FTP"` 屬性的 `<publishProfile>` 標籤。 然後取得其 `userName` 和 `password` 屬性。
這些就是應用程式層級的認證。

    ![](./media/app-service-deployment-credentials/publish_profile_editor.png)

    類似使用者層級認證，FTP 部署使用者名稱的格式是 `<app_name>\<username>`，Git 部署使用者名稱格式是 `<username>`，沒有前置的 `<app_name>\`。

若要重設應用程式層級的認證：

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [App Service] >**&lt;any_app>** > **概觀**。

2. 按一下 [...更多]  >  [重設發行設定檔]。 按一下 [是] 確認重設。

    重設動作會何先前下載的任何 .PublishSettings 檔案失效。

## <a name="next-steps"></a>後續步驟

了解如何使用這些認證從[本機 Git](app-service-deploy-local-git.md)或使用[FTP/S](app-service-deploy-ftp.md) 部署應用程式。
