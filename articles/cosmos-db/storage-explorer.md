---
title: 在 Azure 儲存體總管中管理 Azure Cosmos DB
description: 學習如何在 Azure 儲存體總管中管理 Azure Cosmos DB。
Keywords: Azure Cosmos DB, Azure Storage Explorer, MongoDB
services: cosmos-db
author: Jejiang
manager: kfile
editor: ''
tags: Azure Cosmos DB
ms.service: cosmos-db
ms.custom: Azure Cosmos DB active
ms.devlang: na
ms.topic: conceptual
ms.date: 03/20/2018
ms.author: jejiang
ms.openlocfilehash: b45328425cff978377d5e05de487d42e786c063b
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37345102"
---
# <a name="manage-azure-cosmos-db-in-azure-storage-explorer"></a>在 Azure 儲存體總管中管理 Azure Cosmos DB

在 Azure 儲存體總管中使用 Azure Cosmos DB 可讓使用者管理 Azure Cosmos DB 實體、操縱資料、更新預存程序及觸發程序，以及其他 Azure 實體 (例如儲存體 Blob 及佇列)。 現在您可以使用同一個工具在同一處管理您不同的 Azure 實體。 目前，Azure 儲存體總管支援 SQL、MongoDB、圖表和資料表帳戶。


## <a name="prerequisites"></a>先決條件

SQL API <!--or MongoDB API--> 的 Azure Cosmos DB 帳戶。 若您還沒有帳戶，您可以根據[Azure Cosmos DB：使用 .NET 及 Azure 入口網站建置 SQL API Web 應用程式](create-sql-api-dotnet.md)中的說明，在 Azure 入口網站中建立帳戶。

## <a name="installation"></a>安裝

在這裡安裝最新的 Azure 儲存體總管位元：[Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)，我們現在支援 Windows、Linux 及 MAC 版本。

## <a name="connect-to-an-azure-subscription"></a>連線到 Azure 訂用帳戶

1. 在安裝 **Azure 儲存體總管**之後，按一下左邊的**外掛程式**圖示，如下圖所示：
       
   ![外掛程式圖示](./media/storage-explorer/plug-in-icon.png)
 
2. 選取 [新增 Azure 帳戶]，然後按一下 [登入]。

   ![連線到 Azure 訂用帳戶](./media/storage-explorer/connect-to-azure-subscription.png)

2. 在 [Azure 登入] 對話方塊中，選取 [登入]，然後輸入您的 Azure 認證。

    ![登入](./media/storage-explorer/sign-in.png)

3. 從清單中選取您的訂用帳戶，然後按一下 [套用]。

    ![套用](./media/storage-explorer/apply-subscription.png)

    [總管] 窗格會更新，並顯示選取之訂閱中的帳戶。

    ![帳戶清單](./media/storage-explorer/account-list.png)

    您已成功將您的 **Cosmos DB 帳戶**連線到您的 Azure 訂用帳戶。

## <a name="connect-to-azure-cosmos-db-by-using-a-connection-string"></a>使用連接字串連線到 Azure Cosmos DB

另外一種連線到 Azure Cosmos DB 的方式為使用連接字串。 使用下列步驟來使用連接字串進行連線。

1. 在左邊的樹狀目錄中尋找 [Local and Attached] \(本機與已連結)，以滑鼠右鍵按一下 [Cosmos DB 帳戶]，選擇 [連線到 Cosmos DB...]

    ![透過連接字串連線到 Cosmos DB](./media/storage-explorer/connect-to-db-by-connection-string.png)

2. 目前僅支援 SQL 和資料表 API。 選擇 API、貼上**連接字串**、輸入**帳戶標籤**、按 [下一步] 以檢查摘要，然後按一下 [連線] 以連線到 Azure Cosmos DB 帳戶。 如需擷取連接字串的資訊，請參閱[取得連接字串](https://docs.microsoft.com/azure/cosmos-db/manage-account#get-the--connection-string)。

    ![連接字串](./media/storage-explorer/connection-string.png)

## <a name="connect-to-azure-cosmos-db-by-using-local-emulator"></a>使用本機模擬器連線到 Azure Cosmos DB

若要使用模擬器連線到 Azure Cosmos DB，請使用下列步驟 (目前僅支援 SQL 帳戶)。

1. 安裝模擬器並啟動。 如需了解如何安裝模擬器，請參閱 [Cosmos DB 模擬器](https://docs.microsoft.com/azure/cosmos-db/local-emulator)

2. 在左邊的樹狀目錄中尋找 [Local and Attached] \(本機與已連結)，以滑鼠右鍵按一下 [Cosmos DB 帳戶]，選擇 [連線到 Cosmos DB 模擬器...]

    ![使用模擬器連線到 Cosmos DB](./media/storage-explorer/emulator-entry.png)

3. 目前僅支援 SQL API。 貼上**連接字串**、輸入**帳戶標籤**、按 [下一步] 以檢查摘要，然後按一下 [連線] 以連線到 Azure Cosmos DB 帳戶。 如需擷取連接字串的資訊，請參閱[取得連接字串](https://docs.microsoft.com/azure/cosmos-db/manage-account#get-the--connection-string)。

    ![使用模擬器對話方塊連線到 Cosmos DB](./media/storage-explorer/emulator-dialog.png)


## <a name="azure-cosmos-db-resource-management"></a>Azure Cosmos DB 資源管理

您可以透過執行下列作業來管理 Azure Cosmos DB 帳戶：
* 在 Azure 入口網站中開啟帳戶
* 將資源新增至快速存取清單
* 搜尋和重新整理資源
* 建立和刪除資料庫
* 建立和刪除集合
* 建立、編輯、刪除和篩選文件
* 管理預存程序、觸發程序和使用者定義函式

### <a name="quick-access-tasks"></a>快速存取工作

以滑鼠右鍵按一下 [總管] 窗格中的訂用帳戶，您可以執行許多快速動作工作：

* 以滑鼠右鍵按一下 Azure Cosmos DB 帳戶或資料庫，您可以選擇 [在入口網站中開啟] 並使用瀏覽器在 Azure 入口網站上管理資源。

     ![在入口網站中開啟](./media/storage-explorer/open-in-portal.png)

* 您也可以將 Azure Cosmos DB 帳戶、資料庫及集合新增至 [快速存取]。
* [Search from Here] \(從這裡搜尋) 可在選取的路徑下使用關鍵字搜尋。

    ![從這裡搜尋](./media/storage-explorer/search-from-here.png) 

### <a name="database-and-collection-management"></a>資料庫與集合管理
#### <a name="create-a-database"></a>建立資料庫 
-   以滑鼠右鍵按一下 Azure Cosmos DB 帳戶，選擇 [建立資料庫]，輸入資料庫名稱，然後按  **ENTER 鍵**以完成。
       
    ![建立資料庫](./media/storage-explorer/create-database.png) 

#### <a name="delete-a-database"></a>刪除資料庫
- 以滑鼠右鍵按一下資料庫，按一下 [刪除資料庫]，然後在快顯視窗中按一下 [是]。 資料庫節點隨即會刪除，而 Azure Cosmos DB 帳戶會自動重新整理。

    ![刪除 database1](./media/storage-explorer/delete-database1.png)  

    ![刪除 database2](./media/storage-explorer/delete-database2.png) 

#### <a name="create-a-collection"></a>建立集合
1. 以滑鼠右鍵按一下您的資料庫，選擇 [建立集合]，然後提供下列資訊 (例如**集合識別碼****儲存體容量**等)。按一下 [確定] 以完成。 

    ![建立 collection1](./media/storage-explorer/create-collection.png)

    ![建立 collection2](./media/storage-explorer/create-collection2.png) 

2. 選取 [無限制] 即可指定分割區索引鍵，然後按一下 [確定] 以完成。

    若在建立集合時使用了分割區索引鍵，則一旦建立過程完成後，該分割區索引鍵的值便不能在集合上進行變更。 如需分割區索引鍵設定的資訊，請參閱[設計資料分割](partition-data.md#designing-for-partitioning)。

    ![資料分割索引鍵](./media/storage-explorer/partitionkey.png)

#### <a name="delete-a-collection"></a>刪除集合
- 以滑鼠右鍵按一下集合，按一下 [刪除集合]，然後在快顯視窗中按一下 [是]。 

    集合節點隨即刪除，且資料庫會自動重新整理。

    ![刪除集合](./media/storage-explorer/delete-collection.png) 

### <a name="document-management"></a>文件管理

#### <a name="create-and-modify-documents"></a>建立及修改文件
- 若要建立新文件，請在左邊視窗中開啟 [文件]，按一下 [新增文件]，在右邊窗格中編輯內容，然後按一下 [儲存]。 您可以也更新現有的文件，然後按一下 [儲存]。 按一下 [捨棄] 以捨棄變更。

    ![文件](./media/storage-explorer/document.png)

#### <a name="delete-a-document"></a>刪除文件
- 按一下 [刪除] 按鈕來刪除選取的文件。

#### <a name="query-for-documents"></a>查詢文件
- 輸入 [SQL 查詢](sql-api-sql-query.md)，然後按一下 [套用] 來編輯文件篩選。

    ![文件篩選器](./media/storage-explorer/document-filter.png)



### <a name="graph-management"></a>圖表管理

#### <a name="create-and-modify-vertex"></a>建立及修改頂點
1. 若要建立新的頂點，請從左側視窗中開啟 [圖表]，按一下 [新增頂點] 並編輯內容，然後按一下 [確定]。    
2. 若要修改現有的頂點，按一下右側窗格中的畫筆圖示。   

    ![圖形](./media/storage-explorer/vertex.png)

#### <a name="delete-a-graph"></a>建立圖表
- 若要刪除頂點，按一下頂點名稱旁的 [資源回收筒] 圖示。

#### <a name="filter-for-graph"></a>篩選圖表
- 輸入 [Gremlin 查詢](gremlin-support.md) 來編輯圖表篩選器，然後按一下 [套用篩選條件]。

    ![圖表篩選器](./media/storage-explorer/graph-filter.png)

### <a name="table-management"></a>資料表管理

#### <a name="create-and-modify-table"></a>建立及修改資料表
1. 若要建立新的資料表，請從左側視窗開啟 [實體]，按一下 [新增]、編輯 [新增實體] 對話方塊中的內容、按一下 [新增屬性] 按鈕來新增屬性，然後按一下 [插入]。
2. 若要修改資料表，請按一下 [編輯] 並修改內容，然後按一下 [更新]。

    ![資料表](./media/storage-explorer/table.png)

#### <a name="import-and-export-table"></a>匯入和匯出資料表
1. 若要匯入，請按一下 [匯入] 按鈕，並選擇現有的資料表。
2. 若要匯出，請按一下 [匯出] 按鈕，並選擇目的地。

    ![資料表匯入和匯出](./media/storage-explorer/table-import-export.png)

#### <a name="delete-entities"></a>刪除實體
- 選取實體，然後按一下 [刪除] 按鈕。

    ![資料表刪除](./media/storage-explorer/table-delete.png)

#### <a name="query-table"></a>查詢資料表
- 按一下 [查詢] 按鈕、輸入查詢條件，然後按一下 [執行查詢] 按鈕。 按一下 [關閉查詢] 按鈕即可關閉 [查詢] 窗格。

    ![資料表查詢](./media/storage-explorer/table-query.png)

### <a name="manage-stored-procedures-triggers-and-udfs"></a>管理預存程序、觸發程序和 UDF
* 若要建立預存程序，請在左邊的樹狀目錄中，以滑鼠右鍵按一下 [預存程序]，選擇 [建立預存程序]，在左邊輸入名稱，在右邊的視窗中輸入預存程序指令碼，然後按一下 [建立]。 
* 您也可以透過按兩下現有的預存程序，進行更新，然後按一下 [更新] 以儲存，或按一下 [捨棄] 以取消變更，來編輯現有的預存程序。

    ![預存程序](./media/storage-explorer/stored-procedure.png)
* **觸發程序**及 **UDF** 的作業與**預存程序**雷同。

## <a name="troubleshooting"></a>疑難排解

[Azure 儲存體總管中的 Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/storage-explorer) 是獨立應用程式，可讓您從 Windows、macOS 或 Linux 連線到裝載在 Azure 和主權雲端上的 Azure Cosmos DB 帳戶。 也可讓您管理 Azure Cosmos DB 實體、操縱資料、更新預存程序及觸發程序，以及其他 Azure 實體 (例如儲存體 Blob 及佇列)。

這些是儲存體總管中有關 Azure Cosmos DB 常見問題的解決方案。

### <a name="sign-in-issues"></a>登入問題

在繼續之前，請先嘗試重新啟動您的應用程式，查看是否能修正問題。

#### <a name="self-signed-certificate-in-certificate-chain"></a>信任鏈結中的自我簽署憑證

有幾個原因可能會導致此錯誤，最常見的兩個原因是：

+ 您在「透明 Proxy」背後，這表示有人 (例如您的 IT 部門) 正使用自我簽署憑證攔截 HTTPS 流量、解密再加密。

+ 您正在執行的軟體，例如防毒軟體，會將自我簽署的 SSL 憑證插入您收到的 HTTPS 訊息中。

當儲存體總管遇到這些「自我簽署憑證」的其中一個時，它可能就無法知曉收到的 HTTPS 訊息是否已遭竄改。 如果您有一份自我簽署憑證，則可以告知儲存體總管信任該憑證。 如果您不確定插入憑證的是誰，可以嘗試執行下列步驟來自行尋找：

1. 安裝 Open SSL
     - [Windows](https://slproweb.com/products/Win32OpenSSL.html) (任一輕裝版即可)
     - Mac 和 Linux：應該包含在作業系統中
2. 執行 Open SSL
    - Windows：移至安裝目錄並找到 **/bin/**，然後按兩下 **openssl.exe**。
    - Mac 和 Linux：從終端機執行 **openssl**
3. 執行 `s_client -showcerts -connect microsoft.com:443`
4. 尋找自我簽署憑證。 如果不確定哪些是自我簽署的憑證，請尋找主旨 ("s:") 和簽發者 ("i:") 相同的所有位置。
5.  一旦發現任何自我簽署的憑證，請針對每個憑證，將從 **-----BEGIN CERTIFICATE-----** 到 **-----END CERTIFICATE-----** (含) 的所有內容，複製並貼到新的 .cer 檔案。
6.  開啟儲存體總管，然後移至 [編輯] > [SSL 憑證] > [匯入憑證]。 使用檔案選擇器來尋找、選取及開啟您所建立的 .cer 檔案。

如果使用上述步驟找不到任何自我簽署的憑證，請傳送意見反應給我們，以取得更多協助。

#### <a name="unable-to-retrieve-subscriptions"></a>無法擷取訂用帳戶

如果您成功登入之後，卻無法擷取訂用帳戶：

- 透過登入 [Azure 入口網站](http://portal.azure.com/)確認您的帳戶可存取訂用帳戶的項目
- 確定已使用正確的環境登入 ([Azure](http://portal.azure.com/)、[Azure 中國](https://portal.azure.cn/)、[Azure 德國](https://portal.microsoftazure.de/)、[Azure 美國政府](http://portal.azure.us/)或自訂環境/Azure Stack)
- 如果您是在 Proxy 背景，請確定已正確設定儲存體總管的 Proxy
- 嘗試移除再重新新增帳戶
- 嘗試從主目錄 (例如：C:\Users\ContosoUser) 刪除下列檔案，再重新新增帳戶：
  - .adalcache
  - .devaccounts
  - .extaccounts
- 登入時若出現任何錯誤訊息，請查看開發人員工具主控台 (f12)

![console](./media/storage-explorer/console.png)

#### <a name="unable-to-see-the-authentication-page"></a>看不到驗證頁面 

如果您看不到驗證頁面：

- 載入登入頁面可能需要一些時間，視連線速度而定，請等待至少一分鐘再關閉 [驗證] 對話方塊
- 如果您是在 Proxy 背景，請確定已正確設定儲存體總管的 Proxy
- 按下 F12 鍵，開發人員主控台會隨即顯示。 查看開發人員主控台的回應，看看能否找到任何驗證無法運作的線索

#### <a name="cannot-remove-account"></a>無法移除帳戶

如果無法移除帳戶，或重新驗證連結不執行任何動作

- 嘗試從主目錄刪除下列檔案，再重新新增帳戶：
  - .adalcache
  - .devaccounts
  - .extaccounts
- 如果您想要移除 SAS 連結的儲存體資源，請刪除：
  - Windows 是 %AppData%/StorageExplorer 資料夾
  - Mac 是 /Users/<您的名稱>/Library/Applicaiton SUpport/StorageExplorer
  - Linux 是 ~/.config/StorageExplorer
  - 如果刪除這些檔案，**您必須重新輸入所有認證**


### <a name="httphttps-proxy-issue"></a>Http/Https Proxy 的問題

在 ASE 中設定 Http/Https Proxy 時，無法在左側樹狀目錄中列出 Azure Cosmos DB 節點。 這是已知問題，並將在下一個版本中修正。 目前您可以使用 Azure 入口網站中的 Azure Cosmos DB 資料總管解決此問題。 

### <a name="development-node-under-local-and-attached-node-issue"></a>「本機與已連結」節點下的「開發」節點問題

當您在左側樹狀目錄中，按一下 [本機及連結] 節點下的 [開發] 節點後，系統沒有回應。  這是預期中的行為。 將在下一個版本中提供 Azure DB Cosmos 本機模擬器支援。

![開發節點](./media/storage-explorer/development.png)

### <a name="attaching-azure-cosmos-db-account-in-local-and-attached-node-error"></a>在「本機與已連結」節點中連結 Azure Cosmos DB 帳戶的錯誤

如果您在「本機與已連結」節點中連結 Azure Cosmos DB 帳戶後看到以下錯誤，請檢查您是否使用正確的連接字串。

![在本機與已連結中連結 Azure Cosmos DB 的錯誤](./media/storage-explorer/attached-error.png)

### <a name="expand-azure-cosmos-db-node-error"></a>展開 Azure Cosmos DB 節點錯誤

當您嘗試展開左側的樹狀節點時，可能會看到下列錯誤。 

![展開錯誤](./media/storage-explorer/expand-error.png)

請嘗試下列建議：

- 檢查 Azure Cosmos DB 帳戶是否正在進行佈建，並在帳戶成功建立後再試一次。
- 如果帳戶是在「快速存取」節點或「本機與已連結」節點下，請檢查帳戶是否已遭到刪除。 如果是這樣，您需要以手動方式移除節點。

## <a name="contact-us"></a>與我們連絡

如果沒有適合您的解決方案，請將問題詳細資料以電子郵件傳送給 Azure Cosmos DB 開發工具小組 ([cosmosdbtooling@microsoft.com](mailto:cosmosdbtooling@microsoft.com))，以修正問題。

## <a name="next-steps"></a>後續步驟

* 請觀看下列影片以了解如何在 Azure 儲存體總管中使用 Azure Cosmos DB：[在 Azure 儲存體總管中使用 Azure Cosmos DB](https://www.youtube.com/watch?v=iNIbg1DLgWo&feature=youtu.be)。
* 在[開始使用儲存體總管](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer)中深入了解儲存體總管並連線更多服務。

