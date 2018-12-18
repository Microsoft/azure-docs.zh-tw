---
title: 在 Azure Blob 容器、SQL Server 和 Hive 資料表中進行資料取樣 | Microsoft Docs
description: 如何探索儲存在各種 Azure 環境中的資料。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 80a9dfae-e3a6-4cfb-aecc-5701cfc7e39d
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 0efd754936b67611a747c6c5756de92443a937e4
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837994"
---
# <a name="heading"></a>在 Azure Blob 容器、SQL Server 和 Hive 資料表中進行資料取樣
本文件所連結的文章會說明如何對三個不同 Azure 位置中任一個位置所儲存的資料進行取樣：

* **Azure blob 儲存體資料** ，然後使用 Python 程式碼範例來進行取樣，這是對 Azure blob 儲存體資料進行取樣的方式。
* **SQL Server 資料** 進行取樣。 
* **Hive 資料表** 進行取樣。

以下**功能表**所連結的主題會說明如何從這其中的每一個 Azure 儲存體環境進行資料取樣。 

[!INCLUDE [cap-sample-data-selector](../../../includes/cap-sample-data-selector.md)]

這個取樣工作是 [Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/)中的一個步驟。

**為何要對資料進行取樣？**

如果您規劃分析的資料集很龐大，通常最好是對資料進行向下取樣，將資料縮減為更小但具代表性且更容易管理的大小。 這有助於資料了解、探索和功能工程。 它在 Cortana 分析程序中扮演的角色是能夠快速建立資料處理函式與機器學習服務模型的原型。

