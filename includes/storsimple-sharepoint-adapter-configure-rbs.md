<!--author=SharS last changed: 1/14/2016 -->

> [!NOTE]
> 變更 StorSimple Adapter for SharePoint 的 RBS 組態時，您必須利用屬於 Domain Admins 群組的使用者帳戶登入。 此外，您必須從瀏覽器 (在和管理中心相同的主機上執行) 存取組態頁面。
> 
> 

#### <a name="to-configure-rbs"></a>設定 RBS
1. 開啟 [SharePoint 管理中心] 頁面，並瀏覽至 [系統設定] 。 
2. 在 [Azure StorSimple] 區段中，按一下 [設定 StorSimple Adapter]。
   
    ![設定 StorSimple Adapter](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS1-include.png) 
3. 在 [設定 StorSimple Adapter]  頁面上：
   
   1. 請確定已選取 [啟用編輯路徑]  核取方塊。
   2. 在文字方塊中，輸入 BLOB 存放區的通用命名慣例 (UNC) 路徑。
      
      > [!NOTE]
      > BLOB 存放區磁碟區必須裝載在設定於 StorSimple 裝置上的 iSCSI 磁碟區。

   3. 按一下每個您想要設定遠端存放的內容資料庫下方的 [啟用]  按鈕。
      
      > [!NOTE]
      > BLOB 存放區必須由所有 web 前端 (WFE) 伺服器共用，而且已針對 SharePoint 伺服器陣列設定的使用者帳戶必須可存取此共用權限。
      
      ![啟用 RBS 提供者](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS2-include.png)
      
      當您啟用或停用 RBS 時，也會看到下列訊息。
      
      ![設定 StorSimple Adapter 啟用停用](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_ConfigureStorSimpleAdapterEnableDisableMessage-include.png)

   4. 按一下 [更新]  按鈕以套用組態。 當您按一下 [更新]  按鈕時，所有 WFE 伺服器上的 RBS 組態狀態將會更新，且整個伺服器陣列將會啟用 RBS 功能。 下列訊息隨即出現。
      
      ![配接器組態訊息](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS3-include.png)
      
      > [!NOTE]
      > 如果要設定 RBS 的 SharePoint 伺服器陣列有非常大量的資料庫 (超過 200 個)，SharePoint 管理中心網頁可能會逾時。如果發生這種情況，請重新整理頁面。 這不會影響設定程序。

4. 驗證組態：
   
   1. 登入 SharePoint 管理中心網站，並瀏覽至 [設定 StorSimple Adapter] 頁面。
   2. 請檢查組態詳細資料以確定它們符合您所輸入的設定。 
5. 確認 RBS 正確地運作：
   
   1. 將文件上傳至 SharePoint。 
   2. 瀏覽至您所設定的 UNC 路徑。 請確定已建立 RBS 目錄結構，且它包含已上傳的物件。
6. (選擇性) 您可以使用隨附於 SharePoint 的 Microsoft RBS `Migrate()` PowerShell cmdlet 以移轉現有的 BLOB 內容至 StorSimple 裝置。 如需詳細資訊，請參閱[將內容移入或移出 SharePoint 2013 中的 RBS][6] 或[將內容移入或移出 RBS (SharePoint Foundation 2010)][7]。
7. (選擇性) 在測試安裝上，您可以確認已將 Blob 移出內容資料庫，如下所示： 
   
   1. 啟動 SQL Management Studio
   2. 執行 ListBlobsInDB_2010.sql 或 ListBlobsInDB_2013.sql 查詢，如下所示。
      
      ```
      **ListBlobsInDB_2013.sql**
      
        USE WSS_Content
        GO
      
        SELECT DocStreams.DocId,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               DocStreams.RbsId,
               TimeLastModified
      
        FROM DocStreams
      
             INNER JOIN AllDocs ON DocStreams.DocId = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      
      **ListBlobsInDB_2010.sql**
      
        USE WSS_Content
        GO
      
        SELECT AllDocStreams.Id,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               RbsId,
               TimeLastModified
        FROM AllDocStreams
      
             INNER JOIN AllDocs ON AllDocStreams.Id = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      ```
      
      如果已正確設定 RBS，NULL 值應該會出現在已上傳並使用 RBS 順利外部化之任何物件的 SizeOfContentInDB 資料行。
8. (選擇性) 設定 RBS 並將所有 BLOB 內容都移至 StorSimple 裝置之後，您可以將內容資料庫移至裝置。 如果您選擇移動內容資料庫，建議您在裝置上將內容資料庫儲存體設定為主要磁碟區。 然後使用以建立的 SQL Server 最佳作法，將內容資料庫移轉至 StorSimple 裝置。 
   
   > [!NOTE]
   > 只有 StorSimple 8000 系列支援將內容資料庫移至裝置 (5000 或 7000 系列並不支援)。
   
   如果您在 StorSimple 裝置上的個別磁碟區中儲存 BLOB 和內容資料庫，建議您在相同的磁碟區容器中設定它們。 這樣可確保它們將一起進行備份。
   
   > [!WARNING]
   > 如果您尚未啟用 RBS，我們不建議您將內容資料庫移至 StorSimple 裝置。 這是未經過測試的設定。
   
9. 移至下一個步驟： [設定記憶體回收](#configure-garbage-collection)。

[6]: https://technet.microsoft.com/library/ff628254(v=office.15).aspx
[7]: https://technet.microsoft.com/library/ff628255(v=office.14).aspx
