---
title: Azure Machine Learning 新增功能 | Microsoft Docs
description: Azure Machine Learning 中可用的新功能。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: ddc716ed-2615-4806-bf27-6c9a5662a7f2
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/28/2017
ms.openlocfilehash: 61bea7fde96b239a50ec25a702a73ecfb62ce717
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836067"
---
# <a name="whats-new-in-azure-machine-learning"></a>Azure Machine Learning 新增功能

### <a name="the-march-2017-release-of-microsoft-azure-machine-learning-updates-provides-the-following-feature"></a>2017 年 3 月發行的 Microsoft Azure Machine Learning 更新提供下列功能：



* Azure Machine Learning BES 作業專用的容量

    Machine Learning 批次集區處理會使用 [Azure Batch](../../batch/batch-technical-overview.md) 服務，提供客戶管理的 Azure Machine Learning 批次執行服務級別。 批次集區處理可讓您建立 Azure Batch 集區，以便提交批次作業，並讓它們以預期的方式執行。

    如需詳細資訊，請參閱[適用於 Machine Learning 作業的 Azure Batch 服務](dedicated-capacity-for-bes-jobs.md)。


### <a name="the-august-2016-release-of-microsoft-azure-machine-learning-updates-provide-the-following-features"></a>2016 年 8 月發行的 Microsoft Azure Machine Learning 更新提供下列功能︰
* 新的 [Microsoft Azure Machine Learning Web 服務](https://services.azureml.net/)入口網站可集中管理 Web 服務的所有層面，現在可以在此入口網站中管理傳統 Web 服務。    
  * 它提供 Web 服務 [使用量統計資料](manage-new-webservice.md)。
  * 使用範例資料來簡化測試 Azure Machine Learning 遠端要求呼叫。
  * 提供新的批次執行服務測試頁面，有範例資料和作業提交歷程記錄可用。
  * 提供更輕鬆的端點管理。

### <a name="the-july-2016-release-of-microsoft-azure-machine-learning-updates-provide-the-following-features"></a>2016 年 7 月發行的 Microsoft Azure Machine Learning 更新提供下列功能︰
* 現在 Web 服務是透過 [Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md) 介面管理為 Azure 資源，有下列增強功能︰
  * 有新的 [REST API](https://msdn.microsoft.com/library/azure/Dn950030.aspx) 可部署和管理以 Resource Manager 為基礎的 Web 服務。
  * 新的 [Microsoft Azure Machine Learning Web 服務](https://services.azureml.net/)入口網站可集中管理 Web 服務的所有層面。
* 使用以 Resource Manager 為基礎的 API，運用 Web 服務的 Resource Manager 資源提供者 ，加入以新的訂用帳戶為基礎、多區域 Web 服務的部署模型。
* 使用新的 Resource Manager RP 進行計費，引入新的 [定價方案](https://azure.microsoft.com/pricing/details/machine-learning/) 和方案管理功能。
  * 您現在可以[將您的 Web 服務部署到多個區域](how-to-deploy-to-multiple-regions.md)而不需要在每個區域中建立訂用帳戶。
* 提供 Web 服務 [使用量統計資料](manage-new-webservice.md)。
* 使用範例資料來簡化測試 Azure Machine Learning 遠端要求呼叫。
* 提供新的批次執行服務測試頁面，有範例資料和作業提交歷程記錄可用。

此外，Machine Learning Studio 已更新，可讓您部署到新式 Web 服務模型，或繼續部署到傳統 Web 服務模型。 

