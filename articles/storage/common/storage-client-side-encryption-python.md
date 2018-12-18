---
title: Microsoft Azure 儲存體的用戶端 Python 加密 | Microsoft Docs
description: 適用於 Python 的 Azure 儲存體用戶端程式庫支援適用於您 Azure 儲存體應用程式的最高安全性用戶端加密。
services: storage
author: lakasa
ms.service: storage
ms.devlang: python
ms.topic: article
ms.date: 05/11/2017
ms.author: lakasa
ms.component: common
ms.openlocfilehash: 6a6508393fe935b456cde815d35f2fd4447cd2d4
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528117"
---
# <a name="client-side-encryption-with-python-for-microsoft-azure-storage"></a>Microsoft Azure 儲存體的用戶端 Python 加密
[!INCLUDE [storage-selector-client-side-encryption-include](../../../includes/storage-selector-client-side-encryption-include.md)]

## <a name="overview"></a>概觀
[Azure Storage Client Library for Python (適用於 Python 的 Azure 儲存體用戶端程式庫)](https://pypi.python.org/pypi/azure-storage) 支援在上傳至 Azure 儲存體之前將用戶端應用程式內的資料加密，並在下載至用戶端時解密資料。

> [!NOTE]
> Azure Storage Python 程式庫目前為預覽狀態。
> 
> 

## <a name="encryption-and-decryption-via-the-envelope-technique"></a>透過信封技術進行加密和解密
加密和解密的程序採用信封技術。

### <a name="encryption-via-the-envelope-technique"></a>透過信封技術加密
透過信封技術加密的運作方式如下：

1. Azure 儲存體用戶端程式庫會產生內容加密金鑰 (CEK)，這是使用一次的對稱金鑰。
2. 使用此 CEK 加密使用者資料。
3. 然後使用金鑰加密金鑰 (KEK) 包裝 (加密) CEK。 KEK 是以金鑰識別碼識別，且可以是在本機管理的非對稱金鑰組或對稱金鑰。
   儲存體用戶端程式庫本身永遠沒有 KEK 的存取權。 程式庫會叫用 KEK 所提供的金鑰包裝演算法。 如有需要，使用者可以選擇使用自訂提供者來包裝/取消包裝金鑰。
4. 然後，將加密的資料上傳至 Azure 儲存體服務。 包裝的金鑰及一些其他加密中繼資料會儲存為中繼資料 (在 blob 上)，或插補加密的資料 (訊息佇列和資料表實體)。

### <a name="decryption-via-the-envelope-technique"></a>透過信封技術解密
透過信封技術解密的運作方式如下：

1. 用戶端程式庫假設使用者在本機管理金鑰加密金鑰 (KEK)。 使用者不必知道用於加密的特定金鑰。 相反地，可以設定並使用金鑰解析程式，將不同的金鑰識別碼解析成金鑰。
2. 用戶端程式庫會下載加密的資料，以及儲存在服務上的任何加密資料。
3. 然後，使用金鑰加密金鑰 (KEK) 將已包裝的內容加密金鑰 (CEK) 解除包裝 (解密)。 同樣地，用戶端程式庫在此沒有 KEK 的存取權。 它只會叫用自訂提供者的解除包裝演算法。
4. 然後，使用內容加密金鑰 (CEK) 解密已加密的使用者資料。

## <a name="encryption-mechanism"></a>加密機制
儲存體用戶端程式庫會使用 [AES](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 來加密使用者資料。 具體來說，就是 [加密區塊鏈結 (CBC)](http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) 模式搭配 AES。 每個服務的運作方式稍有不同，我們將在這裡討論每個服務。

### <a name="blobs"></a>Blob
用戶端程式庫目前僅支援整個 Blob 的加密。 明確地說，加密會在使用者使用 **create*** 方法時受到支援。 針對下載項目同時支援完整與範圍下載，也提供上傳與下載的平行處理。

在加密期間，用戶端程式庫會產生 16 位元組的隨機初始化向量 (IV)，以及 32 位元組的隨機內容加密金鑰 (CEK)，並使用這項資訊執行 blob 資料的信封加密。 然後，已包裝的 CEK 和一些其他加密中繼資料會儲存為 blob 中繼資料，並連同加密的 blob 一起儲存在服務上。

> [!WARNING]
> 如果您要為 blob 編輯或上傳您自己的中繼資料，則必須確定保留此中繼資料。 如果您上傳新的中繼資料，但缺少此中繼資料，包裝的 CEK、IV 和其他中繼資料將會遺失，而且永遠無法再擷取 blob 內容。
> 
> 

下載加密的 Blob 牽涉到使用 **get*** 便利性方法擷取整個 Blob 內容。 包裝的 CEK 會解除包裝，並與 IV (在此情況下儲存為 blob 中繼資料) 一起用來傳回解密的資料給使用者。

在加密的 Blob 中下載任意範圍 (包含傳入的範圍參數的**get*** 方法)，牽涉到調整使用者所提供的範圍，以藉此取得少量額外的資料以便用來成功解密所要求的範圍。

區塊 Blob 和分頁 Blob 只可以使用這種配置加密/解密。 目前不支援加密附加 Blob。

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
3. 然後，已包裝的 CEK 和一些其他加密中繼資料會儲存成額外兩個保留的屬性。 第一個保留的屬性 (\_ClientEncryptionMetadata1) 是字串屬性，保存 IV、版本和已包裝的金鑰的相關資訊。 第二個保留的屬性 (\_ClientEncryptionMetadata2) 二進位屬性，其保留已加密之屬性的相關資訊。 此第二個屬性 (\_ClientEncryptionMetadata2) 中的資訊已自行加密。
4. 由於加密需要這些額外保留的屬性，使用者現在可能只有 250 個自訂屬性，而不是 252 個。 實體的總大小必須小於 1MB。
   
   請注意，只有字串屬性可以加密。 如果有其他類型的屬性需要加密，則必須轉換成字串。 加密的字串會做為二進位屬性儲存在服務上，並在解密後轉換回字串 (原始字串，而不是具有 EdmType.STRING 類型的 EntityProperties)。
   
   針對資料表，除了加密原則之外，使用者必須指定要加密的屬性。 這可透過將這些屬性儲存於 TableEntity 物件中並設定類型為 EdmType.STRING 且將加密設定為 true，或是在 tableservice 物件上設定 encryption_resolver_function。 加密解析程式是函式，接受資料分割索引鍵、資料列索引鍵和屬性名稱，然後傳回布林值，指出是否應該加密該屬性。 在加密期間，用戶端程式庫會使用此資訊，決定將屬性在寫到網路時是否應該加密。 委派也提供關於屬性如何加密的可能邏輯。 (例如，如果 X，則加密屬性 A，否則加密屬性 A 和 B。)請注意，讀取或查詢實體時不需要提供這項資訊。

### <a name="batch-operations"></a>批次作業
單一加密原則適用於批次中的所有資料列。 用戶端程式庫會在內部為批次中的每個資料列產生新的隨機 IV 和隨機 CEK。 使用者也可以選擇為批次中的每個作業加密不同的屬性，作法是在加密解析程式中定義此行為。
如果批次透過 tableservice batch() 方法建立為內容管理員，則 tableservice 加密原則會自動套用至批次。 如果批次是由呼叫建構函式而明確建立，則加密原則就必須做為參數傳遞，並在批次的存留期內保持未修改的狀態。
請注意，實體會在使用批次的加密原則插入批次中時受到加密 (實體在使用 tableservice 的加密原則認可批次時將不會受到加密)。

### <a name="queries"></a>查詢
> [!NOTE]
> 因為實體已加密，所以無法執行依加密屬性篩選的查詢。  如果您嘗試這麼做，結果會不正確，因為服務會嘗試比較加密資料與未加密資料。
> 
>
若要執行查詢作業，您必須指定一個能夠解析結果集中的所有金鑰的金鑰解析程式。 如果查詢結果中包含的實體無法解析成提供者，用戶端程式庫會擲回錯誤。 針對執行伺服器端投影的任何查詢，用戶端程式庫會依預設將特殊加密中繼資料屬性 (\_ClientEncryptionMetadata1 和 \_ClientEncryptionMetadata2) 加入選取的資料行。

> [!IMPORTANT]
> 使用用戶端加密時，請留意以下重點：
> 
> * 讀取或寫入加密的 Blob 時，使用整個 Blob 上傳命令及範圍/整個 Blob 下載命令。 避免使用通訊協定作業寫入加密的 Blob，例如放置區塊、放置區塊清單、撰寫頁面或清除頁面；否則您可能會損毀加密的 Blob，並使它無法讀取。
> * 對於資料表，存在類似的條件約束。 請小心在未更新加密中繼資料時即更新加密的內容。
> * 如果您在加密 Blob 上設定中繼資料，您可能會覆寫加密所需的加密相關中繼資料，因為設定中繼資料不是加總解密。 快照集也是如此。在為加密的 Blob 建立快照集時，請避免指定中繼資料。 如果必須設定中繼資料，請務必先呼叫 **get_blob_metadata** 方法，以取得目前的加密中繼資料，並避免在設定中繼資料時並行寫入。
> * 針對僅能使用加密資料的使用者，請在服務物件上啟用 **require_encryption** 旗標。 如需詳細資訊請參閱下方內容。
> 
> 

儲存體用戶端程式庫預期提供的 KEK 與金鑰解析程式會實作下列介面。 [Azure 金鑰保存庫](https://azure.microsoft.com/services/key-vault/) 支援尚未推出，並會在完成時整合到此程式庫。

## <a name="client-api--interface"></a>用戶端 API / 介面
在建立儲存體服務物件 (亦即 blockblobservice) 之後，使用者可以將值指派到構成加密原則的欄位：key_encryption_key、key_resolver_function 和 require_encryption。 使用者可以僅提供 KEK、僅提供解析程式，或是兩者皆提供。 key_encryption_key 是以金鑰識別碼來識別的基本金鑰類型，且提供包裝/解除包裝的邏輯。 key_resolver_function 在解密程序期間用來解析金鑰。 它會傳回指定金鑰識別碼的有效 KEK。 這可讓使用者從多個位置中管理的多個金鑰之中選擇。

KEK 必須實作下列方法以成功加密資料︰

* wrap_key(cek)：使用使用者選擇的演算法包裝指定的 CEK (位元組)。 傳回已包裝的金鑰。
* get_key_wrap_algorithm()：傳回用來包裝金鑰的演算法。
* get_kid()：傳回此 KEK 的字串金鑰識別碼。
  KEK 必須實作下列方法以成功解密資料︰
* unwrap_key(cek, algorithm)：使用字串指定演算法傳回未包裝形式的指定 CEK。
* get_kid()：傳回此 KEK 的字串金鑰識別碼。

金鑰解析程式至少必須實作在指定金鑰識別碼的情況下會傳回實作上述介面之對應 KEK 的方法。 只有這個方法可以指派給服務物件上的 key_resolver_function 屬性。

* 加密時一律使用金鑰，缺少金鑰將會導致錯誤。
* 解密：
  
  * 叫用金鑰解析程式 (如果指定) 以取得金鑰。 如果已指定解析程式，但沒有金鑰識別碼的對應，則會擲回錯誤。
  * 如果未指定解析程式但指定了金鑰，則如果其識別項符合所需的金鑰識別項，就會使用該金鑰。 如果識別項不符合，則會擲回錯誤。
    
    azure.storage.samples <fix URL> 中的加密範例示範了針對 Blob、佇列及資料表更為詳細的端對端案例。
      KEK 與金鑰解析程式的實作範例已在範例檔案中以 KeyWrapper 和 KeyResolver 的形式個別提供。

### <a name="requireencryption-mode"></a>RequireEncryption 模式
使用者可以針對所有上傳和下載都必須加密的作業模式，從中選擇啟用。 在此模式中，在用戶端上嘗試上傳沒有加密原則的資料或下載未在服務上加密的資料將會失敗。 服務物件上的 **require_encryption** 旗標會控制此行為。

### <a name="blob-service-encryption"></a>Blob 服務加密
在 blockblobservice 物件上設定加密原則欄位。 其他一切由用戶端程式庫在內部處理。

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1') # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_block_blob_service.key_encryption_key = kek
my_block_blob_service.key_resolver_funcion = key_resolver.resolve_key

# Upload the encrypted contents to the blob.
my_block_blob_service.create_blob_from_stream(container_name, blob_name, stream)

# Download and decrypt the encrypted contents from the blob.
blob = my_block_blob_service.get_blob_to_bytes(container_name, blob_name)
```

### <a name="queue-service-encryption"></a>佇列服務加密
在 queueservice 物件上設定加密原則欄位。 其他一切由用戶端程式庫在內部處理。

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1') # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_queue_service.key_encryption_key = kek
my_queue_service.key_resolver_funcion = key_resolver.resolve_key

# Add message
my_queue_service.put_message(queue_name, content)

# Retrieve message
retrieved_message_list = my_queue_service.get_messages(queue_name)
```

### <a name="table-service-encryption"></a>資料表服務加密
除了建立加密原則並在要求選項上設定它之外，您必須在 **tableservice** 上指定 **encryption_resolver_function**，或在 EntityProperty 上設定加密屬性。

### <a name="using-the-resolver"></a>使用解析程式

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1') # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Define the encryption resolver_function.
def my_encryption_resolver(pk, rk, property_name):
    if property_name == 'foo':
        return True
    return False

# Set the KEK and key resolver on the service object.
my_table_service.key_encryption_key = kek
my_table_service.key_resolver_funcion = key_resolver.resolve_key
my_table_service.encryption_resolver_function = my_encryption_resolver

# Insert Entity
my_table_service.insert_entity(table_name, entity)

# Retrieve Entity
# Note: No need to specify an encryption resolver for retrieve, but it is harmless to leave the property set.
my_table_service.get_entity(table_name, entity['PartitionKey'], entity['RowKey'])
```

### <a name="using-attributes"></a>使用屬性
如上所述，透過將屬性儲存在 EntityProperty 物件中並設定加密欄位，便可能對該屬性進行加密標記。

```python
encrypted_property_1 = EntityProperty(EdmType.STRING, value, encrypt=True)
```

## <a name="encryption-and-performance"></a>加密和效能
請注意，加密您的儲存體資料會造成額外的效能負擔。 必須產生內容金鑰和 IV，內容本身必須經過加密，而且其他中繼資料必須格式化並上傳。 這個額外負荷會因所加密的資料數量而有所不同。 我們建議客戶一定要在開發期間測試其應用程式的效能。

## <a name="next-steps"></a>後續步驟
* 下載 [適用於 Java PyPi 封裝的 Azure 儲存體用戶端程式庫](https://pypi.python.org/pypi/azure-storage)
* 從 GitHub 下載 [適用於 Python 的 Azure 儲存體用戶端程式庫來源程式碼](https://github.com/Azure/azure-storage-python)
