---
title: 如何使用 Azure Machine Learning 資料準備執行 API 的深度指南 | Microsoft Docs
description: 本文件提供事先執行設計之資料來源和資料準備套件的詳細資料
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
ms.openlocfilehash: 5280ed4143ef2ee980300cebe234de7f2020a14f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968096"
---
# <a name="execute-data-sources-and-data-preparations-packages-from-python"></a>執行來自 Python 的資料來源和資料準備套件

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



若要在 Python 內使用 Azure Machine Learning 資料來源或 Azure Machine Learning 資料準備套件：

1. 移至專案中的 [資料] 索引標籤。

2. 用滑鼠右鍵按一下適當的來源。

3. 選擇 [產生資料存取碼檔案]。

這個動作會建立簡短的 Python 指令碼，可執行封裝並傳回資料框架。

## <a name="data-sources"></a>資料來源

`azureml.dataprep.datasource` 模組包含單一函式以執行資料來源，並傳回資料框架：`load_datasource(path, secrets=None, spark=None)`。
- `path` 是資料來源 (.dsource 檔案) 的路徑。
- `secrets` 是選擇性的字典，將索引鍵對應至祕密。
- `spark` 是選用的布林，指定是否要傳回 Spark 資料框架或 Pandas 資料框架。 根據預設，Azure Machine Learning Workbench 會根據內容決定要在執行階段傳回的資料框架種類。

## <a name="data-preparations-packages"></a>資料準備套件

`azureml.dataprep.package` 模組包含從資料準備套件執行資料流程的三個函式。

### <a name="execution-functions"></a>執行函式

- `submit(package_path, dataflow_idx=0, secrets=None, spark=None)` 送出指定的資料流程以供執行，但不會傳回資料框架。
- `run(package_path, dataflow_idx=0, secrets=None, spark=None)` 執行指定的資料流程，並以資料框架的方式傳回結果。
- `run_on_data(user_config, package_path, dataflow_idx=0, secrets=None, spark=None)` 根據記憶體內部資料來源執行指定的資料流程，並以資料框架的方式傳回結果。 `user_config` 引數是一種字典，將資料來源 (.dsource 檔案) 的絕對路徑對應至表示為其中一份清單的記憶體內部資料來源。

### <a name="common-arguments"></a>常見引數

- `package_path` 是資料準備套件 (.dprep 檔案) 的路徑。
- `dataflow_idx` 是待執行封裝中資料流程之以零為基礎的索引。 如果指定的資料流程參考其他資料流程或資料來源，也會一併執行。
- `secrets` 是選擇性的字典，將索引鍵對應至祕密。
- `spark` 是選用的布林，指定是否要傳回 Spark 資料框架或 Pandas 資料框架。 根據預設，Workbench 會根據內容決定要在執行階段傳回的資料框架種類。
