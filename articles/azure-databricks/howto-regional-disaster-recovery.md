---
title: Azure Databricks 的區域性災害復原
description: 本文說明在 Azure Databricks 中進行災害復原的方法。
services: azure-databricks
author: jasonwhowell
ms.author: jasonh
ms.service: azure-databricks
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/27/2018
ms.openlocfilehash: 671e18346651a40d7f286e984117ce0c9ae62364
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125964"
---
# <a name="regional-disaster-recovery-for-azure-databricks-clusters"></a>Azure Databricks 叢集的區域性災害復原

本文說明適用於 Azure Databricks 叢集的災害復原架構，以及可完成該設計的步驟。

## <a name="azure-databricks-overview"></a>Azure Databricks 概觀

Azure Databricks 是快速、簡單且共同作業的 Apache Spark 架構分析服務。 對於巨量資料管線，資料 (原始或結構化) 會透過 Azure Data Factory 分批內嵌到 Azure 中，或使用 Kafka、事件中樞或 IoT 中樞以近乎即時的方式進行串流處理。 此資料會放置在 Data Lake 中長期持續儲存、在 Azure Blob 儲存體或 Azure Data Lake 儲存體中。 在您的分析工作流程中，使用 Azure Databricks 從多個資料來源 (例如 [Azure Blob 儲存體](../storage/blobs/storage-blobs-introduction.md)、[Azure Data Lake Storage](../data-lake-store/index.md)、[Azure Cosmos DB](../cosmos-db/index.yml) 或 [Azure SQL 資料倉儲](../sql-data-warehouse/index.md)) 讀取資料，並使用 Spark 將它轉換成突破性見解。

![Databricks 管線](media/howto-regional-disaster-recovery/databricks-pipeline.png)

## <a name="azure-databricks-architecture"></a>Azure Databricks 架構

概括而言，當您從 Azure 入口網站建立 Azure Databricks 工作區時，[受控設備](../managed-applications/overview.md)會在所選的 Azure 區域 (例如，美國西部) 部署為您訂用帳戶中的 Azure 資源。 此設備會部署在具有[網路安全性群組](../virtual-network/manage-network-security-group.md)和 Azure 儲存體帳戶的 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md)中 (適用於的您訂用帳戶)。 虛擬網路會提供周邊層級安全性給 Databricks 工作區，而且透過網路安全性群組保護。 在工作區內，提供背景工作角色和驅動程式 VM 類型及 Databricks 執行階段版本，即可建立 Databricks 叢集。 在您的儲存體帳戶 (可以是 Azure Blob 儲存體或 Azure Data Lake Store) 中可取得保存的資料。 建立叢集之後，將 Notebook、REST API、ODBC/JDBC 端點附加到特定叢集，即可透過這些端點執行作業。

Databricks 控制平面可管理和監視 Databricks 工作區環境。 從控制平面可起始任何管理作業，例如建立叢集。 所有中繼資料 (例如排定工作) 都會利用異地複寫儲存在 Azure 資料庫中，以便容錯移轉。

![Databricks 架構](media/howto-regional-disaster-recovery/databricks-architecture.png)

此架構的優點之一就是使用者可以將 Azure Databricks 連線到其帳戶中的任何儲存體資源。 主要優點是計算 (Azure Databricks) 和儲存體均可獨立進行調整。

## <a name="how-to-create-a-regional-disaster-recovery-topology"></a>如何建立區域性災害復原拓撲

如您在先前的架構描述中所見，有許多元件使用於採用 Azure Databricks 的巨量資料管線：Azure 儲存體、Azure 資料庫和其他資料來源。 Azure Databricks 是適用於巨量資料管線的「計算」。 其具有「暫時」性質，也就是說，當您的資料仍可在 Azure 儲存體中取得時，可以終止「計算」 (Azure Databricks 叢集)，您就不必支付不需要的計算費用。 「計算」(Azure Databricks) 和儲存體來源必須位於相同的區域中，作業才不會發生高延遲。  

若要建立您自己的區域性災害復原拓撲，請遵循下列需求：

   1. 在個別的 Azure 區域中佈建多個 Azure Databricks 工作區。 例如，在美國東部 2 中建立主要 Azure Databricks 工作區。 在不同區域 (例如美國西部) 中建立次要災害復原 Azure Databricks 工作區。

   2. 使用[異地備援儲存體](../storage/common/storage-redundancy-grs.md#read-access-geo-redundant-storage)。 根據預設，與 Azure Databrick 相關聯的資料會儲存在 Azure 儲存體中。 Databricks 作業的結果也會儲存在 Azure Blob 儲存體中，因此在終止叢集之後，處理過的資料仍具有耐久性且維持高可用性。 儲存體與 Databricks 叢集會共置，因此您必須使用異地備援儲存體，以便在主要區域不再可供存取時，可以在次要區域中存取資料。

   3. 建立次要地區之後，您必須遷移使用者、使用者資料夾、Notebook、叢集組態、作業組態、程式庫、儲存體、init 指令碼，以及重新設定存取控制。 下一節會概述其他詳細資料。

## <a name="detailed-migration-steps"></a>詳細的移轉步驟

1. **在您的電腦上設定 Databricks 命令列介面**

   本文說明許多程式碼範例，其使用命令列介面進行大部分的自動化步驟，因為它是透過 Azure Databricks REST API 的易用包裝函式。

   執行任何移轉步驟之前，請在您的桌上型電腦或您打算執行此工作的虛擬機器上安裝 databricks-cli。 如需詳細資訊，請參閱[安裝 Databricks CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html)。

   ```bash
   pip install databricks-cli
   ```

   > [!NOTE]
   > 本文中提供的任何 python 指令碼預計會搭配 Python 2.7+ < 3.x 運作。

2. **設定兩個設定檔。**

   為主要工作區設定一個設定檔，為次要工作區設定另一個設定檔：

   ```bash
   databricks configure --profile primary
   databricks configure --profile secondary
   ```

   本文中的程式碼區塊會使用對應的工作區命令，在每個後續步驟中的設定檔之間切換。 確定您建立的設定檔名稱會替代至每個程式碼區塊中。

   ```python
   EXPORT_PROFILE = "primary"
   IMPORT_PROFILE = "secondary"
   ```

   如有需要，您可以在命令列手動切換：

   ```bash
   databricks workspace ls --profile primary
   databricks workspace ls --profile secondary
   ```

3. **遷移 Azure Active Directory 使用者**

   將相同的 Azure Active Directory 使用者手動新增至主要工作區中存在的次要工作區。

4. **遷移使用者資料夾和 Notebook**

   使用下列 python 程式碼來遷移沙箱化使用者環境，其中包含巢狀資料夾結構和每個使用者的 Notebook。

   > [!NOTE]
   > 程式庫不會在此步驟中複製，因為基礎 API 不支援這些程式庫。

   將下列 python 指令碼複製並儲存到檔案，然後在 Databricks 命令列中加以執行。 例如： `python scriptname.py`。

   ```python
   from subprocess import call, check_output

   EXPORT_PROFILE = "primary"
   IMPORT_PROFILE = "secondary"

   # Get a list of all users
   user_list_out = check_output(["databricks", "workspace", "ls", "/Users", "--profile", EXPORT_PROFILE])
   user_list = user_list_out.splitlines()

   # Export sandboxed environment (folders, notebooks) for each user and import into new workspace.
   # Libraries are not included with these APIs / commands.

   for user in user_list:
     print "Trying to migrate workspace for user " + user

     call("mkdir -p " + user, shell=True)
     export_exit_status = call("databricks workspace export_dir /Users/" + user + " ./" + user + " --profile " + EXPORT_PROFILE, shell=True)

     if export_exit_status==0:
       print "Export Success"
       import_exit_status = call("databricks workspace import_dir ./" + user + " /Users/" + user + " --profile " + IMPORT_PROFILE, shell=True)
       if import_exit_status==0:
         print "Import Success"
       else:
         print "Import Failure"
     else:
       print "Export Failure"

   print "All done"
   ```

5. **遷移叢集組態**

   Notebook 完成遷移後 ，您可以選擇性地將叢集組態遷移至新的工作區。 除非您想要執行選擇性叢集組態移轉 (而非全部移轉)，否則使用 databricks-cli 就是幾乎完全自動化的步驟。

   > [!NOTE]
   > 不幸的是，沒有任何建立叢集組態端點，而此指令碼會嘗試立即建立每個叢集。 如果您的訂用帳戶中沒有足夠的核心可用，則叢集建立可能會失敗。 只要組態順利轉移，就可以忽略此失敗。

   下列提供的指令碼會列印新舊叢集識別碼的對應，其稍後會使用於作業移轉 (適用於設定為使用現有叢集的作業)。

   將下列 python 指令碼複製並儲存到檔案，然後在 Databricks 命令列中加以執行。 例如： `python scriptname.py`。

   ```python
   from subprocess import call, check_output import json

   EXPORT_PROFILE = "primary"
   IMPORT_PROFILE = "secondary"

   # Get all clusters info from old workspace 
   clusters_out = check_output(["databricks", "clusters", "list", "--profile", EXPORT_PROFILE]) clusters_info_list = clusters_out.splitlines()

   # Create a list of all cluster ids 
   clusters_list = [] for cluster_info in clusters_info_list:   clusters_list.append(cluster_info.split(None, 1)[0])

   # Optionally filter cluster ids out manually, so as to create only required ones in new workspace

   # Create a list of mandatory / optional create request elements 
   cluster_req_elems = ["num_workers","autoscale","cluster_name","spark_version","spark_conf"," node_type_id","driver_node_type_id","custom_tags","cluster_log_conf","sp ark_env_vars","autotermination_minutes","enable_elastic_disk"]

   # Try creating all / selected clusters in new workspace with same config as in old one.
   cluster_old_new_mappings = {} for cluster in clusters_list:   print "Trying to migrate cluster " + cluster

   cluster_get_out = check_output(["databricks", "clusters", "get", "--cluster-id", cluster, "--profile", EXPORT_PROFILE])
   print "Got cluster config from old workspace"

   # Remove extra content from the config, as we need to build create request with allowed elements only
   cluster_req_json = json.loads(cluster_get_out)    
   cluster_json_keys = cluster_req_json.keys()   

   for key in cluster_json_keys:     
      if key not in cluster_req_elems:       
         cluster_req_json.pop(key, None)
  
   # Create the cluster, and store the mapping from old to new cluster ids
   cluster_create_out = check_output(["databricks", "clusters", "create", "--json", json.dumps(cluster_req_json), "--profile", IMPORT_PROFILE]) 
   cluster_create_out_json = json.loads(cluster_create_out)   
   cluster_old_new_mappings[cluster] = cluster_create_out_json['cluster_id']

   print "Sent cluster create request to new workspace successfully"

   print "Cluster mappings: " + json.dumps(cluster_old_new_mappings)
   print "All done"
   ```

6. **遷移作業組態**

   如果您在上一個步驟中遷移叢集組態，您可選擇將作業組態遷移到新的工作區。 除非您想要執行選擇性作業組態移轉 (而非對所有作業執行)，否則使用 databricks-cli 就是完全自動化的步驟。

   > [!NOTE]
   > 排定作業的組態也包含「排程」資訊，因此該作業預設會在遷移後按照所設定的時機開始運作。 因此，下列程式碼區塊會在移轉期間移除所有排程資訊 (以避免在舊的和新的工作區中重複執行)。 當您準備好進行完全移轉，請設定這類作業的排程。

   需要新的或現有叢集的設定，才能設定作業。 如果使用現有叢集，以下指令碼 / 程式碼會嘗試將舊的叢集識別碼取代為新的叢集識別碼。

   將下列 python 指令碼複製並儲存到檔案。 使用前一個步驟所進行叢集移轉的輸出，取代 `old_cluster_id` 和 `new_cluster_id` 值。 在 databricks-cli 命令列中執行，例如 `python scriptname.py`。

   ```python
   from subprocess import call, check_output
   import json

   EXPORT_PROFILE = "primary"
   IMPORT_PROFILE = "secondary"

   # Please replace the old to new cluster id mappings from cluster migration output
   cluster_old_new_mappings = {"old_cluster_id": "new_cluster_id"}

   # Get all jobs info from old workspace
   try:
     jobs_out = check_output(["databricks", "jobs", "list", "--profile", EXPORT_PROFILE])
     jobs_info_list = jobs_out.splitlines()
   except:
     print "No jobs to migrate"
     sys.exit(0)

   # Create a list of all job ids
   jobs_list = []
   for jobs_info in jobs_info_list:
     jobs_list.append(jobs_info.split(None, 1)[0])

   # Optionally filter job ids out manually, so as to create only required ones in new workspace

   # Create each job in the new workspace based on corresponding settings in the old workspace

   for job in jobs_list:
     print "Trying to migrate " + job

     job_get_out = check_output(["databricks", "jobs", "get", "--job-id", job, "--profile", EXPORT_PROFILE])
     print "Got job config from old workspace"

     job_req_json = json.loads(job_get_out)  
     job_req_settings_json = job_req_json['settings']

     # Remove schedule information so job doesn't start before proper cutover
     job_req_settings_json.pop('schedule', None)

     # Replace old cluster id with new cluster id, if job configured to run against an existing cluster
     if 'existing_cluster_id' in job_req_settings_json:
       if job_req_settings_json['existing_cluster_id'] in cluster_old_new_mappings:
         job_req_settings_json['existing_cluster_id'] = cluster_old_new_mappings[job_req_settings_json['existing_cluster_id']]
       else:
         print "Mapping not available for old cluster id " + job_req_settings_json['existing_cluster_id']
         continue

     call(["databricks", "jobs", "create", "--json", json.dumps(job_req_settings_json), "--profile", IMPORT_PROFILE])
     print "Sent job create request to new workspace successfully"

   print "All done"
   ```

7. **遷移程式庫**

   目前沒有任何直接的方法，可將程式庫從一個工作區遷移到另一個工作區。 反而是，將這些程式庫手動重新安裝到新的工作區中。 有可能自動使用 [DBFS CLI](https://github.com/databricks/databricks-cli#dbfs-cli-examples) 的組合，將自訂程式庫上傳至工作區和 [程式庫 CLI](https://github.com/databricks/databricks-cli#libraries-cli)。

8. **遷移 Azure Blob 儲存體和 Azure Data Lake Store 掛接**

   使用以 Notebook 為基礎的解決方案，手動重新掛接所有的 [Azure Blob 儲存體](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-storage.html)和 [Azure Data Lake Store (Gen 1)](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-datalake.html) 掛接點。 儲存體資源會已掛接在主要工作區中，而且必須在次要工作區中重複。 沒有外部 API 可供掛接使用。

9. **遷移叢集 init 指令碼**

   使用 [DBFS CLI](https://github.com/databricks/databricks-cli#dbfs-cli-examples)，可以將任何叢集初始化指令碼從舊工作區遷移至新工作區。 首先，將所需的指令碼從 `dbfs:/dat abricks/init/..` 複製到本機桌面或虛擬機器。 接下來，將這些指令碼複製到相同路徑中的新工作區。

   ```bash
   // Primary to local
   dbfs cp -r dbfs:/databricks/init ./old-ws-init-scripts --profile primary

   // Local to Secondary workspace
   dbfs cp -r old-ws-init-scripts dbfs:/databricks/init --profile secondary
   ```

10. **手動重新設定並重新套用存取控制。**

   如果現有的主要工作區已設定為使用進階層 (SKU)，您可能也會使用[存取控制功能](https://docs.azuredatabricks.net/administration-guide/admin-settings/index.html#manage-access-control)。

   如果您使用存取控制功能，請將存取控制手動重新套用到資源 (Notebook、叢集、作業、資料表)。

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [Azure Databricks 文件](https://docs.azuredatabricks.net/user-guide/index.html)。
