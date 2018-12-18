---
title: 憑證建立方式
description: 在 Key Vault 中建立憑證的方式。
services: key-vault
documentationcenter: ''
author: bryanla
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: e17b4c9b-4ff3-472f-8c9d-d130eb443968
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/09/2018
ms.author: bryanla
ms.openlocfilehash: d2f9327841e0c6193a89df6459b4d8fffb14c05e
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302838"
---
# <a name="certificate-creation-methods"></a>憑證建立方式

 您可以建立 Key Vault (KV) 憑證，或是將之匯入金鑰保存庫。 當建立 KV 憑證時，會在金鑰保存庫內建立私密金鑰，因此絕不會向憑證擁有者公開金鑰。 以下是在 Key Vault 中建立憑證的方式：  

-   **建立自我簽署憑證：** 這會建立公開-私密金鑰組，並與憑證建立關聯。 憑證會由自己擁有的金鑰簽署。  

-    **手動建立新憑證：** 這會建立公開-私密金鑰組，並產生 X.509 憑證簽署要求。 簽署要求可以由您的登錄授權單位或憑證授權單位簽署。 經過簽署的 x509 憑證能與擱置中的金鑰組合併，在 Key Vault 中完成 KV 憑證。 這個方式的步驟比較繁瑣，不過能確實提高安全性，因為私密金鑰僅限在 Key Vault 中建立。 下圖提供相關說明。  

![使用您自己的憑證授權單位建立憑證](media/certificate-authority-1.png)  

以下說明與可對應至上圖中綠字的步驟。

1. 在上圖中，您的應用程式即將建立憑證，內部程序首先會在金鑰保存庫中建立金鑰。
2. Key Vault 將憑證簽署要求 (CSR) 傳回應用程式
3. 您的應用程式將 CSR 傳遞給您選擇的 CA。
4. 您選擇的 CA 回覆 X509 憑證。
5. 您的應用程式合併 CA 回覆的 X509 憑證，完成新憑證的建立。

-   **利用已知簽發者提供者建立憑證：** 若要使用這個方法，您必須完成一次性的簽發者物件建立工作。 在您的金鑰保存庫中建立簽發者物件後，您就可以在 KV 憑證的原則中參考其名稱。 建立這類 KV 憑證的要求會在保存庫中建立金鑰組，並使用所參考簽發者物件中的資訊與簽發者提供者服務通訊，以取得 x509 憑證。 x509 憑證是從簽發者服務所擷取，並與金鑰組合併以完成 KV 憑證建立工作。  

![使用與 Key Vault 合作的憑證授權單位建立憑證](media/certificate-authority-2.png)  

以下說明與可對應至上圖中綠字的步驟。

1. 在上圖中，您的應用程式即將建立憑證，內部程序首先會在金鑰保存庫中建立金鑰。
2. Key Vault 會將 SSL 憑證要求傳送給 CA。
3. 您的應用程式會以迴圈和等待流程來輪詢 Key Vault，直到憑證完成。 Key Vault 收到含 x509 憑證的 CA 回應時，憑證建立工作即完成。
4. CA 會使用 X509 SSL 憑證來回應 Key Vault 的 SSL 憑證要求。
5. 您的新憑證建立工作是透過合併 CA 的 X509 憑證而完成。

## <a name="asynchronous-process"></a>非同步流程
KV 憑證的建立是非同步流程。 此作業會建立 KV 憑證要求，並傳回 http 狀態碼 202 (已接受)。 透過輪詢該作業所建立的擱置中物件，即可追蹤要求的狀態。 擱置中物件的完整 URI 會在 LOCATION 標頭中傳回。  

當 KV 憑證建立要求完成時，擱置中物件的狀態將從「進行中」變更為「已完成」，而且新版本的 KV 憑證隨即會建立。 此版本將成為目前的版本。  

## <a name="first-creation"></a>首次建立
 首次建立 KV 憑證時，系統會以憑證的名稱一併建立可定址金鑰和祕密。 如果名稱已在使用中，作業將會失敗，並且會傳回 http 狀態碼 409 (衝突)。
可定址金鑰和秘密會從 KV 憑證屬性取得其屬性。 利用這種方式建立的可定址金鑰和秘密會標示為受控金鑰，其存留期會由 Key Vault 管理。 受控金鑰和祕密是唯讀的。 注意：如果 KV 憑證過期或已停用，對應的金鑰和祕密將會無法運作。  

 如果這是建立 KV 憑證的第一個作業，您必須具備原則。  也可以將後續的建立作業提供給原則，取代原則資源。 如果您未提供原則，系統便會以服務上的原則資源建立下一個版本的 KV 憑證。 請注意，當系統正在處理下一個版本的建立要求時，目前的 KV 憑證、對應的可定址金鑰和秘密會保持不變。  

## <a name="self-issued-certificate"></a>自我簽發憑證
 若要建立自我簽發憑證，請在憑證原則中將簽發者名稱設定為「自我」，如以下節錄自憑證原則的程式碼片段所示。  

```  
"issuer": {  
       "name": "Self"  
    }  

```  

 如果您未指定簽發者名稱，系統便會將簽發者名稱設定為「不明」。 當簽發者為「不明」時，憑證擁有者必須手動向選擇的簽發者索取 x509 憑證，然後將公開 x509 憑證與金鑰保存庫憑證擱置中物件合併，以完成憑證建立工作。

```  
"issuer": {  
       "name": "Unknown"  
    }  

```  

## <a name="partnered-ca-providers"></a>合作 CA 提供者
您可以手動完成憑證建立工作，也可以使用「自我」簽發者。 Key Vault 也與特定簽發者提供者合作，藉此簡化憑證的建立工作。 您可以向這些合作夥伴簽發者提供者訂購以下類型的憑證。  

|提供者|憑證類型|  
|--------------|----------------------|  
|DigiCert|Key Vault 透過 DigiCert 提供 OV 或 EV SSL 憑證|
|GlobalCert|Key Vault 透過 GlobalSign 提供 OV 或 EV SSL 憑證|

 憑證簽發者是 Azure Key Vault (KV) 中以 CertificateIssuer 資源表示的實體。 它用來提供 KV 憑證來源相關資訊；簽發者名稱、提供者、認證和其他系統管理詳細資訊。

請注意，向簽發者提供者訂購憑證時，對方可能會根據憑證類型來決定要接受或覆寫 x509 憑證延伸和憑證有效期間。  

 授權：須有憑證/建立權限。

 ## <a name="see-also"></a>另請參閱
 - [關於金鑰、祕密與憑證](about-keys-secrets-and-certificates.md)
 - [監視和管理憑證建立](create-certificate-scenarios.md)
