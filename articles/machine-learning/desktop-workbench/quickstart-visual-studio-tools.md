---
title: Azure 上適用於 Machine Learning 的 Visual Studio Tools 的快速入門文章 | Microsoft Docs
description: 本文說明如何使用適用於 Machine Learning 的 Visual Studio Tools，從建立測試、定型模型以及運用 Web 服務開始。
services: machine-learning
author: chris-lauren
ms.author: clauren
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc, vs-azure
ms.topic: quickstart
ms.date: 11/15/2017
ROBOTS: NOINDEX
ms.openlocfilehash: a32739623f9b4809db6623f0d4d1b9737b0fff51
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984083"
---
# <a name="visual-studio-tools-for-ai"></a>適用於 AI 的 Visual Studio Tools

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

適用於 AI 的 Visual Studio Tools 是開發延伸模組，可建置、測試及部署深入學習和 AI 解決方案。 它的特色在於與 Azure Machine Learning 的緊密整合、值得注意的執行歷程記錄檢視、詳細說明先前的定型與自訂度量的效能。 它提供範例總管檢視，可供使用 [Microsoft 辨識工具組 (先前稱為 CNTK)](http://www.microsoft.com/cognitive-toolkit)、[Google TensorFlow](https://www.tensorflow.org) 與其他深入學習架構瀏覽並啟動新專案。 最後，提供用於計算目標的總管，它可讓您在使用 GPU 的 Azure 虛擬機器或 Linux 伺服器之類的遠端環境上，將作業提交至定型模型。 它也提供 [Azure Batch AI (預覽)](https://docs.microsoft.com/azure/batch-ai/) 的促進式存取。
 
## <a name="getting-started"></a>開始使用 
若要開始使用，您必須先下載並安裝 [Visual Studio](https://www.visualstudio.com/downloads/)。 開啟 Visual Studio 之後，執行下列步驟：
1. 按一下 Visual Studio 中功能表列上的 [工具]，並選取 [擴充功能和更新...]
2. 按一下 [上線] 索引標籤並選取 [搜尋 Visual Studio Marketplace]。
3. 搜尋「適用於 AI 的 Visual Studio」。 
3. 按一下 [ **下載** ] 按鈕。 
4. 安裝之後，重新啟動 Visual Studio。 

一旦重新載入 Visual Studio，擴充功能就會啟動。 [深入了解尋找擴充功能](h https://docs.microsoft.com/visualstudio/ide/finding-and-using-visual-studio-extensions)。

> [!NOTE]
> 適用於 AI 的 Visual Studio Tools 需要 Visual Studio 2015 或 2017 專業或企業版。 它不支援 Apple OSX 版本。 


## <a name="exploring-project-samples"></a>瀏覽專案範例
適用於 AI 的 Visual Studio Tools 隨附範例總管。 範例總管可讓您只按幾下，便輕鬆地探索範例並加以嘗試。 若要開啟總管，請執行下列動作：   
1. 在功能表列中，按一下 [AI 工具]。
2. 按一下 [Azure Machine Learning 資源庫]。

包含所有 Azure ML 範例的索引標籤隨即開啟。

## <a name="creating-a-new-project-from-the-sample-explorer"></a>從範例總管建立新專案 
您可以瀏覽不同的範例，並取得其詳細資訊。 讓我們瀏覽直到找到「分類鳶尾花」範例為止。 若要根據此範例建立新的專案，請執行下列作業：
1. 按一下的專案範例上的 [安裝] 按鈕，這會開啟新的對話方塊。 
2. 選取資源群組、帳戶和工作區。
3. 您可以將專案類型保留為 [一般]。
4. 輸入專案路徑與專案名稱，然後按 Enter 鍵。 
5. 對話方塊隨即開啟，提示您儲存方案，請按一下 [儲存]。 

完成後，新的專案隨即在 Visual Studio 的新執行個體中開啟。 

> [!TIP]
> 您必須登入才能存取您的 Azure 資源。 從內嵌的終端機中，輸入 "az login"，並遵循指示。 

## <a name="submitting-experiment-with-the-new-project"></a>隨著新專案提交測試
新專案會在 Visual Studio 中開啟，請將作業提交到計算目標 (本機或具有 Docker 的 VM)。
若要提交作業，請執行下列作業： 
1. 在方案總管中，以滑鼠右鍵按一下您要提交的檔案，然後選取 [設定為啟動檔案]。
2. 選取專案名稱，以滑鼠右鍵按一下並選取 [提交作業...]。
3. 將會開啟新的對話方塊，讓您選擇叢集 (或計算目標) 來執行您的指令碼。
4. 按一下 [提交]。

一旦提交作業，內嵌的終端機會顯示執行的進度。

## <a name="view-list-of-jobs"></a>檢視作業清單
一旦提交作業，您可以從執行歷程記錄列出作業。
1. 在 [伺服器總管] 中，按一下 [AI 工具]。
2. 然後選取 [Azure Machine Learning]
3. 按一下 [作業] 功能表。

作業總管會列出此專案所有提交的實驗。 

## <a name="view-job-details"></a>檢視工作詳細資料
在作業總管檢視開啟的情況下，按一下清單中的第一個執行。
這會載入 [作業摘要] 面板以及 [記錄和輸出] 面板。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [如何設定 Azure Machine Learning 使用 IDE](./how-to-configure-your-IDE.md)
