---
title: 從另一個實驗匯入資料到 Machine Learning Studio | Microsoft Docs
description: 如何將訓練資料儲存在 Azure Machine Learning Studio 並將它使用於另一個實驗中。
keywords: 匯入資料,資料,資料來源,定型資料
services: machine-learning
documentationcenter: ''
author: bradsev
manager: jhubbard
editor: cgronlun
ms.assetid: 7da9dcec-5693-4bb6-8166-15904e7f75c3
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2017
ms.author: bradsev
ms.openlocfilehash: 8279b13270d4bba990e6670acd714be19118e98c
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834823"
---
# <a name="import-your-data-into-azure-machine-learning-studio-from-another-experiment"></a>從另一個實驗匯入資料到 Azure Machine Learning Studio。
[!INCLUDE [import-data-into-aml-studio-selector](../../../includes/machine-learning-import-data-into-aml-studio.md)]

有時候您會想要從實驗取得中繼結果，並且用來做為其他實驗的一部分。 若要這樣做，您可以將模組另存為資料集：

1. 按一下您要儲存為資料集的模組輸出。
2. 按一下 [ **另存為資料集**]。
3. 系統提示時，請輸入可讓您輕易識別資料集的名稱和說明。
4. 按一下 [ **確定** ] 核取記號。

儲存完成時，資料集可用於工作區中的任何實驗。 您可以在模組調色盤的 [ **儲存的資料集** ] 清單中找到它。

