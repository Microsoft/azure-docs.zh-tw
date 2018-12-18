---
title: 針對 Azure 匯入/匯出匯入作業準備硬碟的範例工作流程 - v1 | Microsoft Docs
description: 請參閱在 Azure 匯入/匯出服務中為匯入作業準備硬碟之完整程序的逐步解說。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: ae792df428d897277e15df9db3ff6f99a5b8859e
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39527282"
---
# <a name="sample-workflow-to-prepare-hard-drives-for-an-import-job"></a>針對匯入作業準備硬碟的範例工作流程
本主題會引導您完成為匯入作業準備硬碟的完整程序。  
  
此範例會將下列資料匯入名為 `mystorageaccount` 的 Windows Azure 儲存體帳戶：  
  
|位置|說明|  
|--------------|-----------------|  
|H:\Video|視訊集合，總共 5 TB。|  
|H:\Photo|相片集合，總共 30 GB。|  
|K:\Temp\FavoriteMovie.ISO|藍光 (Blu-Ray™) 磁碟映像，25 GB。|  
|\\\bigshare\john\music|網路共用上的音樂檔案集合，總共 10 GB。|  
  
匯入作業會將此資料匯入儲存體帳戶中的下列目的地：  
  
|來源|目的地虛擬目錄或 Blob|  
|------------|-------------------------------------------|  
|H:\Video|https://mystorageaccount.blob.core.windows.net/video|  
|H:\Photo|https://mystorageaccount.blob.core.windows.net/photo|  
|K:\Temp\FavoriteMovie.ISO|https://mystorageaccount.blob.core.windows.net/favorite/FavoriteMovies.ISO|  
|\\\bigshare\john\music|https://mystorageaccount.blob.core.windows.net/music|  
  
透過此對應，檔案 `H:\Video\Drama\GreatMovie.mov` 將會匯入 Blob `https://mystorageaccount.blob.core.windows.net/video/Drama/GreatMovie.mov`。  
  
接下來，為了判斷需要幾個硬碟，將會計算資料大小：  
  
`5TB + 30GB + 25GB + 10GB = 5TB + 65GB`  
  
針對此範例，兩個 3 TB 的硬碟應該就已足夠。 不過，由於來源目錄 `H:\Video` 具有 5 TB 的資料，而您的單一硬碟容量只有 3 TB，因此必須在執行 Microsoft Azure 匯入/匯出工具之前，將 `H:\Video` 分為兩個較小的目錄：`H:\Video1` 和 `H:\Video2`。 此步驟將會產生下列來源目錄：  
  
|位置|大小|目的地虛擬目錄或 Blob|  
|--------------|----------|-------------------------------------------|  
|H:\Video1|2.5 TB|https://mystorageaccount.blob.core.windows.net/video|  
|H:\Video2|2.5 TB|https://mystorageaccount.blob.core.windows.net/video|  
|H:\Photo|30 GB|https://mystorageaccount.blob.core.windows.net/photo|  
|K:\Temp\FavoriteMovies.ISO|25 GB|https://mystorageaccount.blob.core.windows.net/favorite/FavoriteMovies.ISO|  
|\\\bigshare\john\music|10 GB|https://mystorageaccount.blob.core.windows.net/music|  
  
 雖然 `H:\Video` 目錄被分為兩個目錄，它們仍然會指向儲存體帳戶中的相同目的地虛擬目錄。 如此一來，所有視訊檔案都會維持在儲存體帳戶中的單一 `video` 容器中。  
  
 接下來，先前的來源目錄將會平均地分散到兩個硬碟：  
  
||||  
|-|-|-|  
|硬碟|來源目錄|總大小|  
|第一個硬碟|H:\Video1|2.5 TB + 30 GB|  
||H:\Photo||  
|第二個硬碟|H:\Video2|2.5 TB + 35 GB|  
||K:\Temp\BlueRay.ISO||  
||\\\bigshare\john\music||  
  
除此之外，您可以針對所有檔案設定下列中繼資料：  
  
-   **UploadMethod：** Windows Azure Import/Export service  
  
-   **DataSetName:** SampleData  
  
-   **CreationDate:** 10/1/2013  
  
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
  
-   **Content-Type:** application/octet-stream  
  
-   **Content-MD5:** Q2hlY2sgSW50ZWdyaXR5IQ==  
  
-   **Cache-Control:** no-cache  
  
若要設定這些屬性，請建立文字檔 (`c:\WAImportExport\SampleProperties.txt`)：  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<Properties>  
    <Content-Type>application/octet-stream</Content-Type>  
    <Content-MD5>Q2hlY2sgSW50ZWdyaXR5IQ==</Content-MD5>  
    <Cache-Control>no-cache</Cache-Control>  
</Properties>  
```
  
現在您已準備好執行 Azure 匯入/匯出工具以準備兩個硬碟。 請注意：  
  
-   第一個硬碟會裝載為硬碟 X。  
  
-   第二個硬碟會裝載為硬碟 Y。  
  
-   儲存體帳戶 `mystorageaccount` 的金鑰是 `8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg==`。  

## <a name="preparing-disk-for-import-when-data-is-pre-loaded"></a>在資料已預先載入時針對匯入準備磁碟
 
 如果要匯入的資料已經存在於磁碟上，請使用 /skipwrite 旗標。 /t 和 /srcdir 的值都應該指向正在針對匯入進行準備的磁碟。 如果要匯入的所有資料不會移至相同的目的地虛擬目錄或儲存體帳戶的根目錄，請分別對每個目的地目錄執行相同的命令，使 /id 的值在每次執行時都相同。

>[!NOTE] 
>請不要指定 /format，因為這將會清除磁碟上所有的資料。 根據磁碟是否已經加密，您可以指定 /encrypt 或 /bk。 
>

```
    When data is already present on the disk for each drive run the following command.
    WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Video1 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:x:\Video1 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt /skipwrite
```

## <a name="copy-sessions---first-drive"></a>複製工作階段 - 第一個硬碟

針對第一個硬碟，執行 Azure 匯入/匯出工具兩次以複製兩個來源目錄：  

**第一個複製工作階段**
  
```
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Video1 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:H:\Video1 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  
```

**第二個複製工作階段**

```  
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Photo /srcdir:H:\Photo /dstdir:photo/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt
```

## <a name="copy-sessions---second-drive"></a>複製工作階段 - 第二個硬碟
 
針對第二個硬碟，執行 Azure 匯入/匯出工具三次 (針對兩個來源目錄各一次，針對獨立藍光 (Blu-Ray™) 映像檔一次)：  
  
**第一個複製工作階段** 

```
WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:Video2 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:y /format /encrypt /srcdir:H:\Video2 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  
```
  
**第二個複製工作階段**

```
WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:Music /srcdir:\\bigshare\john\music /dstdir:music/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  
```  
  
**第三個複製工作階段**  

```
WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:BlueRayIso /srcfile:K:\Temp\BlueRay.ISO /dstblob:favorite/BlueRay.ISO /MetadataFile:c:\WAImportExport\SampleMetadata.txt /PropertyFile:c:\WAImportExport\SampleProperties.txt  
```

## <a name="copy-session-completion"></a>複製工作階段完成

當複製工作階段完成後，您可以將兩個磁碟和複製電腦中斷連線，並將它們寄送到適當的 Microsoft Azure 資料中心。 當您在 [Azure 入口網站](https://portal.azure.com)建立匯入作業時，請上傳兩個日誌檔案 (`FirstDrive.jrn` 和 `SecondDrive.jrn`)。  
  
## <a name="next-steps"></a>後續步驟

* [針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import-v1.md)   
* [常用命令快速參考](../storage-import-export-tool-quick-reference-v1.md) 
