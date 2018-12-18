---
title: Azure Machine Learning 服務的已知問題與疑難排解
description: 取得已知問題、因應措施與疑難排解清單
services: machine-learning
author: j-martens
ms.author: jmartens
ms.reviewer: mldocs
ms.service: machine-learning
ms.component: core
ms.topic: article
ms.date: 09/24/2018
ms.openlocfilehash: d84040dc440c373ae9bae6dbac7a95109a387ba7
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162741"
---
# <a name="known-issues-and-troubleshooting-azure-machine-learning-service"></a>已知問題與針對 Azure Machine Learning 服務進行疑難排解
 
此文章可協助您尋找並修正使用 Azure Machine Learning 服務時遇到的錯誤與失敗。 


## <a name="databricks"></a>Databricks

Databricks 與 Azure Machine Learning 問題。

1. Databricks 叢集建議：
   
   使用 Python 3 將 Azure Databricks 叢集建立為 v4.x。 建議使用高並行存取叢集。
 
1. 已安裝更多套件時，在 Databricks 上安裝 AML SDK 會失敗。

   有些套件 (例如 `psutil upgrade libs`) 會導致發生衝突。 若要避免發生安裝錯誤，請透過凍結 lib 版本來安裝套件。 這個問題與 Databricks 有關，而與 AML SDK 無關。 範例：
   ```python
   pstuil cryptography==1.5 pyopenssl==16.0.0 ipython=2.2.0
   ```

## <a name="gather-diagnostics-information"></a>蒐集診斷資訊
當您在尋求協助時，如果能夠提供診斷資訊，有時可能會相當有幫助。 以下是記錄檔的所在位置：

## <a name="resource-quotas"></a>資源配額

深入了解使用 Azure Machine Learning 時可能會遇到的[資源配額](how-to-manage-quotas.md)。

## <a name="get-more-support"></a>取得更多支援

您可以提交支援要求，並從支援小組、論壇等來源獲得協助。 [深入了解...](support-for-aml-services.md)
