---
title: 使用 Azure CLI 來建立 Azure AD 應用程式，並設定它以存取 Azure 媒體服務 API | Microsoft Docs
description: 此主題說明如何使用 Azure CLI 來建立 Azure AD 應用程式，並設定它以存取「Azure 媒體服務 API」。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/26/2019
ms.author: juliako
ms.openlocfilehash: 76a7cef074dd16a41dee59773aff00d8e58d432d
ms.sourcegitcommit: 77afc94755db65a3ec107640069067172f55da67
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98695936"
---
# <a name="use-azure-cli-to-create-an-azure-ad-app-and-configure-it-to-access-media-services-api"></a>使用 Azure CLI 來建立 Azure AD 應用程式，並設定它以存取媒體服務 API

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> 媒體服務 v2 不會再新增任何新的特性或功能。 <br/>查看最新版本的[媒體服務 v3](../latest/index.yml)。 另請參閱[從 v2 變更為 v3 的移轉指導方針](../latest/migrate-v-2-v-3-migration-introduction.md)

此主題說明如何使用 Azure CLI 來建立 Azure Active Directory (Azure AD) 應用程式和服務主體，以存取「Azure 媒體服務」資源。 

## <a name="prerequisites"></a>必要條件

- Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
- 媒體服務帳戶。 如需詳細資訊，請參閱[使用 Azure 入口網站建立 Azure 媒體服務帳戶](media-services-portal-create-account.md)。

## <a name="use-the-azure-cloud-shell"></a>使用 Azure Cloud Shell

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從入口網站上方的瀏覽窗格啟動 Cloud Shell。

    ![Cloud Shell](./media/media-services-cli-create-and-configure-aad-app/media-services-cli-create-and-configure-aad-app01.png) 

如需詳細資訊，請參閱 [Azure Cloud Shell 概觀](../../cloud-shell/overview.md)。

## <a name="create-an-azure-ad-app-and-configure-access-to-the-media-account-with-azure-cli"></a>使用 Azure CLI 來建立 Azure AD 應用程式並設定對媒體帳戶的存取權
 
```azurecli
az login
az ad sp create-for-rbac --name <appName> 
az role assignment create --assignee < user/app id> --role Contributor --scope <subscription/subscription id>
```

例如：

```azurecli
az role assignment create --assignee a3e068fa-f739-44e5-ba4d-ad57866e25a1 --role Contributor --scope /subscriptions/0b65e280-7917-4874-9fed-1307f2615ea2/resourceGroups/Default-AzureBatch-SouthCentralUS/providers/microsoft.media/mediaservices/sbbash
```

在此範例中，**scope** 是媒體服務帳戶的完整資源路徑。 不過，**scope** 可以是任何層級。

例如，它可以是下列層級之一：
 
* **訂用帳戶** 層級。
* **資源群組** 層級。
* **資源** 層級 (例如，媒體帳戶)。

如需詳細資訊，請參閱[使用 Azure CLI 來建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli)

另請參閱 [使用 Azure CLI 新增或移除 Azure 角色指派](../../role-based-access-control/role-assignments-cli.md)。 

## <a name="next-steps"></a>後續步驟

開始使用[上傳檔案至您的帳戶](media-services-portal-upload-files.md)。
