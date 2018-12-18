# [Data Lake Storage Gen1 文件](index.md)
# [切換為第二代 Data Lake Storage (預覽) 文件](https://docs.microsoft.com/azure/storage/data-lake-storage/introduction)

# 概觀
## [Data Lake Storage Gen1 概觀](data-lake-store-overview.md)
## [與 Azure 儲存體比較](data-lake-store-comparison-with-blob-storage.md)
## [巨量資料處理](data-lake-store-data-scenarios.md)
## [使用開放原始碼應用程式](data-lake-store-compatible-oss-other-applications.md)
## [最佳做法](data-lake-store-best-practices.md)

# 開始使用
## [使用 Azure 入口網站](data-lake-store-get-started-portal.md)
## [使用 Azure PowerShell](data-lake-store-get-started-powershell.md)
## [使用 Azure CLI](data-lake-store-get-started-cli-2.0.md)

# 作法
## 載入和移動資料
### [使用 Azure Data Factory](../data-factory/load-azure-data-lake-store.md)
### [使用儲存體總管](data-lake-store-in-storage-explorer.md)
### [使用 AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)
### [使用 DistCp](data-lake-store-copy-data-wasb-distcp.md)
### [使用 Sqoop](data-lake-store-data-transfer-sql-sqoop.md)
### [從離線來源上傳資料](data-lake-store-offline-bulk-data-upload.md)
### [跨區域遷移 Data Lake Storage Gen1](data-lake-store-migration-cross-region.md)

## 保護資料
### [安全性概觀](data-lake-store-security-overview.md)
### [存取控制](data-lake-store-access-control.md)
### [保護儲存的資料](data-lake-store-secure-data.md)
### [加密](data-lake-store-encryption.md)
### [虛擬網路整合 (預覽)](data-lake-store-network-security.md)

## 使用 Data Lake Storage Gen1 進行驗證
### [驗證選項](data-lakes-store-authentication-using-azure-active-directory.md)
### [使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)
#### [使用 Java](data-lake-store-end-user-authenticate-java-sdk.md)
#### [使用 .NET SDK](data-lake-store-end-user-authenticate-net-sdk.md)
#### [使用 REST API](data-lake-store-end-user-authenticate-rest-api.md)
#### [使用 Python](data-lake-store-end-user-authenticate-python.md)
### [服務對服務驗證](data-lake-store-service-to-service-authenticate-using-active-directory.md)
#### [使用 Java](data-lake-store-service-to-service-authenticate-java.md)
#### [使用 .NET SDK](data-lake-store-service-to-service-authenticate-net-sdk.md)
#### [使用 REST API](data-lake-store-service-to-service-authenticate-rest-api.md)
#### [使用 Python](data-lake-store-service-to-service-authenticate-python.md)

## 與 Data Lake Storage Gen1 搭配使用
### 帳戶管理作業
#### [使用 .NET SDK](data-lake-store-get-started-net-sdk.md)
#### [使用 REST API](data-lake-store-get-started-rest-api.md)
#### [使用 Python](data-lake-store-get-started-python.md)
### 檔案系統作業
#### [使用 .NET SDK](data-lake-store-data-operations-net-sdk.md)
#### [使用 Java SDK](data-lake-store-get-started-java-sdk.md)
#### [使用 REST API](data-lake-store-data-operations-rest-api.md)
#### [使用 Python](data-lake-store-data-operations-python.md)

## 效能
### [概觀](data-lake-store-performance-tuning-guidance.md)
### [使用 Azure PowerShell](data-lake-store-performance-tuning-powershell.md)
### [使用 HDInsight 上的 Spark](data-lake-store-performance-tuning-spark.md)
### [使用 HDInsight 上的 Hive](data-lake-store-performance-tuning-hive.md)
### [在 HDInsight 上使用 MapReduce](data-lake-store-performance-tuning-mapreduce.md)
### [使用 HDInsight 上的 Storm](data-lake-store-performance-tuning-storm.md)

## 與 Azure 服務整合
### 搭配 HDInsight
#### [使用 Azure 入口網站](data-lake-store-hdinsight-hadoop-use-portal.md)
#### [使用 Azure PowerShell (預設儲存體)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
#### [使用 Azure PowerShell (其他儲存體)](data-lake-store-hdinsight-hadoop-use-powershell.md)
#### [使用 Azure 範本](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
### [從 Azure VNET 內的 VM 存取](data-lake-store-connectivity-from-vnets.md)
### [使用 Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
### [搭配使用 Azure 事件中樞](data-lake-store-archive-eventhub-capture.md)
### [搭配 Data Factory 使用](../data-factory/load-azure-data-lake-store.md)
### [搭配串流分析使用](data-lake-store-stream-analytics.md)
### [搭配 Power BI 使用](data-lake-store-power-bi.md)
### [搭配資料目錄使用](data-lake-store-with-data-catalog.md)
### [在 SQL 資料倉儲中使用 PolyBase](../sql-data-warehouse/sql-data-warehouse-load-from-azure-data-lake-store.md)
### [搭配使用 SQL Server 整合服務](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-data-lake-store-connection-manager)
### [更多 Azure 整合選項](data-lake-store-integrate-with-other-services.md)

## 管理
### [存取診斷記錄檔](data-lake-store-diagnostic-logs.md)
### [高可用性規劃](data-lake-store-disaster-recovery-guidance.md)

# 參考
## [程式碼範例](https://azure.microsoft.com/resources/samples/?service=data-lake-store)
## [Azure PowerShell](/powershell/module/azurerm.datalakestore)
## [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/data-lake-store?view=azure-dotnet)
## [Java](/java/api/com.microsoft.azure.datalake.store)
## [Node.js](https://www.npmjs.com/package/azure-arm-datalake-store)
## [Python (帳戶管理)](https://docs.microsoft.com/python/api/azure.mgmt.datalake.store?view=azure-python)
## [Python (檔案系統管理)](http://azure-datalake-store.readthedocs.io/en/latest)
## [REST](/rest/api/datalakestore)
## [Azure CLI](https://docs.microsoft.com/cli/azure/dls)

# 資源
## [Azure 藍圖](https://azure.microsoft.com/roadmap/)
## [Data Lake Store Blog](https://blogs.msdn.microsoft.com/azuredatalake/)
## [在 UserVoice 上提供意見反應](https://feedback.azure.com/forums/327234-data-lake)
## [MSDN 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureDataLake)
## [定價](https://azure.microsoft.com/pricing/details/data-lake-store/)
## [定價計算機](https://azure.microsoft.com/pricing/calculator/)
## [服務更新](https://azure.microsoft.com/updates/?product=data-lake-store)
## [Stack Overflow 論壇](http://stackoverflow.com/questions/tagged/azure-data-lake)
## [影片](https://azure.microsoft.com/documentation/videos/index/?services=data-lake-store)
