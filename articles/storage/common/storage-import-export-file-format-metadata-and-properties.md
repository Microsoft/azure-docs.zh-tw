---
title: Azure 匯入/匯出中繼資料和屬性檔案格式 | Microsoft Docs
description: 了解如何針對匯入或匯出作業中的一或多個 blob 指定中繼資料和屬性。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 5a886244b43ad006a95e9be0350d9c69fd987ad9
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526227"
---
# <a name="azure-importexport-service-metadata-and-properties-file-format"></a>Azure 匯入/匯出服務中繼資料和屬性檔案格式
您可以針對匯入或匯出作業中的一或多個 blob 指定中繼資料和屬性。 若要針對建立為匯入作業一部分的 blob 設定中繼資料或屬性，您可在包含要匯入資料的硬碟上提供中繼資料或屬性檔案。 若為匯出作業，中繼資料和屬性會寫入至中繼資料或屬性檔案 (包含在傳回給您的硬碟上)。  
  
## <a name="metadata-file-format"></a>中繼資料檔案格式  
中繼資料檔案的格式如下所示：  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<Metadata>  
[<metadata-name-1>metadata-value-1</metadata-name-1>]  
[<metadata-name-2>metadata-value-2</metadata-name-2>]  
. . .  
</Metadata>  
```
  
|XML 元素|類型|說明|  
|-----------------|----------|-----------------|  
|`Metadata`|根元素|中繼資料檔案的根項目。|  
|`metadata-name`|字串|選用。 XML 元素可指定 blob 的中繼資料名稱，而其值可指定中繼資料設定的值。|  
  
## <a name="properties-file-format"></a>屬性檔案格式  
屬性檔案的格式如下所示：  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<Properties>  
[<Last-Modified>date-time-value</Last-Modified>]  
[<Etag>etag</Etag>]  
[<Content-Length>size-in-bytes<Content-Length>]  
[<Content-Type>content-type</Content-Type>]  
[<Content-MD5>content-md5</Content-MD5>]  
[<Content-Encoding>content-encoding</Content-Encoding>]  
[<Content-Language>content-language</Content-Language>]  
[<Cache-Control>cache-control</Cache-Control>]  
</Properties>  
```
  
|XML 元素|類型|說明|  
|-----------------|----------|-----------------|  
|`Properties`|根元素|屬性檔案的根元素。|  
|`Last-Modified`|字串|選用。 Blob 上次修改時間。 僅限匯出作業。|  
|`Etag`|字串|選用。 Blob 的 ETag 值。 僅限匯出作業。|  
|`Content-Length`|字串|選用。 Blob 大小 (以位元組為單位)。 僅限匯出作業。|  
|`Content-Type`|字串|選用。 Blob 的內容類型。|  
|`Content-MD5`|字串|選用。 Blob 的 MD5 雜湊。|  
|`Content-Encoding`|字串|選用。 Blob 的內容編碼。|  
|`Content-Language`|字串|選用。 Blob 的內容語言。|  
|`Cache-Control`|字串|選用。 Blob 的快取控制字串。|  

## <a name="next-steps"></a>後續步驟

如需有關設定 blob 中繼資料和屬性的詳細規則，請參閱[設定 Blob 屬性](/rest/api/storageservices/set-blob-properties)、[設定 Blob 中繼資料](/rest/api/storageservices/set-blob-metadata)和[設定及擷取 Blob 資源的屬性和中繼資料](/rest/api/storageservices/setting-and-retrieving-properties-and-metadata-for-blob-resources)。
