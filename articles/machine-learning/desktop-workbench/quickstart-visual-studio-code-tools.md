---
title: Azure 上適用於 Machine Learning 的 Visual Studio Code Tools 的快速入門文章 | Microsoft Docs
description: 本文說明如何使用適用於 Machine Learning 的 Visual Studio Code Tools，從建立測試、定型模型以及運用 Web 服務開始。
services: machine-learning
author: chris-lauren
ms.author: clauren
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: quickstart
ms.date: 11/15/2017
ROBOTS: NOINDEX
ms.openlocfilehash: a2c4455735523b6b726dedb13151627d8ce70f22
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46991754"
---
# <a name="visual-studio-code-tools-for-ai"></a>適用於 AI 的 Visual Studio Code Tools

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

適用於 AI 的 Visual Studio Code Tools 是開發延伸模組，可建置、測試及部署深入學習和 AI 解決方案。 它的特色在於與 Azure Machine Learning 的緊密整合、值得注意的執行歷程記錄檢視、詳細說明先前的定型與自訂度量的效能。 它提供範例總管檢視，可供使用 [Microsoft 辨識工具組 (先前稱為 CNTK)](http://www.microsoft.com/cognitive-toolkit)、[Google TensorFlow](https://www.tensorflow.org) 與其他深入學習架構瀏覽並啟動新專案。 最後，提供用於計算目標的總管，它可讓您在使用 GPU 的 Azure 虛擬機器或 Linux 伺服器之類的遠端環境上，將作業提交至定型模型。 
 
## <a name="getting-started"></a>開始使用 
若要開始，您必須先下載並安裝 [Visual Studio Code](https://code.visualstudio.com/Download)。 開啟 Visual Studio Code 之後，執行下列步驟：
1. 按一下活動列中的延伸模組圖示。 
2. 搜尋「適用於 AI 的 Visual Studio Code Tools」。 
3. 按一下 [安裝] 按鈕。 
4. 安裝之後，按一下 [重新載入] 按鈕。 

一旦重新載入 Visual Studio Code，擴充功能就會啟動。 [深入了解安裝延伸模組](https://code.visualstudio.com/docs/editor/extension-gallery)。

## <a name="exploring-project-samples"></a>瀏覽專案範例
適用於 AI 的 Visual Studio Code Tools 隨附範例總管。 範例總管可讓您只按幾下，便輕鬆地探索範例並加以嘗試。 若要開啟總管，請執行下列動作：   
1. 開啟命令選擇區 ([檢視] > [命令選擇區] 或 **Ctrl+Shift+P**)。
2. 輸入「AI 範例」。 
3. 您會看見「AI：開啟 Azure ML 範例總管」的建議，請選取它然後按 Enter。 

或者，您可以按一下範例總管圖示。

## <a name="creating-a-new-project-from-the-sample-explorer"></a>從範例總管建立新專案 
您可以瀏覽不同的範例，並取得其詳細資訊。 讓我們瀏覽直到找到「分類鳶尾花」範例為止。 若要根據此範例建立新的專案，請執行下列：
1. 按一下專案範例中的 [安裝] 按鈕，注意出現的命令提示，將帶您逐步完成建立新專案的步驟。 
2. 選擇專案的名稱，例如「鳶尾花」。
3. 選擇用來建立您的專案的資料夾路徑，然後按 Enter。 
4. 選取現有的工作區，然後按 Enter。

然後會建立專案。

> [!TIP]
> 您必須已登入才能存取您的 Azure 資源。 從內嵌的終端機中，輸入 "az login"，並遵循指示。 

## <a name="submitting-experiment-with-the-new-project"></a>隨著新專案提交測試
新專案會在 Visual Studio Code 中開啟，我們可以將作業提交到我們的不同計算目標 (本機和具有 Docker 的 VM)。
適用於 AI 的 Visual Studio Code Tools 提供多種方法來提交測試。 
1. 內容功能表 (按一下滑鼠右鍵) - [AI：提交作業]。
2. 從命令選擇區：「AI：提交作業」。
3. 或者，您可以使用 Azure CLI、機器學習命令、使用內嵌的終端機直接執行命令。

開啟 iris_sklearn.py，按一下滑鼠右鍵並選取 [AI：提交作業]。
1. 選取您的平台："Azure Machine Learning"。
2. 選取您的回合組態："Docker-Python"。

> [!NOTE]
> 如果這是您第一次提交作業，您會收到訊息「找不到機器學習服務設定，正在建立...」。 JSON 檔案隨即開啟，將其儲存 (**Ctrl+S**)。

一旦提交作業，內嵌的終端機會顯示執行的進度。 

## <a name="view-list-of-jobs"></a>檢視作業清單
一旦作業提交，您可以從執行歷程記錄列出作業。
1. 開啟命令選擇區 ([檢視] > [命令選擇區] 或 **Ctrl+Shift+P**)。
2. 輸入「AI 清單」。
3. 您會看見「AI：清單作業」的建議，請選取它然後按 Enter。

[作業清單] 檢視會開啟，並顯示所有執行與一些相關資訊。

## <a name="view-job-details"></a>檢視工作詳細資料
在 [作業清單檢視] 仍開啟的情況下，按一下清單中的第一個執行。
若要深入探討作業的結果，按一下最上方的**作業識別碼**來查看詳細資訊。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [如何設定 Azure Machine Learning 使用 IDE](./how-to-configure-your-IDE.md)
