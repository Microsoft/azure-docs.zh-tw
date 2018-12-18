---
title: 如何部署 Azure 檔案服務 | Microsoft Docs
description: 了解如何從頭到尾部署 Azure 檔案服務。
services: storage
author: wmgries
ms.service: storage
ms.topic: article
ms.date: 05/22/2018
ms.author: wgries
ms.component: files
ms.openlocfilehash: d2e09c837597dfd15e6258cbd0100762b098eedf
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39523480"
---
# <a name="how-to-deploy-azure-files"></a>如何部署 Azure 檔案服務

  [Azure 檔案服務](storage-files-introduction.md)可提供在雲端中完全受控的檔案共用，可透過業界標準 SMB 通訊協定加以存取。 本文將說明如何在組織中實際部署 Azure 檔案服務。

強烈建議您先閱讀[規劃 Azure 檔案服務部署](storage-files-planning.md)，再遵循這篇文章中的步驟。

## <a name="prerequisites"></a>必要條件
本文假設您已完成下列步驟：

- 在所要區域中，建立 Azure 儲存體帳戶，並設定所需的復原和加密選項。 如需如何建立儲存體帳戶的逐步指示，請參閱[建立儲存體帳戶](../common/storage-create-storage-account.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。
- 已在儲存體帳戶中建立 Azure 檔案共用，並設定所需的配額。 如需如何建立檔案共用的逐步指示，請參閱[建立檔案共用](storage-how-to-create-file-share.md)。

## <a name="transfer-data-into-azure-files"></a>將資料傳送到 Azure 檔案服務
您可能想要將現有的檔案共用 (例如儲存在內部部署的檔案共用) 移轉至新的 Azure 檔案共用。 本節示範如何透過[規劃指南](storage-files-planning.md#data-transfer-method)詳述的幾個常用方法，將資料移至 Azure 檔案共用

### <a name="azure-file-sync"></a>Azure 檔案同步
Azure 檔案同步可讓您將組織的檔案共用集中在「Azure 檔案服務」中，而不需要犧牲內部部署檔案伺服器的靈活度、效能及相容性。 它會將您的 Windows Server 轉換成 Azure 檔案共用的快速快取來達到這個目的。 您可以使用 Windows Server 上可用的任何通訊協定來存取本機資料 (包括 SMB、NFS 和 FTPS)，並且可以在世界各地擁有任何所需數量的快取。

即使您不需要長期使用同步處理機制，也可以使用 Azure 檔案同步，將資料移轉至 Azure 檔案共用。 如需如何使用 Azure 檔案同步將資料傳送至 Azure 檔案共用的詳細資訊，請參閱[規劃 Azure 檔案同步部署](storage-sync-files-planning.md)和[如何部署 Azure 檔案同步](storage-sync-files-deployment-guide.md)。

### <a name="azure-importexport"></a>Azure 匯入/匯出
您可將硬碟寄送至 Azure 資料中心，透過 Azure 匯入/匯出服務，安全地將大量資料傳送至 Azure 檔案共用。 請參閱[使用 Microsoft Azure 匯入/匯出服務將資料傳送至 Azure 儲存體](../common/storage-import-export-service.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)，以取得服務的詳細概觀。

> [!Note]  
> Azure 匯入/匯出服務目前尚不支援從 Azure 檔案共用匯出檔案。

下列步驟會將資料從內部部署的位置匯入至 Azure 檔案共用。

1. 取得要寄送至 Azure 所需的硬碟數目。 磁碟容量大小不拘，但必須是支援 SATA II 或 SATA III 標準的 2.5 吋或 3.5 吋 SSD 或 HDD。 

2. 將要進行資料傳輸的每個磁碟連線並掛接到伺服器/電腦上。 為了達到最佳效能，建議您在含有資料的伺服器上本機執行內部部署的匯出工作。 某些情況可能無法這麼做，例如提供資料的檔案伺服器是 NAS 裝置時。 這時候，您完全可以將每個磁碟掛接到電腦上。

3. 請確定每個磁碟均已上線、初始化，並指派磁碟機代號。 若要讓磁碟上線、初始化並指派磁碟機代號，請開啟磁碟管理 MMC 嵌入式管理單元 (diskmgmt.ms)。

    - 若要讓磁碟上線 (如果尚未上線的話)，請在磁碟管理 MMC 下方窗格的磁碟上，按一下滑鼠右鍵，並選取 [上線]。
    - 若要初始化磁碟，請以滑鼠右鍵按一下下方窗格的磁碟 (待磁碟上線後)，然後選取 [初始化]。 當系統詢問時，請務必選取 [GPT]。

        ![磁碟管理 MMC 中 [初始化磁碟] 功能表的螢幕擷取畫面](media/storage-files-deployment-guide/transferdata-importexport-1.PNG)

    - 若要將磁碟機代號指派給磁碟，請在已上線並已初始化之磁碟的「未配置」空間中，按一下滑鼠右鍵，然後按一下 [新增簡單磁碟區]。 這可讓您指派磁碟機代號。 請注意，您不需要格式化磁碟區，因為這是稍後要進行的作業。

        ![磁碟管理 MMC 中 [新增簡單磁碟區精靈] 的螢幕擷取畫面](media/storage-files-deployment-guide/transferdata-importexport-2.png)

4. 建立資料集 CSV 檔案。 資料集 CSV 檔案是內部部署資料和所需的 Azure 檔案共用 (要將資料複製過去的目的地) 之間的路徑對應。 比方說，下列資料集 CSV 檔案會將內部部署檔案共用 ("F:\shares\scratch") 對應到 Azure 檔案共用 ("MyAzureFileShare")：
    
    ```
    BasePath,DstItemPathOrPrefix,ItemType,Disposition,MetadataFile,PropertiesFile
    "F:\shares\scratch\","MyAzureFileShare/",file,rename,"None",None
    ```

    一個儲存體帳戶可以指定多個共用。 如需詳細資訊，請參閱[準備資料集 CSV 檔案](../common/storage-import-export-tool-preparing-hard-drives-import.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#prepare-the-dataset-csv-file)。

5. 建立磁碟機集 CSV 檔案。 磁碟機集 CSV 檔案會列出內部部署匯出代理程式可用的磁碟。 例如，下列磁碟機集 CSV 檔案會列出 `X:`、`Y:` 和 `Z:` 磁碟以用於內部部署匯出工作：

    ```
    DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
    X,Format,SilentMode,Encrypt,
    Y,Format,SilentMode,Encrypt,
    Z,Format,SilentMode,Encrypt,
    ```
    
    如需詳細資訊，請參閱[準備磁碟機集 CSV 檔案](../common/storage-import-export-tool-preparing-hard-drives-import.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#prepare-initialdriveset-or-additionaldriveset-csv-file)。

6. 使用 [WAImportExport 工具](https://www.microsoft.com/en-us/download/details.aspx?id=55280)將資料複製到一或多個硬碟。

    ```
    WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>] DataSet:<dataset.csv>
    ```

    > [!Warning]  
    > 完成磁碟準備工作之後，請勿修改硬碟上的資料或日誌檔案。

7. [建立匯入作業](../common/storage-import-export-data-to-files.md#step-2-create-an-import-job)。
    
### <a name="robocopy"></a>Robocopy
Robocopy 是隨附於 Windows 和 Windows Server 的常用複製工具。 Robocopy 可在本機掛接檔案共用，然後將該掛接位置作為 Robocopy 命令中的目的地，以將資料傳輸到 Azure 檔案服務中。 使用 Robocopy 相當簡單：

1. [裝載 Azure 檔案共用](storage-how-to-use-files-windows.md)。 為了達到最佳效能，建議您在含有資料的伺服器上本機裝載 Azure 檔案共用。 某些情況可能無法這麼做，例如提供資料的檔案伺服器是 NAS 裝置時。 這時候，您完全可以將 Azure 檔案共用裝載到電腦上。 此範例會在命令列中使用 `net use` 以掛接檔案共用：

    ```
    net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> <storage-account-key> /user:Azure\<storage-account-name>
    ```

2. 在命令列中使用 `robocopy`，將資料移至 Azure 檔案共用：

    ```
    robocopy <path-to-local-share> <path-to-azure-file-share> /E /Z /MT:32
    ```
    
    Robocopy 有大量的選項，可讓您視需要修改複製行為。 如需詳細資訊，請檢視 [Robocopy](https://technet.microsoft.com/library/cc733145.aspx) 手冊頁面。

### <a name="azcopy"></a>AzCopy
AzCopy 是一種命令列公用程式，專為使用簡單命令高效率地在 Azure 檔案服務及 Azure Blob 儲存體之間複製資料所設計。 使用 AzCopy 非常容易：

1. 下載[最新版本的 AzCopy on Windows](http://aka.ms/downloadazcopy) 或 [AzCopy on Linux](../common/storage-use-azcopy-linux.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#download-and-install-azcopy)。
2. 在命令列中使用 `azcopy`，將資料移至 Azure 檔案共用。 Windows 的語法如下所示： 

    ```
    azcopy /Source:<path-to-local-share> /Dest:https://<storage-account>.file.core.windows.net/<file-share>/ /DestKey:<storage-account-key> /S
    ```

    Linux 的命令語法會有些不同：

    ```
    azcopy --source <path-to-local-share> --destination https://<storage-account>.file.core.windows.net/<file-share>/ --dest-key <storage-account-key> --recursive
    ```

    AzCopy 有大量的選項，可讓您視需要修改複製行為。 如需詳細資訊，請檢視 [AzCopy on Windows](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) 和 [AzCopy on Linux](../common/storage-use-azcopy-linux.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。

## <a name="automatically-mount-on-needed-pcsservers"></a>自動掛接於所需的電腦/伺服器上
若要取代內部部署檔案共用，最好預先在要使用該檔案共用的電腦上掛接共用。 這項作業可依據電腦清單自動完成。

> [!Note]  
> 裝載 Azure 檔案共用時，需要使用儲存體帳戶金鑰作為密碼，因此只建議您在受信任的環境中進行裝載。 

### <a name="windows"></a>Windows
您可以使用 PowerShell，在多部電腦上執行 mount 命令。 在下列範例中，`$computers` 是手動填入的，但您可以產生電腦清單以自動掛接。 例如，您可以填入此變數與 Active Directory 產生的結果。

```PowerShell
$computer = "MyComputer1", "MyComputer2", "MyComputer3", "MyComputer4"
$computer | ForEach-Object { Invoke-Command -ComputerName $_ -ScriptBlock { net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> <storage-account-key> /user:Azure\<storage-account-name> /PERSISTENT:YES } }
```

### <a name="linux"></a>Linux
在下列範例中，結合 SSH 的簡單 Bash 指令碼可能會產生相同的結果。 `$computer` 變數同樣是由使用者填入：

```
computer = ("MyComputer1" "MyComputer2" "MyComputer3" "MyComputer4")
for item in "${computer[@]}"
do
    ssh $item "sudo bash -c 'echo \"//<storage-account-name>.file.core.windows.net/<share-name> /mymountpoint cifs vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino\" >> /etc/fstab'", "sudo mount -a"
done
```

## <a name="next-steps"></a>後續步驟
- [規劃 Azure 檔案同步部署](storage-sync-files-planning.md)
- [針對 Windows 中的 Azure 檔案服務進行疑難排解](storage-troubleshoot-windows-file-connection-problems.md)
- [針對 Linux 中的 Azure 檔案服務進行疑難排解](storage-troubleshoot-linux-file-connection-problems.md)