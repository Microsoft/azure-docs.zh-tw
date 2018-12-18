---
title: 在 Azure Machine Learning 資料準備中衍生新資料行的範例 Python | Microsoft Docs
description: 本文件提供在 Azure Machine Learning 資料準備中建立新資料行的 Python 程式碼範例。
services: machine-learning
author: euangMS
ms.author: euang
manager: lanceo
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: ''
ms.devlang: ''
ms.topic: article
ms.date: 02/01/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 43a66efde2623a848065bbde2be4b56fd6c7fc8b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969303"
---
# <a name="sample-of-custom-column-transforms-python"></a>自訂資料行轉換的範例 (Python) 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


這個轉換在功能表中的名稱是**新增資料行 (指令碼)**。

閱讀本附錄之前，請先參閱 [Python 擴充性概觀](data-prep-python-extensibility-overview.md)。

## <a name="test-equivalence-and-replace-values"></a>等價測試並取代值 
如果 Col1 中的值小於 4，則新資料行的值應該是 1。 如果 Col1 中的值大於 4，則新資料行的值應該是 2。 

```python
    1 if row["Col1"] < 4 else 2
```
## <a name="current-date-and-time"></a>目前的日期和時間 

```python
    datetime.datetime.now()
```
## <a name="typecasting"></a>類型轉型 
```python
    float(row["Col1"]) / float(row["Col2"] - 1)
```
## <a name="evaluate-for-nullness"></a>評估 Null 值 
如果 Col1 包含 Null，則將新的資料行標示為 **Bad**。 如果未包含，則將其標示為 **Good**。 

```python
    'Bad' if pd.isnull(row["Col1"]) else 'Good'
```
## <a name="new-computed-column"></a>新的計算資料行 
```python
    np.log(row["Col1"])
```
## <a name="epoch-computation"></a>Epoch 計算 
自從 Unix Epoch 以來的秒數 (假設 Col1 已經是日期)： 
```python
    row["Col1"] - datetime.datetime.utcfromtimestamp(0)).total_seconds()
```

## <a name="hash-a-column-value-into-a-new-column"></a>將資料行的值雜湊處理到新的資料行
```python
    import hashlib
    hash(row["MyColumnToHashCol1"])

```




