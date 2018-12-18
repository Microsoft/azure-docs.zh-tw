---
title: 使用 Git 設定 API 管理服務 - Azure | Microsoft Docs
description: 了解如何使用 Git 儲存和設定 API 管理服務組態。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: mattfarm
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/02/2018
ms.author: apimpm
ms.openlocfilehash: ce4ff33a8af9cf864ad4902599a0ab18b2b27182
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387220"
---
# <a name="how-to-save-and-configure-your-api-management-service-configuration-using-git"></a>如何使用 Git 儲存和設定 API 管理服務組態

每個 API 管理服務執行個體會維護組態資料庫，包含服務執行個體的組態和中繼資料的相關資訊。 可以對服務執行個體進行變更，方法是使用PowerShell Cmdlet 或進行 REST API 呼叫，變更 Azure 入口網站中的設定。 除了這些方法，您也可以使用 Git 管理服務執行個體組態，啟用下列服務管理案例︰

* 組態版本 - 下載並儲存不同版本的服務組態
* 大量的組態變更 - 對本機儲存機制中服務組態的多個部分進行變更，並且使用單一作業將變更整合回伺服器
* 熟悉的 Git 工具鏈和工作流程 - 使用您已熟悉的 Git 工具和工作流程

下圖顯示設定您的 API 管理服務執行個體的不同方式的概觀。

![Git 設定][api-management-git-configure]

當您使用 Azure 入口網站、PowerShell Cmdlet 或 REST API 對服務進行變更時，您會使用 `https://{name}.management.azure-api.net` 端點管理服務組態資料庫，如圖表右側所示。 圖表左側說明如何針對位於 `https://{name}.scm.azure-api.net`的服務，使用 Git 和 Git 儲存機制管理服務組態。

下列步驟提供使用 Git 管理 API 管理服務執行個體的概觀。

1. 存取服務中的 Git 組態
2. 將您的服務組態資料庫儲存至您的 Git 儲存機制
3. 將 Git 儲存機制複製到本機電腦
4. 將最新的儲存機制提取至您的本機電腦，認可並且將變更推送回您的儲存機制
5. 從您的儲存機制將變更部署至您的服務組態資料庫

本文說明如何啟用及使用 Git 來管理您的服務組態，並提供 Git 儲存機制中檔案和資料夾的參考。

## <a name="access-git-configuration-in-your-service"></a>存取服務中的 Git 組態

若要檢視並設定您的 Git 組態設定，您可以按一下 [安全性] 功能表，然後瀏覽至 [設定存放庫] 索引標籤。

![啟用 GIT][api-management-enable-git]

> [!IMPORTANT]
> 未定義為屬性的任何密碼會儲存在儲存機制，並且仍然保留其歷程記錄，直到您停用然後重新啟用 Git 存取。 屬性會提供一個安全的地方以管理跨所有的 API 組態和原則的常數字串值，包括密碼，因此您不必將它們直接儲存在您的原則陳述式。 如需詳細資訊，請參閱 [如何使用 Azure API 管理原則中的屬性](api-management-howto-properties.md)。
> 
> 

如需使用 REST API 啟用或停用 Git 存取的詳細資訊，請參閱 [使用 REST API 啟用或停用 Git 存取](https://msdn.microsoft.com/library/dn781420.aspx#EnableGit)。

## <a name="to-save-the-service-configuration-to-the-git-repository"></a>將服務組態儲存至 Git 儲存機制

複製儲存機制之前的第一個步驟是將服務組態的目前狀態儲存至儲存機制。 按一下 [儲存到存放庫]。

在 [確認] 畫面上進行任何所需的變更，然後按一下 [確定]  以儲存。

儲存組態一段時間後，儲存機制的組態狀態隨即顯示，包括最後組態變更和上次同步處理服務組態和儲存機制的日期與時間。

一旦組態儲存至儲存機制，就可以複製。

如需使用 REST API 執行此作業的詳細資訊，請參閱 [使用 REST API 認可組態快照集](https://msdn.microsoft.com/library/dn781420.aspx#CommitSnapshot)。

## <a name="to-clone-the-repository-to-your-local-machine"></a>將儲存機制複製到本機電腦

若要複製儲存機制，您需要儲存機制的 URL、使用者名稱和密碼。 若要取得使用者名稱和其他認證，按一下靠近頁面頂端的 [存取認證]。  
 
若要產生密碼，請先確定已將 [到期] 設為所需的到期日期和時間，然後按一下 [產生]。

> [!IMPORTANT]
> 記下此密碼。 一旦您離開此頁面，就不會再次顯示密碼。
> 

下列範例會使用來自 [Git for Windows](http://www.git-scm.com/downloads) 的 Git Bash 工具，但是您可以使用任何您已熟悉的 Git 工具。

在想要的資料夾中開啟 Git 工具，然後執行下列命令，使用 Azure 入口網站提供的命令，將 git 存放庫複製到本機電腦。

```
git clone https://bugbashdev4.scm.azure-api.net/
```

出現提示時，請提供使用者名稱和密碼。

如果您收到任何錯誤，請嘗試修改您的 `git clone` 命令以包含使用者名稱和密碼，如下列範例所示。

```
git clone https://username:password@bugbashdev4.scm.azure-api.net/
```

如果發生錯誤，請嘗試 URL 編碼命令的密碼部分。 完成這項操作的其中一個快速方法是開啟 Visual Studio，並且在 [即時運算視窗] 發出下列命令。 若要開啟 [即時運算視窗]，請在 Visual Studio 中開啟任何解決方案或專案 (或建立新的空白主控台應用程式)，然後從 [偵錯] 功能表選擇 [視窗]、[即時運算]。

```
?System.NetWebUtility.UrlEncode("password from the Azure portal")
```

使用編碼的密碼以及使用者名稱和儲存機制位置以建構 git 命令。

```
git clone https://username:url encoded password@bugbashdev4.scm.azure-api.net/
```

複製儲存機制之後，您可以在本機檔案系統中檢視並加以使用。 如需詳細資訊，請參閱 [本機 Git 儲存機制的檔案和資料夾結構參考](#file-and-folder-structure-reference-of-local-git-repository)。

## <a name="to-update-your-local-repository-with-the-most-current-service-instance-configuration"></a>使用最新的服務執行個體組態更新本機儲存機制

如果您在 Azure 入口網站中或使用 REST API 變更您的 API 管理服務執行個體，您必須先將這些變更儲存至存放庫，才能使用最新的變更來更新本機存放庫。 若要這樣做，請按一下 Azure 入口網站中 [設定存放庫] 索引標籤上的 [將設定儲存到存放庫]，然後在本機存放庫中發出下列命令。

```
git pull
```

在執行 `git pull` 之前，請確認您是位於本機儲存機制的資料夾中。 如果您已完成 `git clone` 命令，則必須執行類似下列的命令，將目錄變更為您的儲存機制。

```
cd bugbashdev4.scm.azure-api.net/
```

## <a name="to-push-changes-from-your-local-repo-to-the-server-repo"></a>將變更從您的本機儲存機制推送至伺服器儲存機制
若要將變更從本機儲存機制推送至伺服器儲存機制，必須認可您的變更，然後再將其推送至伺服器儲存機制。 若要認可變更，請開啟您的 Git 命令工具，切換至您的本機儲存機制的目錄，然後發出下列命令。

```
git add --all
git commit -m "Description of your changes"
```

若要將所有認可推送至伺服器，請執行下列命令。

```
git push
```

## <a name="to-deploy-any-service-configuration-changes-to-the-api-management-service-instance"></a>將服務組態變更部署至 API 管理服務執行個體

一旦您的本機變更被認可並且推送至伺服器儲存機制，您可以將它們部署到您的 API 管理服務執行個體。

如需使用 REST API 執行此作業的詳細資訊，請參閱 [使用 REST API 將 Git 變更部署至組態資料庫](https://docs.microsoft.com/rest/api/apimanagement/tenantconfiguration)。

## <a name="file-and-folder-structure-reference-of-local-git-repository"></a>本機 Git 儲存機制的檔案和資料夾結構參考

本機 git 儲存機制中的檔案和資料夾包含服務執行個體的相關組態資訊。

| Item | 說明 |
| --- | --- |
| 根 api 管理資料夾 |包含服務執行個體的最上層組態 |
| apis 資料夾 |包含服務執行個體中 apis 的組態 |
| 群組資料夾 |包含服務執行個體中群組的組態 |
| 原則資料夾 |包含服務執行個體中的原則 |
| portalStyles 資料夾 |包含服務執行個體中開發人員入口網站自訂的組態 |
| 產品資料夾 |包含服務執行個體中產品的組態 |
| 範本資料夾 |包含服務執行個體中電子郵件範本的組態 |

每個資料夾可以包含一或多個檔案，在某些情況下可以包含一或多個資料夾，例如每個 API、產品或群組的資料夾。 每個資料夾中的檔案是特定於資料夾名稱所描述的實體類型。

| 檔案類型 | 目的 |
| --- | --- |
| json |個別實體的組態資訊 |
| html |實體的相關描述，通常顯示於開發人員入口網站 |
| xml |Policy statements |
| css |開發人員入口網站自訂的樣式表 |

可以在您的本機檔案系統上建立、刪除、編輯和管理這些檔案，並將變更部署回您的 API 管理服務執行個體。

> [!NOTE]
> 下列實體不包含在 Git 儲存機制，因此無法使用 Git 來設定。
> 
> * 使用者
> * 訂用帳戶
> * properties
> * 樣式以外的開發人員入口網站實體
> 

### <a name="root-api-management-folder"></a>根 api 管理資料夾
根 `api-management` 資料夾包含 `configuration.json` 檔案，其中包含服務執行個體的最上層資訊，格式如下。

```json
{
  "settings": {
    "RegistrationEnabled": "True",
    "UserRegistrationTerms": null,
    "UserRegistrationTermsEnabled": "False",
    "UserRegistrationTermsConsentRequired": "False",
    "DelegationEnabled": "False",
    "DelegationUrl": "",
    "DelegatedSubscriptionEnabled": "False",
    "DelegationValidationKey": ""
  },
  "$ref-policy": "api-management/policies/global.xml"
}
```

前四個設定 (`RegistrationEnabled`、`UserRegistrationTerms`、`UserRegistrationTermsEnabled` 和 `UserRegistrationTermsConsentRequired`) 對應至 [安全性] 區段的 [身分識別] 索引標籤中的下列設定。

| 身分識別設定 | 對應至 |
| --- | --- |
| RegistrationEnabled | 核取方塊 |
| UserRegistrationTerms | 文字方塊 |
| UserRegistrationTermsEnabled | 核取方塊 |
| UserRegistrationTermsConsentRequired | 核取方塊 |

接下來四個設定 (`DelegationEnabled`、`DelegationUrl`、`DelegatedSubscriptionEnabled` 和 `DelegationValidationKey`) 對應至 [安全性] 區段的 [委派] 索引標籤中的下列設定。

| 委派設定 | 對應至 |
| --- | --- |
| DelegationEnabled |[委派登入和註冊] 核取方塊 |
| DelegationUrl | 文字方塊 |
| DelegatedSubscriptionEnabled | 核取方塊 |
| DelegationValidationKey | 文字方塊 |

最後的設定 ( `$ref-policy`) 會對應至服務執行個體的全域原則陳述式檔案。

### <a name="apis-folder"></a>apis 資料夾
`apis` 資料夾包含服務執行個體中每個 API 的資料夾，其中包含下列項目。

* `apis\<api name>\configuration.json` - 這是 API 的組態，而且包含後端服務 URL 和作業的相關資訊。 此資訊與當您以 `application/json` 格式的 `export=true` 呼叫[取得特定 API](https://docs.microsoft.com/rest/api/apimanagement/api/get) 時傳回的資訊相同。
* `apis\<api name>\api.description.html` - 這是 API 的說明，並會對應至 [API 實體](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.table._entity_property)的 `description` 屬性。
* `apis\<api name>\operations\` - 此資料夾包含 `<operation name>.description.html` 檔案，該檔案對應至 API 中的作業。 每個檔案包含 API 中單一作業的說明，其會對應至 REST API 中[作業實體](https://docs.microsoft.com/rest/api/visualstudio/operations/list#operationproperties)的 `description` 屬性。

### <a name="groups-folder"></a>群組資料夾
`groups` 資料夾包含服務執行個體中定義的每個群組的資料夾。

* `groups\<group name>\configuration.json` - 這是群組的組態。 此資訊與當您呼叫 [取得特定群組](https://docs.microsoft.com/rest/api/apimanagement/group/get) 作業時傳回的資訊相同。
* `groups\<group name>\description.html` - 這是群組的說明，並會對應至[群組實體](https://docs.microsoft.com/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity)的 `description` 屬性。

### <a name="policies-folder"></a>原則資料夾
`policies` 資料夾包含您的服務執行個體的原則陳述式。

* `policies\global.xml` - 包含在您服務執行個體的全域範圍中定義的原則。
* `policies\apis\<api name>\` - 如果您在 API 範圍中定義了任何原則，它們就會包含在此資料夾中。
* `policies\apis\<api name>\<operation name>\` 資料夾 - 如果您有任何定義於作業範圍中的原則，它們就會包含在此資料夾的 `<operation name>.xml` 檔案中，其會對應至每個作業的原則陳述式。
* `policies\products\` - 如果您有任何定義於產品範圍中的原則，它們就會包含在此資料夾中，其中包含 `<product name>.xml` 檔案，其會對應至每個產品的原則陳述式。

### <a name="portalstyles-folder"></a>portalStyles 資料夾
`portalStyles` 資料夾包含適用於服務執行個體的開發人員入口網站自訂的組態和樣式表。

* `portalStyles\configuration.json` - 包含開發人員入口網站所使用的樣式表名稱
* `portalStyles\<style name>.css` - 每個 `<style name>.css` 檔案包含開發人員入口網站的樣式 (預設為 `Preview.css` 和 `Production.css`)。

### <a name="products-folder"></a>產品資料夾
`products` 資料夾包含服務執行個體中定義的每個產品的資料夾。

* `products\<product name>\configuration.json` - 這是產品的組態。 此資訊與當您呼叫 [取得特定產品](https://docs.microsoft.com/rest/api/apimanagement/product/get) 作業時傳回的資訊相同。
* `products\<product name>\product.description.html` - 這是產品的說明，並會對應至 REST API 中[產品實體](https://docs.microsoft.com/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-product-entity)的 `description` 屬性。

### <a name="templates"></a>範本
`templates` 資料夾包含服務執行個體的 [電子郵件範本](api-management-howto-configure-notifications.md) 的組態。

* `<template name>\configuration.json` - 這是電子郵件範本的組態。
* `<template name>\body.html` - 這是電子郵件範本的主體。

## <a name="next-steps"></a>後續步驟
如需管理您的服務執行個體的其他方法的詳細資訊，請參閱︰

* 使用下列 PowerShell Cmdlet 管理您的服務執行個體
  * [服務部署 PowerShell Cmdlet 參考](https://docs.microsoft.com/powershell/module/wds)
  * [服務管理 PowerShell Cmdlet 參考](https://docs.microsoft.com/powershell/azure/servicemanagement/overview)
* 使用 REST API 管理您的服務執行個體
  * [API 管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn776326.aspx)


[api-management-enable-git]: ./media/api-management-configuration-repository-git/api-management-enable-git.png
[api-management-git-enabled]: ./media/api-management-configuration-repository-git/api-management-git-enabled.png
[api-management-save-configuration]: ./media/api-management-configuration-repository-git/api-management-save-configuration.png
[api-management-save-configuration-confirm]: ./media/api-management-configuration-repository-git/api-management-save-configuration-confirm.png
[api-management-configuration-status]: ./media/api-management-configuration-repository-git/api-management-configuration-status.png
[api-management-configuration-git-clone]: ./media/api-management-configuration-repository-git/api-management-configuration-git-clone.png
[api-management-generate-password]: ./media/api-management-configuration-repository-git/api-management-generate-password.png
[api-management-password]: ./media/api-management-configuration-repository-git/api-management-password.png
[api-management-git-configure]: ./media/api-management-configuration-repository-git/api-management-git-configure.png
[api-management-configuration-deploy]: ./media/api-management-configuration-repository-git/api-management-configuration-deploy.png
[api-management-identity-settings]: ./media/api-management-configuration-repository-git/api-management-identity-settings.png
[api-management-delegation-settings]: ./media/api-management-configuration-repository-git/api-management-delegation-settings.png
[api-management-git-icon-enable]: ./media/api-management-configuration-repository-git/api-management-git-icon-enable.png




