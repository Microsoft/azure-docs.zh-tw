---
title: 預測車輛健全狀態與駕駛習慣 - Azure | Microsoft Docs
description: 使用 Cortana Intelligence 具備的強大功能，取得關於車輛健全狀態與駕駛習慣的即時預測情資。
services: machine-learning
documentationcenter: ''
author: deguhath
ms.author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 09fad60b-2f48-488b-8a7e-47d1f969ec6f
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2018
ms.openlocfilehash: 02a12e917ed36367ffac1ac2e7a1fef1c6098ea7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985362"
---
# <a name="vehicle-telemetry-analytics-solution-playbook"></a>車輛遙測分析解決方案腳本
此功能表連結至此腳本的章節： 

[!INCLUDE [cap-vehicle-telemetry-playbook-selector](../../../includes/cap-vehicle-telemetry-playbook-selector.md)]

## <a name="overview"></a>概觀
超級電腦已步出實驗室，並停在車庫中。 這些電腦現在被放在包含無數感應器的尖端汽車中。 這些感應器讓電腦能夠每秒追蹤及監視數百萬個事件。 在 2020 年前，在上述新銳汽車中，大部分的車輛都將連線至網際網路。 使用此豐富資料可提供更好的安全性、可靠性和更佳的駕駛體驗。 Microsoft 透過 Cortana Intelligence 讓此夢想成真。

Cortana Intelligence 是完全受控的巨量資料與進階分析套件，您可用來將資料轉換成可採取的智慧行動。 Cortana Intelligence 車輛遙測分析解決方案範本示範汽車經銷商、汽車製造商和保險公司如何取得車輛健全狀態與駕駛習慣的即時和預測情資。

此解決方案是以 [lambda 架構模式](https://en.wikipedia.org/wiki/Lambda_architecture)實作，該模式完全展現 Cortana Intelligence 平台在即時批次處理方面的優異潛力。

## <a name="architecture"></a>架構
下圖說明車輛遙測分析解決方案架構：

![方案架構圖表](./media/cortana-analytics-playbook-vehicle-telemetry/fig1-vehicle-telemetry-annalytics-solution-architecture.png)


此解決方案包含下列 Cortana Intelligence 元件，並展示其整合功能：

* **Azure 事件中樞**可將數百萬計的車輛遙測事件擷取至 Azure。
* **Azure 串流分析**可提供關於車輛健全況的即時深入解析，同時以長期儲存的方式保存這些資料，供日後進行更豐富廣泛的批次分析之用。
* **Azure Machine Learning** 可即時偵測異常狀況，並使用批次處理來提供預測性深入解析。
* **Azure HDInsight** 可大規模轉換資料。
* **Azure Data Factory** 可執行批次處理管線的協調流程、排程、資源管理及監控工作。
* **Power BI** 為此解決方案提供具備即時資料與預測性分析視覺效果等豐富功能的儀表板。

此解決方案可存取兩種不同的資料來源： 

* **模擬車輛訊號和診斷**：針對指定時點的車輛狀態與駕駛模式，車輛遠程資訊服務模擬器可發出與其對應的診斷資訊和訊號。 
* **車輛目錄**：此參考資料集會將 VIN 編號對應至模型。

