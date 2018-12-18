---
title: 在 Azure 儲存體總管中管理 Azure Data Lake Storage Gen1 資源
description: 了解如何在 Azure 儲存體總管中存取和管理 Azure Data Lake Storage Gen1 資料與資源
Keywords: Azure Data Lake Store, Azure Storage Explorer
services: Data Lake Store
documentationcenter: ''
author: jejiang
manager: DJ
editor: Jenny Jiang
ms.assetid: ''
ms.service: data-lake-store
ms.custom: Azure Data Lake Store
ms.devlang: dotnet
ms.topic: article
ms.date: 02/05/2018
ms.author: jejiang
ms.openlocfilehash: 537c94976dbb4f7753b7fafb3b6c94e97cc2cc17
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46126321"
---
# <a name="manage-azure-data-lake-storage-gen1-resources-by-using-storage-explorer"></a>使用儲存體總管管理 Azure Data Lake Storage Gen1 資源

[Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview) 是用來儲存大量非結構化資料 (例如文字或二進位資料) 的服務。 您可以透過 HTTP 或 HTTPS 從任何位置存取資料。 Azure 儲存體總管中的 Data Lake Storage Gen1 可讓您存取和管理 Data Lake Storage Gen1 資料和資源，以及如 Blob 和佇列等其他 Azure 實體。 現在您可以使用同一個工具在同一處管理您不同的 Azure 實體。

另一個優點是，您不需擁有訂用帳戶權限即可管理 Data Lake Storage Gen1 資料。 在儲存體總管中，只要他人授與權限，您即可將 Data Lake Storage Gen1 路徑連結至 [本機與已連結的資源] 節點。

## <a name="prerequisites"></a>必要條件
若要完成本文中的步驟，您必須符合下列先決條件︰

*   Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial)。
*   Data Lake Storage Gen1 帳戶。 如需如何建立帳戶的指示，請參閱[開始使用 Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-get-started-portal)。

## <a name="install-storage-explorer"></a>安裝儲存體總管

從[產品網頁](https://azure.microsoft.com/features/storage-explorer/)安裝最新的 Azure 儲存體總管。 安裝支援 Windows、Linux 及 Mac 版本。

## <a name="connect-to-an-azure-subscription"></a>連線到 Azure 訂用帳戶

1. 在儲存體總管中，選取左邊的外掛程式圖示。
       
   ![外掛程式圖示](./media/data-lake-store-in-storage-explorer/plug-in-icon.png)
 
2. 選取 [新增 Azure 帳戶]，然後選取 [登入]。

   ![[連線至 Azure 儲存體] 對話方塊](./media/data-lake-store-in-storage-explorer/connect-to-azure-subscription.png)

2. 在 [登入您的帳戶] 對話方塊中，輸入您的 Azure 認證。

    ![Azure 登入對話方塊](./media/data-lake-store-in-storage-explorer/sign-in.png)

3. 從清單中選取您的訂用帳戶，然後選取 [套用]。

    ![訂用帳戶資訊和 [套用] 按鈕](./media/data-lake-store-in-storage-explorer/apply-subscription.png)

    **[總管]** 窗格會更新，並顯示所選取訂用帳戶中的帳戶。

    ![帳戶清單](./media/data-lake-store-in-storage-explorer/account-list.png)

您已成功將 Data Lake Storage Gen1 連線至您的 Azure 訂用帳戶。

## <a name="connect-to-data-lake-storage-gen1"></a>連線至 Data Lake Storage Gen1
如果有人為您提供資源的 URI，您即可存取您的訂用帳戶中不存在的資源。 您可以在登入後，使用此 URI 連線至 Data Lake Storage Gen1。
1. 開啟儲存體總管。
2. 在左窗格中，展開 [本機與已連結的資源]。
3. 以滑鼠右鍵按一下 [Data Lake Store]，然後選取 [連線至 Data Lake Store]。

      ![捷徑功能表上的 [連線至 Data Lake Store]](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-uri-attach.png)

4. 輸入 URI。 工具會瀏覽至您剛才輸入的 URL 位置。

      ![[連線至 Data Lake Store] 對話方塊，內含可供輸入 URI 的文字方塊](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-uri-attach-dialog.png)

      ![連線至 Data Lake Storage Gen1 的結果](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-attach-finish.png)

## <a name="view-a-data-lake-storage-gen1-accounts-contents"></a>檢視 Data Lake Storage Gen1 帳戶的內容
Data Lake Storage Gen1 帳戶的資源包含資料夾和檔案。

下列步驟說明如何在儲存體總管中檢視 Data Lake Storage Gen1 帳戶的內容：

1. 開啟儲存體總管。
2. 在左窗格中，展開包含您要檢視的 Data Lake Storage Gen1 帳戶之訂用帳戶。
3. 展開 **Data Lake Store**。
4. 以滑鼠右鍵按一下您要檢視的 Data Lake Storage Gen1 帳戶節點，然後選取 [開啟]。 您也可以按兩下 Data Lake Storage Gen1 帳戶來開啟。 
   
   主窗格會顯示 Data Lake Storage Gen1 帳戶的內容。

   ![內含資料夾清單的主窗格](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-toolbar-mainpane.png) 

## <a name="manage-resources-in-data-lake-storage-gen1"></a>管理 Data Lake Storage Gen1 中的資源

您可以透過下列操作來管理 Data Lake Storage Gen1 資源：
*   瀏覽橫跨多個 Data Lake Storage Gen1 帳戶的 Data Lake Storage Gen1 資源。  
*   使用連接字串直接連線和管理 Data Lake Storage Gen1。 
*   透過 [本機與已連結的資源] 下的 ACL 檢視與他人共用的 Data Lake Storage Gen1 資源。
*   執行檔案和資料夾 CRUD 作業：支援遞迴資料夾和複選的檔案。 
*   拖曳、卸除 及新增資料夾，以快速存取最近使用的位置。 這項作業會反映桌面檔案總管體驗。 
*   在儲存體總管中按一下，即可複製和開啟 Data Lake Storage Gen1 超連結。 
*   在右下方的窗格中顯示活動記錄，以檢視活動狀態。
*   顯示資料夾統計資料和檔案屬性。

## <a name="manage-resources-in-azure-storage-explorer"></a>在 Azure 儲存體總管中管理資源
建立 Data Lake Storage Gen1 帳戶後，您可以：

* 上傳資料夾和檔案、下載資料夾和檔案，以及開啟本機電腦上的資源。
* 釘選至 [快速存取]、建立新資料夾、複製 URL 以及全選。
* 複製並貼上、重新命名、刪除、取得資料夾統計資料，以及重新整理。

下列項目說明如何管理 Data Lake Storage Gen1 帳戶內的資源。 遵循您想要執行之工作的步驟。

### <a name="upload-files"></a>上傳檔案

1. 在主窗格的工具列上，選取 [上傳]，然後在下拉式功能表上選取 [上傳檔案]。

   ![[上傳檔案] 功能表項目](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-files-menu.png) 

2. 在 [選取要上傳的檔案] 對話方塊中，選取您要上傳的檔案。

   ![上傳檔案的對話方塊](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-files-dialog.png)

3. 選取 [開啟] 以開始上傳。

### <a name="upload-a-folder"></a>上傳資料夾

1. 在主窗格的工具列上，選取 [上傳]，然後在下拉式功能表上選取 [上傳資料夾]。

   ![[上傳資料夾] 功能表項目](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-folder-menu.png) 
     
2. 在 [選取要上傳的資料夾] 對話方塊中，選取您要上傳的資料夾。 然後按一下 [選取資料夾]。

   ![上傳資料夾的對話方塊](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-folder-dialog.png)      

   上傳隨即開始。

   ![上傳正在進行中的對話方塊](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-folder-drag.png) 

> [!NOTE] 
> 您也可以直接拖曳本機電腦上的檔案和資料夾，以開始上傳。 
       
### <a name="download-folders-or-files-to-your-local-computer"></a>將資料夾或檔案下載到本機電腦

1. 選取您要下載的資料夾或檔案。
2. 在主窗格工具列上選取 [下載] 。
3. 在 [選取要在其中儲存所下載檔案的資料夾] 對話方塊中，指定位置和名稱。
4. 選取 [ **儲存**]。

### <a name="open-a-folder-or-file-from-your-local-computer"></a>從本機電腦開啟資料夾或檔案

1. 選取您要開啟的資料夾或檔案。
2. 在主窗格工具列上選取 [開啟] 。 或以滑鼠右鍵按一下選取的資料夾或檔案，然後選取捷徑功能表上的 [開啟]。

檔案會進行下載，並透過與基礎檔案類型相關聯的應用程式開啟。 或者，資料夾會在主窗格中開啟。

![開啟的日期](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-open.png) 

### <a name="copy-folders-or-files-to-the-clipboard"></a>將資料夾或檔案複製到剪貼簿

1. 選取您要複製的資料夾或檔案。
2. 在主窗格工具列上選取 [複製] 。 或以滑鼠右鍵按一下選取的資料夾或檔案，然後選取捷徑功能表上的 [複製]。
3. 在左窗格中，瀏覽至另一個 Data Lake Storage Gen1 帳戶，然後按兩下該帳戶以在主窗格中檢視。
4. 在主窗格工具列上選取 [貼上]，以建立複本。 或者，在目的地的捷徑功能表上選取 [貼上]。

![複製資料夾的選取項目](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-copy-paste.png)

> [!NOTE] 
> 不支援跨儲存體類型的複製 / 貼上作業。 您可以複製 Data Lake Storage Gen1 資料夾或檔案，並將它們貼到另一個 Data Lake Storage Gen1 帳戶中。 但是您*無法*複製 Data Lake Storage Gen1 資料夾或檔案，並將它們貼到 Azure Blob 儲存體中，反之亦然。
> 
> 複製 / 貼上作業的運作方式，是先將資料夾或檔案下載到本機電腦，然後將它們上傳至目的地。 此工具「不會」在後端執行此動作。 大型檔案的複製 / 貼上作業速度很慢。 高效能檔案的複製 / 移動正在進行最佳化。

### <a name="delete-folders-or-files"></a>刪除資料夾或檔案

1. 選取您要刪除的資料夾或檔案。
2. 在主窗格工具列上選取 [刪除] 。 或以滑鼠右鍵按一下選取的資料夾或檔案，然後選取捷徑功能表上的 [刪除]。
3. 選取確認對話方塊中的 [是]。

![刪除資料夾的選取項目](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-delete.png)

### <a name="pin-to-quick-access"></a>釘選到快速存取

1. 選取您要釘選的資料夾。
2. 在主窗格的工具列上，選取 [釘選到快速存取]。

   在左窗格中，選取的資料夾已新增至 [快速存取] 節點。

   ![將資料夾釘選至 [快速存取] 的選取項目](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-quick-access.png)

將資料夾釘選至 [快速存取] 節點之後，您即可輕鬆存取資源。

### <a name="use-deep-links"></a>使用深層連結
如果您有 URL，您可以將 URL 輸入至 [檔案總管] 或瀏覽器中的位址路徑。 然後 Storage Explorer.exe 會自動執行，以移至 URL 的位置。

![檔案總管中的深層連結](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-deep-link.png)


## <a name="next-steps"></a>後續步驟
* 檢視 [最新的儲存體 Explorer 版本資訊與影片](http://www.storageexplorer.com)。
* 了解如何[在 Azure 儲存體總管中管理 Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/storage-explorer)。
* [開始使用儲存體總管](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer)
* [開始使用 Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview)。
* 觀看[有關如何在 Azure 儲存體總管中使用 Azure Cosmos DB 的 YouTube 影片](https://www.youtube.com/watch?v=iNIbg1DLgWo&feature=youtu.be)。
