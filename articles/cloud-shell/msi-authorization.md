---
title: 在 Azure Cloud Shell 中使用 MSI| Microsoft Docs
description: 在 Azure Cloud Shell 中使用 MSI 驗證程式碼
services: azure
documentationcenter: ''
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 04/14/2018
ms.author: juluk
ms.openlocfilehash: 99577faf7328dc773a9da5f7c1227aa63600aa0a
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31516282"
---
# <a name="use-msi-in-azure-cloud-shell"></a>在 Azure Cloud Shell 中使用 MSI

Azure Cloud Shell 可支援使用受控服務識別 (MSI) 進行驗證。 可利用此方式來擷取存取權杖，以安全地與 Azure 服務進行通訊。

## <a name="about-managed-service-identity-msi"></a>關於受控服務識別 (MSI)
建置雲端應用程式常見的難題是如何安全地管理程式碼中必須存在的認證，以向雲端服務進行驗證。 在 Cloud Shell 中，您可能需要驗證來自 Key Vault 的擷取，才能完成認證，而這可能需要指令碼。

受控服務身分識別 (MSI) 可以輕易地解決此問題，因為 MSI 可在 Azure Active Directory (Azure AD) 中提供自動受控身分識別給 Azure 服務。 您可以使用此身分識別來完成任何支援 Azure AD 驗證的服務驗證 (包括 Key Vault)，不需要任何您程式碼中的認證。

## <a name="acquire-access-token-in-cloud-shell"></a>在 Cloud Shell 中取得存取權杖

執行下列命令以將您的 MSI 存取權杖設定為環境變數 (`access_token`)。
```
response=$(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s)
access_token=$(echo $response | python -c 'import sys, json; print (json.load(sys.stdin)["access_token"])')
echo The MSI access token is $access_token
```

## <a name="handling-token-expiration"></a>處理權杖到期

本機 MSI 子系統會快取權杖。 因此，您可以視需要呼叫它，而只有在下列情況，線上呼叫 Azure AD 才會有結果：
- 由於快取中沒有權杖而發生快取遺漏
- 權杖已過期

如果您在程式碼中快取權杖，則應該妥善處理資源指出權杖過期的情節。

若要處理權杖錯誤，請造訪[以 CURL 取得 MSI 存取權杖的 MSI 頁面](https://docs.microsoft.com/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling)。

## <a name="next-steps"></a>後續步驟
[深入了解此 MSI](https://docs.microsoft.com/azure/active-directory/managed-service-identity/overview)  
[從 MSI VM 取得存取權杖](https://docs.microsoft.com/azure/active-directory/managed-service-identity/how-to-use-vm-token)
