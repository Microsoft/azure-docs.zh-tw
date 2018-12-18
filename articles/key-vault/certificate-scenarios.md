---
title: 開始使用 Key Vault 憑證
description: 下列情節概述 Key Vault 憑證管理服務的數個主要用法 (包括在金鑰保存庫中建立第一個憑證所需的其他步驟)。
services: key-vault
documentationcenter: ''
author: bryanla
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: a788b958-3acb-4bb6-9c94-4776852aeea1
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/09/2018
ms.author: bryanla
ms.openlocfilehash: 60920aaa62d8472eaca8f929c62ce1c0af08f9a4
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49385629"
---
# <a name="get-started-with-key-vault-certificates"></a>開始使用 Key Vault 憑證
下列情節概述 Key Vault 憑證管理服務的數個主要用法 (包括在金鑰保存庫中建立第一個憑證所需的其他步驟)。

概述下列資訊：
- 建立第一個 Key Vault 憑證
- 使用與 Key Vault 合作的憑證授權單位建立憑證
- 使用未與 Key Vault 合作的憑證授權單位建立憑證
- 匯入憑證

## <a name="certificates-are-complex-objects"></a>憑證是複雜物件
憑證是由連結在一起的三個相互關聯資源組成 Key Vault 憑證：憑證中繼資料、金鑰和祕密。


![憑證是複雜的](media/azure-key-vault.png)


## <a name="creating-your-first-key-vault-certificate"></a>建立第一個 Key Vault 憑證  
 必須順利完成必備步驟 1 和 2，而且必須要有此使用者/組織的金鑰保存庫，才能在 Key Vault (KV) 中建立憑證。  

**步驟 1** - 憑證授權單位 (CA) 提供者  
-   以 IT 管理員、PKI 管理員或任何使用 CA 管理帳戶之人的身分上線，針對指定的公司 (例如 Contoso) 是使用 Key Vault 憑證的先決條件。  
    下列 CA 是使用 Key Vault 的目前合作提供者：  
    -   DigiCert - Key Vault 透過 DigiCert 提供 OV SSL 憑證。  
    -   GlobalSign - Key Vault 透過 GlobalSign 提供 OV SSL 憑證  
    -   WoSign - 根據客戶在 WoSign 入口網站上使用其 WoSign 帳戶所設定的設定，Key Vault 會透過 WoSign 提供 OV SSL 或 EV SSL 憑證。  

**Step 2** - CA 提供者的帳戶管理員會建立 Key Vault 用來透過 Key Vault 註冊、更新和使用 SSL 憑證的認證。

**步驟 3** - Contoso 管理員與擁有憑證的 Contoso 員工 (Key Vault 使用者) (根據 CA 而定) 可以向管理員取得憑證或者直接向 CD 從帳戶取得憑證。  

-   [設定憑證簽發者](/rest/api/keyvault/setcertificateissuer)資源，開始將認證作業新增至金鑰保存庫。 憑證簽發者是 Azure Key Vault (KV) 中以 CertificateIssuer 資源表示的實體。 它用來提供 KV 憑證來源相關資訊；簽發者名稱、提供者、認證和其他系統管理詳細資訊。
    -   例如 MyDigiCertIssuer  
        -   提供者  
        -   認證 – CA 帳戶認證。 每個 CA 都有自己的特定資料。  

     如需使用 CA 提供者建立帳戶的詳細資訊，請參閱 [Key Vault 部落格](http://aka.ms/kvcertsblog)上的相關文章。  

**步驟 3.1** - 設定[憑證連絡人](/rest/api/keyvault/setcertificatecontacts)以進行通知。 這是 Key Vault 使用者的連絡人。 Key Vault 不會強制執行此步驟。  

附註 - 透過步驟 3.1 的這個流程是一次性作業。  

## <a name="creating-a-certificate-with-a-ca-partnered-with-key-vault"></a>使用與 Key Vault 合作的 CA 建立憑證

![使用與 Key Vault 合作的憑證授權單位建立憑證](media/certificate-authority-2.png)

**步驟 4** - 下列描述對應至上圖中的綠色編號步驟。  
  (1) - 在上圖中，您的應用程式即將建立憑證，內部流程始於在金鑰保存庫中建立金鑰。  
  (2) - Key Vault 會將 SSL 憑證要求傳送給 CA。  
  (3) - 您的應用程式會以迴圈和等待流程來輪詢 Key Vault，直到憑證完成。 Key Vault 收到 x509 憑證的 CA 回應時，憑證建立即完成。  
  (4) - CA 會使用 X509 SSL 憑證來回應 Key Vault 的 SSL 憑證要求。  
  (5) - 您的新憑證建立是透過合併 CA 的 X509 憑證而完成。  

  Key Vault 使用者 – 指定原則來建立憑證

  -   請視需要重複  
  -   原則條件約束  
      -   X509 屬性  
      -   金鑰屬性  
      -   提供者參考 - > 例如 MyDigiCertIssure  
      -   更新資訊 - > 例如 到期之前的 90 天  

  - 憑證建立流程通常為非同步流程，並且包含輪詢金鑰保存庫中的建立憑證作業狀態。  
[取得憑證作業](https://docs.microsoft.com/rest/api/keyvault/getcertificateoperation)  
      -   狀態：已完成、失敗但有錯誤資訊，或已取消  
      -   因為延遲建立，所以可以起始取消作業。 取消不一定會有作用。  

## <a name="import-a-certificate"></a>匯入憑證  
 或者，可以將憑證匯入至 Key Vault – PFX 或 PEM。  

 如需 PEM 格式的詳細資訊，請參閱[關於金鑰、祕密與憑證](about-keys-secrets-and-certificates.md)的憑證一節。  

 匯入憑證 – 需要磁碟上有 PEM 或 PFX，並且具有私密金鑰。 
-   您必須指定：保存庫名稱和憑證名稱 (原則是選用項目)

-   PEM/PFX 檔案包含 KV 可剖析並用來填入憑證原則的屬性。 如果已指定憑證原則，則 KV 會嘗試比對 PFX/PEM 檔案中的資料。  

-   進行最後一次匯入之後，後續作業將會使用新原則 (新版本)。  

-   如果沒有進一步作業，則 Key Vault 要做的第一件事是傳送到期通知。 

-   此外，使用者可以編輯原則，而此原則是在匯入時作用，但包含匯入時未指定任何資訊的預設值。 例如 無簽發者資訊  

## <a name="creating-a-certificate-with-a-ca-not-partnered-with-key-vault"></a>使用未與 Key Vault 合作的 CA 建立憑證  
 這種方法允許與 Key Vault 合作提供者以外的其他 CA 合作，這表示您的組織可以與它選擇的 CA 合作。  

![利用您自己的憑證授權單位建立憑證](media/certificate-authority-1.png)  

 下列步驟描述對應至上圖中的綠色文字步驟。  

  (1) - 在上圖中，您的應用程式即將建立憑證，內部流程始於在金鑰保存庫中建立金鑰。  

  (2) - Key Vault 將憑證簽署要求 (CSR) 傳回應用程式。  

  (3) - 應用程式將 CSR 傳遞給您選擇的 CA。  

  (4) - 您選擇的 CA 回覆 X509 憑證。  

  (5) - 應用程式合併 CA 的 X509 憑證，來完成新憑證建立。

## <a name="see-also"></a>另請參閱

- [關於金鑰、密碼與憑證](about-keys-secrets-and-certificates.md)
