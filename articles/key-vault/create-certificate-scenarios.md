---
title: 監視和管理憑證建立
description: 情節，示範使用 Key Vault 建立和監視憑證建立流程並與憑證建立流程互動的某範圍選項。
services: key-vault
documentationcenter: ''
author: bryanla
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 0d0995aa-b60d-4811-be12-ba0a45390197
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/09/2018
ms.author: bryanla
ms.openlocfilehash: 0e24bd56123279a24a72b9b52d8cb51e2a3a5eae
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44300625"
---
# <a name="monitor-and-manage-certificate-creation"></a>監視和管理憑證建立
適用對象：Azure  

下列 

本文中所述的情節/作業如下：

- 使用支援的簽發者來要求 KV 憑證
- 取得擱置中要求 - 要求狀態為「進行中」
- 取得擱置中要求 - 要求狀態為「完成」
- 取得擱置中要求 - 擱置中要求狀態為「已取消」或「失敗」
- 取得擱置中要求 - 擱置中要求狀態為「已刪除」或「已覆寫」
- 在存在擱置中要求時建立 (或匯入) - 狀態為「進行中」
- 在使用簽發者建立擱置中要求時合併 (例如，DigiCert)
- 在擱置中要求狀態為「進行中」時要求取消
- 刪除擱置中要求物件
- 手動建立 KV 憑證
- 在建立擱置中要求時合併 - 手動憑證建立

## <a name="request-a-kv-certificate-with-a-supported-issuer"></a>使用支援的簽發者來要求 KV 憑證 

|方法|要求 URI|  
|------------|-----------------|  
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|  

下列範例需要簽發者提供者為 DigiCert 之金鑰保存庫中已有名為 "mydigicert" 的物件。 憑證簽發者是 Azure Key Vault (KV) 中以 CertificateIssuer 資源表示的實體。 它用來提供 KV 憑證來源相關資訊；簽發者名稱、提供者、認證和其他系統管理詳細資訊。  

### <a name="request"></a>要求  

```json
{  
  "policy": {  
    "x509_props": {  
      "subject": "CN=MyCertSubject1"  
    },  
  "issuer": {  
       "name": "mydigicert",  
    "cty": "OV-SSL",  
    }  
  }  
}  

```  

### <a name="response"></a>Response  

```  
StatusCode: 202, ReasonPhrase: 'Accepted'  
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"  
{  
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",  
  "issuer": {  
    "name": "mydigicert"  
  },  
  "csr": "MIICq......DD5Lp5cqXg==",  
  "cancellation_requested": false,  
  "status": "InProgress",  
  "status_details": "Pending certificate created. Certificate request is in progress. This may take some time based on the issuer provider. Please check again later",  
  "request_id": "a76827a18b63421c917da80f28e9913d"  
}  

```  

## <a name="get-pending-request---request-status-is-inprogress"></a>取得擱置中要求 - 要求狀態為「進行中」

|方法|要求 URI|  
|------------|-----------------|  
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|  

### <a name="request"></a>要求  
 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`  

 或  

 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`  

> [!NOTE]
>  如果查詢中指定 *request_id*，則其行為就像篩選。 如果查詢與擱置中物件內所指定的 *request_id* 不同，則會傳回 HTTP 狀態碼 404。  

### <a name="response"></a>Response  

```  
StatusCode: 200, ReasonPhrase: 'OK'  
{  
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",  
  "issuer": {  
    "name": "{issuer-name}"  
  },  
  "csr": "MIICq......DD5Lp5cqXg==",  
  "cancellation_requested": false,  
  "status": "inProgress",  
  "status_details": "…",  
  "request_id": "a76827a18b63421c917da80f28e9913d"  
}  

```  

## <a name="get-pending-request---request-status-is-complete"></a>取得擱置中要求 - 要求狀態為「完成」

### <a name="request"></a>要求  

|方法|要求 URI|  
|------------|-----------------|  
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|  

 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`  

 或  

 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`  

### <a name="response"></a>Response  

```  
StatusCode: 200, ReasonPhrase: 'OK'  
{  
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",  
  "issuer": {  
    "name": "{issuer-name}"  
  },  
  "csr": "MIICq......DD5Lp5cqXg==",  
  "cancellation_requested": false,  
  "status": "completed",  
  "request_id": "a76827a18b63421c917da80f28e9913d",  
   "target": “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"  
}  

```  

## <a name="get-pending-request---pending-request-status-is-canceled-or-failed"></a>取得擱置中要求 - 擱置中要求狀態為「已取消」或「失敗」  

### <a name="request"></a>要求  

|方法|要求 URI|  
|------------|-----------------|  
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|  

 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`  

 或  

 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`  

### <a name="response"></a>Response  

```  
StatusCode: 200, ReasonPhrase: 'OK'  
{  
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",  
  "issuer": {  
    "name": "{issuer-name}"  
  },  
  "csr": "MIICq......DD5Lp5cqXg==",  
  "cancellation_requested": false,  
  "status": "failed",  
  "status_details": "",  
  "request_id": "a76827a18b63421c917da80f28e9913d",  
   "error":  
    {  
        "code": "<errorcode>",    
        "message": "<message>"  
    }  
}  

```  

> [!NOTE]
> 根據簽發者或使用者錯誤，*errorcode* 的值分別可以是「憑證簽發者錯誤」或「已拒絕要求」。  

## <a name="get-pending-request---pending-request-status-is-deleted-or-overwritten"></a>取得擱置中要求 - 擱置中要求狀態為「已刪除」或「已覆寫」  
 建立/匯入作業在其狀態不是「進行中」時，可以刪除或覆寫擱置中物件。

|方法|要求 URI|  
|------------|-----------------|  
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|  

### <a name="request"></a>要求  
 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`  

 或  

 GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`  

### <a name="response"></a>Response  

```  
StatusCode: 404, ReasonPhrase: 'Not Found'  
{  
  "error":  
    {  
         "code": "PendingCertificateNotFound",  
        "message": "…"  
    }  
}  

```  

## <a name="create-or-import-when-pending-request-exists---status-is-inprogress"></a>在存在擱置中要求時建立 (或匯入) - 狀態為「進行中」
 擱置中物件有四個可能狀態：「進行中」、「已取消」、「失敗」或「已完成」。

 擱置中要求的狀態為「進行中」時，建立 (和匯入) 作業將會失敗，HTTP 狀態碼為 409 (衝突)。  

 修正衝突：  

 - 如果手動建立憑證，則可以對擱置中物件執行合併或刪除，以完成 KV 憑證。  

 - 如果正在使用簽發者建立憑證，則您可以等到憑證完成、失敗或取消。 或者，您可以刪除擱置中物件。

> [!NOTE]
> 刪除擱置中物件不一定會向提供者取消 x509 憑證要求。  

|方法|要求 URI|  
|------------|-----------------|  
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|  

### <a name="request"></a>要求  

```json
{  
  "policy": {  
    "x509_props": {  
      "subject": "CN=MyCertSubject1"  
    },  
  "issuer": {  
       "name": "mydigicert"  
    }  
  }  
}  

```  

### <a name="response"></a>Response  

```  
StatusCode: 409, ReasonPhrase: 'Conflict'  
{  
  "error":  
    {  
        "code": "Forbidden",  
        "message": "A new key vault certificate can not be created or imported while a pending key vault certificate's status is inProgress."  
    }  
}  

```  

## <a name="merge-when-pending-request-is-created-with-an-issuer"></a>在使用簽發者建立擱置中要求時合併
 在使用簽發者建立擱置中要求時不允許合併，但在其狀態為「進行中」時允許合併。 

 如果建立 x509 憑證的要求因某個原因而失敗或取消，以及可以透過頻外方式擷取 x509 憑證，則可以執行合併作業來完成 KV 憑證。  

|方法|要求 URI|  
|------------|-----------------|  
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|  

### <a name="request"></a>要求  

```json
{  
  "x5c": [  "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8="  
  ]  
}  

```  

### <a name="response"></a>Response  

```json
StatusCode: 403, ReasonPhrase: 'Forbidden'  
{  
  "error":  
    {  
       "code": "Forbidden",  
       "message": "Merge is forbidden on pending object created with issuer : <issuer-name> while it is in progess."  
    }  
}  

```  

## <a name="request-a-cancellation-while-the-pending-request-status-is-inprogress"></a>在擱置中要求狀態為「進行中」時要求取消  
 只能要求取消。  不一定可以取消要求。 如果要求不是「進行中」，則會傳回「HTTP 狀態 400 (不正確的要求)」。  

|方法|要求 URI|  
|------------|-----------------|  
|PATCH|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|  

### <a name="request"></a>要求  
 PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`  

 或  

 PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`  

```json
{  
  "cancellation_requested": true  
}  

```  

### <a name="response"></a>Response  

```  
StatusCode: 200, ReasonPhrase: 'OK'  
{  
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",  
  "issuer": {  
    "name": "{issuer-name}"  
  },  
  "csr": "MIICq......DD5Lp5cqXg==",  
  "cancellation_requested": true,  
  "status": "inProgress",  
  "status_details": "…",  
  "request_id": "a76827a18b63421c917da80f28e9913d"  
}  

```  

## <a name="delete-a-pending-request-object"></a>刪除擱置中要求物件  

> [!NOTE]
> 刪除擱置中物件不一定會向提供者取消 x509 憑證要求。  

|方法|要求 URI|  
|------------|-----------------|  
|刪除|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|  

### <a name="request"></a>要求  
 DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`  

 或  

 DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`  

### <a name="response"></a>Response  

```  
StatusCode: 200, ReasonPhrase: 'OK'  
{  
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",  
  "issuer": {  
    "name": "{issuer-name}"  
  },  
  "csr": "MIICq......DD5Lp5cqXg==",  
  "cancellation_requested": false,  
  "status": "inProgress",  
  "request_id": "a76827a18b63421c917da80f28e9913d",  
}  
```  

## <a name="create-a-kv-certificate-manually"></a>手動建立 KV 憑證  
 您可以透過手動建立流程，建立利用所選擇的 CA 發出的憑證。 將簽發者名稱設定為「未知」，或未指定簽發者欄位。  

|方法|要求 URI|  
|------------|-----------------|  
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|  

### <a name="request"></a>要求  

```json
{  
  "policy": {  
    "x509_props": {  
      "subject": "CN=MyCertSubject1"  
    }  
  "issuer": {  
       "name": "Unknown"  
    }  
  }  
}  

```  

### <a name="response"></a>Response  

```  
StatusCode: 202, ReasonPhrase: 'Accepted'  
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"  
{  
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",  
  "issuer": {  
    "name": "Unknown"  
  },  
  "csr": "MIICq......DD5Lp5cqXg==",  
  "status": "inProgress",  
  "status_details": "Pending certificate created. Please Perform Merge to complete the request.",  
  "request_id": "a76827a18b63421c917da80f28e9913d"  
}  

```  

## <a name="merge-when-a-pending-request-is-created---manual-certificate-creation"></a>在建立擱置中要求時合併 - 手動憑證建立  

|方法|要求 URI|  
|------------|-----------------|  
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|  

### <a name="request"></a>要求  

```json
{  
  "x5c": [  "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8="  
  ]  
}  

```  

|元素名稱|必要|類型|版本|說明|  
|------------------|--------------|----------|-------------|-----------------|  
|x5c|是|array|\<簡介版本 >|作為 base64 字串陣列的 X509 憑證鏈結。|  

### <a name="response"></a>Response  

```  
StatusCode: 201, ReasonPhrase: 'Created'  
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"  
{  
"id": "https mykeyvault.vault.azure.net/certificates/mycert1/f366e1a9dd774288ad84a45a5f620352",  
    "kid": "https:// mykeyvault.vault.azure.net/keys/mycert1/f366e1a9dd774288ad84a45a5f620352",  
    "sid": " mykeyvault.vault.azure.net/secrets/mycert1/f366e1a9dd774288ad84a45a5f620352",  
    "cer": "……de34534……",  
    "x5t": "n14q2wbvyXr71Pcb58NivuiwJKk",  
    "attributes": {  
        "enabled": true,  
        "exp": 1530394215,  
        "nbf": 1435699215,  
        "created": 1435699919,  
        "updated": 1435699919  
    },  
    "pending": {  
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/pending"  
    },  
    "policy": {  
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/policy",  
        "key_props": {  
            "exportable": false,  
            "kty": "RSA",  
            "key_size": 2048,  
            "reuse_key": false  
        },  
        "secret_props": {  
            "contentType": "application/x-pkcs12"  
        },  
        "x509_props": {  
            "subject": "CN=Mycert1",  
            "ekus": ["1.3.6.1.5.5.7.3.1", "1.3.6.1.5.5.7.3.2"],  
                       "validity_months":12  
        },  
        "lifetime_actions": [{  
            "trigger": {  
                "lifetime_percentage": 80  
            },  
            "action": {  
                "action_type": "EmailContacts"  
            }  
        }],  
        "issuer": {  
            "name": "Unknown"  
        },  
        "attributes": {  
            "enabled": true,  
            "created": 1435699811,  
            "updated": 1435699811  
        }  
    }  
}  

```

## <a name="see-also"></a>另請參閱
- [關於金鑰、密碼與憑證](about-keys-secrets-and-certificates.md)
