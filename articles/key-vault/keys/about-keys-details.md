---
title: 金鑰類型、演算法和作業-Azure Key Vault
description: 支援的金鑰類型、演算法和作業 (詳細資料) 。
services: key-vault
author: amitbapat
manager: msmbaldwin
ms.service: key-vault
ms.subservice: keys
ms.topic: conceptual
ms.date: 10/22/2020
ms.author: ambapat
ms.openlocfilehash: 675c4f04ece322000ae0ebb44d6291c455db9397
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98133271"
---
# <a name="key-types-algorithms-and-operations"></a>金鑰類型、演算法和作業

Key Vault 支援兩種資源類型：保存庫和受管理的 Hsm。 這兩種資源類型都支援不同的加密金鑰。 若要查看支援的金鑰類型摘要、每個資源類型的保護類型，請參閱 [關於金鑰](about-keys.md)。

下表顯示金鑰類型和支援演算法的摘要。

|金鑰類型/大小/曲線| 加密/解密<br>(包裝/解除包裝) | 簽署/驗證 | 
| --- | --- | --- |
|EC-P256, EC-P256K, EC-P384, EC-521|NA|ES256<br>ES256K<br>ES384<br>ES512|
|RSA 2K, 3K, 4K| RSA1_5<br>RSA-OAEP<br>RSA-OAEP-256|PS256<br>PS384<br>PS512<br>RS256<br>RS384<br>RS512<br>RSNULL| 
|AES 128 位元、256 位元 <br/>僅 (受管理的 HSM) | AES-KW<br>AES-GCM<br>AES-CBC| NA| 
|||

##  <a name="ec-algorithms"></a>EC 演算法
 EC-HSM 金鑰支援下列演算法識別碼

### <a name="curve-types"></a>曲線類型

-   **P-256** - NIST 曲線 P-256，定義於 [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)。
-   **P-256K** - SEC 曲線 SECP256K1，其定義在 [SEC 2：建議使用的橢圓曲線網域參數](https://www.secg.org/sec2-v2.pdf)。
-   **P-384** - NIST 曲線 P-384，定義於 [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)。
-   **P-521** - NIST 曲線 P-521，定義於 [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)。

### <a name="signverify"></a>SIGN/VERIFY

-   **ES256** - 適用於 SHA-256 摘要的 ECDSA 與使用曲線 P-256 建立的金鑰。 此演算法說明於 [RFC7518](https://tools.ietf.org/html/rfc7518)。
-   **ES256K** - 適用於 SHA-256 摘要的 ECDSA 與使用曲線 P-256K 建立的金鑰。 此演算法尚待標準化。
-   **ES384** - 適用於 SHA-384 摘要的 ECDSA 與使用曲線 P-384 建立的金鑰。 此演算法說明於 [RFC7518](https://tools.ietf.org/html/rfc7518)。
-   **ES512** - 適用於 SHA-512 摘要的 ECDSA 與使用曲線 P-521 建立的金鑰。 此演算法說明於 [RFC7518](https://tools.ietf.org/html/rfc7518)。

##  <a name="rsa-algorithms"></a>RSA 演算法  
 RSA 與 RSA-HSM 金鑰支援下列演算法識別碼  

### <a name="wrapkeyunwrapkey-encryptdecrypt"></a>包裝金鑰/解除包裝金鑰、加密/解密

-   **RSA1_5** - RSAES-PKCS1-V1_5 [RFC3447] 金鑰加密  
-   **RSA-OAEP** - RSAES 使用「最佳非對稱加密填補 (OAEP)」[RFC3447]，並利用 A.2.1 節中 RFC 3447 所指定的預設參數。 這些預設參數會使用 SHA-1 的雜湊函數及搭配 SHA-1 的 MGF1 遮罩產生函數。  
-  **RSA-OAEP-256** – RSAES 使用最佳非對稱加密填補，具有 SHA-256 的雜湊函式及搭配 SHA-256 的 MGF1 遮罩產生函式

### <a name="signverify"></a>SIGN/VERIFY

-   **PS256** - RSASSA-PSS 使用 SHA-256 及搭配 SHA-256 的 MGF1，如 [RFC7518](https://tools.ietf.org/html/rfc7518) 中所述。
-   **PS384** - RSASSA-PSS 使用 SHA-384 及搭配 SHA-384 的 MGF1，如 [RFC7518](https://tools.ietf.org/html/rfc7518) 中所述。
-   **PS512** - RSASSA-PSS 使用 SHA-512 及搭配 SHA-512 的 MGF1，如 [RFC7518](https://tools.ietf.org/html/rfc7518)中所述。
-   **RS256** - 使用 SHA-256 的 RSASSA-PKCS-v1_5。 提供摘要值的應用程式必須使用 SHA-256 計算，而且長度必須是 32 個位元組。  
-   **RS384** - 使用 SHA-384 的 RSASSA-PKCS-v1_5。 提供摘要值的應用程式必須使用 SHA-384 計算，而且長度必須是 48 個位元組。  
-   **RS512** - 使用 SHA-512 的 RSASSA-PKCS-v1_5。 提供摘要值的應用程式必須使用 SHA-512 計算，而且長度必須是 64 個位元組。  
-   **RSNULL** - 請參閱啟用特定 TLS 案例的特殊使用案例 [RFC2437](https://tools.ietf.org/html/rfc2437)。  

##  <a name="symmetric-key-algorithms-managed-hsm-only"></a>對稱金鑰演算法 (僅限受控 HSM) 
- **AES-KW** - AES 金鑰包裝 ([RFC3394](https://tools.ietf.org/html/rfc3394))。
- **Aes-GCM** -Galois 計數器模式中的 aes 加密 ([NIST SP 800-38d](https://csrc.nist.gov/publications/sp800)) 
- **Aes-CBC** -加密區塊鏈模式中的 aes 加密 ([NIST SP 800-38a](https://csrc.nist.gov/publications/sp800)) 

> [!NOTE] 
> 目前的 AES-GCM 實作和對應的 API 都是實驗性的。 在未來的反覆項目中，實作和 API 可能會大幅變更。 

##  <a name="key-operations"></a>金鑰作業

受控 HSM 支援下列金鑰物件作業：  

-   **建立**：可讓用戶端在 Key Vault 中建立金鑰。 金鑰值會由 Key Vault 產生並儲存，但不會發行給用戶端。 在 Key Vault 中可建立非對稱金鑰。  
-   **Import**：可讓用戶端將現有金鑰匯入至 Key Vault。 非對稱金鑰可使用 JWK 概念中的多種不同封裝方法來匯入至 Key Vault。 
-   **更新**：可讓具有足夠權限的用戶端，修改與先前儲存在 Key Vault 內之金鑰相關的中繼資料 (金鑰屬性)。  
-   **刪除**：可讓具有足夠權限的用戶端從 Key Vault 中刪除金鑰。  
-   **列出**：可讓用戶端列出所指定 Key Vault 中的所有金鑰。  
-   **列出版本**：可讓用戶端列出指定的 Key Vault 中，所指定金鑰的所有版本。  
-   **取得**：可讓用戶端擷取 Key Vault 中所指定金鑰的公開部分。  
-   **備份**：以受保護的形式匯出金鑰。  
-   **還原**：匯入先前備份的金鑰。  

如需詳細資訊，請參閱 [Key Vault REST API 參考中的金鑰作業](/rest/api/keyvault)。  

一旦在 Key Vault 中建立金鑰之後，即可使用該金鑰執行下列加密編譯作業：  

-   **簽署並驗證**：嚴格來說，這項作業是「簽署雜湊」或「驗證雜湊」，因為 Key Vault 不支援在建立簽章時進行內容雜湊。 應用程式應對要在本機上簽署的資料進行雜湊，然後要求 Key Vault 簽署此雜湊資料。 驗證已簽署的雜湊是針對無法存取 [公開] 金鑰內容的應用程式而支援的便利作業。 為達到最佳應用程式效能，驗證作業應在本機上執行。  
-   **金鑰加密/包裝**：儲存在 Key Vault 中的金鑰可用來保護另一個金鑰，通常是對稱內容加密金鑰 (CEK)。 如果 Key Vault 中的金鑰是非對稱的，則會使用金鑰加密。 例如 RSA-OAEP 和包裝金鑰/解除包裝金鑰作業等同於加密/解密。 如果 Key Vault 中的金鑰是對稱的，則使用金鑰包裝。 例如 AES-KW。 包裝金鑰作業是針對無法存取 [公開] 金鑰內容的應用程式而支援的便利機制。 為達到最佳應用程式效能，包裝金鑰作業應在本機上執行。  
-   **加密和解密**：儲存在 Key Vault 中的金鑰可用來加密或解密單一資料區塊。 區塊的大小取決於金鑰類型和選取的加密演算法。 加密作業是針對無法存取 [公開] 金鑰內容的應用程式而提供的便利機制。 為達到最佳應用程式效能，加密作業應在本機上執行。  

儘管使用非對稱金鑰的包裝金鑰/解除包裝金鑰顯得有點多餘 (因為此作業相當於加密/解密)，但使用相異的作業是相當重要的。 這樣的差異性可讓這些作業的語意和授權有所區隔，並且在服務支援其他金鑰類型時帶來一致性。  

Key Vault 不支援匯出作業。 在系統中佈建金鑰後，即無法加以擷取或修改其金鑰內容。 不過，Key Vault 的使用者在其他使用案例中有可能需要其金鑰，例如在金鑰遭刪除後。 在此情況下，他們可以使用備份和還原作業來匯出/匯入受保護的金鑰。 備份作業所建立的金鑰無法在 Key Vault 以外的地方使用。 但是，可以對多個 Key Vault 執行個體使用匯入作業。  

使用者可以使用 JWK 物件的 key_ops 屬性，來對 Key Vault 支援的密碼編譯作業 (以每個金鑰為基礎) 設定限制。  

如需 JWK 物件的詳細資訊，請參閱 [JSON Web 金鑰 (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41)。  

## <a name="key-attributes"></a>金鑰屬性

除了金鑰內容，您可以指定下列屬性。 在 JSON 要求中，屬性關鍵字和括弧「{」「}」是必要的，即使沒有指定任何屬性。  

- enabled：選擇性的布林值，預設值是 **true**。 指定金鑰是否已啟用，並可用於密碼編譯作業。 enabled 屬性會與 nbf 和 exp 一起使用。當作業發生於 nbf 和 exp 之間時，只有在 enabled 設定為 **true** 時，才能允許此作業。 發生於 nbf / exp 範圍外的作業將自動禁止，除了[特定條件](#date-time-controlled-operations)下的特定作業類型。
- *nbf*：選擇性的 IntDate，預設值為現在 (now)。 nbf (不早於) 屬性會定義一個時間，而在此時間之前「絕不可」將金鑰用於密碼編譯作業，除了[特定條件](#date-time-controlled-operations)下的特定作業類型。 若要處理 nbf 屬性，目前的日期/時間「必須」晚於或等同 nbf 屬性中所列的「不早於」日期/時間。 考慮到時鐘誤差，Key Vault 可能會多提供一點時間 (通常都在幾分鐘內)。 其值必須是包含 IntDate 值的數字。  
- *exp*：選擇性的 IntDate，預設值為永久 (forever)。 exp (到期時間) 屬性會定義到期時間，而在此時間點或之後「絕不可」將金鑰用於密碼編譯作業，除了[特定條件](#date-time-controlled-operations)下的特定作業類型。 若要處理 exp 屬性，目前的日期/時間「必須」早於 exp 屬性中所列的到期日期/時間。 考慮到時鐘誤差，Key Vault 可能會多提供一點時間 (通常都在幾分鐘內)。 其值必須是包含 IntDate 值的數字。  

任何包含金鑰屬性的回應中，可包含其他唯讀屬性：  

- *created*：選擇性的 IntDate。 created 屬性會指出建立此金鑰版本的時間。 若金鑰是在新增此屬性之前建立的，則此值為 Null。 其值必須是包含 IntDate 值的數字。  
- *updated*：選擇性的 IntDate。 updated 屬性會指出更新此金鑰版本的時間。 若金鑰是在新增此屬性之前進行最後一次更新，則此值為 Null。 其值必須是包含 IntDate 值的數字。  

如需有關 IntDate 和其他資料類型的詳細資訊，請參閱 [關於金鑰、祕密與憑證：[資料類型](../general/about-keys-secrets-certificates.md#data-types)。

### <a name="date-time-controlled-operations"></a>受到日期時間控制的作業

nbf / exp 範圍外尚未生效和過期的金鑰，將會用於 **解密**、**解除包裝** 和 **驗證** 作業 (不會傳回 403 禁止)。 使用尚未生效狀態的基本原理是，允許金鑰先經過測試，然後才在生產環境使用。 使用過期狀態的基本原理是，允許對在金鑰有效時建立的資料執行復原作業。 此外，您可以使用 Key Vault 原則，或藉由將 enabled 金鑰屬性更新為 **false**，來停用金鑰的存取權。

如需資料類型的詳細資訊，請參閱[資料類型](../general/about-keys-secrets-certificates.md#data-types)。

如需其他可能屬性的詳細資訊，請參閱 [JSON Web 金鑰 (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41)。

## <a name="key-tags"></a>金鑰標記

您可以將其他應用程式專屬的中繼資料指定為標記形式。 Key Vault 支援最多 15 個標記，各標記可以有 256 個字元的名稱和 256 個字元的值。  

> [!NOTE] 
> 如果呼叫者具有該金鑰的 *列出* 或 *取得* 權限，其便可讀取標籤。

##  <a name="key-access-control"></a>金鑰存取控制

對於由 Key Vault 管理的金鑰，其存取控制是在 Key Vault 層級上提供的，Key Vault 則是用來作為金鑰的容器。 在相同 Key Vault 中，金鑰的存取控制原則與秘密的存取控制原則並不相同。 使用者可建立一或多個保存庫來保存秘密，且必須維護適當區分和管理秘密的案例。 金鑰的存取控制獨立於秘密的存取控制。  

您可以在保存庫上的金鑰存取控制項目中授與下列權限 (以每個使用者 / 服務主體為基礎)。 這些權限密切地對映金鑰物件上所允許的作業。  將存取權授與 key vault 中的服務主體是一項 onetime 作業，而且所有 Azure 訂用帳戶的存取都維持不變。 您可以使用此主體來部署任意數目的憑證。 

- 金鑰管理作業的權限
  - *get*：讀取金鑰的公開部分及其屬性
  - *list*：列出儲存在金鑰保存庫中的金鑰或金鑰版本
  - *update*：更新金鑰的屬性
  - *create*：建立新的金鑰
  - *import*：將金鑰匯入至金鑰保存庫
  - *delete*：刪除金鑰物件
  - *recover*：復原已刪除的金鑰
  - *backup*：備份金鑰保存庫中的金鑰
  - *restore*：將備份的金鑰還原至金鑰保存庫

- 密碼編譯作業的權限
  - *decrypt*：使用金鑰來解除對位元組序列的保護
  - *encrypt*：使用金鑰來保護任意的位元組序列
  - *unwrapKey*：使用金鑰來解除對已包裝對稱金鑰的保護
  - *wrapKey*：使用金鑰來保護對稱金鑰
  - *verify*：使用金鑰來驗證摘要  
  - *sign*：使用金鑰來簽署摘要
    
- 特殊權限作業的權限
  - *purge*：清除 (永久刪除) 已刪除的金鑰

如需使用金鑰的詳細資訊，請參閱 [Key Vault REST API 參考中的金鑰作業](/rest/api/keyvault)。 如需建立權限的相關資訊，請參閱[保存庫 - 建立或更新](/rest/api/keyvault/vaults/createorupdate)和[保存庫 - 更新存取原則](/rest/api/keyvault/vaults/updateaccesspolicy)。 

## <a name="next-steps"></a>後續步驟
- [關於 Key Vault](../general/overview.md)
- [關於受控 HSM](../managed-hsm/overview.md)
- [關於秘密](../secrets/about-secrets.md)
- [關於憑證](../certificates/about-certificates.md)
- [Key Vault REST API 概觀](../general/about-keys-secrets-certificates.md)
- [驗證、要求和回應](../general/authentication-requests-and-responses.md)
- [Key Vault 開發人員指南](../general/developers-guide.md)
