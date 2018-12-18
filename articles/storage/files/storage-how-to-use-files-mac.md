---
title: 透過 macOS 的 SMB 掛接 Azure 檔案共用 | Microsoft Docs
description: 了解如何透過 macOS 的 SMB 掛接 Azure 檔案共用。
services: storage
author: RenaShahMSFT
ms.service: storage
ms.topic: get-started-article
ms.date: 09/19/2017
ms.author: renash
ms.component: files
ms.openlocfilehash: ace77b8a15ace71b5b372564bc10c6f4845e1482
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39527213"
---
# <a name="mount-azure-file-share-over-smb-with-macos"></a>透過 macOS 的 SMB 掛接 Azure 檔案共用
[Azure 檔案服務](storage-files-introduction.md)是 Microsoft 易於使用的雲端檔案系統。 您可以使用 macOS El Capitan 10.11+ 所推出的業界標準 SMB 3 通訊協定來掛接 Azure 檔案共用。 本文說明兩種在 macOS 上掛接 Azure 檔案共用的不同方式：使用 Finder UI 和使用終端機。

> [!Note]  
> 在透過 SMB 掛接 Azure 檔案共用之前，建議您停用 SMB 封包簽章。 不這樣做可能會導致從 macOS 存取 Azure 檔案共用時效能不佳。 SMB 連線將會加密，因此這不會影響您連線的安全性。 從終端機中，下列命令會停用 SMB 封包簽章，如同[停用 SMB 封包簽章的 Apple 支援文章](https://support.apple.com/HT205926)所描述：  
>    ```
>    sudo -s
>    echo "[default]" >> /etc/nsmb.conf
>    echo "signing_required=no" >> /etc/nsmb.conf
>    exit
>    ```

## <a name="prerequisites-for-mounting-an-azure-file-share-on-macos"></a>在 macOS 上掛接 Azure 檔案共用的必要條件
* **儲存體帳戶名稱**：若要掛接 Azure 檔案共用，您需要儲存體帳戶的名稱。

* **儲存體帳戶金鑰**：若要掛接 Azure 檔案共用，您需要主要 (或次要) 金鑰。 掛接目前不支援 SAS 金鑰。

* **請確定已開啟連接埠 445**SMB 透過 TCP 通訊埠 445 進行通訊。 在用戶端電腦 (Mac) 中，請檢查並確定您的防火牆不會封鎖 TCP 通訊埠 445。

## <a name="mount-an-azure-file-share-via-finder"></a>透過搜尋工具掛接 Azure 檔案共用
1. **開啟搜尋工具**：搜尋工具在 macOS 中為預設開啟，但是您可以按一下 Dock 上的 [macOS 臉部圖示] 確保它是目前選取的應用程式：  
    ![macOS 臉部圖示](./media/storage-how-to-use-files-mac/mount-via-finder-1.png)

2. **從 [前往] 功能表選取 [連線至伺服器]**：使用[必要條件](#preq)的 UNC 路徑，將開頭的雙反斜線 (`\\`) 轉換至 `smb://`，並將所有其他的反斜線 (`\`) 轉換至正斜線 (`/`)。 您的連結應看起來如下所示：![[連線至伺服器] 對話方塊](./media/storage-how-to-use-files-mac/mount-via-finder-2.png)

3. **提示您輸入使用者名稱和密碼時，使用儲存體帳戶名稱和儲存體帳戶金鑰**：當您在 [連線至伺服器] 對話方塊中按一下 [連線] 時，系統會提示您的使用者名稱和密碼 (這會使用您的 macOS 使用者名稱自動填入)。 您可以選擇將儲存體帳戶名稱/儲存體帳戶金鑰置於您的 macOS 金鑰鏈。

4. **視需要使用 Azure 檔案共用**：在使用共用名稱和儲存體帳戶金鑰替換使用者名稱和密碼之後，系統將掛接共用。 您可以依照平常使用本機資料夾/檔案共用的方式使用，包括將檔案拖放到檔案共用中：

    ![已掛接的 Azure 檔案共用快照集](./media/storage-how-to-use-files-mac/mount-via-finder-3.png)

## <a name="mount-an-azure-file-share-via-terminal"></a>透過終端機掛接 Azure 檔案共用
1. 使用您的儲存體帳戶名稱取代 `<storage-account-name>`。 當系統提示時，請提供儲存體帳戶金鑰作為密碼。 

    ```
    mount_smbfs //<storage-account-name>@<storage-account-name>.file.core.windows.net/<share-name> <desired-mount-point>
    ```

2. **視需要使用 Azure 檔案共用**：Azure 檔案共用將會掛接於先前命令所指定的掛接點。  

    ![已掛接的 Azure 檔案共用快照集](./media/storage-how-to-use-files-mac/mount-via-terminal-1.png)

## <a name="next-steps"></a>後續步驟
請參閱這些連結，以取得 Azure 檔案服務的詳細資訊。

* [Apple 支援文章 - 如何在 Mac 上連線檔案共用](https://support.apple.com/HT204445)
* [常見問題集](../storage-files-faq.md)
* [在 Windows 上進行疑難排解](storage-troubleshoot-windows-file-connection-problems.md)      
* [在 Linux 上進行疑難排解](storage-troubleshoot-linux-file-connection-problems.md)    
