---
title: 使用 Azure Data Lake Storage Gen1 的最佳做法 | Microsoft Docs
description: 了解將 Azure Data Lake Storage Gen1 (先前稱為 Azure Data Lake Store) 用在資料擷取、資料安全性和效能方面的最佳做法
services: data-lake-store
documentationcenter: ''
author: sachinsbigdata
manager: jhubbard
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 06/27/2018
ms.author: sachins
ms.openlocfilehash: ef2b5fe6c9b70eaea5ab4db2d4a0ca59ff82dbb9
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391890"
---
# <a name="best-practices-for-using-azure-data-lake-storage-gen1"></a>使用 Azure Data Lake Storage Gen1 的最佳做法

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

在本文中，您會了解使用 Azure Data Lake Storage Gen1 的最佳做法和考量。 本文提供 Data Lake Storage Gen1 的安全性、效能、恢復功能及監視作業等相關資訊。 尚未使用 Data Lake Storage Gen1 之前，若要在 Azure HDInsight 這類服務中處理如此龐大的資料，過程會十分繁瑣。 您必須將資料分給多個 Blob 儲存體帳戶，才能達到該規模的 PB 儲存量和最佳效能。 Data Lake Storage Gen1 能突破大小和效能等大部分的硬性限制。 不過，為了讓您可取得 Data Lake Storage Gen1 的最佳效能，本文中仍包含了一些考量。 

## <a name="security-considerations"></a>安全性考量

Azure Data Lake Storage Gen1 針對 Azure Active Directory (Azure AD) 使用者、群組和服務主體提供了 POSIX 存取控制與詳細稽核。 您可以對現有檔案和資料夾設定這些存取控制。 存取控制也可用來建立能套用至新檔案或資料夾的預設值。 對現有資料夾及子物件設定權限時，權限必須以遞迴方式在每個物件上傳播。 如果檔案數量很大，則傳播權限可能需要較長的時間。 所費時間大約為每秒處理 30-50 個物件。 因此，請適當地規劃資料夾結構和使用者群組。 否則，當您使用您的資料時，這可能會導致非預期的延遲和問題。 

假設您的資料夾有 100,000 個子物件。 如果以每秒處理 30 個物件的下限來計算，更新整個資料夾的權限可能需要一小時。 如需有關 Data Lake Storage Gen1 ACL 的更多詳細資料，可在[Azure Data Lake Storage Gen1 中的存取控制](data-lake-store-access-control.md)中取得。 如需以遞迴方式指派 ACL 來取得提升的效能，您可以使用 Azure Data Lake 命令列工具。 此工具會建立多個執行緒和遞迴的瀏覽邏輯，以快速將 ACL 套用至數百萬個檔案。 此工具適用於 Linux 和 Windows，而這項工具的[文件](https://github.com/Azure/data-lake-adlstool)和[下載](http://aka.ms/adlstool-download)資訊可在 GitHub 上找到。 透過您自己使用 Data Lake Storage Gen1 [.NET](data-lake-store-data-operations-net-sdk.md) 和 [Java](data-lake-store-get-started-java-sdk.md) SDK 撰寫的工具，也可以獲得相同的效能改進。

### <a name="use-security-groups-versus-individual-users"></a>使用安全性群組與個別使用者 

在 Data Lake Storage Gen1 中使用巨量資料時，最有可能是使用服務主體來允許 Azure HDInsight 這類服務使用資料。 不過，也可能會有個別使用者需要存取資料的情況。 在這種情況下，您必須使用 Azure Active Directory [安全性群組](data-lake-store-secure-data.md#create-security-groups-in-azure-active-directory)，而不是將個別使用者指派至資料夾和檔案。 

指派權限給安全性群組之後，在群組中新增或移除使用者皆不需要更新 Data Lake Storage Gen1。 這也有助於確保您不會超過 [32 個存取和預設 ACL](../azure-subscription-service-limits.md#data-lake-store-limits) 的限制 (該數量包含 4 個 POSIX 樣式 ACL，它們一律與每個檔案和資料夾相關聯：[擁有使用者](data-lake-store-access-control.md#the-owning-user)、[擁有群組](data-lake-store-access-control.md#the-owning-group)、[遮罩](data-lake-store-access-control.md#the-mask)等等)。

### <a name="security-for-groups"></a>群組的安全性 

如同前面所述，當使用者需要存取 Data Lake Storage Gen1 時，最好使用 Azure Active Directory 安全性群組。 建議一開始可以針對帳戶的根目錄使用 **ReadOnlyUsers**、**WriteAccessUsers** 和 **FullAccessUsers** 群組，甚至是針對主要子資料夾各別使用這些群組。 如果預期之後可能會新增任何其他使用者的群組 (但未經過識別)，您可以考慮建立空的安全性群組，讓其可存取特定資料夾。 使用安全性群組可確保您之後將新的權限指派給數千個檔案時，不需要很長的處理時間。 

### <a name="security-for-service-principals"></a>服務主體的安全性 

Azure HDInsight 這類服務通常會使用 Azure Active Directory 服務主體來存取 Data Lake Storage Gen1 中的資料。 因為多個工作負載間的存取需求不同，在確保組織內部和外部的安全性上，可能會有一些考量。 對於許多客戶來說，單一 Azure Active Directory 服務主體可能就已足夠，而且此服務主體可以有 Data Lake Storage Gen1 根目錄的完整權限。 但其他客戶可能需要多個叢集與不同的服務主體，一個叢集有資料的完整存取，而另一個叢集只有讀取存取權。 如同安全性群組，建好 Data Lake Storage Gen1 帳戶後，您可以考慮針對每個預期的狀況 (讀取、寫入、完整) 設定服務主體。 

### <a name="enable-the-data-lake-storage-gen1-firewall-with-azure-service-access"></a>啟用 Data Lake Storage Gen1 防火牆與 Azure 服務存取權 

Data Lake Storage Gen1 支援開啟防火牆，以及限制僅有 Azure 服務具有存取權，此功能建議用在外部入侵中的較小攻擊媒介上。 透過 Azure 入口網站中的 [防火牆] > [啟用防火牆 (開啟)] > [允許存取 Azure 服務] 選項，可啟用 Data Lake Storage Gen1 帳戶上的防火牆。  

![Data Lake Storage Gen1 中的防火牆設定](./media/data-lake-store-best-practices/data-lake-store-firewall-setting.png "Data Lake Storage Gen1 中的防火牆設定")

一旦啟用防火牆後，只有 HDInsight、Data Factory、SQL 資料倉儲等 Azure 服務能夠存取 Data Lake Storage Gen1。 由於 Azure 使用內部網路位址轉譯，因此 Data Lake Storage Gen1 防火牆不支援根據 IP 限制特定服務，而且只適用於限制 Azure 外的端點，例如內部部署環境。 

## <a name="performance-and-scale-considerations"></a>效能和規模考量

Data Lake Storage Gen1 最強大的功能之一就是移除了資料輸送量的硬性限制。 移除這些限制，可讓客戶在不需要將資料分區的情況下，擴增他們的資料大小及隨之而來的效能需求。 最佳化 Data Lake Storage Gen1 效能最重要的考量之一，就是它在提供平行處理原則時，能發揮最好的效能。

### <a name="improve-throughput-with-parallelism"></a>以平行處理原則提升輸送量 

請考慮提供每個核心 8-12 個執行緒，以取得最佳的讀取/寫入輸送量。 這是因為單一執行緒上的讀取/寫入遭到封鎖時，還有其他執行緒可允許 VM 上有較多的並行存取。 為確保層級的狀況良好，且可以增加平行處理原則，請務必監視 VM 的 CPU 使用率。   

### <a name="avoid-small-file-sizes"></a>避免小型檔案大小

Data Lake Storage Gen1 中的 POSIX 權限和稽核會在處理大量小型檔案時，帶來顯而易見的額外負荷。 最佳做法是您必須將資料批次處理成較大的檔案，而不是將成千上萬的小型資料寫入 Data Lake Storage Gen1。 避免小型檔案大小可以有許多好處，例如：

* 減少跨多個檔案的驗證檢查次數
* 減少開啟的檔案連線
* 加快複製/複寫的速度
* 更新 Data Lake Storage Gen1 POSIX 權限時處理的檔案更少 

依據使用資料的是哪些服務和工作負載，對檔案來說良好的大小是 256 MB 以上。 如果檔案大小無法在置入 Data Lake Storage Gen1 時進行批次處理，您可以個別執行壓縮作業，將這些檔案結合成較大的檔案。 如需有關檔案大小及如何在 Data Lake Storage Gen1 中組織資料的詳細資訊和建議，請參閱[建構您的資料集](data-lake-store-performance-tuning-guidance.md#structure-your-data-set)。

### <a name="large-file-sizes-and-potential-performance-impact"></a>大型檔案大小和潛在的效能影響

雖然 Data Lake Storage Gen1 可支援大小達數 PB 的大型檔案，但為獲得最佳效能，以及視讀取資料的程序而定，平均大小最好不要超過 2 GB。 例如，使用 **Distcp** 在多個位置或不同儲存體帳戶之間複製資料時，檔案是用來決定對應工作時，最細微的層級。 因此，如果您複製 10 個 1 TB 的檔案，最多會配置 10 個對應程式。 同樣地，如果您有大量已指派對應程式的檔案，一開始對應程式會以平行方式來移動大型檔案。 不過，因為作業進展會開始變得緩慢，只有少數對應程式維持配置狀態，而指派給大型檔案的單一對應程式可能會造成停滯。 Microsoft 已向 Distcp 提交改善方法，未來的 Hadoop 版本中將會解決此問題。  

另一個要考量的範例是搭配使用 Azure Data Lake Analytics 和 Data Lake Storage Gen1 時。 由於擷取程式完成的處理作業不同，有些無法分割的檔案 (例如 XML、JSON) 若大於 2 GB，可能會導致效能降低。 如果檔案可以由擷取程式分割 (例如 CSV)，則適用大型檔案。

### <a name="capacity-plan-for-your-workload"></a>工作負載的容量規劃 

Azure Data Lake Storage Gen1 會移除位於 Blob 儲存體帳戶上的硬性 IO 節流限制。 不過，仍然必須考量一些軟性限制。 預設的輸入/輸出節流限制可符合大部分案例的需求。 如果您的工作負載需要提高限制，請使用 Microsoft 支援服務。 此外，請在概念證明階段期間查看限制，好讓 IO 不會在生產階段期間達到節流限制。 如果發生這種情況，則可能需要等候 Microsoft 工程小組來執行手動增加。 若發生 IO 節流，Azure Data Lake Storage Gen1 會傳回錯誤碼 429，而理想的情況是能透過適當的指數輪詢原則進行重試。 

### <a name="optimize-writes-with-the-data-lake-storage-gen1-driver-buffer"></a>透過 Data Lake Storage Gen1 驅動程式緩衝區來最佳化「寫入」 

若要最佳化從 Hadoop 寫入 Data Lake Storage Gen1 時的效能，並且減少 IOPS，請儘可能執行接近 Data Lake Storage Gen1 驅動程式緩衝區大小的寫入作業。 例如在使用 Apache Storm 或 Spark 串流工作負載來處理串流時，試著不要在排清之前超過緩衝區大小。 從 HDInsight/Hadoop 寫入 Data Lake Storage Gen1 時，請務必知曉 Data Lake Storage Gen1 的驅動程式有 4 MB 的緩衝區。 如同許多檔案系統驅動程式，此緩衝區可以在達到 4 MB 的大小前手動排清。 如果沒有這麼做，當下次的寫入超過緩衝區的大小上限時，系統就會立即將緩衝區排清至儲存體。 可能的話，您必須避免在根據計數或時間範圍執行同步/排清原則時，發生緩衝區滿溢或明顯的緩衝區不足。

## <a name="resiliency-considerations"></a>恢復功能考量 

使用 Data Lake Storage Gen1 或任何雲端服務建構系統時，您必須考量到可用性需求，以及如何回應服務可能中斷的情況。 系統可能會將問題集中到特定執行個體或甚至是整個區域；因此，務必針對這兩者進行規劃。 根據您工作負載 SLA 的**復原時間目標**和**復原點目標**，您可以針對高可用性和災害復原選擇強度較高或較低的策略。

### <a name="high-availability-and-disaster-recovery"></a>高可用性和災害復原 

高可用性 (HA) 和災害復原 (DR) 有時可以結合在一起，雖然彼此有稍微不同的策略，特別是用在資料的時候。 為不受到本機硬體失敗的影響，Data Lake Storage Gen1 已在幕後執行 3 倍複寫。 不過，因為跨區域複寫並非內建，您必須自行管理此項目。 針對 HA 建置方案時，如果是發生服務中斷的情況，工作負載必須藉由切換至本機或新區域中各別複寫的執行個體，來盡快取得最新資料。  

在 DR 策略中，為了能應付罕見的區域性嚴重失敗事件，將資料複寫至不同區域也是很重要的。 此資料一開始可能與複寫的 HA 資料相同。 不過，您也必須考量到資料損毀這類邊緣案例的需求，您可以建立定期的快照集，以便藉此進行回復。 視資料的大小和重要性而定，您可以依據風險承受度，考慮在本機和/或次要存放區上輪流執行 1、6 和 24 小時週期的差異快照集。 

針對 Data Lake Storage Gen1 的資料恢復功能，建議您使用異地複寫將資料複寫到個別區域，且頻率需滿足您的 HA/DR 需求，最好是每小時一次。 此複寫頻率可將大量資料的移動降到最低，具有比主要系統更具競爭性的輸送量需求，且具有更佳的復原點目標 (RPO)。 此外，您應該考量應用程式使用 Data Lake Storage Gen1 的方式，讓應用程式透過監視觸發程序或失敗嘗試的時間長度，自動容錯移轉至次要帳戶，或至少傳送通知給系統管理員，以手動方式介入。 請注意，您須衡量要使用容錯移轉或等候服務回到線上。 如果資料尚未完成複寫，容錯移轉可能會導致資料遺失、不一致或複雜的資料合併。 

以下是在 Data Lake Storage Gen1 帳戶之間協調複寫的建議選項，以及各個之間的主要差異。


|  |Distcp  |Azure Data Factory  |AdlCopy  |
|---------|---------|---------|---------|
|**調整限制**     | 由背景工作角色節點繫結        | 受限於雲端資料移動單位的上限        | 由分析單位繫結        |
|**支援複製差異**     |   是      | 否         | 否         |
|**內建的協調流程**     |  否 (使用 Oozie、Airflow 或 Cron 作業)       | 是        | 否 (使用 Azure 自動化或 Windows 工作排程器)         |
|**支援的檔案系統**     | ADL、HDFS、WASB、S3、GS、CFS        |很多，請參閱[連接器](../data-factory/connector-azure-blob-storage.md)。         | ADL 至 ADL、WASB 至 ADL (僅限同一區域)        |
|**OS 支援**     |任何執行 Hadoop 的 OS         | N/A          | Windows 10         |
   

### <a name="use-distcp-for-data-movement-between-two-locations"></a>使用 Distcp 在兩個位置之間移動資料 

Distcp 是分散式複製 (distributed copy) 的縮寫，是隨附於 Hadoop 的 Linux 命令列工具，可提供兩個位置之間的分散式資料移動。 兩個位置可以是 Data Lake Storage Gen1、HDFS、WASB 或 S3。 此工具會在 Hadoop 叢集 (例如，HDInsight) 上使用 MapReduce 作業，以在所有節點上相應放大。 Distcp 是公認移動巨量資料最快速的方式，且無須特殊的網路壓縮裝置。 Distcp 也可讓您選擇只更新兩個位置的差異，並可處理自動重試及計算的動態調整。 此方法用於複寫 Hive/Spark 資料表這類項目時就非常有效率，因為這些資料表的單一目錄中可能有許多大型檔案，但您只是要複製修改的資料。 基於這些理由，在巨量資料存放區之間複製資料時，最建議使用 Distcp 工具。 

Apache Oozie 工作流程和 Linux Cron 作業可使用頻率或資料觸發程序來觸發複製作業。 對於需要大量複寫的工作，建議您特別為複製作業啟用個別的 HDInsight Hadoop 叢集，以便進行調整和縮放。 這可確保複製作業不會干擾重要的作業。 如果執行複寫的頻率夠頻繁，叢集甚至可能在每個作業間發生失敗。 如果容錯移轉到次要區域，請確定另一個叢集在次要區域中也是作用中的狀態，如此可在主要 Data Lake Storage Gen1 帳戶回復時，將新資料複寫回去。 如需使用 Distcp 的範例，請參閱[使用 Distcp 在 Azure 儲存體 Blob 與 Data Lake Storage Gen1 之間複製資料](data-lake-store-copy-data-wasb-distcp.md)。

### <a name="use-azure-data-factory-to-schedule-copy-jobs"></a>使用 Azure Data Factory 排程備份作業 

Azure Data Factory 也可使用**複製活動**來排程備份作業，甚至可以透過**複製精靈**來根據頻率進行設定。 請注意，Azure Data Factory 有雲端資料移動單位 (DMU) 限制，最終會達到大型資料工作負載的輸送量/計算上限。 此外，Azure Data Factory 目前並未提供 Data Lake Storage Gen1 帳戶之間的差異更新，因此 Hive 資料表這類資料夾將需要複寫整個複本。 如需有關使用 Data Factory 進行複製的詳細資訊，請參閱[複製活動微調指南](../data-factory/copy-activity-performance.md)。 

### <a name="adlcopy"></a>AdlCopy

AdlCopy 是 Windows 命令列工具，可讓您在兩個 Data Lake Storage Gen1 帳戶之間複製資料，但僅限同一區域內。 AdlCopy 工具提供獨立選項，或使用 Azure Data Lake Analytics 帳戶來執行複製作業的選項。 雖然該工具最初是針對隨選複製而建置，提供的複寫功能並不健全，但它提供的另一個選項，讓您可以在相同區域內的 Data Lake Storage Gen1 帳戶間執行分散式複製。 為達到可靠性，建議您對所有生產工作負載使用進階的 Data Lake Analytics 選項。 此獨立版本可以傳回忙碌回應，而且可使用的調整和監視有限。 

如同 Distcp，AdlCopy 需要由 Azure 自動化或 Windows 工作排程器這類服務來協調。 與 Data Factory 一樣，AdlCopy 不支援只複製更新的檔案，但可重新複製和覆寫現有檔案。 如需詳細資訊和使用 AdlCopy 的範例，請參閱[將資料從 Azure 儲存體 Blob 複製到 Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)。

## <a name="monitoring-considerations"></a>監視功能考量 

Data Lake Storage Gen1 提供詳細的診斷記錄和稽核。 Data Lake Storage Gen1 在 Azure 入口網站的 Data Lake Storage Gen1 帳戶下和 Azure 監視器中提供了一些基本的計量。 Data Lake Storage Gen1 的可用性會顯示在 Azure 入口網站中。 不過，此計量每 7 分鐘會重新整理一次，而且無法透過公開的 API 進行查詢。 若要取得 Data Lake Storage Gen1 帳戶最新的可用性，您必須執行您自己的綜合測試，以驗證可用性。 其他計量資訊，例如總儲存體使用率、讀取/寫入要求及輸入/輸出，可能會持續 24 小時後才重新整理一次。 因此，多個最新計量必須透過 Hadoop 命令列工具或彙總記錄資訊以手動方式計算。 取得最新儲存體使用率最快的方式是從 Hadoop 叢集節點 (例如前端節點) 執行此 HDFS 命令：   

    hdfs dfs -du -s -h adl://<adlsg1_account_name>.azuredatalakestore.net:443/

### <a name="export-data-lake-storage-gen1-diagnostics"></a>匯出 Data Lake Storage Gen1 診斷 

能最快地從Data Lake Storage Gen1 存取可搜尋記錄的其中一個方法，就是在 Data Lake Storage Gen1 帳戶的 [診斷] 刀鋒視窗下，啟用 [Log Analytics] 的記錄傳送。 這提供了以時間和內容篩選來立即存取輸入資料的功能，以及每隔 15 分鐘就會觸發的警示選項 (電子郵件/Webhook)。 如需指示，請參閱[存取 Azure Data Lake Storage Gen1 的診斷記錄](data-lake-store-diagnostic-logs.md)。 

如需更即時的警示，以及更容易控管要置入記錄的位置，請考慮將記錄匯出至 Azure EventHub，此處的內容可以各別進行分析，或針對某一段時間範圍的內容進行分析，以提交即時通知到佇列中。 然後，[邏輯應用程式](../connectors/connectors-create-api-azure-event-hubs.md)這類的應用程式就可以取用警示，並將其傳達至適當的通道，以及將計量提交至 NewRelic、Datadog 或 AppDynamics 等監視工具。 或者，如果您使用 ElasticSearch 這類第三方工具，您可以將記錄匯出到 Blob 儲存體，並使用 [Azure Logstash 外掛程式](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob)將資料取用到 Elasticsearch、Kibana 和 Logstash (ELK) 堆疊中。

### <a name="turn-on-debug-level-logging-in-hdinsight"></a>開啟 HDInsight 中的偵錯層級記錄 

如果 Data Lake Storage Gen1 記錄傳送並未開啟，Azure HDInsight 也可透過 log4j 來開啟[Data Lake Storage Gen1 的用戶端記錄](data-lake-store-performance-tuning-mapreduce.md)。 您必須在 [Ambari] > [YARN] > [Config] > [進階的 yarn-log4j 設定]中，設定下列屬性： 

    log4j.logger.com.microsoft.azure.datalake.store=DEBUG 

設定好屬性並重新啟動節點後，Data Lake Storage Gen1 診斷資料就會寫入節點上的 YARN 記錄 (/tmp/<user>/yarn.log)，並可監視錯誤或節流 (HTTP 429 錯誤代碼) 等重要的詳細資訊。 此相同資訊也可以在 Log Analytics 中監視，或在記錄送達的任何位置上監視 (透過 Data Lake Storage Gen1 帳戶的 [[診斷](data-lake-store-diagnostic-logs.md)] 刀鋒視窗)。 建議您至少開啟用戶端記錄，或利用 Data Lake Storage Gen1 的記錄傳送選項，來提供作業可見性並且使偵錯更容易進行。

### <a name="run-synthetic-transactions"></a>執行綜合交易 

目前，在 Azure 入口網站中，Data Lake Storage Gen1 的服務可用性計量為每 7 分鐘重新整理一次。 此外，它無法使用公開的 API 進行查詢。 因此，建議您建置會對 Data Lake Storage Gen1 執行綜合交易的基本應用程式，其最多每分鐘可提供可用性。 例如可建立 WebJob、邏輯應用程式或 Azure 函數應用程式來對 Data Lake Storage Gen1 執行讀取、建立和更新作業，並將結果傳送給您的監視解決方案。 這些作業會在暫存資料夾中完成，並在完成測試後加以刪除，其可以每 30-60 秒執行一次 (視需求而定)。

## <a name="directory-layout-considerations"></a>目錄配置考量

將資料置入Data Lake 時，務必預先規劃資料結構，以便有效利用安全性、資料分割和處理作業。 您可以使用下列幾個建議項目，這些都可與 Azure Data Lake Storage Gen1、Blob 儲存體或 HDFS 搭配使用。 每個工作負載在資料取用方式上都有不同的需求，但以下是使用 IoT 和批次案例時常考慮使用的配置。

### <a name="iot-structure"></a>IoT 結構 

在 IoT 工作負載中，可能會有大量資料要置入資料存放區，而且範圍橫跨大量產品、裝置、組織和取用者。 請務必為下游取用者預先規劃目錄配置，以利組織、保護和有效率地處理資料。 一般可考慮的範本可能會使用下列配置： 

    {Region}/{SubjectMatter(s)}/{yyyy}/{mm}/{dd}/{hh}/ 

例如，為英國境內的飛機引擎置入遙測資料可能如以下結構所示： 

    UK/Planes/BA1293/Engine1/2017/08/11/12/ 

將日期放在資料夾結構的尾端有很重要的原因。 如果您想要針對使用者/群組鎖定特定區域或內容，那麼您可以輕鬆地透過 POSIX 權限執行此動作。 除此之外，如果必須限定某個安全性群組只能檢視 UK 資料或特定飛機，在個別權限前加入日期結構，對每小時資料夾下的大量資料夾來說是必要的。 另外，前面有日期結構將會使資料夾數目隨著時間呈指數增加。

### <a name="batch-jobs-structure"></a>批次作業結構 

簡單來說，批次處理中常用的方法是將資料置入 "in" 資料夾。 然後，資料處理好後，將新的資料放入 "out" 資料夾，以供下游程序使用。 此目錄結構有時會出現在需要對個別檔案進行處理的作業上，而且可能不需要大量平行處理大型資料集。 如同上述建議的 IoT 結構，好的目錄結構都使用區域和內容這類事項 (例如，組織、產品/生產者) 作為父層級資料夾。 此結構有助於保護跨組織資料，以及更有效率地管理工作負載中的資料。 此外，請考慮在結構中使用日期和時間，可在處理時具有較佳的組織性、可篩選的搜尋、安全性及自動化。 日期結構的細微性層級取決於資料上傳或處理的間隔，例如每小時、每天或甚至是每個月。 

有時後，檔案處理會因為資料損毀或未預期的格式而不成功。 在這種情況下，目錄結構可能就需要 **/bad** 資料夾的功用，才能將檔案移至該資料夾並進行進一步檢查。 批次作業可能也會處理這些「不良」檔案的報告或通知，以進行手動介入。 請參考下列的範本結構： 

    {Region}/{SubjectMatter(s)}/In/{yyyy}/{mm}/{dd}/{hh}/ 
    {Region}/{SubjectMatter(s)}/Out/{yyyy}/{mm}/{dd}/{hh}/ 
    {Region}/{SubjectMatter(s)}/Bad/{yyyy}/{mm}/{dd}/{hh}/ 

例如，一間行銷公司每天從他們位於北美洲的客戶接收消費者更新的資料擷取。 而處理前和處理後的資料可能會看起來像以下程式碼片段： 

    NA/Extracts/ACMEPaperCo/In/2017/08/14/updates_08142017.csv 
    NA/Extracts/ACMEPaperCo/Out/2017/08/14/processed_updates_08142017.csv 
 
對於直接將批次資料處理至 Hive 或傳統 SQL 資料庫的常見案例，則不需要 **/in** 或 **/out** 資料夾，因為輸出已進入 Hive 資料表或外部資料庫的個別資料夾中。 例如，來自客戶的每日擷取會置入個別的資料夾，而 Azure Data Factory、Apache Oozie 或 Apache Airflow 這類服務的協調流程將會觸發每日 Hive 或 Spark 作業來處理資料，並寫入 Hive 資料表。

## <a name="next-steps"></a>後續步驟     

* [Azure Data Lake Storage Gen1 概觀](data-lake-store-overview.md) 
* [Azure Data Lake Storage Gen1 中的存取控制](data-lake-store-access-control.md) 
* [Azure Data Lake Storage Gen1 安全性](data-lake-store-security-overview.md)
* [調整 Azure Data Lake Storage Gen1 的效能](data-lake-store-performance-tuning-guidance.md)
* [HDInsight 和 Azure Data Lake Storage Gen1 上的 Spark 效能微調方針](data-lake-store-performance-tuning-spark.md)
* [HDInsight 和 Azure Data Lake Storage Gen1 上的 Hive 效能微調指導方針](data-lake-store-performance-tuning-hive.md)
* [使用 Azure Data Factory 進行 Azure Data Lake Storage Gen1 的資料協調](https://mix.office.com/watch/1oa7le7t2u4ka)
* [建立搭配 Data Lake Storage Gen1 的 HDInsight 叢集](data-lake-store-hdinsight-hadoop-use-portal.md) 
