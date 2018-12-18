---
title: 可供 Azure Machine Learning 資料準備使用的範例目的地/輸出 | Microsoft Docs
description: 本文件提供可用於 Azure Machine Learning 資料準備之自訂資料目的地/輸出的一組範例
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
ms.openlocfilehash: 3e827c56caa787ae9fc33119810eea0f268f0120
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958404"
---
# <a name="sample-of-destination-connections-python"></a>目的地連線的範例 (Python) 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


閱讀本附錄之前，請先參閱 [Python 擴充性概觀](data-prep-python-extensibility-overview.md)。


## <a name="write-to-excel"></a>寫入至 Excel 


寫入至 Excel 需要其他程式庫。 擴充性概觀中說明如何新增程式庫。 `openpyxl` 是您要新增的程式庫。

寫入至 Excel 之前，可能需要一些其他變更。 資料準備中使用的某些資料類型在部分目的地格式中不支援。 例如，如果 "Error" 物件存在，就無法將這些內容正確序列化至 Excel。 因此在嘗試寫入至 Excel 之前，需要「取代錯誤值」轉換，它會從任何資料行移除錯誤。

如果上述所有工作皆已完成，後續行會將資料資料表寫入 Excel 文件中的單一工作表。 新增轉換資料流程 (指令碼) 轉換。 然後在運算式區段中輸入下列程式碼。


### <a name="on-windows"></a>在 Windows 上 
```python
df.to_excel('c:\dev\data\Output\Customer.xlsx', sheet_name='Sheet1')
```

### <a name="on-macosos-x"></a>在 MacOS/OS X 上 ###
```python
df.to_excel('c:/dev/data/Output/Customer.xlsx', sheet_name='Sheet1')
```
