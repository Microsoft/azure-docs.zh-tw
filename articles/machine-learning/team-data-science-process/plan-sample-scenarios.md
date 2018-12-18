---
title: 識別 Azure Machine Learning 的進階分析案例 | Microsoft Docs
description: 選取適合使用 Team Data Science Process 進行進階預測性分析的案例。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 53aecc1e-5089-42cf-8d44-77678653f92d
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 62e035c0d28a797ae08480b982598d8265b18767
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39423839"
---
# <a name="scenarios-for-advanced-analytics-in-azure-machine-learning"></a>在 Azure 機器學習中的進階分析案例
本文概述可以運用 [Team Data Science Process (TDSP)](overview.md)來處理的各種範例資料來源和目標案例。 TDSP 提供系統化的方法，可讓小組共同建置智慧型應用程式。 此處呈現的案例將根據資料特性、來源位置和在 Azure 中的目標儲存機制，來說明資料處理工作流程中可用的選項。

最後一節提供 **決策樹** ，讓您在選取適合您資料和目標的範例案例時可以使用。

> [!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]
> 
> 

下列各節均提供一個範例案例。 在每個案例中，都會列出可能的資料科學或進階分析流程，以及支援的 Azure 資源。

> [!NOTE]
> **對於下列所有案例，您必須：**
> <br/>
> 
> * [建立儲存體帳戶](../../storage/common/storage-create-storage-account.md)
>   <br/>
> * [建立 Azure Machine Learning 工作區](../studio/create-workspace.md)
> 
> 

## <a name="smalllocal"></a>案例 \#1：本機檔案中的中小型表格式資料集
![中小型本機檔案][1]

#### <a name="additional-azure-resources-none"></a>其他 Azure 資源：無
1. 登入 [Azure 機器學習 Studio](https://studio.azureml.net/)。
1. 上傳資料集。
1. 建置從上傳的資料集開始的 Azure 機器學習實驗流程。

## <a name="smalllocalprocess"></a>案例 \#2：本機檔案中需要處理的中小型資料集
![需要處理的中小型本機檔案][2]

#### <a name="additional-azure-resources-azure-virtual-machine-ipython-notebook-server"></a>其他 Azure 資源：Azure 虛擬機器 (IPython Notebook 伺服器)
1. 建立執行 IPython Notebook 的 Azure 虛擬機器。
1. 將資料上傳至 Azure 儲存體容器。
1. 前置處理和清除 IPython Notebook 中的資料 (從 Azure 儲存體容器存取資料)。
1. 將資料轉換為已清理的表格式格式。
1. 將轉換的資料儲存在 Azure Blob 中。
1. 登入 [Azure 機器學習 Studio](https://studio.azureml.net/)。
1. 使用[匯入資料][import-data]模組從 Azure Blob 讀取資料。
1. 建置從內嵌的資料集開始的 Azure 機器學習實驗流程。

## <a name="largelocal"></a>案例 \#3：本機檔案的大型資料集，以 Azure Blob 為目標
![大型本機檔案][3]

#### <a name="additional-azure-resources-azure-virtual-machine-ipython-notebook-server"></a>其他 Azure 資源：Azure 虛擬機器 (IPython Notebook 伺服器)
1. 建立執行 IPython Notebook 的 Azure 虛擬機器。
1. 將資料上傳至 Azure 儲存體容器。
1. 前置處理和清除 IPython Notebook 中的資料 (從 Azure Blob 存取資料)。
1. 若有需要，請將資料轉換為已清理的表格式格式。
1. 視需要瀏覽資料並建立功能。
1. 擷取中小型資料範例。
1. 將取樣資料儲存在 Azure Blob 中。
1. 登入 [Azure 機器學習 Studio](https://studio.azureml.net/)。
1. 使用[匯入資料][import-data]模組從 Azure Blob 讀取資料。
1. 建置從內嵌的資料集開始的 Azure 機器學習實驗流程。

## <a name="smalllocaltodb"></a>案例 \#4：本機檔案的中小型資料集，以 Azure 虛擬機器中的 SQL Server 為目標
![中小型本機檔案至 Azure 中的 SQL DB][4]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>其他 Azure 資源：Azure 虛擬機器 (SQL Server / IPython Notebook 伺服器)
1. 建立執行 SQL Server + IPython Notebook 的 Azure 虛擬機器。
1. 將資料上傳至 Azure 儲存體容器。
1. 使用 IPython Notebook 前置處理和清除 Azure 儲存體容器中的資料。
1. 若有需要，請將資料轉換為已清理的表格式格式。
1. 將資料儲存至 VM-local 檔案 (IPython Notebook 會在 VM 上執行，本機磁碟是指 VM 磁碟機)。
1. 將資料載入執行於 Azure VM 的 SQL Server 資料庫。
   
   選項 \#1：使用 SQL Server Management Studio。
   
   * 登入 SQL Server VM
   * 執行 SQL Server Management Studio。
   * 建立資料庫和目標資料表。
   * 使用其中一種大量匯入方法，從 VM 本機檔案載入資料。
   
   選項 \#2：使用 IPython Notebook – 不建議用於中型和大型資料集
   
   <!-- -->    
   * 使用 ODBC 連線字串，存取 VM 上的 SQL Server。
   * 建立資料庫和目標資料表。
   * 使用其中一種大量匯入方法，從 VM 本機檔案載入資料。
1. 視需要瀏覽資料並建立功能。 請注意，功能在資料庫資料表中無需具體化。 僅注意建立這些功能的必要查詢。
1. 若有需要，請決定資料範例大小。
1. 登入 [Azure 機器學習 Studio](https://studio.azureml.net/)。
1. 使用[匯入資料][import-data]模組直接從 SQL Server 讀取資料。 視需要，將可擷取欄位、建立功能及對資料取樣的必要查詢，直接貼到[匯入資料][import-data]查詢中。
1. 建置從內嵌的資料集開始的 Azure 機器學習實驗流程。

## <a name="largelocaltodb"></a>案例 \#5：本機資料中的大型資料集，目標 Azure VM 中的 SQL Server
![大型本機檔案至 Azure 中的 SQL DB][5]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>其他 Azure 資源：Azure 虛擬機器 (SQL Server / IPython Notebook 伺服器)
1. 建立執行 SQL Server 和 IPython Notebook 伺服器的 Azure 虛擬機器。
1. 將資料上傳至 Azure 儲存體容器。
1. (選擇性) 前置處理和清除資料。
   
   a.  前置處理和清除 IPython Notebook 中的資料，從 Azure 存取資料
   
       blobs.
   
   b.  若有需要，請將資料轉換為已清理的表格式格式。
   
   c.  將資料儲存至 VM-local 檔案 (IPython Notebook 會在 VM 上執行，本機磁碟是指 VM 磁碟機)。
1. 將資料載入執行於 Azure VM 的 SQL Server 資料庫。
   
   a.  登入 SQL Server VM。
   
   b.  如果資料尚未儲存，請從 Azure 下載資料檔案
   
       storage container to local-VM folder.
   
   c.  執行 SQL Server Management Studio。
   
   d.  建立資料庫和目標資料表。
   
   e.  使用其中一個大量匯入方法來載入資料。
   
   f.  如果需要資料表聯結，請建立索引以加速聯結。
   
   > [!NOTE]
   > 如需加快大型資料的載入速度，建議您建立分割資料表並大量平行匯入資料。 如需詳細資訊，請參閱 [平行資料匯入至 SQL 分割資料表](parallel-load-sql-partitioned-tables.md)。
   > 
   > 
1. 視需要瀏覽資料並建立功能。 請注意，功能在資料庫資料表中無需具體化。 僅注意建立這些功能的必要查詢。
1. 若有需要，請決定資料範例大小。
1. 登入 [Azure 機器學習 Studio](https://studio.azureml.net/)。
1. 使用[匯入資料][import-data]模組直接從 SQL Server 讀取資料。 視需要，將可擷取欄位、建立功能及對資料取樣的必要查詢，直接貼到[匯入資料][import-data]查詢中。
1. 從上傳的資料集開始的簡單 Azure Machine Learning 實驗流程

## <a name="largedbtodb"></a>案例 \#6：SQL Server 資料庫內部部署中的大型資料集，以 Azure 虛擬機器中的 SQL Server 為目標
![大型 SQL DB 內部部署至 Azure 中的 SQL DB][6]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>其他 Azure 資源：Azure 虛擬機器 (SQL Server / IPython Notebook 伺服器)
1. 建立執行 SQL Server 和 IPython Notebook 伺服器的 Azure 虛擬機器。
1. 使用其中一個資料匯出方法來將資料從 SQL Server 匯出成傾印檔案。
   
   > [!NOTE]
   > 如果您決定從內部部署資料庫移動所有資料，一個替代 (較快速) 方法是將整個資料庫移到 Azure 中的 SQL Server 執行個體。 略過匯出資料、建立資料庫，和將資料載入/匯入目標資料庫等步驟，並依照替代方法進行。
   > 
   > 
1. 將傾印檔案上傳至 Azure 儲存體容器。
1. 將資料載入執行於 Azure 虛擬機器的 SQL Server 資料庫。
   
   a.  登入 SQL Server VM。
   
   b.  將資料檔案從 Azure 儲存體容器下載到 local-VM 資料夾。
   
   c.  執行 SQL Server Management Studio。
   
   d.  建立資料庫和目標資料表。
   
   e.  使用其中一個大量匯入方法來載入資料。
   
   f.  如果需要資料表聯結，請建立索引以加速聯結。
   
   > [!NOTE]
   > 如需加快大型資料的載入速度，請建立分割資料表並大量平行匯入資料。 如需詳細資訊，請參閱 [平行資料匯入至 SQL 分割資料表](parallel-load-sql-partitioned-tables.md)。
   > 
   > 
1. 視需要瀏覽資料並建立功能。 請注意，功能在資料庫資料表中無需具體化。 僅注意建立這些功能的必要查詢。
1. 若有需要，請決定資料範例大小。
1. 登入 [Azure 機器學習 Studio](https://studio.azureml.net/)。
1. 使用[匯入資料][import-data]模組直接從 SQL Server 讀取資料。 視需要，將可擷取欄位、建立功能及對資料取樣的必要查詢，直接貼到[匯入資料][import-data]查詢中。
1. 從上傳的資料集開始的簡單 Azure Machine Learning 實驗流程。

### <a name="alternate-method-to-copy-a-full-database-from-an-on-premises--sql-server-to-azure-sql-database"></a>將整個資料庫從內部部署 SQL Server 複製到 Azure SQL Database 的替代方法
![卸離本機 DB 和連結至 Azure 中的 SQL DB][7]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>其他 Azure 資源：Azure 虛擬機器 (SQL Server / IPython Notebook 伺服器)
若要在您的 SQL Server VM 中複寫整個 SQL Server 資料庫，您應將資料庫從一個位置/伺服器複製到另一個位置/伺服器 (假設資料庫可以暫時設定離線)。 您可以在 SQL Server Management Studio Object Explorer 中，或使用對等的 Transact-SQL 命令來執行此作業。

1. 在來源位置卸離資料庫。 如需詳細資訊，請參閱[卸離資料庫](https://technet.microsoft.com/library/ms191491\(v=sql.110\).aspx)。
1. 在 Windows 檔案總管或 Windows 命令提示字元視窗中，將已卸離的資料庫檔案和記錄檔複製到位於 Azure 中 SQL Server VM 上的目標位置。
1. 將複製檔案連結至目標 SQL Server 執行個體。 如需詳細資訊，請參閱[連結資料庫](https://technet.microsoft.com/library/ms190209\(v=sql.110\).aspx)。

[使用卸離和連結來移動資料庫 (Transact-SQL)](https://technet.microsoft.com/library/ms187858\(v=sql.110\).aspx)

## <a name="largedbtohive"></a>案例 \#7：本機檔案中的巨量資料，目標 Azure HDInsight Hadoop 叢集中的 Hive 資料庫
![本機目標 Hive 中的巨量資料][9]

#### <a name="additional-azure-resources-azure-hdinsight-hadoop-cluster-and-azure-virtual-machine-ipython-notebook-server"></a>其他 Azure 資源：Azure HDInsight Hadoop 叢集和 Azure 虛擬機器 (IPython Notebook 伺服器)
1. 建立執行 IPython Notebook 伺服器的 Azure 虛擬機器。
1. 建立 Azure HDInsight Hadoop 叢集。
1. (選擇性) 前置處理和清除資料。
   
   a.  前置處理和清除 IPython Notebook 中的資料，從 Azure 存取資料
   
       blobs.
   
   b.  若有需要，請將資料轉換為已清理的表格式格式。
   
   c.  將資料儲存至 VM-local 檔案 (IPython Notebook 會在 VM 上執行，本機磁碟是指 VM 磁碟機)。
1. 將資料上傳至步驟 2 中已選取 Hadoop 叢集的預設容器。
1. 將資料載入 Azure HDInsight Hadoop 叢集中的 Hive 資料庫。
   
   a.  登入 Hadoop 叢集的前端節點
   
   b.  開啟 Hadoop 命令列。
   
   c.  透過 Hadoop 命令列中的 `cd %hive_home%\bin` 命令進入 Hive 根目錄。
   
   d.  執行 Hive 查詢以建立資料庫和資料表，並將資料從 Blob 儲存體載入 Hive 資料表。
   
   > [!NOTE]
   > 如果資料太大，使用者可以建立包含分割區的 Hive 資料表 接著，使用者可以在前端節點上的 Hadoop 命令列中使用 `for` 迴圈，以將資料載入使用資料分割的 Hive 分割資料表。
   > 
   > 
1. 在 Hadoop 命令列中，視需要瀏覽資料並建立功能。 請注意，功能在資料庫資料表中無需具體化。 僅注意建立這些功能的必要查詢。
   
   a.  登入 Hadoop 叢集的前端節點
   
   b.  開啟 Hadoop 命令列。
   
   c.  透過 Hadoop 命令列中的 `cd %hive_home%\bin` 命令進入 Hive 根目錄。
   
   d.  在 Hadoop 叢集前端節點的 Hadoop 命令列中執行 Hive 查詢，以瀏覽資料並視需要建立功能。
1. 若有需要，取樣資料以符合 Azure Machine Learning Studio 需求。
1. 登入 [Azure 機器學習 Studio](https://studio.azureml.net/)。
1. 使用[匯入資料][import-data]模組直接從 `Hive Queries` 讀取資料。 視需要，將可擷取欄位、建立功能及對資料取樣的必要查詢，直接貼到[匯入資料][import-data]查詢中。
1. 從上傳的資料集開始的簡單 Azure Machine Learning 實驗流程。

## <a name="decisiontree"></a>用於案例選擇的決策樹
- - -
下圖摘要說明上述案例，以及讓您連結到每個分項案例的「進階分析程序和技術」選擇。 請注意，資料處理、資料探索、功能工程和取樣可能會出現在一或多個方法/環境中 (在來源、中繼和/或目標環境中)，且可以視需要反覆處理。 圖表的目的只是說明一些可能的流程，並不提供詳盡的列舉。

![範例 DS 程序逐步解說案例][8]

### <a name="advanced-analytics-in-action-examples"></a>進階分析實務範例
如需透過公用資料集運用進階分析程序和技術的端對端 Azure Machine Learning 逐步解說，請參閱：

* [Team Data Science Process 實務：使用 SQL Server](sql-walkthrough.md)。
* [Team Data Science Process 實務：使用 HDInsight Hadoop 叢集](hive-walkthrough.md)。

[1]: ./media/plan-sample-scenarios/dsp-plan-small-in-aml.png
[2]: ./media/plan-sample-scenarios/dsp-plan-local-with-processing.png
[3]: ./media/plan-sample-scenarios/dsp-plan-local-large.png
[4]: ./media/plan-sample-scenarios/dsp-plan-local-to-db.png
[5]: ./media/plan-sample-scenarios/dsp-plan-large-to-db.png
[6]: ./media/plan-sample-scenarios/dsp-plan-db-to-db.png
[7]: ./media/plan-sample-scenarios/dsp-plan-attach-db.png
[8]: ./media/plan-sample-scenarios/dsp-plan-sample-scenarios.png
[9]: ./media/plan-sample-scenarios/dsp-plan-local-to-hive.png


<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
