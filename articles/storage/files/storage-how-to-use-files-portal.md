---
title: 使用 Azure 入口網站管理 Azure 檔案共用
description: 了解使用 Azure 入口網站來管理 Azure 檔案服務。
services: storage
author: wmgries
ms.service: storage
ms.topic: get-started-article
ms.date: 03/26/2018
ms.author: wgries
ms.component: files
ms.openlocfilehash: b4cace3922983ed93987069ac7cf59f2cb69403b
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45578543"
---
# <a name="managing-azure-file-shares-with-the-azure-portal"></a>使用 Azure 入口網站管理 Azure 檔案共用 
[Azure 檔案](storage-files-introduction.md)是 Microsoft 易於使用的雲端檔案系統。 Azure 檔案共用可在 Windows、Linux 和 macOS 中掛接。 本指南會逐步說明透過 [Azure 入口網站](https://portal.azure.com/)來使用 Azure 檔案共用的基本概念。 了解如何：

> [!div class="checklist"]
> * 建立資源群組和儲存體帳戶
> * 建立 Azure 檔案共用 
> * 建立目錄
> * 上傳檔案 
> * 下載檔案
> * 建立及使用共用快照集

如果您沒有 Azure 訂用帳戶，您可以在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="create-a-storage-account"></a>建立儲存體帳戶
[!INCLUDE [storage-files-create-storage-account-portal](../../../includes/storage-files-create-storage-account-portal.md)]

## <a name="create-a-file-share"></a>建立檔案共用
若要建立檔案共用：

1. 從儀表板中選取儲存體帳戶。
2. 在儲存體帳戶頁面上的 [服務] 區段中，選取 [檔案]。
    ![儲存體帳戶之服務區段的螢幕擷取畫面；選取檔案服務](media/storage-how-to-use-files-portal/create-file-share-1.png)

3. 在 [檔案服務] 頁面頂端的功能表上，按一下 [+ 檔案共用]。 [新增檔案共用] 頁面隨即顯示。
4. 在 [名稱] 中輸入 *myshare*。
5. 按一下 [確定] 以建立 Azure 檔案共用。

## <a name="work-with-the-contents-of-the-azure-file-share"></a>使用 Azure 檔案共用的內容
您現在已建立 Azure 檔案共用，並且可使用 SMB 將檔案共用掛接在 [Windows](storage-how-to-use-files-windows.md)、[Linux](storage-how-to-use-files-linux.md) 或 [macOS](storage-how-to-use-files-mac.md) 上。 或者，您可以使用 Azure 入口網站來處理 Azure 檔案共用。 透過 Azure 入口網站提出的所有要求都是使用 REST API 進行，讓您不需 SMB 存取權，即可在用戶端上建立、修改及刪除檔案和目錄。

### <a name="create-a-directory"></a>建立目錄
若要在 Azure 檔案共用的根目錄建立名為 *myDirectory* 的新目錄：

1. 在 [檔案服務] 頁面上，選取 **myshare** 檔案共用。 檔案共用的頁面隨即開啟。
2. 在頁面頂端的功能表上，選取 [+ 新增目錄]。 [新增目錄] 頁面隨即顯示。
3. 輸入 *myDirectory*，然後按一下 [確定]。

### <a name="upload-a-file"></a>上傳檔案 
若要示範如何上傳檔案，您必須先建立或選取要上傳的檔案。 您可以用您適用的任何方式來執行這項操作。 選取您要上傳的檔案之後：

1. 按一下 **myDirectory** 目錄。 **myDirectory** 面板隨即開啟。
2. 在頂端的功能表中，按一下 [上傳]。 [檔案上傳] 面板隨即開啟。  
    ![上傳檔案面板的螢幕擷取畫面](media/storage-how-to-use-files-portal/upload-file-1.png)

3. 按一下資料夾圖示開啟視窗，以瀏覽您的本機檔案。 
4. 選取檔案，然後按一下 [開啟]。 
5. 在 [上傳檔案] 頁面中確認檔案名稱，然後按一下 [上傳]。
6. 完成時，檔案應會出現在 [myDirectory] 頁面的清單中。

### <a name="download-a-file"></a>下載檔案
您可以在檔案上按一下滑鼠右鍵，以下載您所上傳的檔案複本。 按一下 [下載] 按鈕後，確切的經驗將取決於您所使用的作業系統和瀏覽器。

## <a name="create-and-modify-share-snapshots"></a>建立及修改共用快照集
可以使用 Azure 檔案共用來執行的另一項實用工作，是建立共用快照集。 快照集會保留 Azure 檔案共用的時間點。 共用快照集類似於您可能已經很熟悉的下列作業系統技術：
- Windows 檔案系統的[磁碟區陰影複製服務 (VSS)](https://docs.microsoft.com/windows/desktop/VSS/volume-shadow-copy-service-portal)，例如 NTFS 和 ReFS
- Linux 系統的[邏輯磁碟區管理員 (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) 快照集
- macOS 的 [Apple 檔案系統 (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) 快照集。 

若要建立共用快照集：

1. 從您的儀表板 > [檔案] > **myshare** 開啟儲存體帳戶，以開啟檔案共用的頁面。 
2. 在檔案共用的頁面上，按一下頁面頂端之功能表上的 [快照] 按鈕，然後選取 [建立快照集]。  
    ![說明如何尋找建立快照集按鈕的螢幕擷取畫面](media/storage-how-to-use-files-portal/create-snapshot-1.png)

### <a name="list-and-browse-share-snapshots"></a>列出並瀏覽共用快照集
快照集建立之後，您可以再按一下 [快照集]，然後選取 [檢視快照集] 以列出共用的快照集。 產生的窗格會顯示此共用的快照集。 按一下共用快照集即可加以瀏覽。

### <a name="restore-from-a-share-snapshot"></a>從共用快照集還原
為了示範如何從共用快照集還原檔案，我們必須先從即時的 Azure 檔案共用中刪除檔案。 瀏覽到 *myDirectory* 資料夾中，以滑鼠右鍵按一下您上傳的檔案，然後按一下 [刪除]。 然後，若要從共用快照集還原該檔案：

1. 按一下頂層功能表中的 [快照]，然後選取 [檢視快照集]。 
2. 按一下您先前建立的快照集，它將會在新頁面中開啟內容。 
3. 按一下快照集中的 **myDirectory**，您應會看到您所刪除的檔案。 
4. 以滑鼠右鍵按一下已刪除的檔案，然後選取 [還原]。
5. 此時會出現快顯視窗，供您選擇是要將檔案還原為複本，還是覆寫原始檔案。 由於我們已刪除原始檔案，我們可以選取 [覆寫原始檔案]，將檔案還原至我們將其刪除前的狀態。 按一下 [確定]，將檔案還原至 Azure 檔案共用。  
    ![還原檔案對話方塊的螢幕擷取畫面](media/storage-how-to-use-files-portal/restore-snapshot-1.png)

6. 檔案還原完成後，請關閉快照集的頁面，並返回 **myshare** > **myDirectory**，此時檔案應已回到原來的位置。

### <a name="delete-a-share-snapshot"></a>刪除共用快照集
若要刪除共用快照集，請[瀏覽至共用快照集的清單](#list-and-browse-a-share-snapshot)。 按一下共用快照集名稱旁的核取方塊，然後選取 [刪除] 按鈕。

![刪除共用快照集的螢幕擷取畫面](media/storage-how-to-use-files-portal/delete-snapshot-1.png)

## <a name="clean-up-resources"></a>清除資源
[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>後續步驟
- [使用 Azure PowerShell 管理檔案共用](storage-how-to-use-files-powershell.md)
- [使用 Azure CLI 管理檔案共用](storage-how-to-use-files-cli.md)
- [使用 Azure 儲存體總管來管理檔案共用](storage-how-to-use-files-storage-explorer.md)
- [規劃 Azure 檔案部署](storage-files-planning.md)