---
title: Microsoft Azure 儲存體的用戶端 Java 加密 | Microsoft Docs
description: Azure Storage Client Library for Java 支援用戶端加密以及與 Azure 金鑰保存庫的整合，為您的 Azure 儲存體應用程式提供最大的安全性。
services: storage
author: lakasa
ms.service: storage
ms.devlang: java
ms.topic: article
ms.date: 05/11/2017
ms.author: lakasa
ms.component: common
ms.openlocfilehash: 995b54d8eb53efc6001e29366e22e09b246f0bff
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39522011"
---
# <a name="client-side-encryption-and-azure-key-vault-with-java-for-microsoft-azure-storage"></a>Microsoft Azure 儲存體搭配 Java 的用戶端加密和 Azure Key Vault
[!INCLUDE [storage-selector-client-side-encryption-include](../../../includes/storage-selector-client-side-encryption-include.md)]

## <a name="overview"></a>概觀
[Azure Storage Client Library for Java](http://mvnrepository.com/artifact/com.microsoft.azure/azure-storage) 支援在上傳至 Azure 儲存體之前將用戶端應用程式內的資料加密，並在下載至用戶端時解密資料。 程式庫也支援與 [Azure 金鑰保存庫](https://azure.microsoft.com/services/key-vault/) 整合，以進行儲存體帳戶金鑰管理。

## <a name="encryption-and-decryption-via-the-envelope-technique"></a>透過信封技術進行加密和解密
加密和解密的程序採用信封技術。  

### <a name="encryption-via-the-envelope-technique"></a>透過信封技術加密
透過信封技術加密的運作方式如下：  

1. Azure 儲存體用戶端程式庫會產生內容加密金鑰 (CEK)，這是使用一次的對稱金鑰。  
2. 使用此 CEK 加密使用者資料。   
3. 然後使用金鑰加密金鑰 (KEK) 包裝 (加密) CEK。 KEK 由金鑰識別碼所識別，可以是非對稱金鑰組或對稱金鑰，且可以在本機管理或儲存在 Azure 金鑰保存庫中。  
   儲存體用戶端程式庫本身永遠沒有 KEK 的存取權。 程式庫會叫用金鑰保存庫所提供的金鑰包裝演算法。 如有需要，使用者可以選擇使用自訂提供者來包裝/取消包裝金鑰。  
4. 然後，將加密的資料上傳至 Azure 儲存體服務。 包裝的金鑰及一些其他加密中繼資料會儲存為中繼資料 (在 blob 上)，或插補加密的資料 (訊息佇列和資料表實體)。

### <a name="decryption-via-the-envelope-technique"></a>透過信封技術解密
透過信封技術解密的運作方式如下：  

1. 用戶端程式庫假設使用者在本機或 Azure 金鑰保存庫中管理金鑰加密金鑰 (KEK)。 使用者不必知道用於加密的特定金鑰。 相反地，可以設定並使用金鑰解析程式，將不同的金鑰識別碼解析成金鑰。  
2. 用戶端程式庫會下載加密的資料，以及儲存在服務上的任何加密資料。  
3. 然後，使用金鑰加密金鑰 (KEK) 將已包裝的內容加密金鑰 (CEK) 解除包裝 (解密)。 同樣地，用戶端程式庫在此沒有 KEK 的存取權。 它只會叫用自訂或 Key Vault 提供者的解除包裝演算法。  
4. 然後，使用內容加密金鑰 (CEK) 解密已加密的使用者資料。

## <a name="encryption-mechanism"></a>加密機制
儲存體用戶端程式庫會使用 [AES](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 來加密使用者資料。 具體來說，就是 [加密區塊鏈結 (CBC)](http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) 模式搭配 AES。 每個服務的運作方式稍有不同，我們將在這裡討論每個服務。

### <a name="blobs"></a>Blob
用戶端程式庫目前僅支援整個 Blob 的加密。 更明確地說，會在使用者使用 **upload*** 方法或 **openOutputStream** 方法時支援加密。 針對下載，則皆支援完整與範圍下載。  

在加密期間，用戶端程式庫會產生 16 位元組的隨機初始化向量 (IV)，以及 32 位元組的隨機內容加密金鑰 (CEK)，並使用這項資訊執行 blob 資料的信封加密。 然後，已包裝的 CEK 和一些其他加密中繼資料會儲存為 blob 中繼資料，並連同加密的 blob 一起儲存在服務上。

> [!WARNING]
> 如果您要為 blob 編輯或上傳您自己的中繼資料，則必須確定保留此中繼資料。 如果您上傳新的中繼資料，但缺少此中繼資料，包裝的 CEK、IV 和其他中繼資料將會遺失，而且永遠無法再擷取 blob 內容。
> 
> 

下載已加密的 blob 牽涉到使用 **download*/openInputStream** 便利方法來擷取整個 blob 的內容。 包裝的 CEK 會解除包裝，並與 IV (在此情況下儲存為 blob 中繼資料) 一起用來傳回解密的資料給使用者。

在加密的 Blob 中下載任意範圍 (**downloadRange*** 方法)，包含調整使用者所提供的範圍，藉此取得少量額外的資料以便用來成功解密所要求的範圍。  

所有 Blob 類型 (區塊 Blob、頁面 Blob 和附加 Blob) 都可以使用此機制進行加密/解密。

### <a name="queues"></a>佇列
因為佇列訊息可以是任何格式，用戶端程式庫會定義自訂的格式，其中包含訊息文字中的初始化向量 (IV) 和加密的內容加密金鑰 (CEK)。  

在加密期間，用戶端程式庫會產生 16 位元組的隨機 IV，以及 32 位元組的隨機 CEK，並使用這項資訊執行佇列訊息文字的信封加密。 然後，已包裝的 CEK 和一些其他加密中繼資料會加入至已加密的佇列訊息。 這個修改過的訊息 (如下所示) 會儲存在服務上。

```
<MessageText>{"EncryptedMessageContents":"6kOu8Rq1C3+M1QO4alKLmWthWXSmHV3mEfxBAgP9QGTU++MKn2uPq3t2UjF1DO6w","EncryptionData":{…}}</MessageText>
```

在解密期間，從佇列訊息中擷取已包裝的金鑰，並解除包裝。 IV 也會從佇列訊息中擷取，並與未包裝的金鑰一起用來解密佇列訊息資料。 請注意，加密中繼資料很小 (小於 500 位元組)，雖然計入佇列訊息的 64KB 限制內，但影響仍在可掌控的範圍內。

### <a name="tables"></a>資料表
用戶端程式庫支援在插入和取代作業時進行實體屬性的加密。

> [!NOTE]
> 目前不支援合併。 因為屬性子集先前可能是使用不同的金鑰加密，直接合併新的屬性並更新中繼資料會導致資料遺失。 合併可能需要額外的服務呼叫來從服務讀取預先存在的實體，或在每個屬性上都使用新的金鑰，兩者都不利用於效能。
> 
> 

資料表資料加密的運作方式如下：  

1. 使用者指定要加密的屬性。  
2. 用戶端程式庫會針對每個實體產生 16 位元組的隨機初始化向量 (IV) 以及 32 位元組的隨機內容加密金鑰 (CEK)，並在個別屬性上執行信封加密，使每個屬性衍生新的 IV，藉此進行加密。 加密的屬性會儲存為二進位資料。  
3. 然後，已包裝的 CEK 和一些其他加密中繼資料會儲存成額外兩個保留的屬性。 第一個保留的屬性 (_ClientEncryptionMetadata1) 是字串屬性，保存 IV、版本和已包裝的金鑰的相關資訊。 第二個保留的屬性 (_ClientEncryptionMetadata2) 二進位屬性，其保留已加密之屬性的相關資訊。 此第二個屬性 (_ClientEncryptionMetadata2) 中的資訊已自行加密。  
4. 由於加密需要這些額外保留的屬性，使用者現在可能只有 250 個自訂屬性，而不是 252 個。 實體的總大小必須小於 1MB。  
   
   請注意，只有字串屬性可以加密。 如果有其他類型的屬性需要加密，則必須轉換成字串。 加密的字串儲存在服務上作為二進位屬性，且解密後會轉換回字串。
   
   針對資料表，除了加密原則之外，使用者必須指定要加密的屬性。 作法是指定 [Encrypt] 屬性 (針對衍生自 TableEntity 的 POCO 實體)，或在要求選項中指定加密解析程式。 加密解析程式是委派，接受資料分割索引鍵、資料列索引鍵和屬性名稱，然後傳回布林值，指出是否應該加密該屬性。 在加密期間，用戶端程式庫會使用此資訊，決定將屬性在寫到網路時是否應該加密。 委派也提供關於屬性如何加密的可能邏輯。 (例如，如果 X，則加密屬性 A，否則加密屬性 A 和 B。)請注意，讀取或查詢實體時不需要提供這項資訊。

### <a name="batch-operations"></a>批次作業
在批次作業中，批次作業中的所有資料列上會使用相同的 KEK，因為用戶端程式庫只允許每個批次作業有一個選項物件 (也就是一個原則/KEK)。 不過，用戶端程式庫會在內部為批次中的每個資料列產生新的隨機 IV 和隨機 CEK。 使用者也可以選擇為批次中的每個作業加密不同的屬性，作法是在加密解析程式中定義此行為。

### <a name="queries"></a>查詢
> [!NOTE]
> 因為實體已加密，所以無法執行依加密屬性篩選的查詢。  如果您嘗試這麼做，結果會不正確，因為服務會嘗試比較加密資料與未加密資料。
> 
>
若要執行查詢作業，您必須指定一個能夠解析結果集中的所有金鑰的金鑰解析程式。 如果查詢結果中包含的實體無法解析成提供者，用戶端程式庫會擲回錯誤。 針對執行伺服器端投影的任何查詢，用戶端程式庫會依預設將特殊加密中繼資料屬性 (_ClientEncryptionMetadata1 和 _ClientEncryptionMetadata2) 加入選取的資料行。

## <a name="azure-key-vault"></a>Azure 金鑰保存庫
Azure 金鑰保存庫可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。 使用 Azure 金鑰保存庫時，使用者可以使用受硬體安全模組 (HSM) 保護的金鑰來加密金鑰和密碼 (例如驗證金鑰、儲存體帳戶金鑰、資料加密金鑰、.PFX 檔案和密碼)。 如需詳細資訊，請參閱 [什麼是 Azure 金鑰保存庫？](../../key-vault/key-vault-whatis.md)。

儲存體用戶端程式庫會使用金鑰保存庫核心程式庫，以提供整個 Azure 的通用架構來管理金鑰。 使用者也享有使用金鑰保存庫延伸模組程式庫的額外好處。 延伸模組程式庫提供實用的功能，包括簡單又完善的對稱/RSA 本機和雲端金鑰提供者，以及彙總和快取。

### <a name="interface-and-dependencies"></a>介面和相依性
有三個金鑰保存庫封裝：  

* azure-keyvault-core 包含 IKey 和 IKeyResolver。 這是不具有相依性的小型封裝。 Java 的儲存體用戶端程式庫會將其定義為相依。
* azure-keyvault 包含金鑰保存庫 REST 用戶端。  
* azure-keyvault-extensions 包含延伸模組程式碼，其中包含密碼編譯演算法的實作及 RSAKey 和 SymmetricKey。 它依賴 Core 和 KeyVault 命名空間，並提供功能來定義彙總解析程式 (當使用者想要使用多個金鑰提供者時) 和快取金鑰解析程式。 儲存體用戶端程式庫不直接依賴這個封裝，如果使用者想要使用 Azure 金鑰保存庫來儲存其金鑰，或使用金鑰保存庫延伸模組來取用本機和雲端密碼編譯提供者，則需要此封裝。  
  
  金鑰保存庫是專為高價值的主要金鑰而設計，個別金鑰保存庫的節流限制都以此為設計重點。 使用金鑰保存庫執行用戶端加密時，慣用的模型是使用在金鑰保存庫中儲存為密碼且在本機快取的對稱主要金鑰。 使用者必須執行下列動作：  

1. 離線建立密碼，再上載至金鑰保存庫。  
2. 使用密碼的基底識別碼做為參數，解析用於加密的目前密碼版本，並在本機快取這項資訊。 使用 CachingKeyResolver 進行快取。使用者不需要實作自己的快取邏輯。  
3. 建立加密原則時，使用快取解析程式做為輸入。
   如需有關「金鑰保存庫」使用方式的詳細資訊，請參閱加密程式碼範例。 <fix URL>  

## <a name="best-practices"></a>最佳作法
只有 Java 的儲存體用戶端程式庫才支援加密。

> [!IMPORTANT]
> 使用用戶端加密時，請留意以下重點：
> 
> * 讀取或寫入加密的 Blob 時，使用整個 Blob 上傳命令及範圍/整個 Blob 下載命令。 避免寫入使用通訊協定作業的加密 Blob，例如：放置區塊、放置區塊清單、寫入頁面、清除頁面或附加區塊；否則您可能會破壞加密 Blob 並使它無法讀取。
> * 對於資料表，存在類似的條件約束。 請小心在未更新加密中繼資料時即更新加密的內容。  
> * 如果您在加密 Blob 上設定中繼資料，您可能會覆寫加密所需的加密相關中繼資料，因為設定中繼資料不是加總解密。 快照集也是如此。在為加密的 Blob 建立快照集時，請避免指定中繼資料。 如果必須設定中繼資料，請務必先呼叫 **downloadAttributes** 方法，以取得目前的加密中繼資料，並避免在設定中繼資料時並行寫入。  
> * 在預設的要求選項中啟用 **requireEncryption** 旗標，使用者就應該只能使用加密的資料。 如需詳細資訊請參閱下方內容。  
> 
> 

## <a name="client-api--interface"></a>用戶端 API / 介面
使用者在建立 EncryptionPolicy 物件時，可以只提供金鑰 (實作 IKey)、只提供者解析程式 (實作 IKeyResolver)，或兩者都提供。 IKey 是以金鑰識別碼來識別的基本金鑰類型，且提供包裝/解除包裝的邏輯。 IKeyResolver 在解密程序期間用來解析金鑰。 它定義 ResolveKey 方法，可根據金鑰識別碼傳回 IKey。 這可讓使用者從多個位置中管理的多個金鑰之中選擇。

* 加密時一律使用金鑰，缺少金鑰將會導致錯誤。  
* 解密：  
  
  * 叫用金鑰解析程式 (如果指定) 以取得金鑰。 如果已指定解析程式，但沒有金鑰識別碼的對應，則會擲回錯誤。  
  * 如果未指定解析程式但指定了金鑰，則如果其識別項符合所需的金鑰識別項，就會使用該金鑰。 如果識別項不符合，則會擲回錯誤。  
    
    [加密範例](https://github.com/Azure/azure-storage-net/tree/master/Samples/GettingStarted/EncryptionSamples) <fix URL> 會針對 Blob、佇列和資料表以及金鑰保存庫的整合，示範更詳細的端對端案例。

### <a name="requireencryption-mode"></a>RequireEncryption 模式
使用者可以針對所有上傳和下載都必須加密的作業模式，從中選擇啟用。 在此模式中，在用戶端上嘗試上傳沒有加密原則的資料或下載未在服務上加密的資料將會失敗。 要求選項物件的 **requireEncryption** 旗標會控制此行為。 如果您的應用程式會將所有儲存在 Azure 儲存體中的物件加密，則您可以在服務用戶端服務的預設要求選項上，設定 **requireEncryption** 屬性。   

例如，使用 **CloudBlobClient.getDefaultRequestOptions().setRequireEncryption(true)** 要求加密透過該用戶端物件所執行的所有 Blob 作業。

### <a name="blob-service-encryption"></a>Blob 服務加密
建立 **BlobEncryptionPolicy** 物件，並在要求選項中加以設定 (透過 API 或在用戶端層級使用 **DefaultRequestOptions**)。 其他一切由用戶端程式庫在內部處理。

```java
// Create the IKey used for encryption.
RsaKey key = new RsaKey("private:key1" /* key identifier */);

// Create the encryption policy to be used for upload and download.
BlobEncryptionPolicy policy = new BlobEncryptionPolicy(key, null);

// Set the encryption policy on the request options.
BlobRequestOptions options = new BlobRequestOptions();
options.setEncryptionPolicy(policy);

// Upload the encrypted contents to the blob.
blob.upload(stream, size, null, options, null);

// Download and decrypt the encrypted contents from the blob.
ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
blob.download(outputStream, null, options, null);
```

### <a name="queue-service-encryption"></a>佇列服務加密
建立 **QueueEncryptionPolicy** 物件，並在要求選項中加以設定 (透過 API 或在用戶端層級使用 **DefaultRequestOptions**)。 其他一切由用戶端程式庫在內部處理。

```java
// Create the IKey used for encryption.
RsaKey key = new RsaKey("private:key1" /* key identifier */);

// Create the encryption policy to be used for upload and download.
QueueEncryptionPolicy policy = new QueueEncryptionPolicy(key, null);

// Add message
QueueRequestOptions options = new QueueRequestOptions();
options.setEncryptionPolicy(policy);

queue.addMessage(message, 0, 0, options, null);

// Retrieve message
CloudQueueMessage retrMessage = queue.retrieveMessage(30, options, null);
```

### <a name="table-service-encryption"></a>資料表服務加密
除了建立加密原則並在要求選項上加以設定之外，您必須在 **TableRequestOptions** 中指定 **EncryptionResolver**，或在實體的 getter 與 setter 上設定 [Encrypt] 屬性。

### <a name="using-the-resolver"></a>使用解析程式

```java
// Create the IKey used for encryption.
RsaKey key = new RsaKey("private:key1" /* key identifier */);

// Create the encryption policy to be used for upload and download.
TableEncryptionPolicy policy = new TableEncryptionPolicy(key, null);

TableRequestOptions options = new TableRequestOptions()
options.setEncryptionPolicy(policy);
options.setEncryptionResolver(new EncryptionResolver() {
    public boolean encryptionResolver(String pk, String rk, String key) {
        if (key == "foo")
        {
            return true;
        }
        return false;
    }
});

// Insert Entity
currentTable.execute(TableOperation.insert(ent), options, null);

// Retrieve Entity
// No need to specify an encryption resolver for retrieve
TableRequestOptions retrieveOptions = new TableRequestOptions()
retrieveOptions.setEncryptionPolicy(policy);

TableOperation operation = TableOperation.retrieve(ent.PartitionKey, ent.RowKey, DynamicTableEntity.class);
TableResult result = currentTable.execute(operation, retrieveOptions, null);
```

### <a name="using-attributes"></a>使用屬性
如上所述，如果實體實作 TableEntity，則屬性可以使用 [Encrypt] 屬性裝飾 getter 與 setter 屬性，不需指定 **EncryptionResolver**。

```java
private string encryptedProperty1;

@Encrypt
public String getEncryptedProperty1 () {
    return this.encryptedProperty1;
}

@Encrypt
public void setEncryptedProperty1(final String encryptedProperty1) {
    this.encryptedProperty1 = encryptedProperty1;
}
```

## <a name="encryption-and-performance"></a>加密和效能
請注意，加密您的儲存體資料會造成額外的效能負擔。 必須產生內容金鑰和 IV，內容本身必須經過加密，而且其他中繼資料必須格式化並上傳。 這個額外負荷會因所加密的資料數量而有所不同。 我們建議客戶一定要在開發期間測試其應用程式的效能。

## <a name="next-steps"></a>後續步驟
* 下載 [Azure Storage Client Library for Java Maven package (適用於 Java Maven 封裝的 Azure 儲存體用戶端程式庫)](http://mvnrepository.com/artifact/com.microsoft.azure/azure-storage)  
* 從 GitHub 下載 [Azure Storage Client Library for Java Source Code (適用於 Java 原始程式碼的 Azure 儲存體用戶端程式庫)](https://github.com/Azure/azure-storage-java)   
* 下載適用於 Java Maven 的 Azure 金鑰保存庫 Maven 程式庫封裝：
  * [核心](http://mvnrepository.com/artifact/com.microsoft.azure/azure-keyvault-core) 封裝
  * [用戶端](http://mvnrepository.com/artifact/com.microsoft.azure/azure-keyvault) 封裝
* 請瀏覽 [Azure 金鑰保存庫文件](../../key-vault/key-vault-whatis.md)
