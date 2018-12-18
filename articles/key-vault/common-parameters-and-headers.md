---
title: 通用參數和標頭
description: 參數和標頭通用於針對相關 Key Vault 資源執行的所有作業。
services: key-vault
documentationcenter: ''
author: bryanla
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: a715d13ca9-d6e8-4e54-ac5e-0ed9400fb15b15d13ca9-d6e8-4e54-ac5e-0ed9400fb15b
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/09/2018
ms.author: bryanla
ms.openlocfilehash: a319dc670b5b1dab163b2d3aa623fc4fb9ce1c3a
ms.sourcegitcommit: 0fcd6e1d03e1df505cf6cb9e6069dc674e1de0be
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42145072"
---
# <a name="common-parameters-and-headers"></a>通用參數和標頭

下列資訊通用於針對相關 Key Vault 資源執行的所有作業：

- 將 `{api-version}` 取代為 URI 中的 api-version。
- 將 `{subscription-id}` 取代為 URI 中的訂用帳戶識別碼
- 將 `{resource-group-name}` 取代為資源群組。 如需詳細資訊，請參閱＜使用資源群組管理 Azure 資源＞。
- 將 `{vault-name}` 取代為 URI 中您的金鑰保存庫名稱。
- 將 Content-Type 標頭設定為 application/json。
- 將 Authorization 標頭設定為您從 Azure Active Directory (AAD) 取得的 JSON Web 權杖。 如需詳細資訊，請參閱[驗證 Azure Resource Manager 要求](authentication-requests-and-responses.md)。

## <a name="common-error-response"></a>通用錯誤回應
服務將使用 HTTP 狀態碼指出成功或失敗。 此外，失敗包含下列格式的回應：

   {  
     "error": {  
     "code": "BadRequest",  
     "message": "金鑰保存庫 SKU 無效。"  
     }  
   }  

|元素名稱 | 類型 | 說明 |
|---|---|---|
| code | 字串 | 所發生錯誤的類型。|
| Message | 字串 | 造成錯誤之原因的描述。 |



## <a name="see-also"></a>另請參閱
 [Azure Key Vault REST API 參考](/rest/api/keyvault/)
