---
title: 搭配 Linux 使用 Azure 檔案 | Microsoft Docs
description: 了解如何透過 Linux 上的 SMB 掛接 Azure 檔案共用。
services: storage
author: RenaShahMSFT
ms.service: storage
ms.topic: article
ms.date: 03/29/2018
ms.author: renash
ms.component: files
ms.openlocfilehash: bbd250e9ee987403d670b2605fdb8deda8c19092
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43782040"
---
# <a name="use-azure-files-with-linux"></a>搭配 Linux 使用 Azure 檔案
[Azure 檔案服務](storage-files-introduction.md)是 Microsoft 易於使用的雲端檔案系統。 可以使用 [SMB 核心用戶端](https://wiki.samba.org/index.php/LinuxCIFS)將 Azure 檔案共用裝載在 Linux 發行版本中。 本文將說明掛接 Azure 檔案共用的兩種方式：使用 `mount` 命令的隨選掛接，以及建立項目 `/etc/fstab` 的開機掛接。

> [!NOTE]  
> 若要在 Azure 區域之外掛接 Azure 檔案共用，例如內部部署或是在不同的 Azure 區域，作業系統必須支援 SMB 3.0 的加密功能。

## <a name="prerequisites-for-mounting-an-azure-file-share-with-linux-and-the-cifs-utils-package"></a>以 Linux 掛接 Azure 檔案共用和 cifs-utils 套件的必要條件
<a id="smb-client-reqs"></a>
* **選擇 Linux 發行版本以符合掛接需求。**  
      Azure 檔案服務可以透過 SMB 2.1 和 SMB 3.0 掛接。 對於來自用戶端內部部署或其他 Azure 區域的連線，Azure 檔案服務會拒絕 SMB 2.1 (或是沒有加密的 SMB 3.0)。 如果已針對儲存體帳戶啟用「需要安全傳輸」，則 Azure 檔案服務僅允許使用具有加密的 SMB 3.0 之連線。
    
    SMB 3.0 加密支援是在 Linux 核心版本 4.11 導入，並且針對熱門的 Linux 發行版本反向導入到舊版核心版本。 在本文件發行時，Azure 資源庫的下列發行版本支援掛接此資料表標頭中指定的選項。 

* **具有對應掛接功能的最低建議版本 (SMB 2.1 版與 SMB 3.0 版)。**    
    
    |   | SMB 2.1 <br>(掛接在相同 Azure 區域內的 VM 上) | SMB 3.0 <br>(從內部部署環境和跨區域掛接) |
    | --- | :---: | :---: |
    | Ubuntu Server | 14.04+ | 16.04+ |
    | RHEL | 7+ | 7.5+ |
    | CentOS | 7+ |  7.5+ |
    | Debian | 8+ |   |
    | openSUSE | 13.2+ | 42.3+ |
    | SUSE Linux Enterprise Server | 12 | 12 SP3+ |
    
    如果此處未列出您的 Linux 發行版本，您可以使用下列命令查看 Linux 核心版本：    

   ```bash
   uname -r
   ```    

* <a id="install-cifs-utils"></a>**已安裝 cifs-utils 套件。**  
    可使用套件管理員將 cifs-utils 套件安裝在所選擇的 Linux 發行版本上。 

    在 **Ubuntu** 和 **Debian 型**發行版本上，請使用 `apt-get` 封裝管理員：

    ```bash
    sudo apt-get update
    sudo apt-get install cifs-utils
    ```

    在 **RHEL** 和 **CentOS** 上，請使用 `yum` 封裝管理員：

    ```bash
    sudo yum install cifs-utils
    ```

    在 **openSUSE** 上，請使用 `zypper` 封裝管理員：

    ```bash
    sudo zypper install cifs-utils
    ```

    在其他發行版本上，請使用適當的封裝管理員或[從來源編譯](https://wiki.samba.org/index.php/LinuxCIFS_utils#Download)
    
* **對於掛接的共用決定目錄/檔案權限**：下列範例使用權限 `0777` 提供所有使用者的讀取、寫入和執行權限。 您可以視需要將它取代為其他 [chmod 權限](https://en.wikipedia.org/wiki/Chmod)。 

* **儲存體帳戶名稱**：若要掛接 Azure 檔案共用，您需要儲存體帳戶的名稱。

* **儲存體帳戶金鑰**：若要掛接 Azure 檔案共用，您需要主要 (或次要) 儲存體金鑰。 掛接目前不支援 SAS 金鑰。

* **請確定已開啟連接埠 445**：SMB 透過 TCP 通訊埠 445 進行通訊 - 請檢查您的防火牆不會將 TCP 通訊埠 445 從用戶端電腦封鎖。

## <a name="mount-the-azure-file-share-on-demand-with-mount"></a>使用 `mount` 隨需掛接 Azure 檔案共用
1. **[在您的 Linux 發行版本上安裝 cifs-utils 封裝](#install-cifs-utils)**。

2. **建立掛接點的資料夾**：掛接點的資料夾可以在檔案系統上任何位置建立，但是常見慣例是在 `/mnt` 資料夾底下建立這個資料夾。 例如︰

    ```bash
    mkdir /mnt/MyAzureFileShare
    ```

3. **使用 mount 命令掛接 Azure 檔案共用**：請記得針對您的環境使用適當的資訊來取代 `<storage-account-name>`、`<share-name>`、`<smb-version>`、`<storage-account-key>` 和 `<mount-point>`。 如果您的 Linux 發行版本支援具有加密的 SMB 3.0 (請參閱[了解 SMB 用戶端需求](#smb-client-reqs)以取得詳細資訊)，請針對 `<smb-version>` 使用 `3.0`。 對於不支援具有加密的 SMB 3.0 之 Linux 發行版本，請針對 `<smb-version>` 使用 `2.1`。 請注意，Azure 檔案共用只能使用 SMB 3.0 掛接在 Azure 區域外部 (包括內部部署或不同的 Azure 區域)。 

    ```bash
    sudo mount -t cifs //<storage-account-name>.file.core.windows.net/<share-name> <mount-point> -o vers=<smb-version>,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
    ```

> [!Note]  
> 使用 Azure 檔案共用後，即可使用 `sudo umount <mount-point>` 取消掛接共用。

## <a name="create-a-persistent-mount-point-for-the-azure-file-share-with-etcfstab"></a>使用 `/etc/fstab` 建立 Azure 檔案共用的持續掛接點
1. **[在您的 Linux 發行版本上安裝 cifs-utils 封裝](#install-cifs-utils)**。

2. **建立掛接點的資料夾**：掛接點的資料夾可以在檔案系統上任何位置建立，但是常見慣例是在 `/mnt` 資料夾底下建立這個資料夾。 每當您建立這個資料夾，請記下資料夾的完整路徑。 例如，下列命令會在 `/mnt` 底下建立新的資料夾 (路徑為完整路徑)。

    ```bash
    sudo mkdir /mnt/MyAzureFileShare
    ```

3. **建立認證檔來儲存檔案共用的使用者名稱 (儲存體帳戶名稱) 和密碼 (儲存體帳戶金鑰)。** 請記得將 `<storage-account-name>` 和 `<storage-account-key>` 取代為您環境的適當資訊。 

    ```bash
    if [ ! -d "/etc/smbcredentials" ]; then
        sudo mkdir /etc/smbcredentials
    fi

    if [ ! -f "/etc/smbcredentials/<storage-account-name>.cred" ]; then
        sudo bash -c 'echo "username=<storage-account-name>" >> /etc/smbcredentials/<storage-account-name>.cred'
        sudo bash -c 'echo "password=<storage-account-key>" >> /etc/smbcredentials/<storage-account-name>.cred'
    fi
    ```

4. **變更認證檔的權限，因此只有 root 才能讀取或修改密碼檔。** 由於儲存體帳戶金鑰基本上是儲存體帳戶的超級管理員密碼，因此請務必設定只有 root 才能存取的檔案權限，讓更低權限的使用者無法擷取儲存體帳戶金鑰。   

    ```bash
    sudo chmod 600 /etc/smbcredentials/<storage-account-name>.cred
    ```

5. **使用下列命令將下列一行新增至 `/etc/fstab`**：請記得針對您的環境使用適當的資訊來取代 `<storage-account-name>`、`<share-name>`、`<smb-version>` 和 `<mount-point>`。 如果您的 Linux 發行版本支援具有加密的 SMB 3.0 (請參閱[了解 SMB 用戶端需求](#smb-client-reqs)以取得詳細資訊)，請針對 `<smb-version>` 使用 `3.0`。 對於不支援具有加密的 SMB 3.0 之 Linux 發行版本，請針對 `<smb-version>` 使用 `2.1`。 請注意，Azure 檔案共用只能使用 SMB 3.0 掛接在 Azure 區域外部 (包括內部部署或不同的 Azure 區域)。 

    ```bash
    sudo bash -c 'echo "//<storage-account-name>.file.core.windows.net/<share-name> <mount-point> cifs nofail,vers=<smb-version>,credentials=/etc/smbcredentials/<storage-account-name>.cred,dir_mode=0777,file_mode=0777,serverino" >> /etc/fstab'
    ```

> [!Note]  
> 您可以使用 `sudo mount -a` 在編輯 `/etc/fstab` 之後掛接 Azure 檔案共用而不需要重新開機。

## <a name="feedback"></a>意見反應
Linux 使用者，歡迎您提供相關資訊！

適用於 Linux 的 Azure 檔案使用者群組提供了一個論壇，讓您在 Linux 上評估並採用檔案儲存體之際，得以分享意見反應。 請寄電子郵件至 [Azure 檔案 Linux 使用者](mailto:azurefileslinuxusers@microsoft.com)，以加入使用者的群組。

## <a name="next-steps"></a>後續步驟
請參閱這些連結，以取得 Azure 檔案服務的詳細資訊。
* [Azure 檔案服務簡介](storage-files-introduction.md)
* [規劃 Azure 檔案部署](storage-files-planning.md)
* [常見問題集](../storage-files-faq.md)
* [疑難排解](storage-troubleshoot-linux-file-connection-problems.md)
