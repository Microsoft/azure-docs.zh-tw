---
title: Data Factory 教學課程︰第一個資料管線 | Microsoft Docs
description: 此 Azure Data Factory 教學課程會示範如何使用 Hadoop 叢集上的 Hive 指令碼，建立和排程處理資料的 Data Factory。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.assetid: 81f36c76-6e78-4d93-a3f2-0317b413f1d0
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 63ae8699af5213634eeac7dfc5045a3fc888b6c0
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45734247"
---
# <a name="tutorial-build-your-first-pipeline-to-transform-data-using-hadoop-cluster"></a>教學課程︰使用 Hadoop 叢集建置您的第一個管線來轉換資料
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-build-your-first-pipeline.md)
> * [Azure 入口網站](data-factory-build-your-first-pipeline-using-editor.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Resource Manager 範本](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)


> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[快速入門：使用 Azure Data Factory 來建立資料處理站](../quickstart-create-data-factory-dot-net.md)。

在本教學課程中，您會使用資料管線建立您的第一個 Azure Data Factory。 管線藉由在 Azure HDInsight (Hadoop) 叢集上執行 Hive 指令碼，來將輸入資料轉換成輸出資料。  

本文提供教學課程的概觀與必要條件。 完成必要條件之後，您可以使用下列其中一個工具/SDK 來進行教學課程：Azure 入口網站、Visual Studio、PowerShell、Resource Manager 範本、REST API。 選取文章開頭下拉式清單中的選項，或是選取文章結尾的連結來進行教學課程。    

## <a name="tutorial-overview"></a>教學課程概觀
在本教學課程中，您會執行下列步驟：

1. 建立 **Data Factory**。 資料處理站可以包含一或多個資料管線，可移動和轉換資料。 

    在本教學課程中，您會在資料處理站中建立一個管線。 
2. 建立 **管線**。 管線可以有一或多個活動 (範例︰複製活動、HDInsight Hive 活動)。 本範例使用在 HDInsight Hadoop 叢集上執行 Hive 指令碼的 HDInsight Hive 活動。 指令碼首先會建立一個參照儲存在 Azure Blob 儲存體中的原始 Web 記錄資料的資料表，再依年份或月份分割未經處理資料。

    在本教學課程中，管線使用 Hive 活動來轉換資料，方法是在 Azure HDInsight Hadoop 叢集上執行 Hive 查詢。 
3. 建立 **連結服務**。 您會建立一個連結的服務，以將資料存放區或計算服務連結到 Data Factory。 像是 Azure 儲存體的資料存放區會保留管線中的活動輸入/輸出資料。 計算服務 (例如 HDInsight Hadoop 叢集) 會處理/轉換資料。

    在本教學課程中，您會建立兩個「連結服務」：**Azure 儲存體** 和 **Azure HDInsight**。 Azure 儲存體已連結的服務會連結提供資料處理站輸入/輸出資料的 Azure 儲存體帳戶。 Azure HDInsight 已連結的服務會連將資料轉換到資料處理站的 Azure HDInsight 叢集。 
3. 建立輸入和輸出 **資料集**。 輸入資料集表示管線中的活動輸入，而輸出資料集表示活動的輸出。

    在本教學課程中，輸入和輸出資料集是指定 Azure Blob 儲存體中輸入和輸出資料的位置。 Azure 儲存體連結服務會指定要使用的 Azure 儲存體帳戶。 輸入資料集會指定輸入資料所在的位置，而輸出資料集則指定放置輸出資料的位置。 


如需 Azure Data Factory 的詳細概觀，請參閱 [Azure Data Factory 簡介](data-factory-introduction.md)。
  
以下是您在本教學課程中建置的範例 Data Factory 的 **圖表檢視** 。 **MyFirstPipeline** 有一個 Hive 類型的活動，可接收 **AzureBlobInput** 資料集做為輸入，並輸出 **AzureBlobOutput** 資料集。 

![Data Factory 教學課程中的圖表檢視](media/data-factory-build-your-first-pipeline/data-factory-tutorial-diagram-view.png)


在本教學課程中，**adfgetstarted** Azure Blob 容器的 **inputdata** 資料夾包含一個名為 input.log 的檔案。 此記錄檔包含三個月的項目，分別是：2016 年 1 月、2 月和 3 月。 這裡是輸入檔中每個月份的資料列範例。 

```
2016-01-01,02:01:09,SAMPLEWEBSITE,GET,/blogposts/mvc4/step2.png,X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,53175,871 
2016-02-01,02:01:10,SAMPLEWEBSITE,GET,/blogposts/mvc4/step7.png,X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,30184,871
2016-03-01,02:01:10,SAMPLEWEBSITE,GET,/blogposts/mvc4/step7.png,X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,30184,871
```

當管線使用 HDInsight Hive 活動處理檔案時，活動會在依年份和月份分割輸入資料的 HDInsight 叢集中執行 Hive 指令碼。 指令碼會建立三個輸出資料夾，包含每個月份項目的檔案。  

```
adfgetstarted/partitioneddata/year=2016/month=1/000000_0
adfgetstarted/partitioneddata/year=2016/month=2/000000_0
adfgetstarted/partitioneddata/year=2016/month=3/000000_0
```

如上所示的範例行中，第一行 (標有 2016-01-01) 會寫入 month=1 資料夾中的 000000_0 檔案。 同樣地，第二行會寫入 month=2 資料夾中的檔案；而第三行會寫入 month=3 資料夾中的檔案。  

## <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列必要條件：

1. **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，只需要幾分鐘就可以建立免費試用帳戶。 請參閱 [免費試用](https://azure.microsoft.com/pricing/free-trial/) 一文了解如何取得免費試用帳戶。
2. **Azure 儲存體** – 在本教學課程中，您會使用 Azure 儲存體帳戶來儲存資料。 如果您沒有 Azure 儲存體帳戶，請參閱 [建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md) 一文。 建立儲存體帳戶之後，請記下**帳戶名稱**和**存取金鑰**。 請參閱 [檢視、複製和重新產生儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)。
3. 下載並檢閱位於下列位置的 Hive 查詢檔案 (**HQL**)：[https://adftutorialfiles.blob.core.windows.net/hivetutorial/partitionweblogs.hql](https://adftutorialfiles.blob.core.windows.net/hivetutorial/partitionweblogs.hql)。 這個查詢會轉換輸入資料並產生輸出資料。 
4. 下載並檢閱位於下列位置的範例輸入檔案 (**input.log**)：[https://adftutorialfiles.blob.core.windows.net/hivetutorial/input.log](https://adftutorialfiles.blob.core.windows.net/hivetutorial/input.log)
5. 在您的 Azure Blob 儲存體中建立名為 **adfgetstarted** 的 Blob 容器。 
6. 將 **partitionweblogs.hql** 檔案上傳到 **adfgetstarted** 容器的 [script] 資料夾中。 使用類似 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)的工具。 
7. 將 **input.log** 檔案上傳到 **adfgetstarted** 容器中的 [inputdata] 資料夾。 

完成必要條件之後，選取下列其中一個工具/SDK 來進行教學課程： 

- [Azure 入口網站](data-factory-build-your-first-pipeline-using-editor.md)
- [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
- [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
- [Resource Manager 範本](data-factory-build-your-first-pipeline-using-arm.md)
- [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)

Azure 入口網站和 Visual Studio 可讓您透過 GUI 建置資料處理站。 而 PowerShell、Resource Manager 範本和 REST API，則可讓您選擇以撰寫指令碼/程式碼的方式來建置資料處理站。

> [!NOTE]
> 本教學課程中的資料管線會轉換輸入資料來產生輸出資料， 它不是將資料從來源資料存放區，複製到目的地資料存放區。 如需說明如何使用 Azure Data Factory 複製資料的教學課程，請參閱[教學課程：將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
> 
> 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[在 Data Factory 中排程和執行](data-factory-scheduling-and-execution.md)。 





  
