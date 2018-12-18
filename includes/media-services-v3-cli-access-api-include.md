---
title: 包含檔案
description: 包含檔案
services: media-services
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 05/29/2018
ms.author: juliako
ms.custom: include file
ms.openlocfilehash: f4600413e05950446db71f988c4c4302f0dcacb3
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39062966"
---
## <a name="access-the-media-services-api"></a>存取媒體服務 API

若要連線到 Azure Media Services API，您可使用 Azure AD 服務主體驗證。 下列命令會建立 Azure AD 應用程式，並將服務主體連結到帳戶。 您應使用傳回值來設定您的應用程式。

執行指令碼之前，您可以用建立資源時所選擇的名稱取代 `amsaccount` 和 `amsResourceGroup`。 `amsaccount` 是連結服務主體的 Azure 媒體服務名稱。

此命令會傳回 `json` 輸出︰

```azurecli-interactive
az ams account sp create --account-name amsaccount --resource-group amsResourceGroup
```

此命令會產生如下的回應：

```json
{
  "AadClientId": "00000000-0000-0000-0000-000000000000",
  "AadEndpoint": "https://login.microsoftonline.com",
  "AadSecret": "00000000-0000-0000-0000-000000000000",
  "AadTenantId": "00000000-0000-0000-0000-000000000000",
  "AccountName": "amsaccount",
  "ArmAadAudience": "https://management.core.windows.net/",
  "ArmEndpoint": "https://management.azure.com/",
  "Region": "West US 2",
  "ResourceGroup": "amsResourceGroup",
  "SubscriptionId": "00000000-0000-0000-0000-000000000000"
}
```

如果您想要在回應中取得 `xml`，請使用下列命令：

```azurecli-interactive
az ams account sp create --account-name amsaccount --resource-group amsResourceGroup --xml
```
