---
title: 將資料匯入 Machine Learning Studio | Microsoft Docs
description: 如何從各種資料來源將資料匯入 Azure Machine Learning Studio。 了解支援的資料類型和資料格式。
keywords: 匯入資料,資料格式,資料類型,資料來源,定型資料
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: c194ee3b-838c-4efe-bb2a-c1d052326216
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/29/2017
ms.openlocfilehash: a5750555802489b41b007831164767beb953ebc4
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "34837458"
---
# <a name="import-your-training-data-into-azure-machine-learning-studio-from-various-data-sources"></a>從各種資料來源將訓練資料匯入 Azure Machine Learning Studio
若要在 Machine Learning Studio 中使用您自己的資料來開發和訓練預測性分析方案，您可以： 

* 事先上傳硬碟中**本機檔案**資料，在工作區中建立資料集模組
* 使用[匯入資料][import-data]模組，在實驗進行時，從數個**線上資料來源**其中之一存取資料 
* 使用其他 Azure Machine Learning **實驗**中儲存為資料集的資料
* 使用內部部署 **SQL Server 資料庫**中的資料

上述選項都在下方選單的其中一個主題裡說明。 這些主題說明如何從各種資料來源匯入用於 Machine Learning Studio 的資料。 

[!INCLUDE [import-data-into-aml-studio-selector](../../../includes/machine-learning-import-data-into-aml-studio.md)]

> [!NOTE]
> Machine Learning Studio 中有一些可用於訓練資料的範例資料集。 如需這些資訊，請參閱 [在 Azure Machine Learning Studio 中使用範例資料集](use-sample-datasets.md)。
> 
> 

這個簡介主題也會討論如何備妥資料供 Machine Learning Studio 使用，並描述支援的資料格式和資料類型。 

> [!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]
> 
> 

## <a name="get-data-ready-for-use-in-azure-machine-learning-studio"></a>備妥資料以在 Azure Machine Learning Studio 中使用
Machine Learning Studio 是專為與矩形或表格式資料搭配使用而設計，例如分隔的文字資料，或資料庫的結構化資料，雖然在某些情況下可能會使用非矩形資料。

如果您的資料相對乾淨是最佳狀況。 也就是說，在您將資料上傳至您的實驗之前，您要關注如不具引號的字串的問題。

但是，Machine Learning Studio 中有模組可讓您在實驗中進行一些資料操作。 依據您使用的機器學習演算法，您可能需要決定如何處理資料結構問題，例如遺漏值和疏鬆資料，有模組可以協助處理。 請參閱模組調色盤的 **資料轉換** 區段，以取得執行這些功能的模組。

您可以在實驗的任何位置，檢視或下載模組產生的資料，方法是按一下輸出連接埠。 視模組而定，可能會提供不同的下載選項，或者可以在 Machine Learning Studio 中的網頁瀏覽器將資料視覺化呈現。

## <a name="data-formats-and-data-types-supported"></a>支援的資料格式和資料類型
視您用來匯入資料的機制及其來源而定，您可以將一些資料類型匯入您的實驗：

* 純文字 (.txt)
* 逗號分隔值 (CSV)，具有標頭 (.csv) 或不具標頭 (.nh.csv)
* 定位鍵分隔值 (TSV)，具有標頭 (.tsv) 或不具標頭 (.nh.tsv)
* Excel 檔案
* Azure 資料表
* Hive 資料表
* SQL 資料庫資料表
* OData 值
* SVMLight 資料 (.svmlight) (請參閱 [SVMLight 定義](http://svmlight.joachims.org/) 以了解格式資訊)
* 屬性關係檔案格式 (ARFF) 資料 (.arff) (請參閱 [ARFF 定義](http://weka.wikispaces.com/ARFF) 以了解格式資訊)
* Zip 檔案 (.zip)
* R 物件或工作區檔案 (.RData)

如果您以 ARFF 格式匯入內含中繼資料的資料，Machine Learning Studio 會使用此中繼資料定義每個資料行的標題和資料類型。

如果您以 TSV 或 CSV 格式匯入不包含此中繼資料的資料，Machine Learning Studio 會透過取樣資料來推斷每個資料行的資料類型。 如果資料也沒有資料行標題，Machine Learning Studio 會提供預設名稱。

您可以使用 [編輯中繼資料][edit-metadata]，明確指定或變更資料行的標題和資料類型。

Machine Learning Studio 可辨識下列 **資料類型** ：

* 字串
* 整數 
* 兩倍
* BOOLEAN
* Datetime
* 時間範圍

Machine Learning Studio 使用名為***資料表格***的內部資料類型以在模組之間傳遞資料。 您可以使用[轉換為資料集][convert-to-dataset]模組，明確地將資料轉換為「資料表格」格式。

接受「資料表格」以外格式的任何模組，會在將資料傳遞至下一個模組之前，無訊息地將資料轉換為「資料表格」。

您可以視需要使用其他轉換模組，將「資料表格」格式轉換回 CSV、TSV、ARFF 或 SVMLight 格式。
請參閱模組調色盤的 **資料格式轉換** 區段，以取得執行這些功能的模組。

<!-- Module References -->
[convert-to-dataset]: https://msdn.microsoft.com/library/azure/72bf58e0-fc87-4bb1-9704-f1805003b975/
[edit-metadata]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
