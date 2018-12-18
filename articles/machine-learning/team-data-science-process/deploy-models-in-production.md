---
title: 在生產環境中部署模型 - Azure Machine Learning | Microsoft Docs
description: 如何將模型部署到生產環境，好讓這些模型能夠在進行業務決策時扮演主動角色。
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/30/2017
ms.author: deguhath
ms.openlocfilehash: 0505d58261ec32015f8847b710791249f87f049b
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34838287"
---
# <a name="deploy-models-in-production"></a>在生產環境中部署模型

生產環境部署可讓模型在企業中扮演主動角色。 從已部署模型的預測可用於商業決策。

## <a name="production-platforms"></a>生產環境平台
有許多方法和平台可將模型置入生產環境。 以下提供一些選項：


- [在 Azure Machine Learning 中的模型部署](../desktop-workbench/model-management-overview.md)
- [在 SQL-Server 中部署模型](https://docs.microsoft.com/sql/advanced-analytics/tutorials/sqldev-py6-operationalize-the-model)
- [Microsoft Machine Learning Server](https://docs.microsoft.com/sql/advanced-analytics/r/r-server-standalone)


>[!NOTE]
>在部署之前，必須確保模型計分的延遲夠低，以便在生產環境中使用。
>


>[!NOTE]
>如需使用 Azure Machine Learning Studio 進行部署，請參閱[部署 Azure Machine Learning Web 服務](../studio/publish-a-machine-learning-web-service.md)。
>

## <a name="ab-testing"></a>A/B 測試
多個模型在生產環境中時，執行 [A/B 測試](https://en.wikipedia.org/wiki/A/B_testing)來比較模型的效能會很有用。 

 
## <a name="next-steps"></a>後續步驟

也會提供逐步解說，說明**特定案例**之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)文章中會列出這些逐步解說以及簡短說明的連結。 這些逐步解說說明如何將雲端、內部部署工具及服務組合成工作流程或管線，以建立智慧型應用程式。 
 


