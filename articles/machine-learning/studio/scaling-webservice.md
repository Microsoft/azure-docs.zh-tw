---
title: 如何提高 Azure Machine Learning Web 服務的並行 | Microsoft Docs
description: 了解如何藉由新增其他端點來提高 Azure Machine Learning Web 服務的並行。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
keywords: azure 機器學習服務, web 服務, 作業化, 調整, 端點, 並行要求
ms.assetid: c2c51d7f-fd2d-4f03-bc51-bf47e6969296
ms.service: machine-learning
ms.component: studio
ms.devlang: NA
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/23/2017
ms.openlocfilehash: 2f950d93c0d923e20451eb1622dd4b1393f343a7
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835894"
---
# <a name="scaling-an-azure-machine-learning-web-service-by-adding-additional-endpoints"></a>藉由新增其他端點來調整 Azure Machine Learning Web 服務
> [!NOTE]
> 此主題描述適用於**傳統** Machine Learning Web 服務的技巧。 
> 
> 

根據預設，系統將每個發佈的 Web 服務設定為支援 20 個並行要求，而且最多可達 200 個並行要求。 Azure Machine Learning 會自動最佳化此設定，為您的 Web 服務提供最佳的效能，並忽略入口網站的值。 

如果您打算以超過「並行呼叫數上限」值 200 可支援的負載來呼叫 API，則應該在相同的 Web 服務上建立多個端點。 然後，您就可以將負載隨機分配給所有端點。

調整 Web 服務一件常見的工作。 一些調整理由包括為了支援超過 200 個並行要求、透過多個端點提高可用性，或為 Web 服務提供個別的端點。 您可以透過 [Azure Machine Learning Web 服務](https://services.azureml.net/)入口網站新增更多端點來擴大同一個 Web 服務的規模。

如需有關新增端點的詳細資訊，請參閱[建立端點](create-endpoint.md)。

請記住，如果您未以對應的高比例來呼叫 API，則使用較大的並行處理計數可能有害。 如果您將相對低的負載放在為高負載設定的 API，可能會看見延遲有零星的逾時及 (或) 突增情況。

通常在需要低度延遲的情況下，才會使用同步 API。 這裡所說的延遲意味著 API 完成一個要求所花費的時間，而未計入任何網路延遲的時間。 假設您有一個會延遲 50 毫秒的 API。 其節流層級為 [高] 且 [最大同時呼叫數目] 為 20，您必須每秒呼叫這個 API 20 * 1000 / 50 = 400 次，才能完全耗盡可用的容量。 再進一步延伸，假設會延遲 50 毫秒，則「並行呼叫數上限」200 可讓您每秒呼叫 API 4000 次。

<!--Image references-->
[1]: ./media/scaling-webservice/machlearn-1.png
[2]: ./media/scaling-webservice/machlearn-2.png
