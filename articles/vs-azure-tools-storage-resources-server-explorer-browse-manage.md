---
title: 使用伺服器總管瀏覽和管理儲存體資源 | Microsoft Docs
description: 使用伺服器總管瀏覽和管理儲存體資源
services: visual-studio-online
author: ghogen
manager: douge
assetId: 658dc064-4a4e-414b-ae5a-a977a34c930d
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 8/24/2017
ms.author: ghogen
ms.openlocfilehash: 74f5508586d073bcccc54894cce6fcde1b83fe18
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141513"
---
# <a name="browse-and-manage-storage-resources-by-using-server-explorer"></a>使用伺服器總管瀏覽和管理儲存體資源

[!INCLUDE [storage-try-azure-tools](../includes/storage-try-azure-tools.md)]

## <a name="overview"></a>概觀

如果您已經安裝 Azure Tools for Microsoft Visual Studio，您可以從 Azure 的儲存體帳戶檢視 blob、佇列和資料表資料。 在 [伺服器總管] 中的 Azure **儲存體**節點會顯示位於您的本機儲存體模擬器帳戶和其他 Azure 儲存體帳戶中的資料。

若要在 Visual Studio 中檢視 [伺服器總管]，請在功能表列上選取 [檢視] > [伺服器總管]。 **儲存體**節點會顯示存在於您連接之每個 Azure 訂用帳戶或憑證下的儲存體帳戶。 如果您的儲存體帳戶未出現，您可以遵循 [本文稍後](#add-storage-accounts-by-using-server-explorer)的指示加以新增。

從 Azure SDK 2.7 開始，您也可以使用雲端總管來檢視和管理您的 Azure 資源。 如需詳細資訊，請參閱[使用雲端總管管理 Azure 資源](vs-azure-tools-resources-managing-with-cloud-explorer.md)。

## <a name="view-and-manage-storage-resources-in-visual-studio"></a>檢視和管理 Visual Studio 中的儲存體資源

[伺服器總管] 會自動在您的儲存體模擬器帳戶中顯示 blob、佇列和資料表的清單。 儲存體模擬器帳戶會在**儲存體**節點下的 [伺服器總管] 中列出，作為 **開發** 節點。

若要查看儲存體模擬器帳戶的資源，請展開 **開發** 節點。 當您展開 **開發** 節點時，如果尚未啟動儲存體模擬器，它會自動啟動。 此流程可能需要數秒鐘的時間。 當儲存體模擬器啟動時，您可以繼續在 Visual Studio 的其他區域中運作。

若要檢視儲存體帳戶中的資源，請在 [伺服器總管] 中展開儲存體帳戶的節點，可在其中看見 **Blob**、**佇列**和**表格**節點。

## <a name="work-with-blob-resources"></a>使用 Blob 資源

**Blob** 節點會顯示所選之儲存體帳戶的容器清單。 Blob 容器包含 blob 檔案，您可以將這些 blob 組織成資料夾和子資料夾。 如需詳細資訊，請參閱 [如何從 .NET 使用 Blob 儲存體](storage/blobs/storage-dotnet-how-to-use-blobs.md)。

### <a name="to-create-a-blob-container"></a>建立 Blob 容器

1. 開啟 **Blobs** 節點的捷徑功能表，然後選取 [建立 Blob 容器]。
1. 在 [建立 Blob 容器] 對話方塊中，輸入新容器的名稱。  
1. 選取鍵盤上的 Enter 鍵，也可以按一下或點選名稱欄位以外的地方，以儲存 Blob 容器。

   > [!NOTE]
   > Blob 容器名稱必須以數字 (0-9) 或小寫字母 (a-z) 開頭。

### <a name="to-delete-a-blob-container"></a>刪除 Blob 容器

開啟您想要移除之 Blob 容器的捷徑功能表，然後選取 [刪除]。

### <a name="to-display-a-list-of-the-items-in-a-blob-container"></a>顯示 blob 容器中的項目清單

開啟清單中 Blob 容器名稱的捷徑功能表，然後選取 [開啟]。

當您檢視 blob 容器的內容時，它就會出現在稱為 blob 容器檢視的索引標籤中。

![Blob 容器檢視](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC749016.png)

您可以使用 blob 容器檢視右上角的按鈕在 blob 上執行下列作業：

* 輸入篩選值並加以套用。
* 重新整理容器中的 blob 清單。
* 上傳檔案。
* 刪除 Blob。 (從 blob 容器中刪除檔案並不會刪除基礎的檔案。 只會將它從 blob 容器移除。)
* 開啟 blob。
* 將 blob 儲存到本機電腦。

### <a name="to-create-a-folder-or-subfolder-in-a-blob-container"></a>在 blob 容器中建立資料夾或子資料夾

1. 在 [Cloud Explorer] 中選擇 Blob 容器。 在 [容器] 視窗中，選取 [上傳 Blob] 按鈕。

1. 在 [上傳新的檔案] 對話方塊中，選取 [瀏覽] 按鈕來指定您想要上傳的檔案，然後在 [資料夾 (選擇性)] 方塊中輸入資料夾名稱。

   ![將檔案上傳至 blob 資料夾](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC766037.png)

   您可以遵循相同的步驟將子資料夾加入至容器資料夾。 如果您未指定資料夾名稱，檔案會上傳至 Blob 容器的最上層。 檔案會出現在容器中的特定資料夾。

   ![加入至 blob 容器的資料夾](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC766038.png)

1. 按兩下資料夾或選取 Enter 鍵以查看資料夾的內容。 當您位於容器的資料夾中，您可以藉由選取 [開啟上層目錄] \(箭頭) 按鈕來返回容器的根目錄。

### <a name="to-delete-a-container-folder"></a>刪除容器資料夾

刪除資料夾中的所有檔案。

因為 blob 容器中的資料夾是虛擬資料夾，所以您無法建立空的資料夾。 您也無法以刪除資料夾的方式來刪除其檔案內容，而必須改為刪除資料夾的整個內容來刪除資料夾本身。

### <a name="to-filter-blobs-in-a-container"></a>在容器中篩選 Blob

您可以藉由指定一般的前置詞來篩選顯示的 blob。

例如，如果您在篩選文字方塊中輸入前置詞 **hello**，然後選取 [執行] \(**!**) 按鈕，則只會出現以 "hello" 開頭的 blob。

![篩選文字方塊](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC519076.png)

篩選文字方塊會區分大小寫，並且不支援使用萬用字元篩選。 Blob 只可以利用前置詞篩選。 如果您要使用分隔符號在虛擬階層中組織 blob，前置詞可以包含分隔符號。 例如，篩選前置詞 "HelloFabric/" 會傳回所有以該字串開頭的 blob。

### <a name="to-download-blob-data"></a>下載 blob 資料

在 Cloud Explorer 中，使用下列任何方法：

* 開啟一或多個 blob 的捷徑功能表，然後選取 [開啟]。
* 選擇 blob 名稱，然後選取 [開啟] 按鈕。
* 按兩下 blob 名稱。

Blob 下載進度會顯示在 [Azure 活動記錄檔]  視窗中。

Blob 會在該檔案類型的預設編輯器中開啟。 如果作業系統辨識出檔案類型，該檔案就會以本機安裝的應用程式開啟。 否則，系統會提示您選擇適用於該 blob 檔案類型的應用程式。 下載 blob 時所建立的本機檔案會標示為唯讀。

Blob 資料會在本機快取，並在 Azure Blob 儲存體中針對 blob 的上次修改時間進行檢查。 如果 blob 在上次下載之後已更新過，系統會再次下載該 blob。 否則，系統會從本機磁碟載入該 blob。

根據預設，Blob 會下載至暫存目錄。 若要下載 blob 到特定的目錄中，請開啟所選 blob 名稱的捷徑功能表並選取 [另存新檔]。 當您以這種方式儲存 blob 時，blob 檔案尚未開啟，本機檔案會利用讀/寫屬性建立。

### <a name="to-upload-blobs"></a>上傳 blob

若要上傳 blob，當容器開啟以在 blob 容器檢視中加以檢視時，選取 [上傳 Blob] 按鈕。

您可以選擇一或多個檔案上傳，而且您可以上傳任何類型的檔案。 [Azure 活動記錄] 視窗會顯示上傳的進度。 如需如何使用 blob 資料的詳細資訊，請參閱[如何在 .NET 中使用 Azure Blob 儲存體](http://go.microsoft.com/fwlink/p/?LinkId=267911)。

### <a name="to-view-logs-transferred-to-blobs"></a>檢視傳送輸到 blob 的記錄檔

如果您使用 Azure 診斷記錄來自 Azure 應用程式的資料，且您已將記錄傳輸至您的儲存體帳戶，您會看到 Azure 為這些記錄所建立的容器。 在 [伺服器總管] 中檢視這些記錄檔是利用應用程式識別問題的簡單方法，尤其是在應用程式部署至 Azure 時。

如需 Azure 診斷的詳細資訊，請參閱 [使用 Azure 診斷收集記錄資料](https://msdn.microsoft.com/library/azure/gg433048.aspx)。

### <a name="to-get-the-url-for-a-blob"></a>取得 blob 的 URL

開啟 blob 的捷徑功能表，然後選取 [複製 URL]。

### <a name="to-edit-a-blob"></a>編輯 blob

選取 blob，然後選取 [開啟 Blob] 按鈕。

檔案會下載到暫存位置並在本機電腦上開啟。 在變更之後再次上傳 blob。

## <a name="work-with-queue-resources"></a>使用佇列資源

儲存體服務佇列裝載在 Azure 儲存體帳戶中。 您可以使用它們來讓您的雲端服務角色利用訊息傳遞機制彼此通訊並與其他服務通訊。 您可以透過雲端服務和外部用戶端的 web 服務以程式設計方式存取佇列。 您也可以在 Visual Studio 中使用 [伺服器總管] 直接存取佇列。

當您開發使用佇列的雲端服務時，您可能會想要使用 Visual Studio 來建立佇列，並且在您開發和測試您的程式碼時以互動方式使用它們。

在 [伺服器總管] 中，您可以檢視儲存體帳戶中的佇列、建立和刪除佇列、開啟佇列以檢視其訊息，以及將訊息加入至佇列。 當您開啟佇列進行檢視時，您可以檢視個別訊息，而且您可以使用左上角的按鈕在佇列上執行下列動作：

* 重新整理佇列的檢視。
* 將訊息新增至佇列。
* 清除最上層佇列訊息。
* 清除整個佇列。

下圖顯示包含兩個訊息的佇列：

![檢視佇列](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC651470.png)

如需儲存體服務佇列的詳細資訊，請參閱[以 .NET 開始使用 Azure 佇列儲存體](http://go.microsoft.com/fwlink/?LinkID=264702)。 如需儲存體服務佇列之 Web 服務的詳細資訊，請參閱 [佇列服務概念](http://go.microsoft.com/fwlink/?LinkId=264788)。 如需有關如何使用 Visual Studio 將訊息傳送至儲存體服務佇列的資訊，請參閱 [傳送訊息至儲存體服務佇列](https://docs.microsoft.com/azure/visual-studio/vs-storage-cloud-services-getting-started-queues)。

> [!NOTE]
> 儲存體服務佇列與 Azure 服務匯流排佇列不同。 如需服務匯流排佇列的詳細資訊，請參閱[服務匯流排佇列、主題和訂用帳戶](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-queues-topics-subscriptions)。

## <a name="work-with-table-resources"></a>使用資料表資源

Azure 資料表儲存體可儲存大量的結構化資料。 此服務是一個 NoSQL 資料存放區，接受來自 Azure 雲端內外經過驗證的呼叫。 Azure 資料表很適合儲存結構化、非關聯式資料。

### <a name="to-create-a-table"></a>建立資料表

1. 在 [Cloud Explorer] 中，選取儲存體帳戶的**資料表**節點，然後選取 [建立資料表]。
1. 在 [建立資料表]  對話方塊中，輸入資料表的名稱。

### <a name="to-view-table-data"></a>檢視資料表資料

1. 在 [Cloud Explorer] 中，開啟 **Azure** 節點，然後開啟**儲存體**節點。
1. 開啟您有興趣的儲存體帳戶節點，然後開啟 **資料表** 節點以查看儲存體帳戶的資料表清單。
1. 開啟資料表的捷徑功能表，然後選取 [檢視資料表]。

    ![方案總管中的 Azure 資料表](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC744165.png)

資料表會根據實體 (顯示於資料列) 和屬性 (顯示於資料行) 加以組織。 例如，下圖顯示資料表設計工具中所列的實體。

### <a name="to-edit-table-data"></a>編輯資料表資料

在資料表設計工具中，開啟實體 (單一資料列) 或屬性 (單一儲存格) 的捷徑功能表，然後選取 [編輯]。

    ![Add or edit a table entity](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC656238.png)

單一資料表中的實體不一定要有同一組屬性 (資料行)。 請記住下列檢視和編輯資料表資料的限制：

* 您無法檢視或編輯二進位資料 (`type byte[]`)，但您可以將其儲存在資料表中。
* 您無法編輯 **PartitionKey** 或 **RowKey** 值，因為 Azure 中的資料表儲存體不支援這項操作。
* 您無法建立名為 **Timestamp** 的屬性。 Azure 儲存體服務會使用該名稱的屬性。
* 如果您輸入**日期時間**值，您必須遵循適合您的電腦之區域和語言設定的格式 (例如，MM/DD/YYYY HH:MM:SS [AM|PM] 適用於美國英文)。

### <a name="to-add-entities"></a>加入實體

1. 在資料表設計工具中，選取 [新增實體] 按鈕。

    ![[新增實體] 按鈕](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC655336.png)

1. 在 [新增實體] 對話方塊中，輸入 **PartitionKey** 和 **RowKey** 屬性的值。

    ![[新增實體] 對話方塊](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC655335.png)

    請小心輸入這些值。 在您關閉對話方塊之後，您就無法變更這些值了，除非您刪除此實體並且再次將其加入。

### <a name="to-filter-entities"></a>篩選實體

如果您使用查詢產生器，您就可以自訂會出現在資料表中的實體集。

1. 若要開啟查詢產生器，請開啟資料表進行檢視。
1. 選取資料表檢視工具列上的 [查詢產生器] 按鈕。

    [查詢產生器]  對話方塊會隨即出現。 下圖顯示建置於查詢產生器中的查詢。

    ![查詢產生器](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC652231.png)
1. 當您完成查詢建置時，請關閉對話方塊。 產生的查詢文字格式會出現在文字方塊中做為 WCF Data Services 篩選條件。
1. 若要執行查詢，請選取綠色的三角形圖示。

如果您直接在篩選文字方塊中輸入 WCF Data Services 篩選字串，您也可以篩選出現在資料表設計工具中的實體資料。 這種類型的字串與 SQL WHERE 子句相似，但是會傳送至伺服器做為 HTTP 要求。 如需如何建構篩選字串的資訊，請參閱[建構資料表設計工具的篩選字串](https://docs.microsoft.com/azure/vs-azure-tools-table-designer-construct-filter-strings)。

下圖顯示有效篩選字串的範例：

![篩選字串](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC655337.png)

## <a name="refresh-storage-data"></a>重新整理儲存體資料

當伺服器總管連線到儲存體帳戶或從中取得資料時，此作業最多可能需要一分鐘才能完成。 如果伺服器總管無法連線，此作業可能會逾時。擷取資料時，您可以繼續在 Visual Studio 的其他部分中運作。 如果因為作業時間太長而要將其取消，請選取 [伺服器總管] 工具列上的 [停止重新整理]  按鈕。

### <a name="to-refresh-blob-container-data"></a>重新整理 blob 容器資料

* 選取**儲存體**下的 **Blobs** 節點，然後選取 [伺服器總管] 工具列上的 [重新整理] 按鈕。
* 若要重新整理顯示的 blob 清單，請選取 [執行] 按鈕。

### <a name="to-refresh-table-data"></a>重新整理資料表資料

* 選取**儲存體**下的**資料表**節點，然後選取 [伺服器總管] 工具列上的 [重新整理] 按鈕。
* 若要重新整理資料表設計工具中所顯示的實體清單，請選取資料表設計工具中的 [執行] 按鈕。

### <a name="to-refresh-queue-data"></a>重新整理佇列資料

選取**儲存體**下的**佇列**節點，然後選取 [伺服器總管] 工具列上的 [重新整理] 按鈕。

### <a name="to-refresh-all-items-in-a-storage-account"></a>重新整理儲存體帳戶中的所有項目

選擇帳戶名稱，然後選取 [伺服器總管] 工具列上的 [重新整理] 按鈕。

## <a name="add-storage-accounts-by-using-server-explorer"></a>使用 [伺服器總管] 新增儲存體帳戶

有兩種方式可以使用 [伺服器總管] 新增儲存體帳戶。 您可以在您的 Azure 訂用帳戶中建立儲存體帳戶，也可以連結現有的儲存體帳戶。

### <a name="to-create-a-storage-account-by-using-server-explorer"></a>使用伺服器總管建立儲存體帳戶

1. 在 [伺服器總管] 中，開啟**儲存體**節點的捷徑功能表，然後選取 [建立儲存體帳戶]。

1. 在 [建立儲存體帳戶] 對話方塊中，選取或輸入下列資訊：

   * 您要加入儲存體帳戶的 Azure 訂用帳戶。
   * 您想要用於新儲存體帳戶的名稱。
   * 區域或同質群組 (例如美國西部或東亞)。
   * 您要用於儲存體帳戶的複寫類型，例如本地備援。

   ![建立 Azure 儲存體帳戶](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC744166.png)

1. 選取 [建立] 。

新的儲存體帳戶會出現在 [方案總管] 中的 [儲存體]  清單。

### <a name="to-attach-an-existing-storage-account-by-using-server-explorer"></a>使用伺服器總管附加現有的儲存體帳戶

1. 在 [伺服器總管] 中，開啟 Azure **儲存體**節點的捷徑功能表，然後選取 [連結外部儲存體]。

    ![新增現有的儲存體帳戶](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC766039.png)
1. 在 [建立儲存體帳戶] 對話方塊中，選取或輸入下列資訊：

   * 您想要連結的現有儲存體帳戶名稱。
   * 選取之儲存體帳戶的金鑰。 當您選取儲存體帳戶時，通常會提供這個值給您。 如果您想要 Visual Studio 記住儲存體帳戶金鑰，請選取 [記住帳戶金鑰] 核取方塊。
   * 要用於連接至儲存體帳戶的通訊協定，例如 HTTP、HTTPS 或自訂端點。 如需有關自訂端點的詳細資訊，請參閱[如何設定連接字串](https://msdn.microsoft.com/library/azure/ee758697.aspx) 。

### <a name="to-view-the-secondary-endpoints"></a>檢視次要端點

如果您建立的儲存體帳戶使用**讀取權限異地備援**複寫選項，您可以開啟帳戶名稱的快顯功能表，然後選取 [屬性]，以便檢視其次要端點。

![儲存體次要端點](./media/vs-azure-tools-storage-resources-server-explorer-browse-manage/IC766040.png)

### <a name="to-remove-a-storage-account-from-server-explorer"></a>從伺服器總管移除儲存體帳戶

在 [伺服器總管] 中，開啟帳戶名稱的捷徑功能表，然後選取 [刪除]。 

如果您刪除儲存體帳戶，也會移除該帳戶的所有已儲存金鑰資訊。

如果您從 [伺服器總管] 刪除儲存體帳戶，它並不會影響您的儲存體帳戶或其包含的任何資料。 它只會從 [伺服器總管] 移除參考。 若要永久刪除儲存體帳戶，請使用 [Azure 入口網站](https://portal.azure.com/)。

## <a name="next-steps"></a>後續步驟

若要深入了解如何使用 Azure 儲存體服務，請參閱[存取 Azure 儲存體服務](https://msdn.microsoft.com/library/azure/ee405490.aspx)。
