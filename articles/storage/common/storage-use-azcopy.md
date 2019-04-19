---
title: 使用 AzCopy on Windows 複製或移動資料到 Azure 儲存體 | Microsoft Docs
description: 使用 AzCopy on Windows 公用程式來從 Blob、資料表和檔案內容移動或來回複製資料。 從本機檔案複製資料到 Azure 儲存體，或在儲存體帳戶內或之間複製資料。 輕鬆地將資料移轉至 Azure 儲存體。
services: storage
author: seguler
ms.service: storage
ms.topic: article
ms.date: 01/03/2019
ms.author: seguler
ms.subservice: common
ms.openlocfilehash: 6ae3ec566c05d2460747439d61c87c995a90b19c
ms.sourcegitcommit: c174d408a5522b58160e17a87d2b6ef4482a6694
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2019
ms.locfileid: "58881727"
---
# <a name="transfer-data-with-the-azcopy-on-windows"></a>使用 AzCopy on Windows 傳送資料
AzCopy 是個命令列公用程式，專為使用針對最佳效能而設計的簡單命令來將資料複製到/複製出 Microsoft Azure Blob、檔案和表格儲存體所設計。 您可以在檔案系統和儲存體帳戶之間，或者儲存體帳戶之間複製資料。  

有兩個 AzCopy 版本可供您下載。 AzCopy on Windows 提供了 Windows 樣式的命令列選項。 [AzCopy on Linux](storage-use-azcopy-linux.md) 以 Linux 平台為目標，提供 POSIX 樣式的命令列選項。 本文涵蓋之內容包括 AzCopy on Windows。

## <a name="download-and-install-azcopy-on-windows"></a>在 Windows 上下載並安裝 AzCopy

### <a name="latest-version-v81"></a>最新版本 (v8.1)
下載 [最新版本的 AzCopy on Windows](https://aka.ms/downloadazcopy)。

#### <a name="azcopy-on-windows-81-release-notes"></a>AzCopy on Windows 8.1 版本資訊
- 最新版不再支援資料表服務。 如果您會使用資料表匯出功能，請下載 AzCopy 7.3 版。
- 使用 .NET Core 2.1 建置，且現已在安裝中封裝所有 .NET Core 相依項目。
- 已新增 OAuth 驗證支援。 使用 ```azcopy login``` 以 Azure Active Directory 登入。

### <a name="azcopy-with-table-support-v73"></a>AzCopy (含資料表支援) (v7.3)
下載 [AzCopy 7.3 (含資料表支援)](https://aka.ms/downloadazcopynet)。

### <a name="post-installation-step"></a>後續安裝步驟

使用安裝程式安裝 AzCopy on Windows 之後，請開啟命令視窗並瀏覽至電腦上的 AzCopy 安裝目錄，也就是 `AzCopy.exe` 可執行檔的所在位置。 若有需要，您可以在您的系統路徑中加入 AzCopy 安裝位置。 預設情況下，AzCopy 安裝到 `%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy` 或 `%ProgramFiles%\Microsoft SDKs\Azure\AzCopy`。

## <a name="writing-your-first-azcopy-command"></a>撰寫第一個 AzCopy 命令

AzCopy 命令的基本語法是：

```azcopy
AzCopy /Source:<source> /Dest:<destination> [Options]
```

下列範例會示範各種不同的 Microsoft Azure Blob、檔案和資料表資料複製案例。 如需每個範例中所使用參數的詳細說明，請參閱 [AzCopy 參數](#azcopy-parameters) 一節。

## <a name="download-blobs-from-blob-storage"></a>從 Blob 儲存體下載 blob

讓我們看看使用 AzCopy 來下載 blob 的幾種方式。

### <a name="download-a-single-blob"></a>下載單一 Blob

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:"abc.txt"
```

請注意，如果資料夾 `C:\myfolder` 不存在，AzCopy 會加以建立並將 `abc.txt` 下載到新資料夾。

### <a name="download-a-single-blob-from-the-secondary-region"></a>從次要地區下載單一 Blob

```azcopy
AzCopy /Source:https://myaccount-secondary.blob.core.windows.net/mynewcontainer /Dest:C:\myfolder /SourceKey:key /Pattern:"abc.txt"
```

請注意，您必須啟用讀取權限異地備援儲存體，才能存取次要地區。

### <a name="download-all-blobs-in-a-container"></a>下載容器中所有的 Blob

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /S
```

假定指定的容器中存在以下 blob：  

    abc.txt
    abc1.txt
    abc2.txt
    vd1\a.txt
    vd1\abcd.txt

下載作業之後，目錄 `C:\myfolder` 會包含下列檔案：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt
    C:\myfolder\vd1\a.txt
    C:\myfolder\vd1\abcd.txt

如果您未指定選項 `/S`，則不會下載任何 Blob。

### <a name="download-blobs-with-a-specific-prefix"></a>下載具有指定首碼的 Blob

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:a /S
```

假設下列 Blob 位於指定容器中。 會下載所有以首碼 `a` 開頭的 Blob：

    abc.txt
    abc1.txt
    abc2.txt
    xyz.txt
    vd1\a.txt
    vd1\abcd.txt

下載作業之後，資料夾 `C:\myfolder` 會包含下列檔案：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt

首碼會套用到虛擬目錄，虛擬目錄會構成第一部分的 Blob 名稱。 在上述範例中，虛擬目錄不符合指定的首碼，所以不會被下載。 此外，如果未指定選項 `/S` ，則 AzCopy 不會下載任何 Blob。

### <a name="set-the-last-modified-time-of-exported-files-to-be-same-as-the-source-blobs"></a>將匯出檔案的最後修改時間設定為與來源 Blob 相同的最後修改時間

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT
```

您也可以根據其最後修改時間，將 Blob 從下載作業中排除。 例如，如果您想要排除最後修改時間比目的地檔案還要新或相同的 Blob，請新增 `/XN` 選項：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XN
```

如果您需要排除最後修改時間比目的地檔案還要舊或相同的 Blob，請新增 `/XO` 選項：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XO
```

## <a name="upload-blobs-to-blob-storage"></a>將 blob 上傳至 blob 儲存體

讓我們看看使用 AzCopy 來上傳 blob 的幾種方式。

### <a name="upload-a-single-blob"></a>上傳單一 Blob

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Pattern:"abc.txt"
```

如果指定的目的地容器不存在，則 AzCopy 會建立此容器並將檔案上傳至該容器中。

### <a name="upload-a-single-blob-to-a-virtual-directory"></a>將單一 blob 上傳至虛擬目錄

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer/vd /DestKey:key /Pattern:abc.txt
```

如果指定的虛擬目錄不存在，則 AzCopy 會上傳檔案並在其名稱中加上此虛擬目錄 (例如，上述範例中的 `vd/abc.txt`)。

### <a name="upload-all-blobs-in-a-folder"></a>上傳資料夾中的所有 blob

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /S
```

指定 `/S` 選項以遞迴方式將指定目錄的內容上傳到 Blob 儲存體，這表示也會上傳所有的子資料夾及其檔案。 例如，假設下列檔案位於 `C:\myfolder`資料夾內：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt
    C:\myfolder\subfolder\a.txt
    C:\myfolder\subfolder\abcd.txt

上傳操作完成後，容器中將包括以下檔案：

    abc.txt
    abc1.txt
    abc2.txt
    subfolder\a.txt
    subfolder\abcd.txt

如果您未指定選項 `/S`，AzCopy 不會以遞迴方式上傳。 上傳操作完成後，容器中將包括以下文件：

    abc.txt
    abc1.txt
    abc2.txt

### <a name="upload-blobs-matching-a-specific-pattern"></a>上傳符合特定模式的 blob

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Pattern:a* /S
```

假定以下檔案位于資料夾 `C:\myfolder` 中：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt
    C:\myfolder\xyz.txt
    C:\myfolder\subfolder\a.txt
    C:\myfolder\subfolder\abcd.txt

上傳操作完成後，容器中將包括以下檔案：

    abc.txt
    abc1.txt
    abc2.txt
    subfolder\a.txt
    subfolder\abcd.txt

如果您未指定選項 `/S`，AzCopy 只會上傳沒有位於虛擬目錄的 Blob：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt

### <a name="specify-the-mime-content-type-of-a-destination-blob"></a>指定目標 blob 的 MIME 內容類型

根據預設，AzCopy 會將目的地 blob 的內容類型設定為 `application/octet-stream`。 從 3.1.0 版開始，您可以透過 `/SetContentType:[content-type]`選項明確指定內容類型。 此語法會在上傳作業中設定所有 Blob 的內容類型。

```azcopy
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.windows.net/myContainer/ /DestKey:key /Pattern:ab /SetContentType:video/mp4
```

如果您指定 `/SetContentType` 但未指定任何值，則 AzCopy 會根據副檔名來設定每個 blob 或檔案的內容類型。

```azcopy
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.windows.net/myContainer/ /DestKey:key /Pattern:ab /SetContentType
```

## <a name="copy-blobs-in-blob-storage"></a>複製 Blob 儲存體中的 blob

讓我們看看使用 AzCopy 將 blob 從某個位置複製到另一個位置的幾種方法。

### <a name="copy-a-single-blob-from-one-container-to-another-within-the-same-storage-account"></a>將單一 blob 從某個容器複製到相同儲存體帳戶內的另一個容器 

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1 /Dest:https://myaccount.blob.core.windows.net/mycontainer2 /SourceKey:key /DestKey:key /Pattern:abc.txt
```

當您複製儲存體帳戶內的 Blob 時，系統會執行 [伺服器端複製](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx) 作業。

### <a name="copy-a-single-blob-from-one-storage-account-to-another"></a>將單一 blob 從某個儲存體帳戶複製到另一個儲存體帳戶

```azcopy
AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt
```

當您跨儲存體帳戶複製 Blob 時，系統會執行 [伺服器端複製](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx) 作業。

### <a name="copy-a-single-blob-from-the-secondary-region-to-the-primary-region"></a>將單一 blob 從次要地區複製到主要區域

```azcopy
AzCopy /Source:https://myaccount1-secondary.blob.core.windows.net/mynewcontainer1 /Dest:https://myaccount2.blob.core.windows.net/mynewcontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt
```

請注意，您必須啟用讀取權限異地備援儲存體，才能存取次要儲存體。

### <a name="copy-a-single-blob-and-its-snapshots-from-one-storage-account-to-another"></a>將單一 blob 和其快照集從某個儲存體帳戶複製到另一個儲存體帳戶

```azcopy
AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt /Snapshot
```

複製作業之後，目標容器會包含 Blob 及其快照集。 假設上述範例中的 Blob 有兩份快照，則容器會包含下列 Blob 及快照：

    abc.txt
    abc (2013-02-25 080757).txt
    abc (2014-02-21 150331).txt

### <a name="copy-all-blobs-in-a-container-to-another-storage-account"></a>將容器中所有的 blob 複製到另一個儲存體帳戶 

```azcopy
AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 
/Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /S
```

指定選項 /S 以遞迴方式上傳指定容器的內容。 如需詳細資訊和範例，請參閱[上傳資料夾中的所有 blob](#upload-all-blobs-in-a-folder)。

### <a name="synchronously-copy-blobs-from-one-storage-account-to-another"></a>以非同步方式在儲存體帳戶間複製 blob

根據預設，AzCopy 會以非同步方式在兩個儲存體端點之間複製資料。 因此，複製作業會在背景中執行，並使用在 blob 複製速度方面沒有 SLA 的備用頻寬容量，而 AzCopy 會定期檢查複製狀態，直到複製完成或失敗為止。

`/SyncCopy` 選項可確保複製作業達到一致的速度。 AzCopy 執行同步複製的方式是先將要複製的 blob，從指定的來源下載到本機記憶體，再上傳至 Blob 儲存體目的地。

```azcopy
AzCopy /Source:https://myaccount1.blob.core.windows.net/myContainer/ /Dest:https://myaccount2.blob.core.windows.net/myContainer/ /SourceKey:key1 /DestKey:key2 /Pattern:ab /SyncCopy
```

`/SyncCopy` 可能會產生額外的輸出成本，建議的方法是在與來源儲存體帳戶位於同一區域的 Azure VM 中使用這個選項，以避免產生輸出成本。

## <a name="download-files-from-file-storage"></a>從檔案儲存體下載檔案

讓我們看看使用 AzCopy 來下載檔案的幾種方式。

### <a name="download-a-single-file"></a>下載單一檔案

```azcopy
AzCopy /Source:https://myaccount.file.core.windows.net/myfileshare/myfolder1/ /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt
```

如果指定的來源是 Azure 檔案共用，則您必須指定確切檔案名稱 (例如，`abc.txt`) 以下載單一檔案，或指定 `/S` 選項以遞迴方式下載共用中的所有檔案。 嘗試同時指定檔案模式和 `/S` 選項會造成錯誤。

### <a name="download-all-files-in-a-directory"></a>下載目錄中的所有檔案

```azcopy
AzCopy /Source:https://myaccount.file.core.windows.net/myfileshare/ /Dest:C:\myfolder /SourceKey:key /S
```

請注意，並不會下載空白資料夾。

## <a name="upload-files-to-an-azure-file-share"></a>將檔案上傳至 Azure 檔案服務共用

讓我們看看使用 AzCopy 來上傳檔案的幾種方式。

### <a name="upload-a-single-file"></a>上傳單一檔案

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /Pattern:abc.txt
```

### <a name="upload-all-files-in-a-folder"></a>上傳資料夾中的所有檔案

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /S
```

請注意，並不會上傳空白資料夾。

### <a name="upload-files-matching-a-specific-pattern"></a>上傳符合特定模式的檔案

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /Pattern:ab* /S
```

## <a name="copy-files-in-file-storage"></a>在檔案儲存體中複製檔案

讓我們看看使用 AzCopy 在 Azure 檔案共用中複製檔案的幾種方式。

### <a name="copy-from-one-file-share-to-another"></a>從某個檔案共用複製到另一個檔案共用

```azcopy
AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare1/ /Dest:https://myaccount2.file.core.windows.net/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S
```
當您跨檔案共用複製檔案時，系統會執行[伺服器端複製](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### <a name="copy-from-an-azure-file-share-to-blob-storage"></a>從 Azure 檔案共用複製到 Blob 儲存體

```azcopy
AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare/ /Dest:https://myaccount2.blob.core.windows.net/mycontainer/ /SourceKey:key1 /DestKey:key2 /S
```
當您將檔案從檔案共用複製到 Blob 時，系統會執行[伺服器端複製](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### <a name="copy-a-blob-from-blob-storage-to-an-azure-file-share"></a>將 Blob 從 Blob 儲存體複製到 Azure 檔案共用

```azcopy
AzCopy /Source:https://myaccount1.blob.core.windows.net/mycontainer/ /Dest:https://myaccount2.file.core.windows.net/myfileshare/ /SourceKey:key1 /DestKey:key2 /S
```
當您將檔案從 Blob 複製到檔案共用時，系統會執行[伺服器端複製](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### <a name="synchronously-copy-files"></a>以同步方式複製檔案

您可以指定 `/SyncCopy` 選項，以同步方式從檔案儲存體複製資料到檔案儲存體、從檔案儲存體複製資料到 Blob 儲存體，以及從 Blob 儲存體複製資料到檔案儲存體，AzCopy 會將來源資料下載至本機記憶體，並重新上傳到目的地，來執行此項作業。 這會產生標準輸出成本。

```azcopy
AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare1/ /Dest:https://myaccount2.file.core.windows.net/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S /SyncCopy
```

從檔案儲存體複製到 Blob 儲存體時，預設的 Blob 類型是區塊 Blob，使用者可以指定 `/BlobType:page` 選項來變更目的地 Blob 類型。

請注意，相較於非同步複製，`/SyncCopy` 可能會產生額外的輸出成本。 建議的方法是在與您來源儲存體帳戶位於同一區域的 Azure VM 中使用這個選項，以避免產生輸出成本。

## <a name="export-data-from-table-storage"></a>從資料表儲存體匯出資料

讓我們看看使用 AzCopy 從 Azure 資料表儲存體匯出資料。

### <a name="export-a-table"></a>匯出資料表

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key
```

AzCopy 會將資訊清單檔寫入至指定的目的地資料夾。 匯入程序會使用資訊清單檔案來尋找必要的資料檔案，並執行資料驗證。 資訊清單檔預設會使用下列命令慣例：

    <account name>_<table name>_<timestamp>.manifest

使用者也可以指定 `/Manifest:<manifest file name>` 選項，設定資訊清單檔案名稱。

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /Manifest:abc.manifest
```

### <a name="split-an-export-from-table-storage-into-multiple-files"></a>將資料表儲存體中的匯出分割成多個檔案

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/mytable/ /Dest:C:\myfolder /SourceKey:key /S /SplitSize:100
```

AzCopy 會在分割資料檔案名稱中使用 *磁碟區索引* ，以區分多個檔案。 磁碟區索引由兩部分組成：資料分割索引鍵範圍索引和分割檔案索引。 兩個索引皆以 0 為基礎。

如果使用者未指定 `/PKRS`選項，資料分割索引鍵範圍索引會是 0。

例如，在使用者指定 `/SplitSize`選項之後，假設 AzCopy 產生兩個資料檔。 產生的資料檔案名稱可能會是：

    myaccount_mytable_20140903T051850.8128447Z_0_0_C3040FE8.json
    myaccount_mytable_20140903T051850.8128447Z_0_1_0AB9AC20.json

請注意， `/SplitSize` 選項的最小可能值為 32MB。 如果指定的目的地是 Blob 儲存體，則 AzCopy 會分割已達到 Blob 大小限制 (200GB) 的資料檔，而不論使用者是否已指定 `/SplitSize` 選項。

### <a name="export-a-table-to-json-or-csv-data-file-format"></a>將資料表匯出為 JSON 或 CSV 資料檔案格式

依預設，AzCopy 會將資料表匯出至 JSON 資料檔。 您可以指定選項 `/PayloadFormat:JSON|CSV` 以將資料表匯出為 JSON 或 CSV。

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PayloadFormat:CSV
```

在指定 CSV 裝載格式時，AzCopy 也會產生結構描述檔案，且每個資料檔的副檔名為 `.schema.csv` 。

### <a name="export-table-entities-concurrently"></a>並行匯出資料表實體

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PKRS:"aa#bb"
```

當使用者指定 `/PKRS` 選項時，AzCopy 會啟動並行作業以匯出實體。 每個作業分別會匯出一個資料分割索引鍵範圍。

請注意，並行作業的數目也由 `/NC`選項控制。 在複製資料表實體時，即使未指定 `/NC`，AzCopy 會以核心處理器的數目作為 `/NC` 的預設值。 當使用者指定 `/PKRS`選項時，AzCopy 會從兩個值 (資料分割索引鍵範圍和隱含或明確指定的並行作業) 中選擇較小者來使用，以決定要啟動的並行作業數目。 如需詳細資訊，請在命令列上輸入 `AzCopy /?:NC` 。

### <a name="export-a-table-to-blob-storage"></a>將資料表匯出至 Blob 儲存體

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:https://myaccount.blob.core.windows.net/mycontainer/ /SourceKey:key1 /Destkey:key2
```

AzCopy 會使用下列命令慣例，在 Blob 容器中產生 JSON 資料檔案：

    <account name>_<table name>_<timestamp>_<volume index>_<CRC>.json

產生的 JSON 資料檔案會遵循基本中繼資料的裝載格式。 如需此裝載格式的詳細資訊，請參閱 [資料表服務作業的裝載格式](https://msdn.microsoft.com/library/azure/dn535600.aspx)。

請注意，在將資料表匯出至 Blob 時，AzCopy 會先將資料表實體下載到本機暫存資料檔，再將這些實體上傳至 Blob。 這些暫存資料檔會放入日誌檔案資料夾中，預設路徑為 "`%LocalAppData%\Microsoft\Azure\AzCopy`"，您可以指定 `/Z:[journal-file-folder]` 選項來變更日誌檔案資料夾位置，從而變更暫存資料檔位置。 暫存資料檔大小取決於資料表實體大小和您使用 `/SplitSize` 選項所指定的大小，雖然本機磁碟中的暫存資料檔在上傳至 Blob 之後就立即刪除，但請確定您有足夠的本機磁碟空間，以儲存尚未刪除的暫存資料檔。

## <a name="import-data-into-table-storage"></a>將資料匯入資料表儲存體中

讓我們看看使用 AzCopy 將資料匯入 Azure 資料表儲存體中。

### <a name="import-a-table"></a>匯入資料表

```azcopy
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.core.windows.net/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace
```

選項 `/EntityOperation` 指出如何將實體插入資料表。 可能的值包括：

* `InsertOrSkip`:略過現有實體，或插入新實體 (若不存在於資料表中)。
* `InsertOrMerge`:合併現有實體，或插入新實體 (若不存在於資料表中)。
* `InsertOrReplace`:取代現有實體，或插入新實體 (若不存在於資料表中)。

請注意，您無法在匯入案例中指定 `/PKRS` 選項。 不同於必須指定 `/PKRS` 選項以啟動並行作業的匯出案例，AzCopy 依預設會在您匯入資料表時啟動並行作業。 依預設啟動的並行作業數目等於核心處理器的數目；但您可以使用 `/NC`選項指定不同的並行數目。 如需詳細資訊，請在命令列上輸入 `AzCopy /?:NC` 。

請注意，AzCopy 只支援匯入 JSON 而不支援匯入 CSV。 AzCopy 不支援從使用者建立的 JSON 和資訊清單檔案匯入資料表。 這兩種檔案都必須來自 AzCopy 資料表匯出。 若要避免錯誤，請勿修改匯出的 JSON 或資訊清單檔案。

### <a name="import-entities-into-a-table-from-blob-storage"></a>從 Blob 儲存體將實體匯入資料表

假設有 Blob 容器包含下列：代表 Azure 資料表和其隨附資訊清單檔案的 JSON 檔案。

    myaccount_mytable_20140103T112020.manifest
    myaccount_mytable_20140103T112020_0_0_0AF395F1DC42E952.json

您可以執行下列命令，以使用該 Blob 容器中的資訊清單檔案將實體匯入資料表中：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:https://myaccount.table.core.windows.net/mytable /SourceKey:key1 /DestKey:key2 /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:"InsertOrReplace"
```

## <a name="other-azcopy-features"></a>其他 AzCopy 功能

讓我們看看一些其他的 AzCopy 功能。

### <a name="only-copy-data-that-doesnt-exist-in-the-destination"></a>只複製目的地中沒有的資料

`/XO` 和 `/XN` 參數分別可讓您在複製作業中排除較舊或較新的來源資源。 如果只想複製目標中不存在的來源資源，可以在 AzCopy 命令中指定這兩個參數：

    /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /XO /XN

    /Source:C:\myfolder /Dest:http://myaccount.file.core.windows.net/myfileshare /DestKey:<destkey> /S /XO /XN

    /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:http://myaccount.blob.core.windows.net/mycontainer1 /SourceKey:<sourcekey> /DestKey:<destkey> /S /XO /XN

注意，當來源或目的地其中之一是資料表時，不支援此做法。

### <a name="use-a-response-file-to-specify-command-line-parameters"></a>使用回應檔案指定命令行參數

```azcopy
AzCopy /@:"C:\responsefiles\copyoperation.txt"
```

您可以在回應檔案中包含任何 AzCopy 命令列參數。 AzCopy 處理檔案中的參數，就好像在命令列上指定這些參數一樣，執行使用檔案內容的直接取代。

假設名為 `copyoperation.txt`且包含下列資料行的回應檔案。 每個 AzCopy 參數可以指定在同一行

    /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

每個 AzCopy 參數：

    /Source:http://myaccount.blob.core.windows.net/mycontainer
    /Dest:C:\myfolder
    /SourceKey:<sourcekey>
    /S
    /Y

如果您將參數分割成兩行，如以下的 `/sourcekey` 參數所示，則 AzCopy 會失敗：

    http://myaccount.blob.core.windows.net/mycontainer
     C:\myfolder
    /sourcekey:
    <sourcekey>
    /S
    /Y

### <a name="use-multiple-response-files-to-specify-command-line-parameters"></a>使用多個回應檔案指定命令列參數

假設名為 `source.txt` 且指定來源容器的回應檔案：

    /Source:http://myaccount.blob.core.windows.net/mycontainer

及名為 `dest.txt` 且在檔案系統中指定目的地資料夾的回應檔案：

    /Dest:C:\myfolder

及名為 `options.txt` 且指定 AzCopy 選項的回應檔案：

    /S /Y

若要使用這些回應檔案 (所有這些檔案皆位於 `C:\responsefiles`目錄內) 呼叫 AzCopy，請使用此命令：

```azcopy
AzCopy /@:"C:\responsefiles\source.txt" /@:"C:\responsefiles\dest.txt" /SourceKey:<sourcekey> /@:"C:\responsefiles\options.txt"   
```

AzCopy 處理此命令，就好像您在命令列上包含所有個別參數一樣：

```azcopy
AzCopy /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y
```

### <a name="specify-a-shared-access-signature-sas"></a>指定共用存取簽章 (SAS)

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1 /Dest:https://myaccount.blob.core.windows.net/mycontainer2 /SourceSAS:SAS1 /DestSAS:SAS2 /Pattern:abc.txt
```

您也可以在容器 URI 上指定 SAS：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1/?SourceSASToken /Dest:C:\myfolder /S
```

### <a name="journal-file-folder"></a>日誌檔案資料夾

每次向 AzCopy 發出命令时，它都會檢查預設資料夾中是否存在日誌檔案，或者通過此選項指定的資料夾中是否存在日誌檔案。 如果在這兩個地方都找不到日誌檔案，AzCopy 會將此作業視為新的作業，並產生新的日誌檔案。

如果找到日誌檔案，則 AzCopy 會檢查所輸入的命令列是否符合日誌檔案中的命令列。 如果這兩個命令列相符，AzCopy 便會繼續未完成的作業。 如果這兩個命令列不符，系統會提示您覆寫日誌檔案並開始新的作業，或取消目前作業。

如果您想要使用預設的日誌檔案位置：

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Z
```

如果省略 `/Z`，或指定 `/Z` 選項但沒有指定資料夾路徑 (如上所示)，則 AzCopy 會在預設位置上建立日誌檔案，預設位置是 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。 如果日誌檔案已存在，則 AzCopy 會根據此日誌檔案繼續作業。

如果您想要指定自訂的日誌檔案位置：

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Z:C:\journalfolder\
```

如果日誌檔案不存在，本範例將建立日誌檔案。 如果日誌檔案已存在，則 AzCopy 會根據此日誌檔案繼續作業。

如果您想要繼續 AzCopy 作業：

```azcopy
AzCopy /Z:C:\journalfolder\
```

本範例會繼續最後一個無法完成的作業。

### <a name="generate-a-log-file"></a>產生記錄檔

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /V
```

如果指定 `/V` 選項，但沒有提供詳細資訊記錄的檔案路徑，則 AzCopy 會在預設位置上建立記錄檔，預設位置是 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。

否則，您可以在自訂位置建立記錄檔：

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /V:C:\myfolder\azcopy1.log
```

請注意，如果您在 `/V` 選項後面指定相對路徑 (例如 `/V:test/azcopy1.log`)，則系統會在目前工作目錄中的一個名為 `test` 的子資料夾內建立詳細資訊記錄。

### <a name="specify-the-number-of-concurrent-operations-to-start"></a>指定要啟動的並行作業數目

`/NC` 選項可指定並行複製作業的數目。 根據預設，AzCopy 依預設會啟動特定數量的並行作業，以提高資料傳輸的輸送量。 若為資料表作業，並行作業數目等於您所擁有的處理器數目。 若為 Blob 和檔案作業，並行作業數目等於您所擁有處理器數目的 8 倍。 如果您在低頻寬的網路上執行 AzCopy，則您可以在 /NC 中指定較低的數字，以避免因為資源競爭所導致的失敗。

### <a name="run-azcopy-against-the-azure-storage-emulator"></a>針對 Azure 儲存體模擬器執行 AzCopy

您可以針對 Blob，對 [Azure 儲存體模擬器](storage-use-emulator.md) 執行 AzCopy：

```azcopy
AzCopy /Source:https://127.0.0.1:10000/myaccount/mycontainer/ /Dest:C:\myfolder /SourceKey:key /SourceType:Blob /S
```

您也可以針對資料表加以執行：

```azcopy
AzCopy /Source:https://127.0.0.1:10002/myaccount/mytable/ /Dest:C:\myfolder /SourceKey:key /SourceType:Table
```

### <a name="automatically-determine-content-type-of-a-blob"></a>自動判斷 Blob 的內容類型

AzCopy 會根據儲存副檔名對應內容類型的 JSON 檔案，以判斷 Blob 的內容類型。 此 JSON 檔案名為 AzCopyConfig.json 且位在 AzCopy 目錄中。 如果清單中沒有您的檔案類型，可以將該對應附加至 JSON 檔案：

```
{
  "MIMETypeMapping": {
    ".myext": "text/mycustomtype",
    .
    .
  }
}
```     

## <a name="azcopy-parameters"></a>AzCopy 參數

以下內容說明 AzCopy 的參數。 您也可以從命令列中輸入下列其中一個命令，以取得 AzCopy 的使用說明：

* 如需 AzCopy 的詳細命令列說明： `AzCopy /?`
* 如需任何 AzCopy 參數的詳細說明： `AzCopy /?:SourceKey`
* 如需命令列範例： `AzCopy /?:Sample`

### <a name="sourcesource"></a>/Source:"source"

指定要複製的來源資料。 來源可以是檔案系統目錄、Blob 容器、Blob 虛擬目錄、儲存體檔案共用、儲存體檔案目錄或 Azure 資料表。

**適用於：** Blob、檔案、資料表

### <a name="destdestination"></a>/Dest:"destination"

指定複製的目的地。 目的地可以是檔案系統目錄、Blob 容器、Blob 虛擬目錄、儲存體檔案共用、儲存體檔案目錄或 Azure 資料表。

**適用於：** Blob、檔案、資料表

### <a name="patternfile-pattern"></a>/Pattern:"file-pattern"

指定一個檔案模式以指出所要複製的檔案。 `/Pattern` 參數的行為取決於來源資料的位置以及是否有遞迴模式選項。 遞迴模式可透過選項 `/S` 來指定。

如果指定來源是檔案系統中的目錄，則標準萬用字元便會立即生效，且所提供的檔案模式會與目錄中的檔案做比對。 如果已指定選項 `/S`，則 AzCopy 也會將指定模式與該目錄下任何子資料夾中的所有檔案做比對。

如果指定來源是 Blob 容器或虛擬目錄，則萬用字元並不適用。 如果已指定選項 `/S`，則 AzCopy 會將指定的檔案模式解譯為 Blob 首碼。 如果未指定選項 `/S`，則 AzCopy 會將檔案模型與確切 Blob 名稱做比對。

如果指定來源是 Azure 檔案共用，則您必須指定確切檔案名稱 (例如 abc.txt) 以複製單一檔案，或指定選項 `/S` 以遞迴方式複製共用中的所有檔案。 嘗試同時指定檔案模式和 `/S` 選項會造成錯誤。

當 `/Source` 是 Blob 容器或 Blob 的虛擬目錄時，AzCopy 會使用區分大小寫比對，並在所有其他情況下使用不區分大小寫比對。

未指定檔案模式時，使用的預設檔案模式如下：針對檔案系統位置，會使用 \*.\* ，針對「Azure 儲存體」位置，則是使用空白首碼。 不支援指定多個檔案模式。

**適用於：** Blob、檔案

### <a name="destkeystorage-key"></a>/DestKey:"storage-key"

指定目的地資源的儲存體帳戶金鑰。

**適用於：** Blob、檔案、資料表

### <a name="destsassas-token"></a>/DestSAS:"sas-token"

以讀取和寫入權限指定目的地的共用存取簽章 (SAS) (如果適用的話)。 為 SAS 加上雙引號，因為它可能包含特殊命令列字元。

如果目的地資源是 Blob 容器、檔案共用或資料表，您可以指定此選項後面接著 SAS 權杖，或者您可以不使用此選項，直接將 SAS 指定為目的地 Blob 容器、檔案共用或資料表 URI 的一部分。

如果來源和目的地都是 Blob，則目的地 Blob 必須與來源 Blob 位於相同的儲存體帳戶中。

**適用於：** Blob、檔案、資料表

### <a name="sourcekeystorage-key"></a>/SourceKey:"storage-key"

指定來源資源的儲存體帳戶金鑰。

**適用於：** Blob、檔案、資料表

### <a name="sourcesassas-token"></a>/SourceSAS:"sas-token"

以讀取和清單權限指定來源的共用存取簽章 (如果適用的話)。 為 SAS 加上雙引號，因為它可能包含特殊命令列字元。

如果來源資源是 Blob 容器，且未提供金鑰及 SAS，則可透過匿名存取讀取該 Blob 容器。

如果來源是檔案共用或資料表，則必須提供金鑰或 SAS。

**適用於：** Blob、檔案、資料表

### <a name="s"></a>/S

指定複製作業的遞迴模式。 在遞迴模式中，AzCopy 會複製所有符合指定檔案模式的 Blob 或檔案，包括子資料夾中的 Blob 或檔案。

**適用於：** Blob、檔案

### <a name="blobtypeblock--page--append"></a>/BlobType:"block" | "page" | "append"

指定目的地 Blob 是區塊 Blob、分頁 Blob 或附加 Blob。 此選項僅在上傳 Blob 時才適用。 否則會產生錯誤。 如果目的地是 Blob 且未指定此選項，則 AzCopy 預設會建立區塊 Blob。

**適用於：** Blob

### <a name="checkmd5"></a>/CheckMD5

計算下載資料的 MD5 雜湊，並驗證 MD5 雜湊是否儲存在符合計算之雜湊的 Blob 或檔案的 Content-MD5 屬性中。 如果值不相符，AzCopy 便無法下載資料。 MD5 檢查依預設為關閉，因此您必須指定此選項，才能在下載資料時執行 MD5 檢查。

請注意，Azure 儲存體並不保證儲存供 Blob 或檔案使用的 MD5 雜湊是最新的版本。 每次修改 Blob 或檔案時將 MD5 更新是用戶端的責任。 如果是磁碟映像 (受控或非受控磁碟)，Azure VM 不會隨著磁碟內容變更而更新 MD5 值，因此 /CheckMD5 會在下載磁碟映像時擲回錯誤。

在上傳至服務之後，AzCopy v8 一律會為 Azure Blob 或檔案設定 Content-MD5 屬性。  

**適用於：** Blob、檔案

### <a name="snapshot"></a>/Snapshot

指出是否傳輸快照。 此選項僅在來源是 Blob 才有效。

傳輸的 Blob 快照集會以下列格式重新命名：blob-name (snapshot-time).extension

依預設，不會複製快照。

**適用於：** Blob

### <a name="vverbose-log-file"></a>/V:[verbose-log-file]

將詳細資訊狀態訊息輸出至記錄檔。

根據預設，在 `%LocalAppData%\Microsoft\Azure\AzCopy`中詳細資訊記錄檔會被命名為 AzCopyVerbose.log。 如果您在此選項中指定現有檔案位置，則系統會將詳細資訊記錄附加到該檔案。  

**適用於：** Blob、檔案、資料表

### <a name="zjournal-file-folder"></a>/Z:[journal-file-folder]

指定用於繼續作業的日誌檔案資料夾。

如果作業遭到中斷，AzCopy 絕對支援繼續作業。

如果未指定此選項，或指定此選項但沒有指定資料夾路徑，則 AzCopy 會在預設位置上建立日誌檔案，預設位置是 `%LocalAppData%\Microsoft\Azure\AzCopy`。

每次發佈命令至 AzCopy 時，它會檢查預設資料夾或透過此選項指定的資料夾中是否有日誌檔案存在。 如果在這兩個地方都找不到日誌檔案，AzCopy 會將此作業視為新的作業，並產生新的日誌檔案。

如果找到日誌檔案，則 AzCopy 會檢查所輸入的命令列是否符合日誌檔案中的命令列。 如果這兩個命令列相符，AzCopy 便會繼續未完成的作業。 如果這兩個命令列不符，系統會提示您覆寫日誌檔案並開始新的作業，或取消目前作業。

成功完成作業時，系統便會將日誌檔案刪除。

請注意，不支援根據舊版的 AzCopy 所建立的日誌檔案繼續作業。

**適用於：** Blob、檔案、資料表

### <a name="parameter-file"></a>/@:"parameter-file"

指定包含參數的檔案。 AzCopy 處理檔案中的參數，就好像在命令列上指定這些參數一樣。

在回應檔案中，您可以在單行中指定多個參數，或每一行各自指定一個參數。 請注意，各個參數無法橫跨多行。

回應檔案可包含開頭為 # 符號的命令列。

您可以指定多個回應檔案。 不過，請記住 AzCopy 不支援巢狀回應檔案。

**適用於：** Blob、檔案、資料表

### <a name="y"></a>/Y

隱藏所有 AzCopy 確認提示。 當 `/XO` 和 `/XN` 皆未指定時，此選項也允許針對資料上傳案例使用唯寫的 SAS 權杖。

**適用於：** Blob、檔案、資料表

### <a name="l"></a>/L

僅指定清單作業，不會複製資料。

AzCopy 會解譯使用這個選項為模擬不使用 `/L` 選項來執行此命令列，而且會計算有多少物件會被複製，您可以同時指定 /V 選項，檢查要複製哪些物件到詳細記錄檔中。

這個選項的行為還取決於來源資料的位置，以及是否有遞迴模式選項 `/S` 和檔案模式選項 `/Pattern`。

使用此選項時，AzCopy 會需要此來源位置的清單和讀取權限。

**適用於：** Blob、檔案

### <a name="mt"></a>/MT

將下載檔案的最後修改時間設定為與來源 Blob 或檔案相同的最後修改時間。

**適用於：** Blob、檔案

### <a name="xn"></a>/XN

排除較新的來源資源。 如果最後修改時間比目的地還要新或同時間，則不會複製資源。

**適用於：** Blob、檔案

### <a name="xo"></a>/XO
排除較舊的來源資源。 如果最後修改時間比目的地還要舊或同時間，則不會複製資源。

**適用於：** Blob、檔案

### <a name="a"></a>/A

僅上傳已設定 [封存] 屬性的檔案。

**適用於：** Blob、檔案

### <a name="iarashcnetoi"></a>/IA:[RASHCNETOI]

僅上傳已設定任何指定屬性的檔案。

可用的属性包括：

* R = 唯讀檔案
* A = 準備封存的檔案
* S = 系統檔案
* H = 隱藏檔案
* C = 壓縮檔案
* N = 正常檔案
* E = 加密檔案
* T = 暫存檔案
* O = 離線檔案
* I = 未編製索引的檔案

**適用於：** Blob、檔案

### <a name="xarashcnetoi"></a>/XA:[RASHCNETOI]

排除已設定任何指定屬性的檔案。

可用屬性包括：

* R = 唯讀檔案
* A = 準備封存的檔案
* S = 系統檔案
* H = 隱藏檔案
* C = 壓縮檔案
* N = 正常檔案
* E = 加密檔案
* T = 暫存檔案
* O = 離線檔案
* I = 未編製索引的檔案

**適用於：** Blob、檔案

### <a name="delimiterdelimiter"></a>/Delimiter:"delimiter"

指出在 Blob 名稱中，用來分隔虛擬目錄的分隔符號字元。

預設情況下，AzCopy 使用 / 作為分隔符字符。 不過，AzCopy 支援使用任何常見字元 (例如 @、# 或 %) 作為分隔符號。 如果您必須在命令列中包含其中一個特殊字元，請為檔案名稱加上雙引號。

此選項僅適用於下載 Blob。

**適用於：** Blob

### <a name="ncnumber-of-concurrent-operations"></a>/NC:"number-of-concurrent-operations"

指定並行作業的數目。

AzCopy 依預設會啟動特定數量的並行作業，以提高資料傳輸的輸送量。 請注意，在低頻寬環境中的大量並行作業有可能會拖垮網路連線，並使作業無法完成。 根據實際可用網路頻寬來節流處理並行作業。

並行作業數的上限為 512。

**適用於：** Blob、檔案、資料表

### <a name="sourcetypeblob--table"></a>/SourceType:"Blob" | "Table"

指定 `source` 資源是本機開發環境中可用、於儲存體模擬器中執行的 Blob。

**適用於：** Blob、資料表

### <a name="desttypeblob--table"></a>/DestType:"Blob" | "Table"

指定 `destination` 資源是本機開發環境中可用、於儲存體模擬器中執行的 Blob。

**適用於：** Blob、資料表

### <a name="pkrskey1key2key3"></a>/PKRS:"key1#key2#key3#..."

分割資料分割索引鍵範圍以支援平行匯出資料表資料的作業，這樣可以加快匯出作業的速度。

若未指定此選項，AzCopy 會使用單一執行緒來匯出資料表實體。 例如，如果使用者指定 /PKRS:"aa#bb"，則 AzCopy 會啟動三個並行作業。

每個作業分別會匯出三個資料分割索引鍵範圍的其中一個，如下所示：

  [first-partition-key, aa)

  [aa, bb)

  [bb, last-partition-key]

**適用於：** 資料表

### <a name="splitsizefile-size"></a>/SplitSize:"file-size"

指定匯出的檔案分割大小 (以 MB 為單位)，允許的最小值為 32。

若未指定此選項，則 AzCopy 會將資料表資料匯出至單一檔案。

如果資料表資料匯出至 Blob，且匯出的檔案大小達到 Blob 的大小限制 200 GB，則 AzCopy 會分割匯出的檔案，即使未指定此選項亦然。

**適用於：** 資料表

### <a name="entityoperationinsertorskip--insertormerge--insertorreplace"></a>/EntityOperation:"InsertOrSkip" | "InsertOrMerge" | "InsertOrReplace"

指定資料表資料匯入行為。

* InsertOrSkip - 略過現有實體，或插入新實體 (若不存在於資料表中)。
* InsertOrMerge - 合併現有實體，或插入新實體 (若不存在於資料表中)。
* InsertOrReplace - 取代現有實體，或插入新實體 (若不存在於資料表中)。

**適用於：** 資料表

### <a name="manifestmanifest-file"></a>/Manifest:"manifest-file"

指定資料表匯出和匯入作業的資訊清單檔。

在匯出作業期間，此選項是選擇性的，如果沒有指定這個選項，AzCopy 便會產生具有預先定義名稱的資訊清單檔案。

在匯入作業期間，為了尋找資料檔案，這是必要選項。

**適用於：** 資料表

### <a name="synccopy"></a>/SyncCopy

指出在兩個 Azure 儲存體端點之間是否以同步方式複製 blob 或檔案。

AzCopy 依預設會使用伺服器端的非同步複本。 指定此選項可執行同步複製，也就是將 blog 或檔案下載至本機記憶體，再上傳至 Azure 儲存體。

當您複製檔案是在 Blob 儲存體內、在檔案儲存體內，或從 Blob 儲存體到檔案儲存體 (或反過來) 時，您可以使用此選項。

**適用於：** Blob、檔案

### <a name="setcontenttypecontent-type"></a>/SetContentType:"content-type"

指定目的地 blob 或檔案的 MIME 內容類型。

AzCopy 預設會將 blob 或檔案的內容類型設定為 application/octet-stream。 您可以明確指定這個選項的值來設定所有 blob 或檔案的內容類型。

如果您指定這個選項但未指定任何值，則 AzCopy 會根據副檔名來設定每個 blob 或檔案的內容類型。

**適用於：** Blob、檔案

### <a name="payloadformatjson--csv"></a>/PayloadFormat:"JSON" | "CSV"

指定資料表匯出的資料檔格式。

如果未指定此選項，根據預設，AzCopy 會匯出 JSON 格式的資料表資料檔案。

**適用於：** 資料表

## <a name="known-issues-and-best-practices"></a>已知問題和最佳作法

讓我們看看一些已知的問題和最佳做法。

### <a name="limit-concurrent-writes-while-copying-data"></a>限制複製資料時的並行寫入

使用 AzCopy 複製 Blob 或檔案時，請留意當您在複製資料時，另一個應用程式可能正在修改該資料。 請儘可能地確定在複製作業過程中，您正要複製的資料並不在修改中。 例如，當複製與 Azure 虛擬機器相關聯的 VHD 時，請確定目前沒有其他應用程式正在寫入此 VHD。 要這樣做的一個好方法是租用要複製的資源。 此外，您可以首先建立 VHD 的快照，然後複製此快照。

如果您無法在複製時防止其他應用程式寫入 Blob 或檔案，請記住，工作完成時，複製的資源可能不再與來源資源完全相同。

### <a name="enable-fips-compliant-md5-algorithms-for-azcopy-when-you-use-fips-compliant-algorithms-for-encryption-hashing-and-signing"></a>當您「使用 FIPS 相容演算法於加密、雜湊及簽章」時，請為 AzCopy 啟用 FIPS 相容的 MD5 演算法。

複製物件時，AzCopy 預設會使用 .NET MD5 實作來計算此 MD5，但仍有一些需要 AzCopy 啟用 FIPS 相容 MD5 設定的安全性需求。

您可以建立具有 `AzureStorageUseV1MD5` 屬性的 app.config 檔案 `AzCopy.exe.config`，並將它放在 AzCopy.exe 旁邊。

    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
      <appSettings>
        <add key="AzureStorageUseV1MD5" value="false"/>
      </appSettings>
    </configuration>

針對屬性 "AzureStorageUseV1MD5"：

* True - 預設值，AzCopy 會使用 .NET MD5 實作。
* False – AzCopy 會使用 FIPS 相容的 MD5 演算法。

依預設會停用 Windows 上的 FIPS 相容演算法。 您可以在電腦上變更這個原則設定。 在 [執行] 視窗 (Windows+R) 中，輸入 secpol.msc 以開啟 [本機安全性原則] 視窗。 在 [安全性設定] 視窗中，瀏覽至 [安全性設定] > [本機原則] > [安全性選項]。 找出**系統加密：使用 FIPS 相容演算法進行加密、雜湊和簽署**原則。 按兩下原則可查看 [安全性設定] 資料行中顯示的值。

## <a name="next-steps"></a>後續步驟

如需關於 Azure 儲存體和 AzCopy 的詳細資訊，請參閱下列資源：

### <a name="azure-storage-documentation"></a>Azure 儲存體文件：
* [Azure 存储简介](../storage-introduction.md)
* [如何通過 .NET 使用 Blob 存储](../blobs/storage-dotnet-how-to-use-blobs.md)
* [如何使用 .NET 的檔案儲存體](../storage-dotnet-how-to-use-files.md)
* [如何使用 .NET 的資料表儲存體](../../cosmos-db/table-storage-how-to-use-dotnet.md)
* [如何创建、管理或删除存储帐户](../storage-create-storage-account.md)
* [使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)

### <a name="azure-storage-blog-posts"></a>Azure 存储博客文章：
* [Azure 儲存體資料移動文件庫預覽簡介](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)
* [AzCopy：簡介同步複製和自訂內容類型](https://blogs.msdn.com/b/windowsazurestorage/archive/2015/01/13/azcopy-introducing-synchronous-copy-and-customized-content-type.aspx)
* [AzCopy：宣布正式發行 AzCopy 3.0 和具有資料表和檔案支援的 AzCopy 4.0 預覽版本](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/10/29/azcopy-announcing-general-availability-of-azcopy-3-0-plus-preview-release-of-azcopy-4-0-with-table-and-file-support.aspx)
* [AzCopy：針對大規模複製案例最佳化](https://go.microsoft.com/fwlink/?LinkId=507682)
* [AzCopy：支援讀取存取異地備援儲存體](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/07/azcopy-support-for-read-access-geo-redundant-account.aspx)
* [AzCopy：使用可重新啟動模式和 SAS 權杖傳輸資料](https://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/azcopy-transfer-data-with-re-startable-mode-and-sas-token.aspx)
* [AzCopy：使用跨帳戶複製 Blob](https://blogs.msdn.com/b/windowsazurestorage/archive/2013/04/01/azcopy-using-cross-account-copy-blob.aspx)
* [AzCopy：上傳/下載 Azure Blob 的檔案](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx)
