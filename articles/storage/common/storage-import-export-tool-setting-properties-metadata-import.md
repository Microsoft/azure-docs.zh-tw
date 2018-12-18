---
title: 使用 Azure 匯入/匯出來設定屬性和中繼資料 | Microsoft Docs
description: 了解執行 Azure 匯入/匯出工具準備磁碟機時，如何指定要在目的地 blob 上設定的屬性和中繼資料。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: d68de35591d1b17a62278a0bc4adf2b9ee20cf02
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39524996"
---
# <a name="setting-properties-and-metadata-during-the-import-process"></a>在匯入程序期間設定屬性和中繼資料

當您執行 Microsoft Azure 匯入/匯出工具準備磁碟機時，可以指定要在目的地 blob 上設定的屬性和中繼資料。 請遵循下列步驟：

1.  若要設定 blob 屬性，建立本機電腦上指定屬性名稱和值的文字檔案。
2.  若要設定 blob 中繼資料，建立本機電腦上指定中繼資料名稱和值的文字檔案。
3.  將完整路徑傳遞至一或兩個 Azure 匯入/匯出工具，做為 `PrepImport` 作業的一部分。

> [!NOTE]
>  當您指定屬性或中繼資料檔案做為複製工作階段的一部分時，會設定每個做為該複製工作階段一部分的 blob 之屬性或中繼資料。 如果您想要指定要匯入之 blob 部分的一組不同的屬性或中繼資料，您必須使用不同的屬性或中繼資料檔案建立個別複製工作階段。

## <a name="specify-blob-properties-in-a-text-file"></a>在文字檔案中指定 blob 屬性

若要指定 blob 屬性，建立本機文字檔，並包含指定屬性名稱做為元素的 XML 以及做為值的屬性值。 以下是指定一些屬性值的範例︰

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Properties>
    <Content-Type>application/octet-stream</Content-Type>
    <Content-MD5>Q2hlY2sgSW50ZWdyaXR5IQ==</Content-MD5>
    <Cache-Control>no-cache</Cache-Control>
</Properties>
```

將檔案儲存至本機位置，例如 `C:\WAImportExport\ImportProperties.txt`。

## <a name="specify-blob-metadata-in-a-text-file"></a>指定文字檔案中的 blob 中繼資料

同樣地，若要指定 blob 中繼資料，建立指定中繼資料名稱做為項目的本機文字檔，以及做為值的中繼資料值。 以下是指定一些中繼資料值的範例︰

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Metadata>
    <UploadMethod>Windows Azure Import/Export service</UploadMethod>
    <DataSetName>SampleData</DataSetName>
    <CreationDate>10/1/2013</CreationDate>
</Metadata>
```

將檔案儲存至本機位置，例如 `C:\WAImportExport\ImportMetadata.txt`。

## <a name="add-the-path-to-properties-and-metadata-files-in-datasetcsv"></a>在 dataset.csv 中新增屬性和中繼資料檔案的路徑

```
BasePath,DstBlobPathOrPrefix,BlobType,Disposition,MetadataFile,PropertiesFile
H:\Video\,https://mystorageaccount.blob.core.windows.net/video/,BlockBlob,rename,None,H:\mydirectory\properties.xml
H:\Photo\,https://mystorageaccount.blob.core.windows.net/photo/,BlockBlob,rename,None,H:\mydirectory\properties.xml
K:\Temp\FavoriteVideo.ISO,https://mystorageaccount.blob.core.windows.net/favorite/FavoriteVideo.ISO,BlockBlob,rename,None,H:\mydirectory\properties.xml
\\myshare\john\music\,https://mystorageaccount.blob.core.windows.net/music/,BlockBlob,rename,None,H:\mydirectory\properties.xml
```

## <a name="next-steps"></a>後續步驟

* [匯入/匯出服務中繼資料和屬性檔案格式](../storage-import-export-file-format-metadata-and-properties.md)
