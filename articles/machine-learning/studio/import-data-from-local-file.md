---
title: 將資料從檔案匯入到 Azure Machine Learning Studio | Microsoft Docs
description: 了解如何將訓練資料檔案從硬碟上傳到 Azure Machine Learning Studio。 這樣會在工作區中建立資料集模組。
keywords: 匯入資料、資料格式、資料類型、資料來源、定型資料
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: c0dd9e90-23c4-4f64-8b8f-489ad79f047b
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2017
ms.openlocfilehash: 0365492b2814d686dd0bfa099e94717137b51725
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834765"
---
# <a name="import-training-data-from-a-file-on-your-hard-drive-into-machine-learning-studio"></a>將訓練資料從硬碟上的檔案匯入到 Machine Learning Studio
[!INCLUDE [import-data-into-aml-studio-selector](../../../includes/machine-learning-import-data-into-aml-studio.md)]

了解如何上傳硬碟上的資料檔案，以在 Azure Machine Learning Studio 中做為訓練資料使用。 透過匯入資料檔案，您就有可在工作區中使用的資料集模組。

## <a name="steps-to-import-data-from-a-local-file"></a>從本機檔案匯入資料的步驟
若要從本機硬碟匯入資料，請執行下列作業︰

1. 在 [Machine Learning Studio] 視窗底部，按一下 [ **+新增** ]。
2. 選取 [資料集] 和 [從本機檔案]。
3. 在 [ **上傳新的資料集** ] 對話方塊中，瀏覽至您要上傳的檔案。
4. 輸入名稱、識別資料類型，然後選擇性地輸入說明。 建議輸入說明 - 它可以讓您記錄資料的任何特性，您會希望在未來使用資料時記住這些特性。
5. 核取方塊 [ **這是現有資料集的新版本** ] 可讓您使用新資料更新現有資料集。 按一下此核取方塊然後輸入現有資料集的名稱即可。

![上傳新的資料集](./media/import-data-from-local-file/upload-dataset.png)

在上傳期間，您會看見訊息，指出您的檔案正在上傳。 上傳時間取決於資料大小和連接至服務的速度。 如果您知道上傳檔案將耗費很長的時間，您可以在等待時在 ML Studio 中執行其他動作。 但是，關閉瀏覽器會導致資料上傳失敗。

## <a name="dataset-module-is-ready-for-use"></a>資料集模組已可供使用
資料上傳之後，會儲存在資料集模組中，並且可供工作區中的任何實驗使用。

當您在編輯實驗時，您可以在模組調色盤的 [儲存的資料集] 清單底下的 [我的資料集] 清單內，找到您所建立的資料集。 當您想將資料集用於進一步的分析和機器學習時，可以在實驗畫布上拖放資料集。
