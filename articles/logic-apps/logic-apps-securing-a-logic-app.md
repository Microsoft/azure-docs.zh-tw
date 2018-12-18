---
title: 安全存取 Azure Logic Apps | Microsoft Docs
description: 保護對 Azure Logic Apps 工作流程中觸發程序、輸入和輸出、動作參數和服務的存取
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, LADocs
ms.assetid: 9fab1050-cfbc-4a8b-b1b3-5531bee92856
ms.topic: article
ms.date: 11/22/2016
ms.openlocfilehash: 1307b6df22c51af9710d44abb23178d65e3507aa
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44377389"
---
# <a name="secure-access-in-azure-logic-apps"></a>在 Azure Logic Apps 中安全存取

以下是您可以安全存取邏輯應用程式中不同元件的方式：

* 保護存取，以便透過 HTTP 要求觸發程序觸發邏輯應用程式工作流程。
* 保護存取，以便管理、編輯或讀取邏輯應用程式。
* 保護邏輯應用程式執行的輸入和輸出內容的存取。
* 保護邏輯應用程式工作流程中動作的參數或輸入。
* 保護可接收邏輯應用程式工作流程要求的服務存取。

## <a name="secure-access-to-trigger"></a>保護觸發程序的存取

當您使用透過 HTTP 要求 ([要求](../connectors/connectors-native-reqres.md)或 [Webhook](../connectors/connectors-native-webhook.md)) 引發的邏輯應用程式時，您可以限制存取，讓只有經過授權的用戶端可以引發該邏輯應用程式。 所有傳入邏輯應用程式的要求都會透過 SSL 加密並保護。

### <a name="shared-access-signature"></a>共用存取簽章

邏輯應用程式的每個要求端點都會在 URL 中包含[共用存取簽章](../storage/common/storage-dotnet-shared-access-signature-part-1.md) (SAS)。 每個 URL 都包含 `sp`、`sv` 和 `sig` 查詢參數。 `sp` 會指定權限，並對應至允許的 HTTP 方法，`sv` 是用來產生的版本，而 `sig` 是用來驗證對觸發程序的存取。 簽章是使用 SHA256 演算法搭配秘密金鑰所產生，位於所有的 URL 路徑和屬性上。 秘密金鑰永遠不會公開及發佈，並且會維持加密狀態，儲存於邏輯應用程式中。 您的邏輯應用程式只會針對包含使用秘密金鑰建立之有效簽章的觸發程序進行授權。

#### <a name="regenerate-access-keys"></a>重新產生存取金鑰

您可以隨時透過 REST API 或 Azure 入口網站重新產生新的安全金鑰。 之前使用舊金鑰產生的所有目前 URL 都將失效，並且不再有引發邏輯應用程式的授權。

1. 在 Azure 入口網站中，開啟您要重新產生金鑰的邏輯應用程式
1. 按一下 [設定] 之下的 [存取金鑰] 功能表項目
1. 選擇要重新產生的金鑰並完成程序

重新產生之後，您所擷取的 URL 會使用新的存取金鑰進行簽署。

#### <a name="creating-callback-urls-with-an-expiration-date"></a>使用具有到期日期的回呼 URL

如果您與其他人共用 URL，您可以視需要產生具有特定金鑰和到期日期的 URL。 您之後就能順暢地輪替金鑰，或是確保引發應用程式的存取權限制於特定的時間範圍。 您可以透過[邏輯應用程式 REST API](https://docs.microsoft.com/rest/api/logic/workflowtriggers) 指定 URL 的到期日期：

``` http
POST 
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{workflowName}/triggers/{triggerName}/listCallbackUrl?api-version=2016-06-01
```

在主體中，包含屬性 `NotAfter` 做為 JSON 日期字串，這會傳回僅在 `NotAfter` 日期與時間之前有效的回呼 URL。

#### <a name="creating-urls-with-primary-or-secondary-secret-key"></a>建立具有主要或次要秘密金鑰的 URL

當您針對要求式觸發程序產生或是列出回呼 URL 時，您也可以指定要用來簽署 URL 的金鑰。  您可以透過[邏輯應用程式 REST API](https://docs.microsoft.com/rest/api/logic/workflowtriggers) 產生以特定金鑰簽署的 URL，如下所示：

``` http
POST 
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{workflowName}/triggers/{triggerName}/listCallbackUrl?api-version=2016-06-01
```

在主體中，將屬性 `KeyType` 包含為 `Primary` 或 `Secondary`。  這會傳回以指定之安全金鑰簽署的 URL。

### <a name="restrict-incoming-ip-addresses"></a>限制傳入的 IP 位址

除了共用存取簽章，您應該會想要限制只從特定用戶端呼叫邏輯應用程式。  例如，如果您透過 Azure API 管理來管理端點，您可以限制邏輯應用程式僅接受來自 API 管理實體之 IP 位址的要求。

此設定可在邏輯應用程式設定中進行設定：

1. 在 Azure 入口網站中，開啟您要新增 IP 位址限制的邏輯應用程式
1. 按一下 [設定] 下的 [工作流程設定] 功能表項目
1. 指定觸發程序可接受的 IP 位址範圍清單

有效的 IP 位址範圍格式為 `192.168.1.1/32`。 如果您只想讓邏輯應用程式做為巢狀邏輯應用程式進行引發，請選取 [僅其他邏輯應用程式] 選項。 這個選項會將空陣列寫入資源，表示只能成功引發來自服務本身 (父邏輯應用程式) 的呼叫。

> [!NOTE]
> 您仍然可以透過 REST API / Management `/triggers/{triggerName}/run` 來執行含有要求觸發程序的邏輯應用程式，不論 IP 位址為何。 在此情況下，會要求針對 Azure REST API 進行驗證，而所有事件都會出現在 Azure 稽核記錄中。 請適當地設定存取控制原則。

#### <a name="setting-ip-ranges-on-the-resource-definition"></a>在資源定義上設定 IP 範圍

如果您使用[部署範本](logic-apps-create-deploy-template.md)來自動化您的部署，您可以在資源範本上設定 IP 範圍。  

``` json
{
    "properties": {
        "definition": {
        },
        "parameters": {},
        "accessControl": {
            "triggers": {
                "allowedCallerIpAddresses": [
                    {
                        "addressRange": "192.168.12.0/23"
                    },
                    {
                        "addressRange": "2001:0db8::/64"
                    }
                ]
            }
        }
    },
    "type": "Microsoft.Logic/workflows"
}

```

### <a name="adding-azure-active-directory-oauth-or-other-security"></a>新增 Azure Active Directory、OAuth 或其他安全性

為了在邏輯應用程式最上層新增更多授權通訊協定，[Azure API 管理](https://azure.microsoft.com/services/api-management/)針對任何能夠將邏輯應用程式公開為 API 的端點，提供豐富的監控、安全性、原則和文件。 Azure API 管理可針對邏輯應用程式將公用或私人端點公開，並且可以使用 Azure Active Directory、憑證、OAuth 或其他的安全性標準。 收到要求時，Azure API 管理會將要求轉送至邏輯應用程式 (並在過程中執行任何必要的轉換或限制)。 您可以在邏輯應用程式上使用傳入 IP 範圍設定，讓邏輯應用程式只能透過 API 管理觸發。

## <a name="secure-access-to-manage-or-edit-logic-apps"></a>保護管理或編輯邏輯應用程式的存取

您可以限制邏輯應用程式的管理作業存取，以便只有特定的使用者或群組能夠在資源上執行操作。 邏輯應用程式使用 Azure [角色型存取控制 (RBAC)](../role-based-access-control/role-assignments-portal.md) 功能，並且能夠使用相同的工具進行自訂。  您也可以將一些內建的角色指派給訂用帳戶的成員：

* **邏輯應用程式參與者**：提供邏輯應用程式檢視、編輯和更新的存取權限。  無法移除資源或執行管理作業。
* **邏輯應用程式操作員**：可以檢視邏輯應用程式和執行歷程記錄，以及啟用/停用。  無法編輯或更新定義。

您也可以使用 [Azure 資源鎖定](../azure-resource-manager/resource-group-lock-resources.md)，來防止變更或刪除邏輯應用程式。 這個功能對於防止生產資源遭到修改或刪除非常有用。

## <a name="secure-access-to-contents-of-the-run-history"></a>保護執行歷程記錄內容的存取

您可以將輸入的內容，或是先前所執行的輸出，限制為只有特定 IP 位址範圍可以存取。  

工作流程執行中的所有資料在傳輸及靜止時都會加密。 對執行歷程記錄進行呼叫時，服務會驗證要求、提供要求的連結，並回應輸入並輸出。 此連結會受到保護，以確保只有來自指定 IP 位址範圍的檢視內容要求才會傳回內容。 您可以使用此功能來進行其他存取控制。 您甚至可以指定如 `0.0.0.0` 的 IP 位址，這樣就沒有人可以存取輸入/輸出。 只有具備管理員權限的人才能移除此限制，提供工作流程內容「即時」存取的可能性。

這個設定可在 Azure 入口網站的資源設定中進行設定：

1. 在 Azure 入口網站中，開啟您要新增 IP 位址限制的邏輯應用程式
2. 按一下 [設定] 之下的 [存取控制設定] 功能表項目
3. 針對內容的存取指定 IP 位址範圍清單

#### <a name="setting-ip-ranges-on-the-resource-definition"></a>在資源定義上設定 IP 範圍

如果您使用[部署範本](logic-apps-create-deploy-template.md)來自動化您的部署，您可以在資源範本上設定 IP 範圍。  

``` json
{
    "properties": {
        "definition": {
        },
        "parameters": {},
        "accessControl": {
            "contents": {
                "allowedCallerIpAddresses": [
                    {
                        "addressRange": "192.168.12.0/23"
                    },
                    {
                        "addressRange": "2001:0db8::/64"
                    }
                ]
            }
        }
    },
    "type": "Microsoft.Logic/workflows"
}
```

## <a name="secure-parameters-and-inputs-within-a-workflow"></a>保護工作流程中的參數和輸入

您可能想要將工作流程定義的某些層面參數化，以便跨環境進行部署。 此外，部分參數可能是您不想在編輯工作流程時顯示的安全參數，例如 HTTP 動作的 [Azure Active Directory 驗證](../connectors/connectors-native-http.md#authentication)的用戶端識別碼和用戶端密碼。

### <a name="using-parameters-and-secure-parameters"></a>使用參數和安全參數

為了在執行階段存取資源參數的值，[工作流程定義語言 (英文)](http://aka.ms/logicappsdocs) 提供 `@parameters()` 作業。 此外，您可以[指定資源部署範本中的參數](../azure-resource-manager/resource-group-authoring-templates.md#parameters)。 但是，如果您將參數類型指定為 `securestring`，將無法透過其餘的資源定義傳回此參數，而且在部署之後，將無法藉由檢視資源存取此資源。

> [!NOTE]
> 如果您在要求的標頭或主體中使用參數，可藉由存取執行歷程記錄及連出 HTTP 要求來檢視該參數。 請務必適當地設定您的內容存取原則。
> 授權標頭絕對不會透過輸入或輸出來顯示。 因此，如果該處使用了密碼，就無法擷取該密碼。

#### <a name="resource-deployment-template-with-secrets"></a>含有密碼的資源部署範本

下列範例示範在執行階段中參考 `secret` 安全參數的部署。 在個別的參數檔中，您可以指定 `secret` 的環境值，或是利用 [Azure Resource Manager Key Vault](../azure-resource-manager/resource-manager-keyvault-parameter.md)，在部署階段擷取密碼。

``` json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "secretDeploymentParam": {
         "type": "securestring"
      }
   },
   "variables": {},
   "resources": [ {
      "name": "secret-deploy",
      "type": "Microsoft.Logic/workflows",
      "location": "westus",
      "tags": {
         "displayName": "LogicApp"
      },
      "apiVersion": "2016-06-01",
      "properties": {
         "definition": {
            "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
            "actions": {
               "Call_External_API": {
                  "type": "Http",
                  "inputs": {
                     "headers": {
                        "Authorization": "@parameters('secret')"
                     },
                     "body": "This is the request"
                  },
                  "runAfter": {}
               }
            },
            "parameters": {
               "secret": {
                  "type": "SecureString"
               }
            },
            "triggers": {
               "manual": {
                  "type": "Request",
                  "kind": "Http",
                  "inputs": {
                     "schema": {}
                  }
               }
            },
            "contentVersion": "1.0.0.0",
            "outputs": {}
         },
         "parameters": {
            "secret": {
               "value": "[parameters('secretDeploymentParam')]"
            }
         }
      }
   } ],
   "outputs": {}
}
```

## <a name="secure-access-to-services-receiving-requests-from-a-workflow"></a>保護接受工作流程要求之服務的存取

有許多方法可以協助保護邏輯應用程式需要存取的所有端點。

### <a name="using-authentication-on-outbound-requests"></a>在外送要求上使用驗證

當使用 HTTP、HTTP + Swagger (Open API)，或 Webhook 動作時，您可以將驗證新增到所傳送的要求。 您可以包括基本驗證、憑證驗證或 Azure Active Directory 驗證。 如需如何設定此驗證的詳細資料，請參閱[此文章](../connectors/connectors-native-http.md#authentication)。

### <a name="restricting-access-to-logic-app-ip-addresses"></a>限制對邏輯應用程式 IP 位址的存取

邏輯應用程式的所有呼叫都來自每個區域一組特定的 IP 位址。 您可以加入其他篩選以僅允許接受來自那些指定 IP 位址的要求。 如需這些 IP 位址清單，請參閱[邏輯應用程式的限制和設定](logic-apps-limits-and-config.md#configuration)。

### <a name="on-premises-connectivity"></a>內部部署連線能力

邏輯應用程式提供與數個服務的整合，以提供安全可靠的內部部署通訊。

#### <a name="on-premises-data-gateway"></a>內部部署資料閘道

邏輯應用程式的許多受控連接器提供對內部部署系統 (包括檔案系統、SQL、SharePoint、DB2 等等) 的安全連線能力。 閘道會在加密通道上經過 Azure 服務匯流排轉送來自內部部署來源的資料。 源自閘道代理程式的所有流量都是安全輸出流量。 深入了解[資料閘道的運作方式](logic-apps-gateway-install.md#gateway-cloud-service)。

#### <a name="azure-api-management"></a>Azure API 管理

[Azure API 管理](https://azure.microsoft.com/services/api-management/)具有內部部署連線選項，包括站對站 VPN 和 ExpressRoute 整合，以對內部部署系統提供安全的 Proxy 和通訊。 在邏輯應用程式設計工具中，您可以在工作流程中快速地選取從 Azure API 管理公開的 API，提供對內部部署系統的快速存取。

## <a name="next-steps"></a>後續步驟
[建立部署範本](logic-apps-create-deploy-template.md)  
[例外狀況處理](logic-apps-exception-handling.md)  
[監視邏輯應用程式](logic-apps-monitor-your-logic-apps.md)  
[診斷邏輯應用程式失敗和問題](logic-apps-diagnosing-failures.md)  
