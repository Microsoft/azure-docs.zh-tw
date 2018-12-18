---
title: 使用憑證保護 B2B 訊息 - Azure Logic Apps | Microsoft Docs
description: 使用企業整合套件在 Azure Logic Apps 中新增憑證以保護 B2B 訊息
services: logic-apps
ms.service: logic-apps
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, LADocs
manager: jeconnoc
ms.assetid: 4cbffd85-fe8d-4dde-aa5b-24108a7caa7d
ms.suite: integration
ms.topic: article
ms.date: 08/17/2018
ms.openlocfilehash: 5ae69d365a183f7d2a219d853241e73c1e27212b
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42144815"
---
# <a name="secure-b2b-messages-with-certificates"></a>使用憑證保護 B2B 訊息

如果您需要將 B2B 通訊保密，您可以藉由將憑證新增至整合帳戶，來保護企業整合應用程式 (特別是邏輯應用程式) 的 B2B 通訊。 憑證是數位文件，可檢查電子通訊參與者的身分識別，還能協助您透過下列方式保護通訊：

* 將訊息內容加密。
* 以數位方式簽署訊息。 

您可以在企業整合應用程式中使用這些憑證：

* [公開憑證](https://en.wikipedia.org/wiki/Public_key_certificate)，您必須從公用網路的[憑證授權單位 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 購買這些憑證，但不需要任何金鑰。 

* 私用憑證或[自我簽署憑證](https://en.wikipedia.org/wiki/Self-signed_certificate)，這些憑證由您自行建立並核發，但也需要私密金鑰。 

## <a name="upload-a-public-certificate"></a>上傳公開憑證

若要在具備 B2B 功能的 Logic Apps 中使用「公開憑證」，您必須先將憑證上傳到整合帳戶。 當您在自己建立的[合約](logic-apps-enterprise-integration-agreements.md)中定義屬性之後，就能使用憑證來幫助您保護 B2B 訊息。

1. 登入 [Azure 入口網站](https://portal.azure.com)。 在主要 Azure 功能表上，選取 [所有資源]。 在 [搜尋] 方塊中，輸入您的整合帳戶名稱，然後選取您要的整合帳戶。

   ![尋找並選取您的整合帳戶](media/logic-apps-enterprise-integration-certificates/select-integration-account.png)  

2. 在 [元件] 下方選擇 [憑證] 圖格。

   ![選擇 [憑證]](media/logic-apps-enterprise-integration-certificates/add-certificates.png)

3. 在 [憑證] 之下，選擇 [新增]。 在 [新增憑證] 之下，提供憑證的以下詳細資料。 完成時，選擇 [確定]。

   | 屬性 | 值 | 說明 | 
   |----------|-------|-------------|
   | **名稱** | <*certificate-name*> | 您的憑證名稱，在本例中為 "publicCert" | 
   | **憑證類型** | 公開 | 您的憑證類型 |
   | **憑證** | <*certificate-file-name*> | 若要尋找及選取要上傳的憑證檔案，請選擇 [憑證] 方塊旁的資料夾圖示。 |
   ||||

   ![選擇 [新增]，並提供憑證的詳細資料](media/logic-apps-enterprise-integration-certificates/public-certificate-details.png)

   Azure 會在驗證您的選取項目之後，將憑證上傳。

   ![Azure 會顯示新的憑證](media/logic-apps-enterprise-integration-certificates/new-public-certificate.png) 

## <a name="upload-a-private-certificate"></a>上傳私人憑證

若要在具備 B2B 功能的 Logic Apps 中使用「私人憑證」，您必須先將憑證上傳到整合帳戶。 您也必須要有先加入至 [Azure Key Vault](../key-vault/key-vault-get-started.md) 的私密金鑰。 

當您在自己建立的[合約](logic-apps-enterprise-integration-agreements.md)中定義屬性之後，就能使用憑證來幫助您保護 B2B 訊息。

> [!NOTE]
> 對於私人憑證，務必要新增對應的公開憑證，讓其出現在 [AS2 合約](logic-apps-enterprise-integration-as2.md)的**傳送和接收**設定以用於簽署和加密訊息。

1. [將私密金鑰新增至 Key Vault](../key-vault/key-vault-get-started.md#add) 並提供 [金鑰名稱]。
   
2. 授權 Azure Logic Apps 以便對 Azure Key Vault 執行作業。 若要對 Logic Apps 服務主體授與存取權，請使用 PowerShell 命令 [Set-AzureRmKeyVaultAccessPolicy](https://docs.microsoft.com/powershell/module/azurerm.keyvault/set-azurermkeyvaultaccesspolicy)，例如：

   `Set-AzureRmKeyVaultAccessPolicy -VaultName 'TestcertKeyVault' -ServicePrincipalName 
   '7cd684f4-8a78-49b0-91ec-6a35d38739ba' -PermissionsToKeys decrypt, sign, get, list`
 
3. 登入 [Azure 入口網站](https://portal.azure.com)。 在主要 Azure 功能表上，選取 [所有資源]。 在 [搜尋] 方塊中，輸入您的整合帳戶名稱，然後選取您要的整合帳戶。

   ![尋找您的整合帳戶](media/logic-apps-enterprise-integration-certificates/select-integration-account.png) 

4. 在 [元件] 下方選擇 [憑證] 圖格。  

   ![選擇 [憑證] 圖格](media/logic-apps-enterprise-integration-certificates/add-certificates.png)

5. 在 [憑證] 之下，選擇 [新增]。 在 [新增憑證] 之下，提供憑證的以下詳細資料。 完成時，選擇 [確定]。

   | 屬性 | 值 | 說明 | 
   |----------|-------|-------------|
   | **名稱** | <*certificate-name*> | 您的憑證名稱，在本例中為 "privateCert" | 
   | **憑證類型** | 私人 | 您的憑證類型 |
   | **憑證** | <*certificate-file-name*> | 若要尋找及選取要上傳的憑證檔案，請選擇 [憑證] 方塊旁的資料夾圖示。 | 
   | **資源群組** | <*integration-account-resource-group*> | 整合帳戶的資源群組，在本例中為 "MyResourceGroup" | 
   | **金鑰保存庫** | <*key-vault-name*> | Azure 金鑰保存庫的名稱 |
   | **金鑰名稱** | <*key-name*> | 金鑰名稱 |
   ||||

   ![選擇 [新增]，並提供憑證的詳細資料](media/logic-apps-enterprise-integration-certificates/private-certificate-details.png)

   Azure 會在驗證您的選取項目之後，將憑證上傳。

   ![Azure 會顯示新的憑證](media/logic-apps-enterprise-integration-certificates/new-private-certificate.png) 

## <a name="next-steps"></a>後續步驟

* [建立 B2B 合約](logic-apps-enterprise-integration-agreements.md)
