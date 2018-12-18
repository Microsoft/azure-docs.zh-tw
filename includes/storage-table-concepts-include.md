## <a name="what-is-table-storage"></a>什麼是資料表儲存體
Azure 資料表儲存體可儲存大量的結構化資料。 此服務是一個 NoSQL 資料存放區，接受來自 Azure 雲端內外經過驗證的呼叫。 Azure 資料表很適合儲存結構化、非關聯式資料。 資料表儲存體的一般用途包括：

* 儲存好幾 TB 的結構化資料，足以供應 Web 規模的應用程式所需
* 儲存資料集，不需要複雜的聯結、外部索引鍵或預存程序，還可以反正規化以加速存取
* 使用叢集索引來快速查詢資料
* 使用 OData 通訊協定和 LINQ 查詢，搭配 WCF Data Service .NET Libraries 來存取資料

您可以使用資料表儲存體來儲存和查詢龐大的結構化、非關聯式資料集，且資料表會隨著需求增加而調整。

## <a name="table-storage-concepts"></a>資料表儲存體概念
資料表儲存體包含下列元件：

![資料表儲存體元件圖表][Table1]

* **URL 格式：** Azure 資料表儲存體帳戶會使用下列格式：`http://<storage account>.table.core.windows.net/<table>`

  Azure Cosmos DB 資料表 API 帳戶會使用下列格式：`http://<storage account>.table.cosmosdb.azure.com/<table>`  

  您可以使用此位址搭配 OData 通訊協定來直接定址 Azure 資料表。 如需詳細資訊，請參閱 [OData.org][OData.org]。
* **帳戶：** 所有對 Azure 儲存體的存取都會透過儲存體帳戶執行。 如需關於儲存體帳戶容量的詳細資訊，請參閱＜ [Azure 儲存體延展性和效能目標](../articles/storage/common/storage-scalability-targets.md) ＞(英文)。 

    所有對 Azure Cosmos DB 的存取都會透過資料表 API 帳戶執行。 如需建立資料表 API 帳戶的詳細資訊，請參閱[建立資料表 API 帳戶](../articles/cosmos-db/create-table-dotnet.md#create-a-database-account)。
* **資料表**：資料表是一組實體。 資料表不強制規定實體的結構描述，這表示單一資料表包含的實體可以有幾組不同的屬性。  
* **實體**：實體是一組屬性，類似於資料庫的資料列。 Azure 儲存體中的實體大小上限為 1MB。 Azure Cosmos DB 儲存體中的實體大小上限為 2MB。
* **屬性**：屬性是名稱/值組。 每個實體最多可包含 252 個屬性來儲存資料。 每個實體也有 3 個系統屬性，可指定資料分割索引鍵、資料列索引鍵和時間戳記。 具有相同資料分割索引鍵的實體，查詢起來更快，還能在不可部分完成的作業中插入/更新。 實體的資料列索引鍵是資料分割內的唯一識別碼。

如需有關命名資料表和屬性的詳細資訊，請參閱 [了解表格服務資料模型](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model)。

[Table1]: ./media/storage-table-concepts-include/table1.png
[OData.org]: http://www.odata.org/
