---
title: 使用儲存體總管來管理 Azure Blob 儲存體資源 | Microsoft Docs
description: 使用儲存體總管來管理 Azure Blob 容器和 Blob
services: storage
documentationcenter: na
author: cawa
manager: paulyuk
editor: ''
ms.assetid: 2f09e545-ec94-4d89-b96c-14783cc9d7a9
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/18/2016
ms.author: cawa
ms.openlocfilehash: 2207bac1ef7cb6393d7dc173f96ddda719bcdfdf
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31528523"
---
# <a name="manage-azure-blob-storage-resources-with-storage-explorer"></a>使用儲存體總管來管理 Azure Blob 儲存體資源
## <a name="overview"></a>概觀
[Azure Blob 儲存體](storage/blobs/storage-dotnet-how-to-use-blobs.md)是一項儲存大量非結構化資料的服務 (例如文字或二進位資料)，全球任何地方都可透過 HTTP 或 HTTPS 來存取這些資料。
您可以使用 Blob 儲存體向全球公開資料，或私下儲存應用程式資料。 在本文中，您將學習如何使用儲存體總管 來處理 Blob 容器和 Blob。

## <a name="prerequisites"></a>先決條件
若要完成這篇文章中的步驟，您需要下列項目︰

* [下載並安裝儲存體總管](http://www.storageexplorer.com)
* [連接到 Azure 儲存體帳戶或服務](vs-azure-tools-storage-manage-with-storage-explorer.md#connect-to-a-storage-account-or-service)

## <a name="create-a-blob-container"></a>建立 Blob 容器
所有 blob 必須都位於 blob 容器，這是 blob 的邏輯群組。 帳戶可以包含無限數量的容器，每個容器可以儲存無限數量的 blob。

下列步驟說明如何在儲存體總管中建立 blob 容器。

1. 開啟儲存體總管。
2. 在左窗格中，展開您要在其中建立 blob 容器的儲存體帳戶。
3. 以滑鼠右鍵按一下 [Blob 容器]，從內容功能表中，選取 [建立 Blob 容器]。

   ![建立 Blob 容器內容功能表][0]
4. [Blob 容器]  資料夾底下會出現文字方塊。 輸入 blob 容器的名稱。 請參閱[建立容器與設定權限](storage/blobs/storage-quickstart-blobs-dotnet.md#create-the-container-and-set-permissions)，以取得為 Blob 容器命名的規則和限制。

   ![建立 Blob 容器文字方塊][1]
5. 完成建立 blob 容器時按下 **Enter**鍵，或按下 **Esc** 鍵取消。 一旦成功建立 blob 容器，它就會顯示在所選的儲存體帳戶的 [Blob 容器]  資料夾底下。

   ![Blob 容器已建立][2]

## <a name="view-a-blob-containers-contents"></a>檢視 blob 容器的內容
Blob 容器包含 blob 和資料夾 (也包含 blob)。

下列步驟說明如何在儲存體總管中檢視 blob 容器的內容：

1. 開啟儲存體總管。
2. 在左窗格中，展開儲存體帳戶，其中包含您要檢視的 blob 容器。
3. 展開儲存體帳戶的 [Blob 容器] 。
4. 以滑鼠右鍵按一下您想要檢視的 blob 容器，從內容功能表中，選取 [開啟 Blob 容器編輯器] 。
   您也可以按兩下想要檢視的 blob 容器。

   ![開啟 blob 容器編輯器內容功能表][19]
5. 主窗格會顯示 blob 容器的內容。

   ![Blob 容器編輯器][3]

## <a name="delete-a-blob-container"></a>刪除 Blob 容器
Blob 容器可以輕鬆地建立並視需要刪除。 (若要查看如何刪除個別的 blob，請參閱[管理 blob 容器中的 blob 區段](#managing-blobs-in-a-blob-container)。)

下列步驟說明如何在儲存體總管中刪除 blob 容器：

1. 開啟儲存體總管。
2. 在左窗格中，展開儲存體帳戶，其中包含您要檢視的 blob 容器。
3. 展開儲存體帳戶的 [Blob 容器] 。
4. 以滑鼠右鍵按一下您想要刪除的 blob 容器，從內容功能表中，選取 [刪除] 。
   您也可以按 [刪除]  以刪除目前選取的 blob 容器。

   ![刪除 Blob 容器內容功能表][4]
5. 選取確認對話方塊上的 [是]  。

   ![刪除 Blob 容器確認][5]

## <a name="copy-a-blob-container"></a>複製 Blob 容器
儲存體總管可讓您將 blob 容器複製到剪貼簿，然後將該 blob 容器貼到另一個儲存體帳戶。 (若要查看如何複製個別的 blob，請參閱[管理 blob 容器中的 blob 區段](#managing-blobs-in-a-blob-container)。)

下列步驟說明如何將 blob 容器從某個儲存體帳戶複製到另一個儲存體帳戶。

1. 開啟儲存體總管。
2. 在左窗格中，展開儲存體帳戶，其中包含您要複製的 blob 容器。
3. 展開儲存體帳戶的 [Blob 容器] 。
4. 以滑鼠右鍵按一下您想要複製的 blob 容器，從內容功能表中，選取 [複製 Blob 容器編輯器] 。

   ![複製 Blob 容器內容功能表][6]
5. 以滑鼠右鍵按一下想要貼上至 blob 容器的「目標」儲存體帳戶，從內容功能表中，選取 [貼上 Blob 容器] 。

   ![貼上 Blob 容器內容功能表][7]

## <a name="get-the-sas-for-a-blob-container"></a>取得 Blob 容器的 SAS
[共用存取簽章 (SAS)](storage/common/storage-dotnet-shared-access-signature-part-1.md) 可提供您儲存體帳戶中資源的委派存取。
這表示您可以在無需分享您帳戶存取金鑰的情況下，將您儲存體帳戶中的物件有限權限授與用戶端，該用戶端便可在指定的時間期間內及使用指定的權限集來進行存取。

下列步驟說明如何建立 blob 容器的 SAS：

1. 開啟儲存體總管。
2. 在左窗格中，展開儲存體帳戶，其中包含您要取得 SAS 的 blob 容器。
3. 展開儲存體帳戶的 [Blob 容器] 。
4. 以滑鼠右鍵按一下想要的 blob 容器，從內容功能表中，選取 [取得共用存取簽章] 。

   ![取得 SAS 內容功能表][8]
5. 在 [共用存取簽章]  對話方塊中，指定您要用於資源的原則、開始和到期日期、時區和存取層級。

   ![取得 SAS 選項][9]
6. 當您完成指定 SAS 選項時，選取 [建立] 。
7. 第二個 [共用存取簽章]  對話方塊會顯示，列出您可以用來存取儲存體資源的 blob 容器及 URL 和 QueryStrings。
   選取您想要複製到剪貼簿的 URL 旁邊的 [複製]  。

   ![複製 SAS URL][10]
8. 完成時，選取 [關閉] 。

## <a name="manage-access-policies-for-a-blob-container"></a>管理 blob 容器的存取原則
下列步驟說明如何管理 (新增和移除) blob 容器的存取原則︰

1. 開啟儲存體總管。
2. 在左窗格中，展開儲存體帳戶，其中包含您要管理其存取原則的 blob 容器。
3. 展開儲存體帳戶的 [Blob 容器] 。
4. 選取想要的 blob 容器，從內容功能表中，選取 [管理存取原則] 。

   ![管理存取原則內容功能表][11]
5. [存取原則]  對話方塊會列出已針對所選的 blob 容器建立的任何存取原則。

   ![存取原則選項][12]        
6. 根據存取原則管理工作遵循下列步驟︰

   * **新增新的存取原則** -選取 [新增]。 一旦產生，[存取原則]  對話方塊將會顯示新加入的存取原則 (具有預設設定)。
   * **編輯存取原則** -進行任何所需的編輯，然後選取 [儲存]。
   * **移除存取原則** -選取您想要移除的存取原則旁邊的 [移除]。

## <a name="set-the-public-access-level-for-a-blob-container"></a>設定 blob 容器的公用存取層級
根據預設，每個 blob 容器設定為「沒有公用存取」。

下列步驟說明如何指定 blob 容器的公用存取層級。

1. 開啟儲存體總管。
2. 在左窗格中，展開儲存體帳戶，其中包含您要管理其存取原則的 blob 容器。
3. 展開儲存體帳戶的 [Blob 容器] 。
4. 選取想要的 blob 容器，從內容功能表中，選取 [設定公用存取層級] 。

   ![設定公用存取層級內容功能表][13]
5. 在 [設定容器公用存取層級]  對話方塊中，指定所需的存取層級。

   ![設定公用存取層級選項][14]
6. 選取 [套用] 。

## <a name="managing-blobs-in-a-blob-container"></a>管理 blob 容器中的 blob
建立 blob 容器之後，您可以將 blob 上傳至 blob 容器、將 blob 下載到本機電腦或在本機電腦上開啟 blob 等等。

下列步驟說明如何管理 blob 容器中的 blob (及資料夾)。

1. 開啟儲存體總管。
2. 在左窗格中，展開儲存體帳戶，其中包含您要管理的 blob 容器。
3. 展開儲存體帳戶的 [Blob 容器] 。
4. 按兩下您想要檢視的 blob 容器。
5. 主窗格會顯示 blob 容器的內容。

   ![檢視 Blob 容器][3]
6. 主窗格會顯示 blob 容器的內容。
7. 根據您想要執行的工作遵循下列步驟︰

   * **將檔案上傳至 blob 容器**

     1. 在主窗格工具列上，選取 [上傳]，然後從下拉式功能表選取 [上傳檔案]。

        ![上傳檔案功能表][15]
     2. 在 [上傳檔案] 對話方塊中，選擇 [檔案] 文字方塊右側的省略符號 (**…**) 按鈕，以選取您想要上傳的檔案。

        ![上傳檔案選項][16]
     3. 指定 [Blob 類型] 的類型。 如需詳細資訊，請參閱[建立容器並設定權限](storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-the-container)。
     4. 選擇性地指定選取的檔案將會上傳至其中的目標資料夾。 如果目標資料夾不存在，系統就會加以建立。
     5. 選取 [上傳] 。
   * **將資料夾上傳至 blob 容器**

     1. 在主窗格工具列上，選取 [上傳]，然後從下拉式功能表選取 [上傳資料夾]。

        ![上傳資料夾功能表][17]
     2. 在 [上傳資料夾] 對話方塊中，選擇 [資料夾] 文字方塊右側的省略符號 (**…**) 按鈕，以選取您想要上傳其內容的資料夾。

        ![上傳資料夾選項][18]
     3. 指定 [Blob 類型] 的類型。 如需詳細資訊，請參閱[建立容器並設定權限](storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-the-container)。
     4. 選擇性地指定選取的資料夾的內容將會上傳至其中的目標資料夾。 如果目標資料夾不存在，系統就會加以建立。
     5. 選取 [上傳] 。
   * **將 blob 下載到本機電腦**

     1. 選取您想要下載的 blob。
     2. 在主窗格工具列上選取 [下載] 。
     3. 在 [指定儲存下載的 blob 的位置]  對話方塊中，指定要下載 blob 的位置，和您想要給予它的名稱。  
     4. 選取 [ **儲存**]。
   * **在您的本機電腦上開啟 blob**

     1. 選取您想要開啟的 blob。
     2. 在主窗格工具列上選取 [開啟] 。
     3. Blob 會被下載，並使用與 blob 的基礎檔案類型相關聯的應用程式開啟。
   * **將 blob 複製到剪貼簿**

     1. 選取您想要複製的 blob。
     2. 在主窗格工具列上選取 [複製] 。
     3. 在左窗格中，瀏覽至另一個 blob 容器，然後在主窗格中按兩下加以檢視。
     4. 在主窗格工具列上選取 [貼上]  以建立 blob 的複本。
   * **刪除 Blob**

     1. 選取您想要刪除的 blob。
     2. 在主窗格工具列上選取 [刪除] 。
     3. 選取確認對話方塊上的 [是]  。

## <a name="next-steps"></a>後續步驟
* 檢視 [最新的儲存體 Explorer 版本資訊與影片](http://www.storageexplorer.com)。
* 了解如何 [利用 Azure Blob、資料表、佇列和檔案建立應用程式](https://azure.microsoft.com/documentation/services/storage/)。

[0]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-create-context-menu.png
[1]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create.png
[2]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create-done.png
[3]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-editor.png
[4]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-context-menu.png
[5]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-confirmation.png
[6]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-copy-context-menu.png
[7]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-paste-context-menu.png
[8]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-context-menu.png
[9]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-options.png
[10]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-urls.png
[11]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-context-menu.png
[12]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-options.png
[13]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-context-menu.png
[14]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-options.png
[15]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-menu.png
[16]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-options.png
[17]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-menu.png
[18]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-options.png
[19]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-open-editor-context-menu.png
