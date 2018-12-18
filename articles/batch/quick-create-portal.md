---
title: Azure 快速入門 - 執行 Batch 作業 - 入口網站
description: 快速了解如何使用 Azure 入口網站執行 Batch 作業。
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.devlang: na
ms.topic: quickstart
ms.date: 07/03/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 7daaf042d22ba4ac0369b732b586a3760d8cd51c
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37859569"
---
# <a name="quickstart-run-your-first-batch-job-in-the-azure-portal"></a>快速入門：在 Azure 入口網站中執行您的第一個 Batch 作業

本快速入門示範如何使用 Azure 入口網站建立 Batch 帳戶、計算節點 (虛擬機器) 的「集區」，以及在集區上執行基本「工作」的「作業」。 完成本快速入門之後，您將了解 Batch 服務的重要概念，並可準備使用更多真實的工作負載來大規模試用 Batch。

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="sign-in-to-azure"></a>登入 Azure 

在 https://portal.azure.com 登入 Azure 入口網站。

## <a name="create-a-batch-account"></a>建立批次帳戶：

請依照下列步驟來建立用於測試的範例 Batch 帳戶。 您需有 Batch 帳戶才能建立集區和作業。 如這裡所示，您可以連結 Azure 儲存體帳戶與 Batch 帳戶。 本快速入門雖然不要求這麼做，但儲存體帳戶很適合用於部署應用程式以及儲存大部分真實工作負載的輸入和輸出資料。


1. 選取 [建立資源] > [計算] > [Batch 服務]。 

  ![Marketplace 中的批次][marketplace_portal]

2. 輸入 [帳戶名稱] 和 [資源群組] 的值。 帳戶名稱必須是所選取 [Azure 位置] 中的唯一名稱，只能使用小寫字元或數字，而且包含 3-24 個字元。 

3. 在 [儲存體帳戶] 中，選取現有的儲存體帳戶或建立新的儲存體帳戶。

4. 保留其餘設定的預設值，然後選取 [建立] 以建立帳戶。

  ![建立批次帳戶：][account_portal]  

當 [部署成功] 訊息出現時，請移至入口網站中的 Batch 帳戶。

## <a name="create-a-pool-of-compute-nodes"></a>建立計算節點的集區

您現在已有一個 Batch 帳戶，請建立 Windows 計算節點的範例集區，以便用於測試。 這個快速範例的集區是由執行 Azure Marketplace 中 Windows Server 2012 R2 映像的 2 個節點所組成。


1. 在 Batch 帳戶中，選取 [集區] > [新增]。

2. 輸入名為 mypool 的 [集區識別碼]。 

3. 在 [作業系統] 中，選取下列設定 (您可以瀏覽其他選項)。
  
  |設定  |值  |
  |---------|---------|
  |**映像類型**|Marketplace (Linux/Windows)|
  |**發行者**     |MicrosoftWindowsServer|
  |**供應項目**     |WindowsServer|
  |**Sku**     |2012-R2-Datacenter-smalldisk|

  ![選取集區作業系統][pool_os] 

4. 向下捲動以輸入 [節點大小] 和 [調整大小] 設定。 建議的節點大小可為此快速範例提供良好的效能與成本平衡。
  
  |設定  |值  |
  |---------|---------|
  |**節點定價層**     |Standard_A1|
  |**目標專用節點**     |2|

  ![選取集區大小][pool_size] 

5. 保留其餘設定的預設值，然後選取 [確定] 以建立集區。

Batch 會立即建立集區，但需花費數分鐘的時間來配置和啟動計算節點。 在這段期間，集區的 [配置狀態] 為 [調整大小]。 當集區在調整大小時，您可以繼續建立作業和工作。 

![調整大小狀態的集區][pool_resizing]

幾分鐘之後，集區的狀態為 [穩定]，而節點會啟動。 選取 [節點] 以檢查節點的狀態。 當節點的狀態為 [閒置] 時，它就準備好執行工作。 

## <a name="create-a-job"></a>建立工作

既然您有集區，請建立要在其中執行的作業。 Batch 作業是一或多項工作的邏輯群組。 作業包含工作通用的設定，例如優先順序以及要執行工作的集區。 一開始作業沒有任何工作。 

1. 在 Batch 帳戶檢視中，選取 [作業] > [新增]。 

2. 輸入名為 myjob 的 [作業識別碼]。 在 [集區] 中，選取 mypool。 保留其餘設定的預設值，然後選取 [確定]。

  ![建立工作][job_create]

建立作業之後，[工作] 頁面隨即開啟。

## <a name="create-tasks"></a>建立工作

現在建立要在作業中執行的工作範例。 您通常會建立多個工作，而 Batch 會將這些工作排入佇列並分散在計算節點上執行。 在此範例中，您可建立兩個完全相同的工作。 每個工作都會執行命令列，以顯示計算節點上的 Batch 環境變數，然後等候 90 秒。 

當您使用 Batch 時，您會在此命令列中指定您的應用程式或指令碼。 Batch 提供數種方法來將應用程式和指令碼部署至計算節點。 

若要建立第一個工作：

1. 選取 [新增] 。

2. 輸入名為 mytask 的 [工作識別碼]。 

3. 在 [命令列] 中，輸入 `cmd /c "set AZ_BATCH & timeout /t 90 > NUL"`。 保留其餘設定的預設值，然後選取 [確定]。

  ![建立工作][task_create]

建立工作之後，Batch 會將它排入佇列以在集區上執行。 當節點可用來執行工作時，此工作就會執行。

若要建立第二個工作，請回到步驟 1。 請輸入不同的 [工作識別碼]，但指定相同的命令列。 如果第一個工作仍在執行中，Batch 會在集區中的其他節點上啟動第二個工作。

## <a name="view-task-output"></a>檢視工作輸出

上述的工作範例會在幾分鐘內完成。 若要檢視已完成工作的輸出，請選取 [節點上的檔案]，然後選取 `stdout.txt` 檔案。 此檔案會顯示工作的標準輸出。 內容如下所示：

![檢視工作輸出][task_output]

內容會顯示在節點上設定的 Azure Batch 環境變數。 當您建立自己的 Batch 作業和工作時，您可以在工作命令列中，以及由命令列執行的應用程式和指令碼中，參照這些環境變數。

## <a name="clean-up-resources"></a>清除資源

如果您想要繼續執行 Batch 教學課程和範例，請使用本快速入門中建立的 Batch 帳戶和連結的儲存體帳戶。 Batch 帳戶本身不收費。

即使沒有排定的作業，您仍需支付節點執行時的集區費用。 當您不再需要集區時，請將它刪除。 在帳戶檢視中，選取 [集區] 和集區名稱。 然後選取 [刪除]。  當您刪除集區時，節點上的所有工作輸出也會跟著刪除。 

若不再需要，可刪除資源群組、Batch 帳戶和所有相關資源。 若要這麼做，請選取 Batch 帳戶的資源群組，然後選取 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在本快速入門中，您建立了 Batch 帳戶、Batch 集區和 Batch 作業。 此作業執行了範例工作，而您檢視了在其中一個節點上建立的輸出。 您既然了解 Batch 服務的重要概念，即可準備使用更多真實的工作負載來大規模試用 Batch。 若要深入了解 Azure Batch，請繼續進行 Azure Batch 教學課程。 

> [!div class="nextstepaction"]
> [Azure Batch 教學課程](./tutorial-parallel-dotnet.md)

[marketplace_portal]: ./media/quick-create-portal/marketplace-batch.png

[account_portal]: ./media/quick-create-portal/batch-account-portal.png

[account_keys]: ./media/quick-create-portal/batch-account-keys.png

[pool_os]: ./media/quick-create-portal/pool-operating-system.png

[pool_size]: ./media/quick-create-portal/pool-size.png

[pool_resizing]: ./media/quick-create-portal/pool-resizing.png

[job_create]: ./media/quick-create-portal/job-create.png

[task_create]: ./media/quick-create-portal/task-create.png

[task_output]: ./media/quick-create-portal/task-output.png