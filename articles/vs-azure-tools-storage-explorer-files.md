---
title: 搭配使用儲存體總管與 Azure 檔案儲存體 | Microsoft Docs
description: 了解如何透過儲存體總管來使用檔案共用和檔案。
services: storage
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: ''
ms.assetid: ''
ms.service: storage
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 03/09/2017
ms.author: cawa
ms.openlocfilehash: 536092c1ee50045e6df4e2933c7d6eb0ee7a8b42
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39429782"
---
# <a name="using-storage-explorer-with-azure-file-storage"></a>搭配使用儲存體總管與 Azure 檔案儲存體

Azure 檔案儲存體是使用標準伺服器訊息區塊 (SMB) 通訊協定，在雲端中提供檔案共用功能的服務。 SMB 2.1 和 SMB 3.0 皆受到支援。 使用 Azure 檔案儲存體時，您可以快速地將依賴檔案共用功能的舊式應用程式移轉至 Azure，而不必浪費成本來重新撰寫程式。 您可以使用檔案儲存體向全球公開資料，或私下儲存應用程式資料。 在本文中，您將學習如何使用儲存體總管來使用檔案共用和檔案。

## <a name="prerequisites"></a>必要條件

若要完成這篇文章中的步驟，您需要下列項目︰

- [下載並安裝儲存體總管](http://www.storageexplorer.com/)

- [連接到 Azure 儲存體帳戶或服務](https://docs.microsoft.com//azure/vs-azure-tools-storage-manage-with-storage-explorer#connect-to-a-storage-account-or-service)

## <a name="create-a-file-share"></a>建立檔案共用

所有檔案必須都位於檔案共用，這是檔案的邏輯群組。 帳戶可以包含無限數量的檔案共用，每個共用可以儲存無限數量的檔案。

下列步驟說明如何在儲存體總管中建立檔案共用。

1. 開啟儲存體總管。

1. 在左窗格中，展開您要在其中建立檔案共用的儲存體帳戶

1. 以滑鼠右鍵按一下 [檔案共用]，然後從內容功能表中 - 選取 [建立檔案共用]。

    ![建立檔案共用](media/vs-azure-tools-storage-explorer-files/image1.png)

1. [檔案共用] 資料夾底下會出現文字方塊。 輸入檔案共用的名稱。 請參閱 [共用命名規則](https://docs.microsoft.com//azure/storage/storage-dotnet-how-to-use-blobs#create-a-container) 區段，以取得命名檔案共用的規則和限制的清單。

    ![命名共用](media/vs-azure-tools-storage-explorer-files/image2.png)

1. 完成建立檔案共用時按下 **Enter**鍵，或按下 **Esc** 鍵取消。 一旦成功建立檔案共用，它就會顯示在所選的儲存體帳戶的 [檔案共用] 資料夾底下。

    ![新增共用](media/vs-azure-tools-storage-explorer-files/image3.png)

## <a name="view-a-file-shares-contents"></a>檢視檔案共用的內容

檔案共用包含檔案和資料夾 (也包含檔案)。

下列步驟說明如何在儲存體總管中檢視檔案共用的內容：+

1. 開啟儲存體總管。

1. 在左窗格中，展開儲存體帳戶，其中包含您要檢視的檔案共用。

1. 展開儲存體帳戶的 [檔案共用]。

1. 以滑鼠右鍵按一下您想要檢視的檔案共用，從內容功能表中，選取 [開啟]。 您也可以按兩下想要檢視的檔案共用。

    ![開啟共用](media/vs-azure-tools-storage-explorer-files/image4.png)

1. 主窗格會顯示檔案共用的內容。
    
    ![檔案共用的內容](media/vs-azure-tools-storage-explorer-files/image5.png)

## <a name="delete-a-file-share"></a>刪除檔案共用

檔案共用可以輕鬆地建立並視需要刪除。 (若要查看如何刪除個別的檔案，請參閱[管理檔案共用中的檔案](https://docs.microsoft.com//azure/vs-azure-tools-storage-explorer-blobs#managing-blobs-in-a-blob-container)。)

下列步驟說明如何在儲存體總管中刪除檔案共用：

1. 開啟儲存體總管。

1. 在左窗格中，展開儲存體帳戶，其中包含您要檢視的檔案共用。

1. 展開儲存體帳戶的 [檔案共用]。

1. 以滑鼠右鍵按一下您想要刪除的檔案共用，從內容功能表中，選取 [刪除]。 您也可以按 [刪除] 以刪除目前選取的檔案共用。

    ![刪除](media/vs-azure-tools-storage-explorer-files/image6.png)

1. 選取確認對話方塊上的 [是]  。
    
    ![確認對話方塊](media/vs-azure-tools-storage-explorer-files/image7.png)

## <a name="copy-a-file-share"></a>複製檔案共用

儲存體總管可讓您將檔案共用複製到剪貼簿，然後將該檔案共用貼到另一個儲存體帳戶。 (若要查看如何複製個別的檔案，請參閱[管理檔案共用中的檔案](https://docs.microsoft.com//azure/vs-azure-tools-storage-explorer-blobs#managing-blobs-in-a-blob-container)。)

下列步驟說明如何將檔案共用從某個儲存體帳戶複製到另一個儲存體帳戶。

1. 開啟儲存體總管。

1. 在左窗格中，展開儲存體帳戶，其中包含您要複製的檔案共用。

1. 展開儲存體帳戶的 [檔案共用]。

1. 以滑鼠右鍵按一下您想要複製的檔案共用，從內容功能表中，選取 [複製檔案共用]。

    ![複製檔案共用](media/vs-azure-tools-storage-explorer-files/image8.png)

1. 以滑鼠右鍵按一下想要貼上至檔案共用的「目標」儲存體帳戶，從內容功能表中，選取 [貼上檔案共用]。

    ![貼上檔案共用](media/vs-azure-tools-storage-explorer-files/image9.png)

## <a name="get-the-sas-for-a-file-share"></a>取得檔案共用的 SAS

[共用存取簽章 (SAS)](https://docs.microsoft.com//azure/storage/storage-dotnet-shared-access-signature-part-1) 可提供您儲存體帳戶中資源的委派存取。 這表示您可以在無需分享您帳戶存取金鑰的情況下，將您儲存體帳戶中的物件有限權限授與用戶端，該用戶端便可在指定的時間期間內及使用指定的權限集來進行存取。

下列步驟說明如何建立檔案共用的 SAS：

1. 開啟儲存體總管。

1. 在左窗格中，展開儲存體帳戶，其中包含您要取得 SAS 的檔案共用。

1. 展開儲存體帳戶的 [檔案共用]。

1. 以滑鼠右鍵按一下想要的檔案共用，從內容功能表中，選取 [取得共用存取簽章]。

    ![取得共用存取簽章](media/vs-azure-tools-storage-explorer-files/image10.png)

1. 在 [共用存取簽章]  對話方塊中，指定您要用於資源的原則、開始和到期日期、時區和存取層級。

    ![SAS 對話方塊](media/vs-azure-tools-storage-explorer-files/image11.png)

1. 當您完成指定 SAS 選項時，選取 [建立] 。

1. 第二個 [共用存取簽章] 對話方塊會顯示，列出您可以用來存取儲存體資源的檔案共用及 URL 和 QueryStrings。 選取您想要複製到剪貼簿的 URL 旁邊的 [複製]  。
    
    ![第二個 SAS 對話方塊](media/vs-azure-tools-storage-explorer-files/image12.png)

1. 完成時，選取 [關閉] 。

## <a name="manage-access-policies-for-a-file-share"></a>管理檔案共用的存取原則

下列步驟說明如何管理 (新增和移除) 檔案共用的存取原則︰+。 存取原則用來建立 SAS URL，使用者可藉此使用在定義的時間期間存取儲存體檔案資源。

1. 開啟儲存體總管。

1. 在左窗格中，展開儲存體帳戶，其中包含您要管理其存取原則的檔案共用。

1. 展開儲存體帳戶的 [檔案共用]。

1. 選取想要的檔案共用，從內容功能表中，選取 [管理存取原則]。

    ![管理存取原則內容功能表](media/vs-azure-tools-storage-explorer-files/image13.png)

1. [存取原則] 對話方塊會列出已針對所選的檔案共用建立的任何存取原則。
    
    ![存取原則](media/vs-azure-tools-storage-explorer-files/image14.png)

1. 根據存取原則管理工作遵循下列步驟︰
    
    - **新增新的存取原則** -選取 [新增]。 一旦產生，[存取原則]  對話方塊將會顯示新加入的存取原則 (具有預設設定)。

    - **編輯存取原則** - 進行任何所需的編輯，然後選取 [儲存]。

    - **移除存取原則** -選取您想要移除的存取原則旁邊的 [移除]。

1. 使用您稍早建立的存取原則建立新的 SAS URL︰
    
    ![取得 SAS](media/vs-azure-tools-storage-explorer-files/image15.png)
    
    ![SAS 名稱和屬性](media/vs-azure-tools-storage-explorer-files/image16.png)

## <a name="managing-files-in-a-file-share"></a>檔案共用中的管理檔案

建立檔案共用之後，您可以將檔案上傳至檔案共用、將檔案下載到本機電腦或在本機電腦上開啟檔案等等。

下列步驟說明如何管理檔案共用中的檔案 (及資料夾)。

1.  開啟儲存體總管。

1.  在左窗格中，展開儲存體帳戶，其中包含您要管理的檔案共用。

1.  展開儲存體帳戶的 [檔案共用]。

1.  按兩下想要檢視的檔案共用。

1.  主窗格會顯示檔案共用的內容。

    ![檔案共用的內容](media/vs-azure-tools-storage-explorer-files/image17.png)

1.  主窗格會顯示檔案共用的內容。

1.  根據您想要執行的工作遵循下列步驟︰

    - **將檔案上傳至檔案共用**

        a.  在主窗格工具列上，選取 [上傳]，然後從下拉式功能表選取 [上傳檔案]。

        ![上傳檔案](media/vs-azure-tools-storage-explorer-files/image18.png)
        
        b. 在 [上傳檔案] 對話方塊中，選擇 [檔案] 文字方塊右側的省略符號 (**…**) 按鈕，以選取您想要上傳的檔案。

        ![新增檔案](media/vs-azure-tools-storage-explorer-files/image19.png)

        c. 選取 [上傳] 。

    - **將資料夾上傳至檔案共用**
        
        a. 在主窗格工具列上，選取 [上傳]，然後從下拉式功能表選取 [上傳資料夾]。

        ![上傳資料夾功能表](media/vs-azure-tools-storage-explorer-files/image20.png)

        b. 在 [上傳資料夾] 對話方塊中，選擇 [資料夾] 文字方塊右側的省略符號 (**…**) 按鈕，以選取您想要上傳其內容的資料夾。

        c. 選擇性地指定選取的資料夾的內容將會上傳至其中的目標資料夾。 如果目標資料夾不存在，系統就會加以建立。

        d. 選取 [上傳] 。

    - **將檔案下載到本機電腦**
        
        a. 選取您想要下載的檔案。
        
        b. 在主窗格工具列上選取 [下載] 。
        
        c. 在 [指定儲存下載的檔案的位置] 對話方塊中，指定要下載檔案的位置，和您想要給予它的名稱。

        d. 選取 [ **儲存**]。

    - **在您的本機電腦上開啟檔案**
        
        a.  選取您想要開啟的檔案。
        
        b.  在主窗格工具列上選取 [開啟] 。
        
        c.  檔案會被下載，並使用與檔案的基礎檔案類型相關聯的應用程式開啟。

    - **將檔案複製到剪貼簿**

        a. 選取您想要複製的檔案。

        b. 在主窗格工具列上選取 [複製] 。

        c. 在左窗格中，瀏覽至另一個檔案共用，然後在主窗格中按兩下加以檢視。

        d. 在主窗格工具列上選取 [貼上] 以建立檔案的複本。

    - **刪除檔案**

        a. 選取您想要刪除的檔案。

        b. 在主窗格工具列上選取 [刪除] 。

        c. 選取確認對話方塊上的 [是]  。

## <a name="next-steps"></a>後續步驟

- 檢視 [最新的儲存體 Explorer 版本資訊與影片](http://www.storageexplorer.com/)。

- 了解如何 [利用 Azure Blob、資料表、佇列和檔案建立應用程式](https://azure.microsoft.com/documentation/services/storage/)。
