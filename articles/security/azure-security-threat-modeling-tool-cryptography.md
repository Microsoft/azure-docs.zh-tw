---
title: 密碼編譯 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
description: 降低威脅模型化工具所暴露的威脅
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.openlocfilehash: 4128d113535c67c0b440dc3fb275af05b5c1c1ae
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306140"
---
# <a name="security-frame-cryptography--mitigations"></a>安全性架構︰密碼編譯 | 風險降低 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **Web 應用程式** | <ul><li>[只使用核准的對稱區塊編碼器和金鑰長度](#cipher-length)</li><li>[針對對稱編碼器使用核准的區塊編碼器模式和初始化向量](#vector-ciphers)</li><li>[使用核准的非對稱演算法、金鑰長度和填補](#padding)</li><li>[使用核准的亂數產生器](#numgen)</li><li>[請勿使用對稱串流編碼器](#stream-ciphers)</li><li>[使用核准的 MAC/HMAC/索引雜湊演算法](#mac-hash)</li><li>[只使用核准的密碼編譯雜湊函式](#hash-functions)</li></ul> |
| **資料庫** | <ul><li>[使用增強式加密演算法來加密資料庫中的資料](#strong-db)</li><li>[SSIS 套件應予以加密和數位簽章](#ssis-signed)</li><li>[在重要的資料庫安全性實體中新增數位簽章](#securables-db)</li><li>[使用 SQL Server EKM 來保護加密金鑰](#ekm-keys)</li><li>[如果不應對資料庫引擎顯示加密金鑰，請使用 AlwaysEncrypted 功能](#keys-engine)</li></ul> |
| **IoT 裝置** | <ul><li>[在 IoT 裝置上安全地儲存密碼編譯金鑰](#keys-iot)</li></ul> | 
| **IoT 雲端閘道** | <ul><li>[產生長度足夠的隨機對稱金鑰以向 IoT 中樞進行驗證](#random-hub)</li></ul> | 
| **Dynamics CRM 行動用戶端** | <ul><li>[確定已備妥需要使用 PIN 並允許遠端抹除的裝置管理原則](#pin-remote)</li></ul> | 
| **Dynamics CRM Outlook 用戶端** | <ul><li>[確定已備妥需要 PIN/密碼/自動鎖定並會加密所有資料的裝置管理原則 (例如 Bitlocker)](#bitlocker)</li></ul> | 
| **Identity Server** | <ul><li>[確定在使用 Identity Server 時會變換簽署金鑰](#rolled-server)</li><li>[確定 Identity Server 會使用密碼編譯增強式用戶端識別碼和用戶端祕密](#client-server)</li></ul> | 

## <a id="cipher-length"></a>只使用核准的對稱區塊編碼器和金鑰長度

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>產品只能使用組織的密碼編譯顧問已明確核准的對稱區塊編碼器和相關聯的金鑰長度。 Microsoft 核准的對稱演算法包括下列區塊編碼器︰</p><ul><li>對於新的程式碼，可接受 AES-128、AES-192 和 AES-256</li><li>對於現有程式碼的回溯相容性，可接受三金鑰 3DES</li><li>對於使用對稱區塊編碼器的產品︰<ul><li>新的程式碼需要進階加密標準 (AES)</li><li>為了回溯相容性，現有程式碼允許三金鑰三重資料加密標準 (3DES)</li><li>其他所有區塊編碼器 (包括 RC2、DES、兩金鑰 3DES、DESX 和 Skipjack) 只能用於解密舊資料，若用於加密，則必須加以取代</li></ul></li><li>對稱區塊加密演算法需要至少 128 位元的金鑰長度。 新程式碼唯一建議使用的區塊加密演算法是 AES (AES-128、AES-192 和 AES-256 均可接受)</li><li>如果現有程式碼已使用三金鑰 3DES，則目前仍可接受；但建議您轉換為 AES。 DES、DESX、RC2 和 Skipjack 不再被視為是安全的。 這些演算法只能用於解密現有資料以便實現回溯相容性，並且應該使用建議的區塊編碼器重新加密資料</li></ul><p>請注意，所有的對稱區塊編碼器都必須搭配核准的編碼器模式使用，此模式需要使用適當的初始化向量 (IV)。 適當的 IV 通常是亂數，而絕不會是常數值</p><p>在經過貴組織的密碼編譯委員會審核之後，或許也會允許您使用舊版或其他未經核准的密碼編譯演算法和較短的金鑰長度來讀取現有資料 (而不是撰寫新的資料)。 不過，您必須針對這項需求申請例外狀況。 此外，在企業部署中，產品應考慮在有人使用弱式密碼編譯讀取資料時警告系統管理員。 此類警告應清楚說明原因並提供可採取的動作。 在某些情況下，讓群組原則控制弱式密碼編譯的使用可能是適當措施</p><p>為了實現受控密碼編譯靈活性所允許的 .NET 演算法 (依偏好順序)</p><ul><li>AesCng (符合 FIPS 規範)</li><li>AuthenticatedAesCng (符合 FIPS 規範)</li><li>AESCryptoServiceProvider (符合 FIPS 規範)</li><li>AESManaged (不符合 FIPS 規範)</li></ul><p>請注意，若要透過 `SymmetricAlgorithm.Create` 或 `CryptoConfig.CreateFromName` 方法指定上述演算法，就一定要變更 machine.config 檔案。 另請注意，在 .NET 3.5 之前的 .NET 版本中，AES 是命名為 `RijndaelManaged`，而 `AesCng` 和 `AuthenticatedAesCng` 則可透過 CodePlex 來使用，並且需要基礎作業系統中的 CNG</p>

## <a id="vector-ciphers"></a>針對對稱編碼器使用核准的區塊編碼器模式和初始化向量

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 所有的對稱區塊編碼器都必須搭配核准的對稱編碼器模式使用。 核准的模式只有 CBC 和 CTS。 特別是，應避免電子密碼本 (ECB) 模式的作業；要使用 ECB 需要通過組織的密碼編譯委員會審核。 OFB、CFB、CTR、CCM 和 GCM 或其他任何加密模式的使用，全都必須通過組織的密碼編譯委員會審核。 在區塊編碼器處於「串流編碼器模式」(例如 CTR) 時重複使用相同的初始化向量 (IV)，可能會導致加密資料顯示出來。 所有的對稱區塊編碼器也必須搭配適當的初始化向量 (IV) 使用。 適當的 IV 是指密碼編譯增強式亂數，而絕不會是常數值。 |

## <a id="padding"></a>使用核准的非對稱演算法、金鑰長度和填補

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>使用遭到禁用的密碼編譯演算法會導致產品安全性出現重大風險，因此必須加以避免。 產品只能使用組織的密碼編譯委員會已明確核准的密碼編譯演算法及相關聯的金鑰長度和填補。</p><ul><li>**RSA-** 可用於加密、金鑰交換和簽章。 RSA 加密只能使用 OAEP 或 RSA-KEM 填補模式。 現有程式碼只能在為了相容性因素時使用 PKCS #1 v1.5 填補模式。 明確禁止使用 null 填補。 新的程式碼需要 > = 2048 位元的金鑰。 在經過貴組織的密碼編譯委員會審核之後，現有程式碼也可支援 < 2048 位元的金鑰，但只能用於實現回溯相容性。 < 1024 位元的金鑰只能用於解密/驗證舊資料，若用於加密或簽署作業，則必須加以取代</li><li>**ECDSA-** 只能用於簽章。 新的程式碼需要金鑰位元數 > = 256 的 ECDSA。 以 ECDSA 為基礎的簽章必須使用三個 NIST 核准曲線 (P-256、P-384 或 P521) 的其中一個。 在經過貴組織的密碼編譯委員會審核之後，才能使用已經過徹底分析的曲線。</li><li>**ECDH-** 只能用於金鑰交換。 新的程式碼需要金鑰位元數 > = 256 的 ECDH。 以 ECDH 為基礎的金鑰交換必須使用三個 NIST 核准曲線 (P-256、P-384 或 P521) 的其中一個。 在經過貴組織的密碼編譯委員會審核之後，才能使用已經過徹底分析的曲線。</li><li>**DSA-** 在經過貴組織的密碼編譯委員會審核並核准之後即可接受。 請連絡您的安全性顧問來安排貴組織的密碼編譯委員會進行審核。 如果您獲准使用 DSA，請注意您必須禁止使用長度小於 2048 位元的金鑰。 自 Windows 8 開始，CNG 支援長度為 2048 位元 (含) 以上的金鑰。</li><li>**Diffie-Hellman-** 只能用於工作階段金鑰管理。 新的程式碼需要長度 > = 2048 位元的金鑰。 在經過貴組織的密碼編譯委員會審核之後，現有程式碼也可支援長度 < 2048 位元的金鑰，但只能用於實現回溯相容性。 不得使用 < 1024 位元的金鑰。</li><ul>

## <a id="numgen"></a>使用核准的亂數產生器

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>產品必須使用核准的亂數產生器。 因此，虛擬隨機函式 (例如 C 執行階段函式 rand)、.NET Framework 類別 System.Random 或系統函式 (例如 GetTickCount) 絕不會用於這類程式碼中。 禁止使用雙橢圓曲線亂數產生器 (DUAL_EC_DRBG) 演算法</p><ul><li>**CNG-** BCryptGenRandom (建議使用 BCRYPT_USE_SYSTEM_PREFERRED_RNG 旗標，除非呼叫端可能執行於任何大於 0 的 IRQL [也就是 PASSIVE_LEVEL])</li><li>**CAPI-** cryptGenRandom</li><li>**Win32/64-** RtlGenRandom (新的實作應該使用 BCryptGenRandom 或 CryptGenRandom) * rand_s * SystemPrng (適用於核心模式)</li><li>**.NET-** RNGCryptoServiceProvider 或 RNGCng</li><li>**Windows 市集應用程式-** Windows.Security.Cryptography.CryptographicBuffer.GenerateRandom 或 .GenerateRandomNumber</li><li>**Apple OS X (10.7+)/iOS(2.0+)-** int SecRandomCopyBytes (SecRandomRef random, size_t count, uint8_t \*bytes )</li><li>**Apple OS X (<10.7)-** 使用 /dev/random 來取出隨機數字</li><li>**Java (包括 Google Android Java 程式碼)-** java.security.SecureRandom 類別。 請注意，在 Android 4.3 (Jelly Bean) 中，開發人員必須遵循 Android 建議的因應措施，並將其應用程式更新為使用來自 /dev/urandom 或 /dev/random 的 Entropy 明確初始化 PRNG</li></ul>|

## <a id="stream-ciphers"></a>請勿使用對稱串流編碼器

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 不得使用對稱串流編碼器，例如 RC4。 產品不該使用對稱串流編碼器，而該使用區塊編碼器，特別是金鑰長度至少 128 位元的 AES。 |

## <a id="mac-hash"></a>使用核准的 MAC/HMAC/索引雜湊演算法

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>產品只能使用核准的訊息驗證碼 (MAC) 或雜湊式訊息驗證碼 (HMAC) 演算法。</p><p>訊息驗證碼 (MAC) 是附加到訊息的一條資訊，可供其收件者驗證寄件者的真實性和使用秘密金鑰之訊息的完整性。 只要獲准使用所有基礎雜湊或對稱加密演算法，則也允許使用雜湊式 MAC ([HMAC](http://csrc.nist.gov/publications/nistpubs/800-107-rev1/sp800-107-rev1.pdf)) 或[區塊編碼器式 MAC](http://csrc.nist.gov/publications/nistpubs/800-38B/SP_800-38B.pdf)；目前來說，這包括 HMAC-SHA2 函式 (HMAC-SHA256、HMAC-SHA384 和 HMAC-SHA512) 以及 CMAC/OMAC1 和 OMAC2 區塊編碼器式 MAC (這些是以 AES 為基礎)。</p><p>為了平台相容性，或許會允許您使用 HMAC-SHA1，但您必須針對這項程序申請例外狀況，並接受貴組織的密碼編譯委員會審核。 不允許將 HMAC 截斷為少於 128 位元。 使用客戶的方法對金鑰和資料進行雜湊處理是不被允許的，且必須先經過貴組織的密碼編譯委員會審核後才能使用。</p>|

## <a id="hash-functions"></a>只使用核准的密碼編譯雜湊函式

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>產品必須使用 SHA-2 系列的雜湊演算法 (SHA256、SHA384 及 SHA512)。 如果需要較短的雜湊，例如 128 位元的輸出長度，以配合設計時預設使用較短 MD5 雜湊的資料結構，產品小組可截斷其中一個 SHA2 雜湊 (通常是 SHA256)。 請注意，SHA384 是 SHA512 的截斷版本。 不允許基於安全性考量而將密碼編譯雜湊截斷為少於 128 位元。 新的程式碼不得使用 MD2、MD4、MD5、SHA-0、SHA-1 或 RIPEMD 雜湊演算法。 這些演算法在計算時可能會發生雜湊衝突，而結果便是打斷演算法。</p><p>為了實現受控密碼編譯靈活性所允許的 .NET 雜湊演算法 (依偏好順序)：</p><ul><li>SHA512Cng (符合 FIPS 規範)</li><li>SHA384Cng (符合 FIPS 規範)</li><li>SHA256Cng (符合 FIPS 規範)</li><li>SHA512Managed (不符合 FIPS 規範) (使用 SHA512 作為 HashAlgorithm.Create 或 CryptoConfig.CreateFromName 之呼叫中的演算法名稱)</li><li>SHA384Managed (不符合 FIPS 規範) (使用 SHA384 作為 HashAlgorithm.Create 或 CryptoConfig.CreateFromName 之呼叫中的演算法名稱)</li><li>SHA256Managed (不符合 FIPS 規範) (使用 SHA256 作為 HashAlgorithm.Create 或 CryptoConfig.CreateFromName 之呼叫中的演算法名稱)</li><li>SHA512CryptoServiceProvider (符合 FIPS 規範)</li><li>SHA256CryptoServiceProvider (符合 FIPS 規範)</li><li>SHA384CryptoServiceProvider (符合 FIPS 規範)</li></ul>| 

## <a id="strong-db"></a>使用增強式加密演算法來加密資料庫中的資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [選擇加密演算法](https://technet.microsoft.com/library/ms345262(v=sql.130).aspx) |
| **步驟** | 加密演算法會定義未經授權的使用者無法輕鬆反轉的資料轉換。 SQL Server 可讓系統管理員和開發人員選擇數種演算法，包括 DES、Triple DES、TRIPLE_DES_3KEY、RC2、RC4、128 位元 RC4、DESX、128 位元 AES、192 位元 AES 和 256 位元 AES |

## <a id="ssis-signed"></a>SSIS 套件應予以加密和數位簽章

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [使用數位簽章識別套件的來源](https://msdn.microsoft.com/library/ms141174.aspx)、[威脅和弱點風險降低 (整合服務)](https://msdn.microsoft.com/library/bb522559.aspx) |
| **步驟** | 套件的來源是建立了套件的個人或組織。 執行來自未知或未受信任來源的套件可能有風險。 為了防止有人未經授權就竄改 SSIS 套件，您應該使用數位簽章。 此外，為了確保套件能在儲存/傳輸期間保持機密，您必須加密 SSIS 套件 |

## <a id="securables-db"></a>在重要的資料庫安全性實體中新增數位簽章

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [新增簽章 (Transact-SQL)](https://msdn.microsoft.com/library/ms181700) |
| **步驟** | 在必須驗證重要資料庫的安全性實體是否完整的情況下，您應該使用數位簽章。 資料庫安全性實體 (例如預存程序、函式、組件或觸發程序) 可以加上數位簽章。 以下是此簽章適用時機的範例︰假設 ISV (獨立軟體廠商) 已針對傳遞給其中一個客戶的軟體提供支援。 在提供支援之前，ISV 會想要確保軟體中的資料庫安全性實體並未因為有人不小心或惡意嘗試而遭到竄改。 如果安全性實體有加上數位簽章，ISV 就可以驗證其數位簽章，並驗證其完整性。| 

## <a id="ekm-keys"></a>使用 SQL Server EKM 來保護加密金鑰

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [SQL Server 可延伸金鑰管理 (EKM)](https://msdn.microsoft.com/library/bb895340)、[使用 Azure Key Vault 的可延伸金鑰管理 (SQL Server)](https://msdn.microsoft.com/library/dn198405) |
| **步驟** | SQL Server 可延伸金鑰管理可讓用以保護資料庫檔案的加密金鑰儲存在外部裝置，例如智慧卡、USB 裝置或 EKM/HSM 模組。 這也會讓資料庫管理員 (系統管理員群組的成員除外) 能夠保護資料。 資料可以使用外部 EKM/HSM 模組上只有資料庫使用者可存取的加密金鑰來加密。 |

## <a id="keys-engine"></a>如果不應對資料庫引擎顯示加密金鑰，請使用 AlwaysEncrypted 功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | SQL Azure、OnPrem |
| **屬性**              | SQL 版本 - V12、MsSQL2016 |
| **參考**              | [永遠加密 (資料庫引擎)](https://msdn.microsoft.com/library/mt163865) |
| **步驟** | Always Encrypted 功能是設計用來保護 Azure SQL Database 或 SQL Server 資料庫中儲存的敏感性資料，例如信用卡號碼或身分證號碼 (例如，美國的社會安全號碼)。 Always Encrypted 可讓用戶端在用戶端應用程式中將敏感性資料加密，且永遠不會對資料庫引擎 (SQL Database 或 SQL Server) 顯示加密金鑰。 因此，Always Encrypted 可將資料擁有者 (和可檢視者) 及資料管理者 (但無法存取資料) 分隔開來 |

## <a id="keys-iot"></a>在 IoT 裝置上安全地儲存密碼編譯金鑰

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 裝置 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | 裝置作業系統 - Windows IoT 核心版、裝置連線能力 - Azure IoT 裝置 SDK |
| **參考**              | [Windows IoT 核心版上的 TPM](https://developer.microsoft.com/windows/iot/docs/tpm)、[設定 Windows IoT 核心版上的 TPM](https://developer.microsoft.com/windows/iot/win10/setuptpm)、[Azure IoT 裝置 SDK TPM](https://github.com/Azure/azure-iot-hub-vs-cs/wiki/Device-Provisioning-with-TPM) |
| **步驟** | 在硬體保護的儲存體 (如 TPM 或智慧卡晶片) 中安全地對稱或憑證私密金鑰。 Windows 10 IoT Core 支援使用 TPM，而且有數個相容 TPM 可供使用︰ https://developer.microsoft.com/windows/iot/win10/tpm。 建議您使用韌體或離散 TPM。 軟體 TPM 只應用於開發和測試用途。 一旦 TPM 可供使用而且在其中佈建了金鑰，應該就能撰寫權杖產生程式碼而不必在其中硬式編碼任何敏感性資訊。 | 

### <a name="example"></a>範例
```
TpmDevice myDevice = new TpmDevice(0);
// Use logical device 0 on the TPM 
string hubUri = myDevice.GetHostName(); 
string deviceId = myDevice.GetDeviceId(); 
string sasToken = myDevice.GetSASToken(); 

var deviceClient = DeviceClient.Create( hubUri, AuthenticationMethodFactory. CreateAuthenticationWithToken(deviceId, sasToken), TransportType.Amqp); 
```
如您所見，裝置的主要金鑰不在程式碼中。 而是儲存在插槽 0 的 TPM 中。 TPM 裝置會產生短期 SAS 權杖，然後用來連線到 IoT 中樞。 

## <a id="random-hub"></a>產生長度足夠的隨機對稱金鑰以向 IoT 中樞進行驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 雲端閘道 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | 閘道選擇 - Azure IoT 中樞 |
| **參考**              | N/A  |
| **步驟** | IoT 中樞包含裝置身分識別登錄，在佈建裝置時，會自動產生隨機的對稱金鑰。 建議您使用 Azure IoT 中樞身分識別登錄的這項功能來產生用於驗證的金鑰。 IoT 中樞還可在建立裝置時指定金鑰。 如果在佈建裝置期間，您是在 IoT 中樞外部產生金鑰，建議您建立隨機對稱金鑰或至少 256 位元。 |

## <a id="pin-remote"></a>確定已備妥需要使用 PIN 並允許遠端抹除的裝置管理原則

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM 行動用戶端 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確定已備妥需要使用 PIN 並允許遠端抹除的裝置管理原則 |

## <a id="bitlocker"></a>確定已備妥需要 PIN/密碼/自動鎖定並會加密所有資料的裝置管理原則 (例如 Bitlocker)

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Dynamics CRM Outlook 用戶端 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確定已備妥需要 PIN/密碼/自動鎖定並會加密所有資料的裝置管理原則 (例如 Bitlocker) |

## <a id="rolled-server"></a>確定在使用 Identity Server 時會變換簽署金鑰

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Identity Server | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Identity Server - 金鑰、簽章和密碼編譯](https://identityserver.github.io/Documentation/docsv2/configuration/crypto.html) |
| **步驟** | 確定在使用 Identity Server 時會變換簽署金鑰。 [參考] 區段中的連結會說明該如何進行規劃，而不會造成仰賴 Identity Server 的應用程式發生中斷。 |

## <a id="client-server"></a>確定 Identity Server 會使用密碼編譯增強式用戶端識別碼和用戶端祕密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Identity Server | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>確定 Identity Server 會使用密碼編譯增強式用戶端識別碼和用戶端祕密。 在產生用戶端識別碼和祕密時，應該使用下列指導方針︰</p><ul><li>產生隨機 GUID 做為用戶端識別碼</li><li>產生密碼編譯隨機 256 位元金鑰作為祕密</li></ul>|
