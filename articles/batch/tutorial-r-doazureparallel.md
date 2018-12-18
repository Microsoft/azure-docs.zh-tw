---
title: 使用 Azure Batch 執行平行 R 模擬
description: 教學課程 - 使用 R doAzureParallel 套件在 Azure Batch 中執行 Monte Carlo 財務模擬的逐步指示
services: batch
author: dlepow
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: r
ms.topic: tutorial
ms.date: 01/23/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: fb616dc95cc7dd7dbb25f2deb832b517d0747ae4
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30315445"
---
# <a name="tutorial-run-a-parallel-r-simulation-with-azure-batch"></a>教學課程：使用 Azure Batch 執行平行 R 模擬 

使用 [doAzureParallel](http://www.github.com/Azure/doAzureParallel) 大規模執行平行 R 工作負載，這個輕量型 R 套件可讓您直接從 R 工作階段使用 Azure Batch。 doAzureParallel 套件是以熱門的 [foreach](http://cran.r-project.org/web/packages/foreach/index.html) R 套件作為建置基礎。 doAzureParallel 會採用 foreach 迴圈的每次反覆運算，並將它提交為 Azure Batch 工作。

本教學課程示範如何部署 Batch 集區，並直接在 RStudio 內執行 Azure Batch 中的平行 R 作業。 您會了解如何：
 

> [!div class="checklist"]
> * 安裝 doAzureParallel 並將它設定為存取您的 Batch 和儲存體帳戶
> * 建立 Batch 集區作為 R 工作階段的平行後端
> * 在集區上執行平行模擬範例

## <a name="prerequisites"></a>先決條件

* 已安裝 [R](https://www.r-project.org/) 散發版本，例如 [Microsoft R Open](https://mran.microsoft.com/open)。 使用 R 3.3.1 版或更新版本。

* [RStudio](https://www.rstudio.com/)，商務版本或開放原始碼 [RStudio Desktop](https://www.rstudio.com/products/rstudio/#Desktop)。 

* Azure Batch 帳戶和 Azure 儲存體帳戶。 若要建立這些帳戶，請參閱使用 [Azure 入口網站](quick-create-portal.md)或 [Azure CLI](quick-create-cli.md) 的 Batch 快速入門。 

## <a name="sign-in-to-azure"></a>登入 Azure

在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)] 
## <a name="install-doazureparallel"></a>安裝 doAzureParallel

在 RStudio 主控台中，安裝 [doAzureParallel Github 套件](http://www.github.com/Azure/doAzureParallel)。 下列命令會在目前的 R 工作階段中，下載並安裝此套件及其相依項目： 

```R
# Install the devtools package  
install.packages("devtools") 

# Install rAzureBatch package
devtools::install_github("Azure/rAzureBatch") 

# Install the doAzureParallel package 
devtools::install_github("Azure/doAzureParallel") 
 
# Load the doAzureParallel library 
library(doAzureParallel) 
```
安裝可能需要數分鐘的時間。

若要使用您先前取得的帳戶認證設定 doAzureParallel，請在工作目錄中產生名為 credentials.json 的設定檔： 

```R
generateCredentialsConfig("credentials.json") 
``` 

在此檔案中填入您的 Batch 和儲存體帳戶名稱及金鑰。 讓 `githubAuthenticationToken` 設定保持不變。

完成時，credentials 檔案會如下所示： 

```json
{
  "batchAccount": {
    "name": "mybatchaccount",
    "key": "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==",
    "url": "https://mybatchaccount.mybatchregion.batch.azure.com"
  },
  "storageAccount": {
    "name": "mystorageaccount",
    "key": "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ=="
  },
  "githubAuthenticationToken": ""
}
```

儲存檔案。 然後，執行下列命令來設定目前 R 工作階段的認證： 

```R
setCredentials("credentials.json") 
```

## <a name="create-a-batch-pool"></a>建立 Batch 集區 

doAzureParallel 包含一個函式，其可產生要執行平行 R 作業的 Azure Batch 集區 (叢集)。 節點會執行以 Ubuntu 為基礎的 [Azure 資料科學虛擬機器](../machine-learning/data-science-virtual-machine/overview.md)。 Microsoft R Open 和熱門的 R 套件都已預先安裝於此映像。 您可以檢視或自訂特定叢集設定，例如節點的數目和大小。 

若要在工作目錄中產生叢集 JSON 設定檔： 
 
```R
generateClusterConfig("cluster.json")
``` 
 
開啟此檔案來檢視預設設定，其中包含 3 個專用節點和 3 個[低優先順序](batch-low-pri-vms.md)節點。 這些設定只是可供您試驗或修改的範例。 專用節點會保留給您的集區使用。 低優先順序節點則會以較低的價格從 Azure 中的剩餘容量提供。 如果 Azure 沒有足夠的容量，便無法使用低優先順序節點。 

針對此教學課程變更設定，如下所示：

* 將 `maxTasksPerNode` 增加至 2，以利用每個節點上的兩個核心
* 將 `dedicatedNodes` 設定為 0，即可嘗試 Batch 可用的低優先順序 VM。 將 `lowPriorityNodes` 的 `min` 設定為 5， 以及將 `max` 設定為 10，或者視需要選擇較小的數字。 

保留其餘設定的預設值，然後儲存檔案。 看起來應該會像下面這樣：

```json
{
  "name": "myPoolName",
  "vmSize": "Standard_D2_v2",
  "maxTasksPerNode": 4,
  "poolSize": {
    "dedicatedNodes": {
      "min": 0,
      "max": 0
    },
    "lowPriorityNodes": {
      "min": 5,
      "max": 10
    },
    "autoscaleFormula": "QUEUE"
  },
  "containerImage": "rocker/tidyverse:latest",
  "rPackages": {
    "cran": [],
    "github": [],
    "bioconductor": []
  },
  "commandLine": []
}
```

立即建立叢集。 Batch 會立即建立集區，但需花費數分鐘的時間來配置和啟動計算節點。 在叢集可用之後，將它註冊為 R 工作階段的平行後端。 

```R
# Create your cluster if it does not exist; this takes a few minutes
cluster <- makeCluster("cluster.json") 
  
# Register your parallel backend 
registerDoAzureParallel(cluster) 
  
# Check that the nodes are running 
getDoParWorkers() 
```

輸出會顯示 doAzureParallel 的「執行背景工作角色」數目。 這個數字是節點數目乘以 `maxTasksPerNode` 的值。 如果您已如先前所述修改叢集設定，則此數字為 10。 
 
## <a name="run-a-parallel-simulation"></a>執行平行模擬

既然建立了叢集，您就能夠使用已註冊的平行後端 (Azure Batch 集區) 執行 foreach 迴圈。 例如，執行 Monte Carlo 財務模擬時，先在本機使用標準 foreach 迴圈，然後使用 Batch 執行 foreach。 這個範例藉由模擬 5 年後大量不同的結果來簡化股價預測。

假設 Contoso Corporation 的股價每天平均上漲 1.001 倍的開盤價，但其波動率 (標準差) 為 0.01。 假設開盤價為 $100，請使用 Monte Carlo 定價模擬來找出5 年後的 Contoso 股價。

Monte Carlo 模擬的參數：

```R
mean_change = 1.001 
volatility = 0.01 
opening_price = 100 
```

若要模擬收盤價，請定義下列函式：

```R
getClosingPrice <- function() { 
  days <- 1825 # ~ 5 years 
  movement <- rnorm(days, mean=mean_change, sd=volatility) 
  path <- cumprod(c(opening_price, movement)) 
  closingPrice <- path[days] 
  return(closingPrice) 
} 
```

先使用標準 foreach 迴圈搭配 `%do%` 關鍵字，在本機執行 10,000 次模擬：

```R
start_s <- Sys.time() 
# Run 10,000 simulations in series 
closingPrices_s <- foreach(i = 1:10, .combine='c') %do% { 
  replicate(1000, getClosingPrice()) 
} 
end_s <- Sys.time() 
```


在長條圖中繪製收盤價，以顯示結果的分佈情形：

```R
hist(closingPrices_s)
``` 

輸出大致如下：

![收盤價的分佈情形](media/tutorial-r-doazureparallel/closing-prices-local.png)
  
本機模擬會在數秒或更短的時間內完成：

```R
difftime(end_s, start_s) 
```

本機得到 1000 萬筆結果的估計執行時間 (使用線性逼近) 大約是 30 分鐘：

```R 
1000 * difftime(end_s, start_s, unit = "min") 
```


現在使用 `foreach` 搭配 `%dopar%` 關鍵字來執行程式碼，以比較在 Azure 中執行 1000 萬次模擬需要多久時間。 若要使用 Batch 平行處理模擬，請執行 100,000 次模擬的 100 次反覆運算：

```R
# Optimize runtime. Chunking allows running multiple iterations on a single R instance.
opt <- list(chunkSize = 10) 
start_p <- Sys.time()  
closingPrices_p <- foreach(i = 1:100, .combine='c', .options.azure = opt) %dopar% { 
  replicate(100000, getClosingPrice()) 
} 
end_p <- Sys.time() 
```

模擬作業會將工作分配給 Batch 集區中的節點。 您可以在 Azure 入口網站中看見此集區的熱度圖活動。 移至 **Batch 帳戶** > myBatchAccount。 按一下 [集區] > myPoolName。 

![執行平行 R 工作之集區的熱度圖](media/tutorial-r-doazureparallel/pool.png)

幾分鐘之後，模擬作業就完成了。 套件會自動合併結果，並將它們從節點中提取出來。 然後，您就能夠在 R 工作階段中使用結果。 

```R
hist(closingPrices_p) 
```

輸出大致如下：

![收盤價的分佈情形](media/tutorial-r-doazureparallel/closing-prices.png)

平行模擬花費了多久時間？ 

```R
difftime(end_p, start_p, unit = "min")  
```

您應會發現在 Batch 集區上執行模擬，可讓您在本機執行模擬所需的預期時間內擁有大幅提升的效能。 

## <a name="clean-up-resources"></a>清除資源

作業完成之後，就會自動刪除。 若不再需要叢集，請呼叫 doAzureParallel 套件中的 `stopCluster` 函式來刪除該叢集：

```R
stopCluster(cluster)
```

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何：

> [!div class="checklist"]
安裝 doAzureParallel 並將它設定為存取您的 Batch 和儲存體帳戶
> * 建立 Batch 集區作為 R 工作階段的平行後端
> * 在集區上執行平行模擬範例


如需 doAzureParallel 的詳細資訊，請參閱 GitHub 上的文件和範例。

> [!div class="nextstepaction"]
> [doAzureParallel 套件](https://github.com/Azure/doAzureParallel/)




