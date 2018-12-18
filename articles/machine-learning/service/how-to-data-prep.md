---
title: 使用適用於 Python 的 Machine Learning 資料準備 SDK 來準備資料 - Azure
description: 了解如何使用適用於 Python 的 Azure Machine Learning 資料準備 SDK 來載入各種格式的資料、將資料轉換為更實用的格式，然後該資料寫入到某個位置供您的模型存取。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: cforbe
author: cforbe
manager: cgronlun
ms.reviewer: jmartens
ms.date: 09/24/2018
ms.openlocfilehash: 673b8ddf3ed07507cbce4b54e9b8a488cb3ac689
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166600"
---
# <a name="prepare-data-for-modeling-with-azure-machine-learning"></a>使用 Azure Machine Learning 準備模型用的資料
 
資料準備是機器學習工作流程的一個重要部分。 若您的模型可以存取容易取用之格式的乾淨資料，您的模型將更精確、更有效。 

您可以使用 [Azure Machine Learning 資料準備 SDK](https://docs.microsoft.com/python/api/overview/azure/dataprep?view=azure-dataprep-py)，以 Python 準備資料。 

## <a name="data-preparation-pipeline"></a>資料準備管道

主要的資料準備步驟如下：

1. [載入資料](how-to-load-data.md)，它可以是各種格式
2. [轉換](how-to-transform-data.md)成更有用的結構
3. 將資料[寫入](how-to-write-data.md)到您模型可以存取的位置

![資料準備程序](./media/concept-data-preparation/data-prep-process.png)

## <a name="next-steps"></a>後續步驟
檢閱使用「Azure Machine Learning 資料準備 SDK」進行之資料準備的[範例 Notebook](https://github.com/Microsoft/PendletonDocs/blob/master/Scenarios/GettingStarted/getting-started.ipynb)。
