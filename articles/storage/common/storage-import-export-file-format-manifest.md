---
title: Azure 匯入/匯出資訊清單檔案格式 | Microsoft Docs
description: 了解磁碟機資訊清單檔案的格式，此檔案描述在 Azure Blob 儲存體中的 Blob 以及匯入/匯出服務內匯入或匯出作業中磁碟機上的檔案之間的對應。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 920f350ab5ba1e9e1703ffcc32dc8c7153624c0b
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525149"
---
# <a name="azure-importexport-service-manifest-file-format"></a>Azure 匯入/匯出服務資訊清單檔案格式
磁碟機資訊清單檔案描述 Azure Blob 儲存體中的 Blob 與包含匯入或匯出工作的磁碟機上的檔案之間的對應。 針對匯入作業，在磁碟機準備程序期間會建立資訊清單檔案，此檔案在磁碟機傳送至 Azure 資料中心之前會儲存在磁碟機上。 匯出作業期間，Azure 匯入/匯出服務會建立資訊清單，並將其儲存在磁碟機上。  
  
針對匯入和匯出工作，磁碟機資訊清單檔案會儲存在匯入或匯出磁碟機；它不會透過任何 API 作業傳送至服務。  
  
以下描述磁碟機資訊清單檔案的一般格式︰  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<DriveManifest Version="2014-11-01">  
  <Drive>  
    <DriveId>drive-id</DriveId>  
    import-export-credential  
  
    <!-- First Blob List -->  
    <BlobList>  
      <!-- Global properties and metadata that applies to all blobs -->  
      [<MetadataPath Hash="md5-hash">global-metadata-file-path</MetadataPath>]  
      [<PropertiesPath   
        Hash="md5-hash">global-properties-file-path</PropertiesPath>]  
  
      <!-- First Blob -->  
      <Blob>  
        <BlobPath>blob-path-relative-to-account</BlobPath>  
        <FilePath>file-path-relative-to-transfer-disk</FilePath>  
        [<ClientData>client-data</ClientData>]  
        [<Snapshot>snapshot</Snapshot>]  
        <Length>content-length</Length>  
        [<ImportDisposition>import-disposition</ImportDisposition>]  
        page-range-list-or-block-list          
        [<MetadataPath Hash="md5-hash">metadata-file-path</MetadataPath>]  
        [<PropertiesPath Hash="md5-hash">properties-file-path</PropertiesPath>]  
      </Blob>  
  
      <!-- Second Blob -->  
      <Blob>  
      . . .  
      </Blob>  
    </BlobList>  
  
    <!-- Second Blob List -->  
    <BlobList>  
    . . .  
    </BlobList>  
  </Drive>  
</DriveManifest>  
  
import-export-credential ::=   
  <StorageAccountKey>storage-account-key</StorageAccountKey> | <ContainerSas>container-sas</ContainerSas>  
  
page-range-list-or-block-list ::=   
  page-range-list | block-list  
  
page-range-list ::=   
    <PageRangeList>  
      [<PageRange Offset="page-range-offset" Length="page-range-length"   
       Hash="md5-hash"/>]  
      [<PageRange Offset="page-range-offset" Length="page-range-length"   
       Hash="md5-hash"/>]  
    </PageRangeList>  
  
block-list ::=  
    <BlockList>  
      [<Block Offset="block-offset" Length="block-length" [Id="block-id"]  
       Hash="md5-hash"/>]  
      [<Block Offset="block-offset" Length="block-length" [Id="block-id"]   
       Hash="md5-hash"/>]  
    </BlockList>  

```

## <a name="manifest-xml-elements-and-attributes"></a>資訊清單的 XML 元素和屬性

下表指定磁碟機資訊清單 XML 格式的資料元素和屬性。  
  
|XML 元素|類型|說明|  
|-----------------|----------|-----------------|  
|`DriveManifest`|根元素|資訊清單檔案的根元素。 檔案中的其他所有元素則列於此元素下方。|  
|`Version`|屬性、字串|資訊清單檔案的版本。|  
|`Drive`|巢狀的 XML 元素|包含每個磁碟機的資訊清單。|  
|`DriveId`|字串|磁碟機的唯一磁碟機識別元。 藉由查詢磁碟機序號，即可找出磁碟機識別元。 磁碟機序號通常也會印在磁碟機外部。 `DriveID` 元素必須顯示在資訊清單檔案中的任何 `BlobList` 元素之前。|  
|`StorageAccountKey`|字串|如果未指定 (且只有在未指定) `ContainerSas` 時，才需為匯入工作指定。 和工作相關聯的 Azure 儲存體帳戶的帳戶金鑰。<br /><br /> 匯出作業的資訊清單中會省略這個元素。|  
|`ContainerSas`|字串|如果未指定 (且只有在未指定) `StorageAccountKey` 時，才需為匯入工作指定。 存取與工作相關聯的 Blob 所用的容器 SAS。 請參閱 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 了解其格式。匯出作業的資訊清單中會省略這個元素。|  
|`ClientCreator`|字串|指定建立 XML 檔案的用戶端。 匯入/匯出服務無法解譯這個值。|  
|`BlobList`|巢狀的 XML 元素|包含屬於匯入或匯出工作的 Blob 的清單。 Blob 清單中的每個 Blob 會共用相同的中繼資料和屬性。|  
|`BlobList/MetadataPath`|字串|選用。 指定磁碟上的檔案的相對路徑，該檔案包含將在匯入作業的 Blob 清單中的 Blob 設定的預設中繼資料。 您可以選擇以個別的 Blob 為基礎來覆寫此中繼資料。<br /><br /> 匯出作業的資訊清單中會省略這個元素。|  
|`BlobList/MetadataPath/@Hash`|屬性、字串|為中繼資料檔案指定 Base16 式編碼的 MD5 雜湊值。|  
|`BlobList/PropertiesPath`|字串|選用。 指定磁碟上的檔案的相對路徑，該檔案包含將在匯入作業的 Blob 清單中的 Blob 設定的預設屬性。 您可以選擇以個別的 Blob 為基礎來覆寫這些屬性。<br /><br /> 匯出作業的資訊清單中會省略這個元素。|  
|`BlobList/PropertiesPath/@Hash`|屬性、字串|為屬性檔案指定 Base16 式編碼的 MD5 雜湊值。|  
|`Blob`|巢狀的 XML 元素|包含每個 Blob 清單中的每個 Blob 的相關資訊。|  
|`Blob/BlobPath`|字串|Blob 的相對 URI，開頭為容器名稱。 如果 blob 位於根容器中，其開頭需為 `$root`。|  
|`Blob/FilePath`|字串|指定磁碟機上檔案的相對路徑。 針對匯出工作，Blob 路徑將用於檔案路徑 (如果可行)；例如，`pictures/bob/wild/desert.jpg` 將匯出到 `\pictures\bob\wild\desert.jpg`。 不過，由於 NTFS 名稱的限制，Blob 可能會匯出至路徑不像 Blob 路徑的檔案。|  
|`Blob/ClientData`|字串|選用。 包含來自客戶的意見。 匯入/匯出服務無法解譯這個值。|  
|`Blob/Snapshot`|Datetime|如為匯出工作則為選擇性。 指定匯出的 Blob 快照集的快照集識別碼。|  
|`Blob/Length`|整數 |指定 Blob 的總長度 (以位元組為單位)。 區塊 Blob 的這個值可能高達 200 GB，分頁 Blob 的這個值則可能高達 1 TB。 對於分頁 Blob，此值需為 512 的倍數。|  
|`Blob/ImportDisposition`|字串|如為匯入工作則為選擇性，匯出工作可省略。 這會指定匯入/匯出服務應如何處理匯入工作中已存在同名的 Blob 的案例。 如果匯入資訊清單中省略此值，則預設值為 `rename`。<br /><br /> 此元素的值包括︰<br /><br /> -   `no-overwrite`︰如果已經有同名的目的地 Blob 存在，匯入作業會略過匯入這個檔案。<br />-   `overwrite`︰新匯入的檔案會完全覆寫任何現有的目的地 Blob。<br />-   `rename`︰新的 Blob 將使用修改過的名稱上傳。<br /><br /> 重新命名規則如下︰<br /><br /> - 如果 Blob 名稱未包含點，將 `(2)` 附加到原始 Blob 名稱會產生新的名稱；如果這個新名稱也與現有的 Blob 名稱衝突，則會附加 `(3)` 來取代 `(2)`；依此類推。<br />- 如果 Blob 名稱包含點，最後一個點之後的部分會被視為副檔名。 類似於上述的程序，`(2)` 會插入到最後一個點之前以產生新名稱；如果新名稱仍與現有的 Blob 名稱衝突，服務會嘗試 `(3)`、`(4)`，依此類推，直到找到沒有衝突的名稱。<br /><br /> 部分範例如下：<br /><br /> Blob `BlobNameWithoutDot` 會重新命名為︰<br /><br /> `BlobNameWithoutDot (2)  // if BlobNameWithoutDot exists`<br /><br /> `BlobNameWithoutDot (3)  // if both BlobNameWithoutDot and BlobNameWithoutDot (2) exist`<br /><br /> Blob `Seattle.jpg` 會重新命名為︰<br /><br /> `Seattle (2).jpg  // if Seattle.jpg exists`<br /><br /> `Seattle (3).jpg  // if both Seattle.jpg and Seattle (2).jpg exist`|  
|`PageRangeList`|巢狀的 XML 元素|分頁 Blob 的必要元素。<br /><br /> 針對匯入作業，請指定要匯入檔案的位元組範圍清單。 每個頁面範圍是由一個位移和描述頁面範圍的來源檔案中的長度及區域的 MD5 雜湊所描述。 頁面範圍的 `Hash` 屬性是必要屬性。 服務會驗證 Blob 中的資料雜湊是否符合從頁面範圍計算的 MD5 雜湊。 用來描述要匯入的檔案的頁面範圍數目沒有限制，總大小可高達 1 TB。 所有頁面範圍必須按照位移排序，且不允許任何重疊。<br /><br /> 針對匯出作業，請指定已匯出至磁碟機的 Blob 的一組位元組範圍。<br /><br /> 頁面範圍可以只覆蓋 Blob 或檔案的子範圍。  可能會有任何頁面範圍未覆蓋的其餘檔案部分，而且其內容可以是未定義。|  
|`PageRange`|XML 元素|代表頁面範圍。|  
|`PageRange/@Offset`|屬性、整數|指定傳輸檔案中的位移和指定的頁面範圍開始處的 Blob。 此值需為 512 的倍數。|  
|`PageRange/@Length`|屬性、整數|指定頁面範圍的長度。 此值需為 512 的倍數，且不得超過 4 MB。|  
|`PageRange/@Hash`|屬性、字串|為頁面範圍指定 Base16 式編碼的 MD5 雜湊值。|  
|`BlockList`|巢狀的 XML 元素|含有具名區塊的區塊 Blob 的必要元素。<br /><br /> 針對匯入作業，區塊清單會指定將匯入到 Azure 儲存體的一組區塊。 針對匯出作業，區塊清單會指定匯出磁碟機上的檔案中已儲存的每個區塊的位置。 每個區塊是由該檔案中的位移和一個區塊長度所描述；每個區塊再由一個區塊識別碼屬性命名，並且包含區塊的 MD5 雜湊。 最多可使用 50,000 個區塊來描述一個 Blob。  所有區塊必須按照位移排序，並能覆蓋檔案的完整範圍，，也就是說，區塊之間必須沒有任何間距。 如果 Blob 小於 64 MB，每個區塊的區塊識別碼必須全部不存在或全部存在。 區塊識別碼必須是 Base64 式編碼的字串。 請參閱 [Put Block](/rest/api/storageservices/put-block)以了解區塊識別碼的進一步需求。|  
|`Block`|XML 元素|代表區塊。|  
|`Block/@Offset`|屬性、整數|指定指定的區塊開始處的位移。|  
|`Block/@Length`|屬性、整數|指定區塊中的位元組數目；此值不得超過 4 MB。|  
|`Block/@Id`|屬性、字串|指定代表區塊的區塊識別碼的字串。|  
|`Block/@Hash`|屬性、字串|指定區塊的 Base16 式編碼的 MD5 雜湊。|  
|`Blob/MetadataPath`|字串|選用。 指定中繼資料檔案的相對路徑。 匯入期間，會在目的地 Blob 上設定中繼資料。 匯出作業期間，Blob 的中繼資料會儲存在磁碟機上的中繼資料檔案。|  
|`Blob/MetadataPath/@Hash`|屬性、字串|為 Blob 的中繼資料檔案指定 Base16 式編碼的 MD5 雜湊。|  
|`Blob/PropertiesPath`|字串|選用。 指定屬性檔案的相對路徑。 匯入期間，會在目的地 Blob 上設定屬性。 匯出作業期間，Blob 屬性會儲存在磁碟機上的屬性檔案。|  
|`Blob/PropertiesPath/@Hash`|屬性、字串|為 Blob 的屬性檔案指定 Base16 式編碼的 MD5 雜湊。|  
  
## <a name="next-steps"></a>後續步驟
 
* [儲存體匯入/匯出 REST API](/rest/api/storageimportexport/)
