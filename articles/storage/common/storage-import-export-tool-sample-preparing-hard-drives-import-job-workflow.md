---
title: 針對 Azure 匯入/匯出匯入作業準備硬碟的範例工作流程 | Microsoft Docs
description: 請參閱在 Azure 匯入/匯出服務中為匯入作業準備硬碟之完整程序的逐步解說。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 04/07/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: b21c378d58590e33c7b6aeffe627ce5602074fa2
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39524615"
---
# <a name="sample-workflow-to-prepare-hard-drives-for-an-import-job"></a>針對匯入作業準備硬碟的範例工作流程

本文章會引導您完成為匯入作業準備硬碟的完整程序。

## <a name="sample-data"></a>範例資料

此範例會將下列資料匯入名為 `mystorageaccount` 的 Azure 儲存體帳戶：

|位置|說明|資料大小|
|--------------|-----------------|-----|
|H:\Video\ |視訊集合|12 TB|
|H:\Photo\ |相片集合|30 GB|
|K:\Temp\FavoriteMovie.ISO|藍光 (Blu-Ray™) 磁碟映像|25 GB|
|\\\bigshare\john\music\ |網路共用上的音樂檔案集合|10 GB|

## <a name="storage-account-destinations"></a>儲存體帳戶目的地

匯入作業會將資料匯入儲存體帳戶中的下列目的地：

|來源|目的地虛擬目錄或 Blob|
|------------|-------------------------------------------|
|H:\Video\ |video/|
|H:\Photo\ |photo/|
|K:\Temp\FavoriteMovie.ISO|favorite/FavoriteMovies.ISO|
|\\\bigshare\john\music\ |music|

透過此對應，檔案 `H:\Video\Drama\GreatMovie.mov` 將會匯入 Blob `https://mystorageaccount.blob.core.windows.net/video/Drama/GreatMovie.mov`。

## <a name="determine-hard-drive-requirements"></a>判斷硬碟需求

接下來，為了判斷需要幾個硬碟，將會計算資料大小：

`12TB + 30GB + 25GB + 10GB = 12TB + 65GB`

針對此範例，兩個 8TB 的硬碟應該就已足夠。 不過，由於來源目錄 `H:\Video` 具有 12TB 的資料，而您的單一硬碟容量只有 8TB，您將可以在 **driveset.csv** 檔案中以下列方法指定這個情況：

```
DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
X,Format,SilentMode,Encrypt,
Y,Format,SilentMode,Encrypt,
```
該工具將會把資料以最佳化的方式分散到兩個硬碟。

## <a name="attach-drives-and-configure-the-job"></a>附加磁碟並設定作業
您將兩個磁碟連接到機器，並建立磁碟區。 然後撰寫 **dataset.csv** 檔案：
```
BasePath,DstBlobPathOrPrefix,BlobType,Disposition,MetadataFile,PropertiesFile
H:\Video\,video/,BlockBlob,rename,None,H:\mydirectory\properties.xml
H:\Photo\,photo/,BlockBlob,rename,None,H:\mydirectory\properties.xml
K:\Temp\FavoriteVideo.ISO,favorite/FavoriteVideo.ISO,BlockBlob,rename,None,H:\mydirectory\properties.xml
\\myshare\john\music\,music/,BlockBlob,rename,None,H:\mydirectory\properties.xml
```

除此之外，您可以針對所有檔案設定下列中繼資料：

* **UploadMethod：** Windows Azure Import/Export service
* **DataSetName:** SampleData
* **CreationDate:** 10/1/2013

若要針對匯入檔案設定中繼資料，請建立具有下列內容的文字檔 (`c:\WAImportExport\SampleMetadata.txt`)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Metadata>
    <UploadMethod>Windows Azure Import/Export service</UploadMethod>
    <DataSetName>SampleData</DataSetName>
    <CreationDate>10/1/2013</CreationDate>
</Metadata>
```

您也可以設定 `FavoriteMovie.ISO` Blob 的部分屬性：

* **Content-Type:** application/octet-stream
* **Content-MD5:** Q2hlY2sgSW50ZWdyaXR5IQ==
* **Cache-Control:** no-cache

若要設定這些屬性，請建立文字檔 (`c:\WAImportExport\SampleProperties.txt`)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Properties>
    <Content-Type>application/octet-stream</Content-Type>
    <Content-MD5>Q2hlY2sgSW50ZWdyaXR5IQ==</Content-MD5>
    <Cache-Control>no-cache</Cache-Control>
</Properties>
```

## <a name="run-the-azure-importexport-tool-waimportexportexe"></a>執行 Azure 匯入/匯出工具 (WAImportExport.exe)

現在您已準備好執行 Azure 匯入/匯出工具以準備兩個硬碟。

**第一個工作階段︰**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset-1.csv /DataSet:dataset-1.csv /logdir:F:\logs
```

如果需要新增額外的資料，請建立另一個資料集檔案 (和初始資料集相同格式)。

**第二個工作階段︰**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
```

當複製工作階段完成後，您可以將兩個磁碟和複製電腦中斷連線，並將它們寄送到適當的 Azure 資料中心。 當您在 Azure 入口網站建立匯入作業時，您將會上傳兩個日誌檔案 (`<FirstDriveSerialNumber>.xml` 和 `<SecondDriveSerialNumber>.xml`)。

## <a name="next-steps"></a>後續步驟

* [針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import.md)
* [常用命令快速參考](../storage-import-export-tool-quick-reference.md)
