---
title: 關於金鑰、祕密與憑證
description: REST 介面概觀與 KV 開發人員詳細資料
services: key-vault
documentationcenter: ''
author: BryanLa
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: abd1b743-1d58-413f-afc1-d08ebf93828a
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/14/2018
ms.author: bryanla
ms.openlocfilehash: 60b8689e602f0716cdd1a383ab6568979a37e7c0
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302940"
---
# <a name="about-keys-secrets-and-certificates"></a>關於金鑰、祕密與憑證
Azure Key Vault 可讓使用者在 Microsoft Azure 環境中儲存和使用密碼編譯金鑰。 Key Vault 支援多種金鑰類型和演算法，並可針對高價值金鑰使用硬體安全模組 (HSM)。 此外，Key Vault 可讓使用者安全地儲存秘密。 秘密是有大小限制的八位元物件，其中不包含特定語意。 Key Vault 也支援建立於金鑰和秘密之上，且會新增自動更新功能的憑證。

如需有關 Azure Key Vault 的一般詳細資訊，請參閱 [什麼是 Azure Key Vault？](/azure/key-vault/key-vault-whatis)

**Key Vault 一般詳細資料**

-   [支援標準](#BKMK_Standards)
-   [資料類型](#BKMK_DataTypes)  
-   [物件、識別碼和版本控制](#BKMK_ObjId)  

**關於金鑰**

-   [金鑰與金鑰類型](#BKMK_KeyTypes)  
-   [RSA 演算法](#BKMK_RSAAlgorithms)  
-   [RSA-HSM 演算法](#BKMK_RSA-HSMAlgorithms)  
-   [密碼編譯保護](#BKMK_Cryptographic)
-   [金鑰作業](#BKMK_KeyOperations)  
-   [金鑰屬性](#BKMK_KeyAttributes)  
-   [金鑰標記](#BKMK_Keytags)  

**關於秘密** 

-   [使用秘密](#BKMK_WorkingWithSecrets)  
-   [秘密屬性](#BKMK_SecretAttrs)  
-   [秘密標記](#BKMK_SecretTags)  
-   [秘密存取控制](#BKMK_SecretAccessControl)  

**關於憑證**

-   [憑證的構成要素](#BKMK_CompositionOfCertificate)  
-   [憑證屬性與標記](#BKMK_CertificateAttributesAndTags)  
-   [憑證原則](#BKMK_CertificatePolicy)  
-   [憑證簽發者](#BKMK_CertificateIssuer)  
-   [憑證連絡人](#BKMK_CertificateContacts)  
-   [憑證存取控制](#BKMK_CertificateAccessControl)  

--

## <a name="key-vault-general-details"></a>Key Vault 一般詳細資料

下列各節會提供適用於 Azure Key Vault 服務實作的一般資訊。

###  <a name="BKMK_Standards"></a> 支援標準

JavaScript 物件標記法 (JSON) 和 JavaScript 物件簽章與加密 (JOSE) 規格是重要的背景資訊。  

-   [JSON Web 金鑰 (JWK)](http://tools.ietf.org/html/draft-ietf-jose-json-web-key)  
-   [JSON Web 加密 (JWE)](http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption)  
-   [JSON Web 演算法 (JWA)](http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms)  
-   [JSON Web 簽章 (JWS)](http://tools.ietf.org/html/draft-ietf-jose-json-web-signature)  

### <a name="BKMK_DataTypes"></a> 資料類型

請參閱 [JOSE 規格](#BKMK_Standards)，以了解金鑰、加密及簽章的相關資料類型。  

-   **演算法** - 支援的金鑰作業演算法，例如 RSA1_5  
-   **加密文字值** - 加密文字的八位元組，使用 Base64URL 編碼  
-   **摘要值** - 雜湊演算法的輸出，使用 Base64URL 編碼  
-   **金鑰類型** - 其中一種支援的金鑰類型，例如 RSA。  
-   **純文字值** - 純文字的八位元組，使用 Base64URL 編碼  
-   **簽章值** - 簽章演算法的輸出，使用 Base64URL 編碼  
-   **base64URL** - Base64URL [RFC4648] 編碼的二進位值  
-   **布林值** - True 或 False  
-   **身分識別** - Azure Active Directory (AAD) 中的身分識別。  
-   **IntDate** - JSON 十進位值，代表從 1970-01-01T0:0:0Z UTC 到指定 UTC 日期/時間的 的秒數。 如需有關一般日期/時間和特定 UTC 時間的詳細資料，請參閱 [RFC3339]。  

###  <a name="BKMK_ObjId"></a> 物件、識別碼和版本控制

儲存在 Azure Key Vault 中的物件會在每次建立物件的新執行個體時保留版本，而且每個版本都有唯一識別碼和 URL。 第一次建立的物件會獲得唯一的版本識別碼，並標示為物件的目前版本。 以相同物件名稱建立新的執行個體，會使新物件獲得唯一版本識別碼，並使其成為目前的版本。  

您可以使用目前的識別碼或版本專屬的識別碼，來處理 Azure Key Vault 中的物件。 例如，假設某個金鑰的名稱為 "MasterKey"，若以目前識別碼來執行作業，則系統就會使用最新的可用版本。 若以版本專屬的識別碼來執行作業，則系統就會使用該特定版本的物件。  

Azure Key Vault 內的物件會使用 URL 來進行唯一識別，因此，系統中不會有相同 URL (無論地理位置在哪)。 物件的完整 URL 稱為「物件識別碼」，其包含可識別 Key Vault 的首碼部分、物件類型、使用者提供的物件名稱和物件版本。 物件名稱不區分大小寫且無法變更。 不包含物件版本的識別碼稱為「基底識別碼」。  

如需詳細資訊，請參閱[驗證、要求與回應](authentication-requests-and-responses.md)

物件識別碼具有下列一般格式：  

`https://{keyvault-name}.vault.azure.net/{object-type}/{object-name}/{object-version}`  

其中：  

|||  
|-|-|  
|`keyvault-name`|Microsoft Azure Key Vault 服務中的金鑰保存庫名稱。<br /><br /> Key Vault 名稱由使用者選取，且具有全域唯一性。<br /><br /> Key Vault 名稱必須是包含 3-24 個字元的字串，而且只可使用 (0-9、a-z、A-Z 及 -)。|  
|`object-type`|物件類型為「金鑰」或「秘密」。|  
|`object-name`|`object-name` 是由使用者提供的名稱，而且在 Key Vault 中必須是唯一的。 名稱必須是包含 1-127 個字元的字串，而且只可使用 0-9、a-z、A-Z 及 -。|  
|`object-version`|`object-version` 是由系統產生並包含 32 個字元的字串識別碼，可選擇性地用來當作物件的唯一版本。|  

## <a name="key-vault-keys"></a>Key Vault 金鑰

###  <a name="BKMK_KeyTypes"></a>金鑰與金鑰類型

Azure Key Vault 中的密碼編譯金鑰會表示為 JSON Web 金鑰 [JWK] 物件。 您也可以藉由擴充基本 JWK/JWA 規格，讓金鑰類型專屬於 Azure Key Vault 實作，例如將金鑰匯入至 Azure Key Vault 時，可以使用 HSM 廠商 (Thales) 專用的封裝來啟用金鑰的安全傳輸，如此一來，這些金鑰只可用於 Azure Key Vault HSM。  

- **「軟式」金鑰**：由 Key Vault 在軟體中進行處理，但會在待用期間使用 HSM 中的系統金鑰來加密儲存。 用戶端可能會匯入現有的 RSA 或 EC 金鑰，或要求 Azure Key Vault 產生此金鑰。
- **「硬式」金鑰**：在 HSM (硬體安全模組) 中處理的金鑰。 這些金鑰會在其中一個 Azure Key Vault HSM Security Worlds 中受到保護 (為保有獨立性，每個地理位置中會有一個 Security World)。 用戶端若要匯入 RSA 或 EC 金鑰，可以透過軟式形式或從相容的 HSM 裝置匯出 RSA 金鑰，或要求 Azure Key Vault 產生此金鑰。 此金鑰類型會將 T 屬性新增至為執行 HSM 金鑰內容而取得的 JWK。

     如需地理界限的詳細資訊，請參閱 [Microsoft Azure 信任中心](https://azure.microsoft.com/support/trust-center/privacy/)  

Azure Key Vault 只支援 RSA 與橢圓曲線 (EC) 金鑰；未來版本可能會支援其他金鑰類型，例如「對稱」。

-   **EC**：「軟式」橢圓曲線金鑰。
-   **EC-HSM**：「硬式」橢圓曲線金鑰。
-   **RSA**：「軟式」RSA 金鑰。
-   **RSA-HSM**：「硬式」RSA 金鑰。

Azure Key Vault 支援大小為 2048、3072 與 4096 的 RSA 金鑰，以及類型為 P-256、P-384、P-521 與 P-256K 的橢圓曲線金鑰。

### <a name="BKMK_Cryptographic"></a>密碼編譯保護

Azure Key Vault 使用的密碼編譯模組 (HSM 或軟體) 皆經過 FIPS 驗證。 您不需要在 FIPS 模式中執行任何特殊動作。 如果您在**建立**或**匯入**金鑰時是使用 HSM 進行保護，則這些金鑰保證會在以 FIPS 140-2 層級 2 或更高層級驗證的 HSM 內進行處理。 如果您在**建立**或**匯入**金鑰時是使用軟體進行保護，則這些金鑰會在以 FIPS 140-2 層級 1 或更高層級驗證的加密模組內進行處理。 如需詳細資訊，請參閱[金鑰與金鑰類型](#BKMK_KeyTypes)。

###  <a name="BKMK_ECAlgorithms"></a> EC 演算法
 Azure Key Vault 中的 EC 與 EC-HSM 金鑰支援下列演算法識別碼。 

#### <a name="signverify"></a>SIGN/VERIFY

-   **ES256** - 適用於 SHA-256 摘要的 ECDSA 與使用曲線 P-256 建立的金鑰。 [RFC7518] 說明此演算法。
-   **ES256K** - 適用於 SHA-256 摘要的 ECDSA 與使用曲線 P-256K 建立的金鑰。 此演算法尚待標準化。
-   **ES384** - 適用於 SHA-384 摘要的 ECDSA 與使用曲線 P-384 建立的金鑰。 [RFC7518] 說明此演算法。
-   **ES512** - 適用於 SHA-512 摘要的 ECDSA 與使用曲線 P-521 建立的金鑰。 [RFC7518] 說明此演算法。

###  <a name="BKMK_RSAAlgorithms"></a>RSA 演算法  
 Azure Key Vault 中的 RSA 與 RSA-HSM 金鑰支援下列演算法識別碼。  

#### <a name="wrapkeyunwrapkey-encryptdecrypt"></a>包裝金鑰/解除包裝金鑰、加密/解密

-   **RSA1_5** - RSAES-PKCS1-V1_5 [RFC3447] 金鑰加密  
-   **RSA-OAEP** - RSAES 使用「最佳非對稱加密填補 (OAEP)」[RFC3447]，並利用 A.2.1 節中 RFC 3447 所指定的預設參數。 這些預設參數會使用 SHA-1 的雜湊函數及搭配 SHA-1 的 MGF1 遮罩產生函數。  

#### <a name="signverify"></a>SIGN/VERIFY

-   **RS256** - 使用 SHA-256 的 RSASSA-PKCS-v1_5。 提供摘要值的應用程式必須使用 SHA-256 計算，而且長度必須是 32 個位元組。  
-   **RS384** - 使用 SHA-384 的 RSASSA-PKCS-v1_5。 提供摘要值的應用程式必須使用 SHA-384 計算，而且長度必須是 48 個位元組。  
-   **RS512** - 使用 SHA-512 的 RSASSA-PKCS-v1_5。 提供摘要值的應用程式必須使用 SHA-512 計算，而且長度必須是 64 個位元組。  
-   **RSNULL** - 請參閱啟用特定 TLS 案例的特殊使用案例 [RFC2437]。  

###  <a name="BKMK_KeyOperations"></a>金鑰作業

Azure Key Vault 支援以下關於金鑰物件的作業：  

-   **建立**：可讓用戶端在 Azure Key Vault 中建立金鑰。 金鑰值會由 Azure Key Vault 產生並儲存，但不會發行給用戶端。 Azure Key Vault 中可建立非對稱 (未來還有橢圓曲線和對稱) 金鑰。  
-   **匯入**：可讓用戶端將現有金鑰匯入至 Azure Key Vault。 非對稱 (未來還有橢圓曲線和對稱) 金鑰可使用 JWK 概念中的多種不同封裝方法來匯入至 Azure Key Vault。  
-   **更新**：針對先前已儲存在 Azure Key Vault 中的金鑰，具有足夠權限的用戶端可修改與此金鑰相關聯的中繼資料 (金鑰屬性)。  
-   **刪除**：可讓具有足夠權限的用戶端從 Azure Key Vault 中刪除金鑰。  
-   **清單**：可讓用戶端列出指定 Azure Key Vault 中的所有金鑰。  
-   **列出版本**：可讓用戶端從指定的 Azure Key Vault 中，列出指定金鑰的所有版本。  
-   **取得**：可讓用戶端擷取 Azure Key Vault 中指定金鑰的公開部分。  
-   **備份**：以受保護的形式匯出金鑰。  
-   **還原**：匯入先前備份的金鑰。  

如需詳細資訊，請參閱 [Key Vault REST API 參考中的金鑰作業](/rest/api/keyvault)。  

一旦在 Azure Key Vault 中建立金鑰之後，即可使用該金鑰執行下列加密編譯作業：  

-   **簽章與驗證**：嚴格來說，這項作業是「簽署雜湊」或「驗證雜湊」，因為 Azure Key Vault 不支援在建立簽章時執行內容雜湊。 應用程式應對要在本機上簽署的資料進行雜湊，然後要求 Azure Key Vault 簽署此雜湊資料。 為方便起見，無法存取 [公開] 金鑰內容的應用程式可支援對已簽署的雜湊資料進行驗證；為達到最佳應用程式效能，我們建議您在本機上執行驗證作業。  
-   **金鑰加密 / 包裝**：儲存在 Azure Key Vault 中的金鑰可能會用來保護其他金鑰，通常是對稱內容加密金鑰 (CEK)。 如果 Azure Key Vault 中的金鑰是非對稱，則會使用金鑰加密，例如 RSA-OAEP 和包裝金鑰/解除包裝金鑰作業等同於加密/解密。 如果 Azure Key Vault 中的金鑰是對稱的，則使用金鑰包裝。例如 AES-KW。 為方便起見，無法存取 [公開] 金鑰內容的應用程式可支援包裝金鑰作業；為達到最佳應用程式效能，建議您在本機執行包裝金鑰作業。  
-   **加密和解密**：儲存在 Azure Key Vault 中的金鑰可用來加密或解密單一資料區塊，其大小取決於金鑰類型和所選取的加密演算法。 為方便起見，無法存取 [公開] 金鑰內容的應用程式可使用「加密」作業；為達到最佳應用程式效能，建議您在本機執行加密作業。  

使用非對稱金鑰時，包裝金鑰/解除包裝金鑰似乎是多餘的，因為此作業相當於加密/解密，但在提供這些作業的語意和授權區隔上，以及當服務支援其他金鑰類型而需要一致性時，使用不同的作業是相當重要的。  

Azure Key Vault 不支援 EXPORT 作業：一旦金鑰佈建在系統中，即無法擷取或修改其金鑰內容。  不過，Azure Key Vault 的使用者可能需要將其金鑰用於其他使用案例，或從 Azure Key Vault 刪除金鑰後，使用者可能會使用 BACKUP 和 RESTORE 作業，以受保護的形式來匯出/匯入金鑰。 BACKUP 作業所建立的金鑰無法在 Azure Key Vault 以外的地方使用。 但是，可以對多個 Azure Key Vault 執行個體使用 IMPORT 作業。  

使用者可以使用 JWK 物件的 key_ops 屬性，來對 Azure Key Vault 支援的密碼編譯作業 (以每個金鑰為基礎) 設定限制。  

如需 JWK 物件的詳細資訊，請參閱 [JSON Web 金鑰 (JWK)](http://tools.ietf.org/html/draft-ietf-jose-json-web-key)。  

###  <a name="BKMK_KeyAttributes"></a>金鑰屬性

除了金鑰內容，您可以指定下列屬性。 在 JSON 要求中，屬性關鍵字和括弧「{」「}」是必要的，即使沒有指定任何屬性。  

- enabled：選擇性的布林值，預設值是 **true**。 指定金鑰是否已啟用，並可用於密碼編譯作業。 enabled 屬性會與 nbf 和 exp 一起使用。當作業發生於 nbf 和 exp 之間時，只有在 enabled 設定為 **true** 時，才能允許此作業。 發生於 nbf / exp 範圍外的作業將自動禁止，除了[特定條件](#BKMK_key-date-time-ctrld-ops)下的特定作業類型。
- nbf：選擇性的 IntDate，預設值為現在 (now)。 nbf (不早於) 屬性會定義一個時間，而在此時間之前「絕不可」將金鑰用於密碼編譯作業，除了[特定條件](#BKMK_key-date-time-ctrld-ops)下的特定作業類型。 若要處理 nbf 屬性，目前的日期/時間「必須」晚於或等同 nbf 屬性中所列的「不早於」日期/時間。 考慮到時鐘誤差，Azure Key Vault 可能會多提供一點時間 (通常都在幾分鐘內)。 其值必須是包含 IntDate 值的數字。  
- exp：選擇性的 IntDate，預設值為永久 (forever)。 exp (到期時間) 屬性會定義到期時間，而在此時間點或之後「絕不可」將金鑰用於密碼編譯作業，除了[特定條件](#BKMK_key-date-time-ctrld-ops)下的特定作業類型。 若要處理 exp 屬性，目前的日期/時間「必須」早於 exp 屬性中所列的到期日期/時間。 考慮到時鐘誤差，Azure Key Vault 可能會多提供一點時間 (通常都在幾分鐘內)。 其值必須是包含 IntDate 值的數字。  

任何包含金鑰屬性的回應中，可包含其他唯讀屬性：  

- created：選擇性的 IntDate。 created 屬性會指出建立此金鑰版本的時間。 若金鑰是在新增此屬性之前建立的，則此值為 Null。 其值必須是包含 IntDate 值的數字。  
- updated：選擇性的 IntDate。 updated 屬性會指出更新此金鑰版本的時間。 若金鑰是在新增此屬性之前進行最後一次更新，則此值為 Null。 其值必須是包含 IntDate 值的數字。  

如需有關 IntDate 和其他資料類型的詳細資訊，請參閱[資料類型](#BKMK_DataTypes)  

#### <a name="BKMK_key-date-time-ctrld-ops"></a>受到日期時間控制的作業

nbf / exp 範圍外的尚未生效和過期金鑰，將會用於**解密**、**解除包裝**和**驗證**作業 (不會傳回 403 禁止)。 使用尚未生效狀態的基本原理是，允許金鑰先經過測試，然後才在生產環境使用。 使用過期狀態的基本原理是，允許對在金鑰有效時建立的資料執行復原作業。 此外，您可以使用 Key Vault 原則，或藉由將 enabled 金鑰屬性更新為 **false**，來停用金鑰的存取權。

如需有關資料類型的詳細資訊，請參閱[資料類型](#BKMK_DataTypes)。

如需其他可能屬性的詳細資訊，請參閱 [JSON Web 金鑰 (JWK)](http://tools.ietf.org/html/draft-ietf-jose-json-web-key)。

### <a name="BKMK_Keytags"></a>金鑰標記

您可以將其他應用程式專屬的中繼資料指定為標記形式。 Azure Key Vault 支援最多 15 個標記，其中每一個都可以有 256 個字元的名稱和 256 個字元的值。  

>[!Note]
>如果標記具有可對物件類型 (金鑰、秘密或憑證) 執行「列出」或「取得」的權限，則呼叫者可讀取這些標記。

###  <a name="BKMK_KeyAccessControl"></a>金鑰存取控制

對於由 Key Vault 管理的金鑰，其存取控制是在 Key Vault 層級上提供的，Key Vault 則是用來作為金鑰的容器。 在相同 Key Vault 中，金鑰的存取控制原則與秘密的存取控制原則不同。 使用者可建立一個或多個保存庫來保存金鑰，而且必須用保存庫來維護適當的案例分割和金鑰管理。 金鑰的存取控制獨立於秘密的存取控制。  

您可以在保存庫上的金鑰存取控制項目中授與下列權限 (以每個使用者 / 服務主體為基礎)。 這些權限密切地對映金鑰物件上所允許的作業：  

-   建立：建立新的金鑰
-   取得：讀取金鑰的公開部分及其屬性
-   列出：列出金鑰保存庫中儲存的金鑰或金鑰版本
-   更新：更新金鑰屬性
-   刪除：刪除金鑰物件
-   簽署：使用金鑰來簽署摘要
-   驗證：使用金鑰來驗證摘要
-   包裝金鑰：使用金鑰來保護對稱金鑰
-   解除包裝金鑰：使用金鑰來取消保護已包裝的對稱金鑰
-   加密：使用金鑰來保護任意位元組序列
-   解密：使用金鑰來取消保護位元組序列
-   匯入：將金鑰匯入金鑰保存庫
-   備份：備份金鑰保存庫中的金鑰
-   還原：將備份的金鑰還原至金鑰保存庫
-   所有：所有權限

## <a name="key-vault-secrets"></a>Key Vault 祕密 

###  <a name="BKMK_WorkingWithSecrets"></a>使用秘密

Azure Key Vault 中的每個秘密都是大小上限為 25k 位元組的八位元序列。 Azure Key Vault 服務未針對秘密提供任何語意；它只接受資料，然後對其加密並儲存，而傳回的秘密識別碼 “id” 可用來在之後擷取秘密。  

針對高度敏感性的資料，用戶端應考慮對儲存在 Azure Key Vault 中的資料使用其他保護層級；例如，使用個別保護金鑰來預先加密資料。  

Azure Key Vault 也支援秘密的 contentType 欄位。 用戶端可以指定祕密的內容類型 "contentType"，以協助在擷取秘密資料時進行解譯。 此欄位的長度上限為 255 個字元。 沒有任何預先定義的值。 建議的用法是作為解譯秘密資料的提示。 比方說，實作可能會將密碼和憑證儲存為秘密，然後使用此欄位來表示。 沒有任何預先定義的值。  

###  <a name="BKMK_SecretAttrs"></a>秘密屬性

除了秘密內容，您可以指定下列屬性：  

- exp：選擇性的 IntDate，預設值為**永久 (forever)**。 Exp (到期時間) 屬性會識別到期時間，在此時間點或之後「不應」擷取秘密資料，除非在[特定情況下](#BKMK_secret-date-time-ctrld-ops)。 此欄位僅供**參考**用，因為它告知金鑰保存庫的使用者服，特定祕密可能無法使用。 其值必須是包含 IntDate 值的數字。   
- nbf：選擇性的 IntDate，預設值為**現在 (now)**。 nbf (不早於) 屬性會識別一個時間，在此時間之前「不應」擷取秘密資料，除非在[特定情況下](#BKMK_secret-date-time-ctrld-ops)。 此欄位僅供**參考**用。 其值必須是包含 IntDate 值的數字。 
- enabled：選擇性的布林值，預設值是 **true**。 此屬性會指定是否可以擷取秘密資料。 當作業發生於 nbf 和 exp 之間時，enabled 屬性會用來搭配 nbf 和 exp 使用，只有在 enabled 設定為 **true** 時，才能允許此作業。 發生於 nbf 和 exp 範圍外的作業將自動禁止，除了在[特定情況下](#BKMK_secret-date-time-ctrld-ops)。  

任何包含秘密屬性的回應中，可包含其他唯讀屬性：  

- created：選擇性的 IntDate。 created 屬性會指出建立此秘密版本的時間。 若秘密是在新增此屬性之前建立的，則此值為 Null。 其值必須是包含 IntDate 值的數字。  
- updated：選擇性的 IntDate。 updated 屬性會指出更新此秘密版本的時間。 若秘密是在新增此屬性之前進行最後一次更新，則此值為 Null。 其值必須是包含 IntDate 值的數字。

#### <a name="BKMK_secret-date-time-ctrld-ops"></a>受到日期時間控制的作業

秘密的**取得**作業將會用於尚未生效和過期的秘密 (在 nbf  / exp 範圍外)。 呼叫祕密的**取得**作業來取得尚未生效的秘密，可以用於測試。 擷取 (**取得**) 過期的密碼，可用於復原作業。

如需有關資料類型的詳細資訊，請參閱[資料類型](#BKMK_DataTypes)。  

###  <a name="BKMK_SecretAccessControl"></a>密碼存取控制

對於在 Azure Key Vault 中管理的秘密，其存取控制是在 Key Vault 層級上提供的，Key Vault 則是用來作為這些秘密的容器。 在相同 Key Vault 中，秘密的存取控制原則與金鑰的存取控制原則不同。 使用者可建立一個或多個保存庫來保存秘密，而且必須用保存庫來維護適當的案例分割和秘密管理。 秘密的存取控制獨立於金鑰的存取控制。  

下列權限可以在保存庫上的祕密存取控制項目中使用 (以每個主體為基礎)，且密切地對映秘密物件上所允許的作業：  

-   設定：建立新的秘密  
-   取得：讀取密碼  
-   列出：列出 Key Vault 中儲存的秘密或秘密版本  
-   刪除：刪除秘密  
-   所有：所有權限  

如需使用秘密的詳細資訊，請參閱 [Key Vault REST API 參考中的秘密作業](/rest/api/keyvault)。  

###  <a name="BKMK_SecretTags"></a>秘密標記  
您可以將其他應用程式專屬的中繼資料指定為標記形式。 Azure Key Vault 支援最多 15 個標記，其中每一個都可以有 256 個字元的名稱和 256 個字元的值。  

>[!Note]
>如果標記具有可對物件類型 (金鑰、秘密或憑證) 執行「列出」或「取得」的權限，則呼叫者可讀取這些標記。

## <a name="key-vault-certificates"></a>Key Vault 憑證

金鑰保存庫憑證支援提供管理 x509 憑證和下列行為︰  

-   允許憑證擁有者透過金鑰保存庫建立程序或匯入現有憑證來建立憑證。 這同時包含自我簽署和憑證授權單位所產生的憑證。
-   允許金鑰保存庫憑證擁有者實作 X509 憑證的安全儲存和管理，而不需要與私密金鑰資料互動。  
-   允許憑證擁有者建立原則，以指示金鑰保存庫管理憑證的生命週期。  
-   允許憑證擁有者提供有關憑證到期和更新生命週期事件的通知的連絡資訊。  
-   支援與所選簽發者的自動更新 - 金鑰保存庫夥伴 X509 憑證提供者/憑證授權單位。

>[!Note]
>也允許無合作關係的提供者/授權單位，但不支援自動更新功能。

###  <a name="BKMK_CompositionOfCertificate"></a>憑證的構成要素

建立 Key Vault 憑證時，可定址的金鑰和秘密也會以相同名稱建立。 Key Vault 金鑰可讓金鑰作業和 Key Vault 秘密允許擷取憑證值作為秘密。 Key Vault 憑證也會包含公用 x509 憑證的中繼資料。  

憑證的識別碼和版本類似於金鑰和密碼。 以 Key Vault 憑證版本建立的特定可定址金鑰和秘密版本，可用於 Key Vault 憑證的回應。
 
![憑證是複雜物件](media/azure-key-vault.png)

###  <a name="BKMK_CertificateExportableOrNonExportableKey"></a> 可匯出或不可匯出金鑰

建立 Key Vault 憑證時，如果用來建立憑證的原則指出金鑰可匯出，則您即可以 PFX 或 PEM 格式擷取可定址秘密中的憑證和私密金鑰。 如果用來建立 Key Vault 憑證的原則指出金鑰不可匯出，則擷取為祕密的值不包含私密金鑰。  

可定址的金鑰會變成與不可匯出的 KV 憑證有較高相關性。 用來建立 KV 憑證的 KV 憑證原則欄位 keyusage，會對應至可定址 KV 金鑰的作業。  

支援金鑰類型有兩種 – 具有憑證的 RSA 或 RSA HSM。 可匯出金鑰允許使用 RSA，不支援 RSA HSM。  

###  <a name="BKMK_CertificateAttributesAndTags"></a>憑證屬性與標記

除了憑證中繼資料、可定址的金鑰和可定址的秘密，Key Vault 憑證也會包含屬性和標記。  

#### <a name="attributes"></a>屬性

憑證屬性會對映至建立 KV 憑證時建立的可定址金鑰和秘密。  

Key Vault 憑證具有下列屬性：  

-   enabled：選擇性的布林值，預設值是 **true**。 您可以指定此屬性來指出憑證資料是否可以擷取為秘密，或是否可以作為金鑰執行。 當作業發生於 nbf 和 exp 之間時，此屬性會用來搭配 nbf 和 exp 使用，只有在 enabled 設定為 true 時，才能允許此作業。 在 nbf 和 exp 範圍以外的作業會自動禁止。  

回應中會包含其他唯讀屬性：

-   created：IntDate：指出建立此憑證版本的時間。  
-   updated：IntDate：指出更新此憑證版本的時間。  
-   exp：IntDate：包含 x509 憑證的到期日值。  
-   nbf：IntDate：包含 x509 憑證的日期值。  

> [!Note] 
> 如果 Key Vault 憑證到期，其可定址金鑰和秘密將變得無法使用。  

#### <a name="tags"></a>標記

 用戶端指定的金鑰值組字典，類似於金鑰與秘密中的標記。  

 > [!Note]
> 如果標記具有可對物件類型 (金鑰、秘密或憑證) 執行「列出」或「取得」的權限，則呼叫者可讀取這些標記。

###  <a name="BKMK_CertificatePolicy"></a>憑證原則

憑證原則包含如何建立和管理 KV 憑證生命週期的資訊。 當憑證與私密金鑰匯入金鑰保存庫時，系統即會透過讀取 x509 憑證來建立預設的原則。  

如果從頭建立 KV 憑證，則需要將原則提供給 Key Vault，向其說明如何建立此 KV 憑證版本或下一個 KV 憑證版本。 一旦建立原則後，之後將不需要使用建立作業來建立下一個 KV 憑證版本。  

所有 KV 憑證版本只會有一個原則執行個體。  

概括而言，憑證原則包含下列內容：  

-   X509 憑證的屬性：包含主體名稱、主體替代名稱等等。 用來建立 x509 憑證要求。  
-   金鑰屬性：包含金鑰類型、金鑰長度及可匯出和重複使用的金鑰欄位。 這些欄位會指示金鑰保存庫如何產生金鑰。  
-   秘密屬性：包含用來產生秘密值的秘密屬性，例如可定址秘密的內容類型，進而擷取憑證作為秘密。  
-   存留期動作：包含 KV 憑證的存留期動作。 每個存留期動作包含：  

     - 觸發程序：透過到期或存留期範圍百分比前的天數來指定  

     - 動作：指定動作類型 – emailContacts 或 autoRenew  

-   簽發者：要用於發行 x509 憑證的憑證簽發者相關參數。  
-   原則屬性：包含與原則相關聯的屬性  

#### <a name="x509-to-key-vault-usage-mapping"></a>X509 與 Key Vault 使用方式的對應

下表代表 x509 金鑰使用原則與有效金鑰 (Key Vault 憑證建立時一起建立的金鑰) 作業的對應。

|**X509 金鑰使用方式旗標**|**Key Vaul 金鑰作業**|**預設行為**|
|----------|--------|--------|
|DataEncipherment|加密、解密| N/A |
|DecipherOnly|解密| N/A  |
|DigitalSignature|簽署、驗證| Key Vault 的預設是憑證建立時不包含使用規格 | 
|EncipherOnly|加密| N/A |
|KeyCertSign|簽署、驗證|N/A|
|KeyEncipherment|包裝金鑰、解除包裝金鑰| Key Vault 的預設是憑證建立時不包含使用規格 | 
|NonRepudiation|簽署、驗證| N/A |
|crlsign|簽署、驗證| N/A |

###  <a name="BKMK_CertificateIssuer"></a>憑證簽發者

Key Vault 憑證物件會保存用來與所選憑證簽發者之提供者通訊的組態，以訂購 x509 憑證。  

-   Key Vault 與下列憑證簽發者的提供者有夥伴關係 (適用於 SSL 憑證)

|**提供者名稱**|**位置**|
|----------|--------|
|DigiCert|支援所有位在公用雲端和 Azure Government 中的金鑰保存庫服務|
|GlobalSign|支援所有位在公用雲端和 Azure Government 中的金鑰保存庫服務|

您必須成功完成下列先決條件的步驟 1 和 2，才可在 Key Vault 中建立憑證簽發者。  

1. 上架至憑證授權單位 (CA) 提供者  

    -   組織系統管理員必須至少使用一個 CA 提供者來讓其公司 ( 例如 Contoso) 上架。  

2. 管理員會建立 Key Vault 的要求者憑證，以註冊 (和更新) SSL 憑證  

    -   提供組態，以用來建立金鑰保存庫中提供者的簽發者物件  

如需有關從憑證入口網站建立簽發者物件的詳細資訊，請參閱 [Key Vault 憑證部落格](http://aka.ms/kvcertsblog)  

Key Vault 可讓您以不同簽發者的提供者組態，來建立多個簽發者物件。 簽發者物件建立之後，其名稱可以在一個或多個憑證原則中進行參考。 在建立與更新憑證期間，從 CA 提供者中要求 x509 憑證時，參考簽發者物件會指示 Key Vault 使用簽發者物件中所指定的組態。  

簽發者物件會在保存庫中建立，而且只能搭配相同保存庫中的 KV 憑證使用。  

###  <a name="BKMK_CertificateContacts"></a>憑證連絡人

憑證連絡人中包含連絡資訊，用來傳送由憑證存留期事件觸發的通知。 連絡人資訊會由金鑰保存庫中的所有憑證共用。 若金鑰保存庫中發生任何憑證事件，系統會將通知傳送給所有指定的連絡人。  

如果將憑證的原則設定為自動更新，則發生下列事件時會傳送通知。  

-   憑證更新之前
-   在憑證更新之後，指出是否已成功更新憑證，或是如果發生錯誤，需要手動更新憑證。  

 如果憑證的原則設定為以手動方式更新 (僅限電子郵件)，則會在需更新憑證時傳送通知。  

###  <a name="BKMK_CertificateAccessControl"></a>憑證存取控制

 憑證的存取控制是由 Key Vault 管理且在 Key Vault 層級上提供的，Key Vault 則是用來作為這些憑證的容器。 在相同 Key Vault 中，憑證的存取控制原則與金鑰及秘密的存取控制原則不同。 使用者可建立一個或多個保存庫來保存憑證，而且必須用保存庫來維護適當的案例分割和憑證管理。  

 下列權限可以在金鑰保存庫上的祕密存取控制項目中使用 (以每個主體為基礎)，且密切地對映秘密物件上所允許的作業：  

-   取得：允許取得目前版本的憑證或任何版本的憑證 
-   列出：允許列出目前憑證或憑證版本  
-   刪除：允許刪除憑證、憑證的原則和其所有版本  
-   建立：允許建立 Key Vault 憑證。  
-   匯入：允許將憑證內容匯入 Key Vault 憑證。  
-   更新：允許更新憑證。  
-   managecontacts：允許管理 Key Vault 憑證連絡人  
-   getissuers：允許取得憑證的簽發者  
-   listissuers：允許列出憑證的簽發者  
-   setissuers：允許建立或更新 Key Vault 憑證簽發者  
-   deleteissuers：允許刪除 Key Vault 憑證簽發者  
-   所有：授與所有權限  

如需詳細資訊，請參閱 [Key Vault REST API 參考中的憑證作業](/rest/api/keyvault)。 

## <a name="see-also"></a>另請參閱

- [驗證、要求和回應](authentication-requests-and-responses.md)
- [Key Vault 版本](key-vault-versions.md)
- [Key Vault 開發人員指南](/azure/key-vault/key-vault-developers-guide)
