---
title: 使用 AzCopy on Linux 複製或移動資料到 Azure 儲存體 | Microsoft Docs
description: 使用 AzCopy on Linux 公用程式來從 Blob 和檔案內容移動或來回複製資料。 從本機檔案複製資料到 Azure 儲存體，或在儲存體帳戶內或之間複製資料。 輕鬆地將資料移轉至 Azure 儲存體。
services: storage
author: seguler
ms.service: storage
ms.topic: article
ms.date: 04/26/2018
ms.author: seguler
ms.component: common
ms.openlocfilehash: 1aff2ecdec83886d40d12b76e64a14a81451e89e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386683"
---
# <a name="transfer-data-with-azcopy-on-linux"></a>使用 AzCopy on Linux 傳送資料

AzCopy 是個命令列公用程式，專為使用針對最佳效能而設計的簡單命令來將資料複製到/複製出 Microsoft Azure Blob 和檔案儲存體所設計。 您可以在檔案系統和儲存體帳戶之間，或者儲存體帳戶之間複製資料。  

有兩個 AzCopy 版本可供您下載。 Linux 上的 AzCopy 以 Linux 平台為目標，提供 POSIX 樣式的命令列選項。 [Windows 上的 AzCopy](../storage-use-azcopy.md) 提供了 Windows 樣式的命令列選項。 本文涵蓋之內容包括 AzCopy on Linux。 

> [!NOTE]  
> 從 AzCopy 7.2 版開始，.NET Core 相依項目會與 AzCopy 套件一起封裝。 如果您使用 7.2 版本或更新版本，您不再需要安裝作為先決條件的 .NET Core。

## <a name="download-and-install-azcopy"></a>下載並安裝 AzCopy

### <a name="installation-on-linux"></a>在 Linux 上安裝

> [!NOTE]
> 您可能需要安裝在此 [.NET Core 先決條件文章](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)中醒目提示的 .NET Core 2.1 相依項目 (視您的散發套件而定)。 
>
> 針對 RHEL 7 發行版，請安裝 ICU 和 libunwind 相依性：```yum install -y libunwind icu```

安裝 Linux 上的 AzCopy (v7.2 或更新版本) 相當容易，只要將 tar 套件解壓縮，然後執行安裝指令碼。 

**RHEL 6 架構的散發套件**：[下載連結](https://aka.ms/downloadazcopylinuxrhel6)
```bash
wget -O azcopy.tar.gz https://aka.ms/downloadazcopylinuxrhel6
tar -xf azcopy.tar.gz
sudo ./install.sh
```

**其他所有 Linux 散發套件**：[下載連結](https://aka.ms/downloadazcopylinux64)
```bash
wget -O azcopy.tar.gz https://aka.ms/downloadazcopylinux64
tar -xf azcopy.tar.gz
sudo ./install.sh
```

安裝 AzCopy on Linux 之後，您便可移除解壓縮後的檔案。 或者，如果您沒有 superuser 權限，也可以使用解壓縮資料夾中的 shell 指令碼 azcopy 執行 `azcopy`。

### <a name="alternative-installation-on-ubuntu"></a>Ubuntu 上的替代安裝

**Ubuntu 14.04**

新增適用於 Microsoft Linux 產品存放庫的 apt 來源並安裝 AzCopy：

```bash
sudo echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-trusty-prod/ trusty main" > azure.list
sudo cp ./azure.list /etc/apt/sources.list.d/
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys EB3E94ADBE1229CF
```

```bash
sudo apt-get update
sudo apt-get install azcopy
```

**Ubuntu 16.04**

新增適用於 Microsoft Linux 產品存放庫的 apt 來源並安裝 AzCopy：

```bash
echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod/ xenial main" > azure.list
sudo cp ./azure.list /etc/apt/sources.list.d/
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys EB3E94ADBE1229CF
```

```bash
sudo apt-get update
sudo apt-get install azcopy
```

## <a name="writing-your-first-azcopy-command"></a>撰寫第一個 AzCopy 命令
AzCopy 命令的基本語法是：

```azcopy
azcopy --source <source> --destination <destination> [Options]
```

下列範例會示範各種不同的 Microsoft Azure Blob 和檔案資料複製案例。 如需每個範例中所使用參數的詳細說明，請參閱 `azcopy --help`功能表。

## <a name="blob-download"></a>Blob：下載
### <a name="download-single-blob"></a>下載單一 Blob

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer/abc.txt \
    --destination /mnt/myfiles/abc.txt \
    --source-key <key> 
```

如果資料夾 `/mnt/myfiles` 不存在，AzCopy 會加以建立並將 `abc.txt ` 下載到新資料夾。 

### <a name="download-single-blob-from-secondary-region"></a>從次要地區下載單一 Blob

```azcopy
azcopy \
    --source https://myaccount-secondary.blob.core.windows.net/mynewcontainer/abc.txt \
    --destination /mnt/myfiles/abc.txt \
    --source-key <key>
```

請注意，您必須啟用讀取權限異地備援儲存體。

### <a name="download-all-blobs"></a>下載所有 Blob

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer \
    --destination /mnt/myfiles \
    --source-key <key> \
    --recursive
```

假設下列 Blob 位於指定容器中：  

```
abc.txt
abc1.txt
abc2.txt
vd1/a.txt
vd1/abcd.txt
```

下載作業之後，目錄 `/mnt/myfiles` 會包含下列檔案：

```
/mnt/myfiles/abc.txt
/mnt/myfiles/abc1.txt
/mnt/myfiles/abc2.txt
/mnt/myfiles/vd1/a.txt
/mnt/myfiles/vd1/abcd.txt
```

如果您未指定選項 `--recursive`，則不會下載任何 Blob。

### <a name="download-blobs-with-specified-prefix"></a>下載具有指定首碼的 Blob

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer \
    --destination /mnt/myfiles \
    --source-key <key> \
    --include "a" \
    --recursive
```

假設下列 Blob 位於指定容器中。 會下載所有以首碼 `a` 開頭的 Blob。

```
abc.txt
abc1.txt
abc2.txt
xyz.txt
vd1\a.txt
vd1\abcd.txt
```

下載作業之後，資料夾 `/mnt/myfiles` 會包含下列檔案：

```
/mnt/myfiles/abc.txt
/mnt/myfiles/abc1.txt
/mnt/myfiles/abc2.txt
```

首碼會套用到虛擬目錄，虛擬目錄會構成第一部分的 Blob 名稱。 在上述範例中，虛擬目錄不符合指定的首碼，所以不會下載任何 Blob。 此外，如果未指定選項 `--recursive` ，則 AzCopy 不會下載任何 Blob。

### <a name="set-the-last-modified-time-of-exported-files-to-be-same-as-the-source-blobs"></a>將匯出檔案的最後修改時間設定為與來源 Blob 相同的最後修改時間

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer \
    --destination "/mnt/myfiles" \
    --source-key <key> \
    --preserve-last-modified-time
```

您也可以根據其最後修改時間，將 Blob 從下載作業中排除。 例如，如果您想要排除最後修改時間比目的地檔案還要新或相同的 Blob，請新增 `--exclude-newer` 選項：

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer \
    --destination /mnt/myfiles \
    --source-key <key> \
    --preserve-last-modified-time \
    --exclude-newer
```

或者，如果您想要排除最後修改時間比目的地檔案還要舊或相同的 Blob，請新增 `--exclude-older` 選項：

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer \
    --destination /mnt/myfiles \
    --source-key <key> \
    --preserve-last-modified-time \
    --exclude-older
```

## <a name="blob-upload"></a>Blob：上傳
### <a name="upload-single-file"></a>上傳單一檔案

```azcopy
azcopy \
    --source /mnt/myfiles/abc.txt \
    --destination https://myaccount.blob.core.windows.net/mycontainer/abc.txt \
    --dest-key <key>
```

如果指定的目的地容器不存在，則 AzCopy 會建立此容器並將檔案上傳至該容器中。

### <a name="upload-single-file-to-virtual-directory"></a>上傳單一檔案到虛擬目錄

```azcopy
azcopy \
    --source /mnt/myfiles/abc.txt \
    --destination https://myaccount.blob.core.windows.net/mycontainer/vd/abc.txt \
    --dest-key <key>
```

如果指定的虛擬目錄不存在，則 AzCopy 會上傳檔案並在 Blob 名稱中加上此虛擬目錄 (例如，上述範例中的 `vd/abc.txt`)。

### <a name="redirect-from-stdin"></a>從 stdin 重新導向

```azcopy
gzip myarchive.tar -c | azcopy \
    --destination https://myaccount.blob.core.windows.net/mycontainer/mydir/myarchive.tar.gz \
    --dest-key <key>
```

### <a name="upload-all-files"></a>上傳所有檔案

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.blob.core.windows.net/mycontainer \
    --dest-key <key> \
    --recursive
```

指定 `--recursive` 選項以遞迴方式將指定目錄的內容上傳到 Blob 儲存體，這表示也會上傳所有的子資料夾及其檔案。 例如，假設下列檔案位於 `/mnt/myfiles`資料夾內：

```
/mnt/myfiles/abc.txt
/mnt/myfiles/abc1.txt
/mnt/myfiles/abc2.txt
/mnt/myfiles/subfolder/a.txt
/mnt/myfiles/subfolder/abcd.txt
```

上傳作業之後，容器會包含下列檔案：

```
abc.txt
abc1.txt
abc2.txt
subfolder/a.txt
subfolder/abcd.txt
```

未指定選項 `--recursive` 時，只會上傳下列三個檔案：

```
abc.txt
abc1.txt
abc2.txt
```

### <a name="upload-files-matching-specified-pattern"></a>上傳符合指定模式的檔案

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.blob.core.windows.net/mycontainer \
    --dest-key <key> \
    --include "a*" \
    --recursive
```

假設下列檔案位於 `/mnt/myfiles`資料夾內：

```
/mnt/myfiles/abc.txt
/mnt/myfiles/abc1.txt
/mnt/myfiles/abc2.txt
/mnt/myfiles/xyz.txt
/mnt/myfiles/subfolder/a.txt
/mnt/myfiles/subfolder/abcd.txt
```

上傳作業之後，容器會包含下列檔案：

```
abc.txt
abc1.txt
abc2.txt
subfolder/a.txt
subfolder/abcd.txt
```

未指定選項 `--recursive` 時，AzCopy 會跳過子目錄中的檔案：

```
abc.txt
abc1.txt
abc2.txt
```

### <a name="specify-the-mime-content-type-of-a-destination-blob"></a>指定目的地 blob 的 MIME 內容類型
根據預設，AzCopy 會將目的地 blob 的內容類型設定為 `application/octet-stream`。 但是，您可以透過 `--set-content-type [content-type]` 選項明確指定內容類型。 此語法會在上傳作業中設定所有 Blob 的內容類型。

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.blob.core.windows.net/myContainer/ \
    --dest-key <key> \
    --include "ab" \
    --set-content-type "video/mp4"
```

若已指定選項 `--set-content-type` 但未指定任何值，則 AzCopy 會根據副檔名來設定每個 blob 或檔案的內容類型。

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.blob.core.windows.net/myContainer/ \
    --dest-key <key> \
    --include "ab" \
    --set-content-type
```

### <a name="customizing-the-mime-content-type-mapping"></a>自訂 MIME 內容類型對應
AzCopy 所使用的設定檔中會包含副檔名與內容類型的對應。 您可以視需要自訂這個對應並新增配對。 此對應位於 ```/usr/lib/azcopy/AzCopyConfig.json```

## <a name="blob-copy"></a>Blob：複製
### <a name="copy-single-blob-within-storage-account"></a>複製儲存體帳戶內的單一 Blob

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer1/abc.txt \
    --destination https://myaccount.blob.core.windows.net/mycontainer2/abc.txt \
    --source-key <key> \
    --dest-key <key>
```

當您未以 --sync-copy 選項複製 Blob 時，系統會執行[伺服器端複製](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx) \(英文\) 作業。

### <a name="copy-single-blob-across-storage-accounts"></a>跨儲存體帳戶複製單一 Blob

```azcopy
azcopy \
    --source https://sourceaccount.blob.core.windows.net/mycontainer1/abc.txt \
    --destination https://destaccount.blob.core.windows.net/mycontainer2/abc.txt \
    --source-key <key1> \
    --dest-key <key2>
```

當您未以 --sync-copy 選項複製 Blob 時，系統會執行[伺服器端複製](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx) \(英文\) 作業。

### <a name="copy-single-blob-from-secondary-region-to-primary-region"></a>將單一 Blob 從次要地區複製到主要區域

```azcopy
azcopy \
    --source https://myaccount1-secondary.blob.core.windows.net/mynewcontainer1/abc.txt \
    --destination https://myaccount2.blob.core.windows.net/mynewcontainer2/abc.txt \
    --source-key <key1> \
    --dest-key <key2>
```

請注意，您必須啟用讀取權限異地備援儲存體。

### <a name="copy-single-blob-and-its-snapshots-across-storage-accounts"></a>跨儲存體帳戶複製單一 Blob 及其快照

```azcopy
azcopy \
    --source https://sourceaccount.blob.core.windows.net/mycontainer1/ \
    --destination https://destaccount.blob.core.windows.net/mycontainer2/ \
    --source-key <key1> \
    --dest-key <key2> \
    --include "abc.txt" \
    --include-snapshot
```

複製作業之後，目標容器會包含 Blob 及其快照集。 容器包含下列 Blob 及其快照集：

```
abc.txt
abc (2013-02-25 080757).txt
abc (2014-02-21 150331).txt
```

### <a name="synchronously-copy-blobs-across-storage-accounts"></a>以同步方式跨儲存體帳戶複製 Blob
根據預設，AzCopy 會以非同步方式在兩個儲存體端點之間複製資料。 因此，複製作業會在背景中使用並未以 SLA 規範應以多快速度複製 Blob 的備用頻寬能力執行。 

`--sync-copy` 選項可確保複製作業達到一致的速度。 AzCopy 執行同步複製的方式是先將要複製的 blob，從指定的來源下載到本機記憶體，再上傳至 Blob 儲存體目的地。

```azcopy
azcopy \
    --source https://myaccount1.blob.core.windows.net/myContainer/ \
    --destination https://myaccount2.blob.core.windows.net/myContainer/ \
    --source-key <key1> \
    --dest-key <key2> \
    --include "ab" \
    --sync-copy
```

相較於非同步複製，`--sync-copy` 可能會產生額外的輸出成本。 建議的方法是在與您來源儲存體帳戶位於同一區域的 Azure VM 中使用這個選項，以避免產生輸出成本。

## <a name="file-download"></a>檔案：下載
### <a name="download-single-file"></a>下載單一檔案

```azcopy
azcopy \
    --source https://myaccount.file.core.windows.net/myfileshare/myfolder1/abc.txt \
    --destination /mnt/myfiles/abc.txt \
    --source-key <key>
```

如果指定的來源是 Azure 檔案共用，則您必須指定確切檔案名稱 (例如，`abc.txt`) 以下載單一檔案，或指定 `--recursive` 選項以遞迴方式下載共用中的所有檔案。 嘗試同時指定檔案模式和 `--recursive` 選項會造成錯誤。

### <a name="download-all-files"></a>下載所有檔案

```azcopy
azcopy \
    --source https://myaccount.file.core.windows.net/myfileshare/ \
    --destination /mnt/myfiles \
    --source-key <key> \
    --recursive
```

請注意，並不會下載任何空白資料夾。

## <a name="file-upload"></a>檔案：上傳
### <a name="upload-single-file"></a>上傳單一檔案

```azcopy
azcopy \
    --source /mnt/myfiles/abc.txt \
    --destination https://myaccount.file.core.windows.net/myfileshare/abc.txt \
    --dest-key <key>
```

### <a name="upload-all-files"></a>上傳所有檔案

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.file.core.windows.net/myfileshare/ \
    --dest-key <key> \
    --recursive
```

請注意，並不會上傳任何空白資料夾。

### <a name="upload-files-matching-specified-pattern"></a>上傳符合指定模式的檔案

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.file.core.windows.net/myfileshare/ \
    --dest-key <key> \
    --include "ab*" \
    --recursive
```

## <a name="file-copy"></a>檔案：複製
### <a name="copy-across-file-shares"></a>跨檔案共用複製

```azcopy
azcopy \
    --source https://myaccount1.file.core.windows.net/myfileshare1/ \
    --destination https://myaccount2.file.core.windows.net/myfileshare2/ \
    --source-key <key1> \
    --dest-key <key2> \
    --recursive
```
當您跨檔案共用複製檔案時，系統會執行[伺服器端複製](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### <a name="copy-from-file-share-to-blob"></a>從檔案共用複製到 Blob

```azcopy
azcopy \ 
    --source https://myaccount1.file.core.windows.net/myfileshare/ \
    --destination https://myaccount2.blob.core.windows.net/mycontainer/ \
    --source-key <key1> \
    --dest-key <key2> \
    --recursive
```
當您將檔案從檔案共用複製到 Blob 時，系統會執行[伺服器端複製](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### <a name="copy-from-blob-to-file-share"></a>從 Blob 複製到檔案共用

```azcopy
azcopy \
    --source https://myaccount1.blob.core.windows.net/mycontainer/ \
    --destination https://myaccount2.file.core.windows.net/myfileshare/ \
    --source-key <key1> \
    --dest-key <key2> \
    --recursive
```
當您將檔案從 Blob 複製到檔案共用時，系統會執行[伺服器端複製](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### <a name="synchronously-copy-files"></a>以同步方式複製檔案
您可以指定 `--sync-copy` 選項，以同步方式從檔案儲存體複製資料到檔案儲存體、從檔案儲存體複製資料到 Blob 儲存體，以及從 Blob 儲存體複製資料到檔案儲存體。 AzCopy 會將來源資料下載至本機記憶體，然後上傳到目的地，來執行此項作業。 在此情況下，會產生標準輸出成本。

```azcopy
azcopy \
    --source https://myaccount1.file.core.windows.net/myfileshare1/ \
    --destination https://myaccount2.file.core.windows.net/myfileshare2/ \
    --source-key <key1> \
    --dest-key <key2> \
    --recursive \
    --sync-copy
```

從檔案儲存體複製到 Blob 儲存體時，預設的 Blob 類型是區塊 Blob，使用者可以指定 `--blob-type page` 選項來變更目的地 Blob 類型。 可用的類型為 `page | block | append`。

請注意，相較於非同步複製，`--sync-copy` 可能會產生額外的輸出成本。 建議的方法是在與您來源儲存體帳戶位於同一區域的 Azure VM 中使用這個選項，以避免產生輸出成本。

## <a name="other-azcopy-features"></a>其他 AzCopy 功能
### <a name="only-copy-data-that-doesnt-exist-in-the-destination"></a>只複製目的地中沒有的資料
`--exclude-older` 和 `--exclude-newer` 參數分別可讓您在複製作業中排除較舊或較新的來源資源。 如果您只想複製目的地中沒有的來源資源，則可以在 AzCopy 命令中同時指定這兩個參數：

    --source http://myaccount.blob.core.windows.net/mycontainer --destination /mnt/myfiles --source-key <sourcekey> --recursive --exclude-older --exclude-newer

    --source /mnt/myfiles --destination http://myaccount.file.core.windows.net/myfileshare --dest-key <destkey> --recursive --exclude-older --exclude-newer

    --source http://myaccount.blob.core.windows.net/mycontainer --destination http://myaccount.blob.core.windows.net/mycontainer1 --source-key <sourcekey> --dest-key <destkey> --recursive --exclude-older --exclude-newer

### <a name="use-a-configuration-file-to-specify-command-line-parameters"></a>使用組態檔指定命令列參數

```azcopy
azcopy --config-file "azcopy-config.ini"
```

您可以在組態檔中包含任何 AzCopy 命令列參數。 AzCopy 處理檔案中的參數，就好像在命令列上指定這些參數一様，執行使用檔案內容的直接取代。

假設名為 `copyoperation` 且包含下列資料行的組態檔。 每個 AzCopy 參數可以指定在同一行。

    --source http://myaccount.blob.core.windows.net/mycontainer --destination /mnt/myfiles --source-key <sourcekey> --recursive --quiet

或不同行：

    --source http://myaccount.blob.core.windows.net/mycontainer
    --destination /mnt/myfiles
    --source-key<sourcekey>
    --recursive
    --quiet

如果您將參數分割成兩行，如以下的 `--source-key` 參數所示，則 AzCopy 會失敗：

    http://myaccount.blob.core.windows.net/mycontainer
    /mnt/myfiles
    --sourcekey
    <sourcekey>
    --recursive
    --quiet

### <a name="specify-a-shared-access-signature-sas"></a>指定共用存取簽章 (SAS)

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer1/abc.txt \
    --destination https://myaccount.blob.core.windows.net/mycontainer2/abc.txt \
    --source-sas <SAS1> \
    --dest-sas <SAS2>
```

您也可以在容器 URI 上指定 SAS：

```azcopy
azcopy \
    --source https://myaccount.blob.core.windows.net/mycontainer1/?SourceSASToken \
    --destination /mnt/myfiles \
    --recursive
```

### <a name="journal-file-folder"></a>日誌檔案資料夾
每次發佈命令至 AzCopy 時，它會檢查預設資料夾或透過此選項指定的資料夾中是否有日誌檔案存在。 如果在這兩個地方都找不到日誌檔案，AzCopy 會將此作業視為新的作業，並產生新的日誌檔案。

如果找到日誌檔案，則 AzCopy 會檢查所輸入的命令列是否符合日誌檔案中的命令列。 如果這兩個命令列相符，AzCopy 便會繼續未完成的作業。 如果這兩個命令列不符，AzCopy 會提示使用者覆寫日誌檔案並開始新的作業，或取消目前作業。

如果您想要使用預設的日誌檔案位置：

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.blob.core.windows.net/mycontainer \
    --dest-key <key> \
    --resume
```

如果省略 `--resume`，或指定 `--resume` 選項但沒有指定資料夾路徑 (如上所示)，則 AzCopy 會在預設位置上建立日誌檔案，預設位置是 `~\Microsoft\Azure\AzCopy`。 如果日誌檔案已存在，則 AzCopy 會根據此日誌檔案繼續作業。

如果您想要指定自訂的日誌檔案位置：

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.blob.core.windows.net/mycontainer \
    --dest-key key \
    --resume "/mnt/myjournal"
```

如果日誌檔案不存在，本範例將建立日誌檔案。 如果日誌檔案已存在，則 AzCopy 會根據此日誌檔案繼續作業。

如果您想要繼續 AzCopy 作業，請重複相同的命令。 然後 AzCopy on Linux 會提示您確認：

```azcopy
Incomplete operation with same command line detected at the journal directory "/home/myaccount/Microsoft/Azure/AzCopy", do you want to resume the operation? Choose Yes to resume, choose No to overwrite the journal to start a new operation. (Yes/No)
```

### <a name="output-verbose-logs"></a>輸出詳細資訊記錄檔

```azcopy
azcopy \
    --source /mnt/myfiles \
    --destination https://myaccount.blob.core.windows.net/mycontainer \
    --dest-key <key> \
    --verbose
```

### <a name="specify-the-number-of-concurrent-operations-to-start"></a>指定要啟動的並行作業數目
`--parallel-level` 選項可指定並行複製作業的數目。 根據預設，AzCopy 依預設會啟動特定數量的並行作業，以提高資料傳輸的輸送量。 並行作業數目等於您所擁有處理器數目的八倍。 如果您在低頻寬的網路上執行 AzCopy，則您可以針對平行層級指定較低的數字，以避免因為資源競爭所導致的失敗。

>[!TIP]
>若要檢視 AzCopy 參數的完整清單，請參閱 [azcopy --說明] 功能表。

## <a name="installation-steps-for-azcopy-71-and-earlier-versions"></a>AzCopy 7.1 和更早版本的安裝步驟

Linux 上的 AzCopy (僅限 v7.1 和更早版本) 需要 .NET Core 架構。 您可以在 [.NET Core 安裝](https://www.microsoft.com/net/core#linuxubuntu)頁面上取得安裝指示。

例如，從安裝 Ubuntu 16.10 上的 .NET Core 開始。 如需最新安裝指南，請造訪 [.NET Core on Linux](https://www.microsoft.com/net/core#linuxubuntu) \(英文\) 安裝頁面。


```bash
sudo sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/dotnet-release/ yakkety main" > /etc/apt/sources.list.d/dotnetdev.list' 
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 417A0893
sudo apt-get update
sudo apt-get install dotnet-sdk-2.0.0
```

安裝 .NET Core 之後，下載並安裝 AzCopy。

```bash
wget -O azcopy.tar.gz https://aka.ms/downloadazcopyprlinux
tar -xf azcopy.tar.gz
sudo ./install.sh
```

安裝 AzCopy on Linux 之後，您便可移除解壓縮後的檔案。 或者，如果您沒有 superuser 權限，也可以使用解壓縮資料夾中的 shell 指令碼 azcopy 執行 `azcopy`。

## <a name="known-issues-and-best-practices"></a>已知問題和最佳作法
### <a name="error-installing-azcopy"></a>安裝 AzCopy 時發生錯誤
如果您遇到 AzCopy 安裝的問題，可以嘗試使用解壓縮 `azcopy` 資料夾中的 bash 指令碼執行 AzCopy。

```bash
cd azcopy
./azcopy
```

### <a name="limit-concurrent-writes-while-copying-data"></a>限制複製資料時的並行寫入
使用 AzCopy 複製 Blob 或檔案時，請留意當您在複製資料時，另一個應用程式可能正在修改該資料。 請儘可能地確定在複製作業過程中，您正要複製的資料並不在修改中。 例如，當複製與 Azure 虛擬機器相關聯的 VHD 時，請確定目前沒有其他應用程式正在寫入此 VHD。 要這樣做的一個好方法是租用要複製的資源。 此外，您可以首先建立 VHD 的快照，然後複製此快照。

如果您無法在複製時防止其他應用程式寫入 Blob 或檔案，請記住，工作完成時，複製的資源可能不再與來源資源完全相同。

### <a name="running-multiple-azcopy-processes"></a>執行多個 AzCopy 處理程序
如果您使用不同日誌資料夾，您可以在單一用戶端上執行多個 AzCopy 處理程序。 不支援對多個 AzCopy 處理程序使用單一日誌資料夾。

第一個處理程序：
```azcopy
azcopy \
    --source /mnt/myfiles1 \
    --destination https://myaccount.blob.core.windows.net/mycontainer/myfiles1 \
    --dest-key <key> \
    --resume "/mnt/myazcopyjournal1"
```

第二個處理程序：
```azcopy
azcopy \
    --source /mnt/myfiles2 \
    --destination https://myaccount.blob.core.windows.net/mycontainer/myfiles2 \
    --dest-key <key> \
    --resume "/mnt/myazcopyjournal2"
```

## <a name="next-steps"></a>後續步驟
如需關於 Azure 儲存體和 AzCopy 的詳細資訊，請參閱下列資源：

### <a name="azure-storage-documentation"></a>Azure 儲存體文件：
* [Azure 儲存體簡介](../storage-introduction.md)
* [建立儲存體帳戶](../storage-create-storage-account.md)
* [使用儲存體總管來管理 Blob](https://docs.microsoft.com/azure/vs-azure-tools-storage-explorer-blobs) \(英文\)
* [使用 Azure CLI 搭配 Azure 儲存體](../storage-azure-cli.md)
* [如何使用 C++ 的 Blob 儲存體](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
* [如何使用 Java 的 Blob 儲存體](../blobs/storage-java-how-to-use-blob-storage.md)
* [如何使用 Node.js 的 Blob 儲存體](../blobs/storage-nodejs-how-to-use-blob-storage.md)
* [如何使用 Python 的 Blob 儲存體](../blobs/storage-python-how-to-use-blob-storage.md)

### <a name="azure-storage-blog-posts"></a>Azure 儲存體部落格文章：
* [宣告 AzCopy on Linux 預覽](https://azure.microsoft.com/blog/announcing-azcopy-on-linux-preview/) \(英文\)
* [Azure 儲存體資料移動文件庫預覽簡介](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)
* [AzCopy：簡介同步複製和自訂內容類型](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/01/13/azcopy-introducing-synchronous-copy-and-customized-content-type.aspx)
* [AzCopy: 宣布正式發行 AzCopy 3.0 和具有資料表和檔案支援的 AzCopy 4.0 預覽版本](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/10/29/azcopy-announcing-general-availability-of-azcopy-3-0-plus-preview-release-of-azcopy-4-0-with-table-and-file-support.aspx)
* [AzCopy: 針對大規模複製案例最佳化](http://go.microsoft.com/fwlink/?LinkId=507682)
* [AzCopy: 支援讀取存取異地備援儲存體](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/07/azcopy-support-for-read-access-geo-redundant-account.aspx)
* [AzCopy：使用可重新啟動模式和 SAS 權杖傳輸資料](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/azcopy-transfer-data-with-re-startable-mode-and-sas-token.aspx) \(英文\)
* [AzCopy: 使用跨帳戶複製 Blob](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/04/01/azcopy-using-cross-account-copy-blob.aspx)
* [AzCopy: 上傳/下載 Azure Blob 的檔案](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx)
