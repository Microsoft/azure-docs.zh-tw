---
title: 自動調整和 App Service 環境 v1
description: 自動調整和 App Service 環境
services: app-service
documentationcenter: ''
author: btardif
manager: erikre
editor: ''
ms.assetid: c23af2d8-d370-4b1f-9b3e-8782321ddccb
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.openlocfilehash: 0feb6e4862f643c35a58c0321181bdda22b032e9
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22987563"
---
# <a name="autoscaling-and-app-service-environment-v1"></a>自動調整和 App Service 環境 v1

> [!NOTE]
> 這篇文章是關於 App Service 環境 v1。  有較新版本的 App Service 環境，更易於使用，並且可以在功能更強大的基礎結構上執行。 若要深入了解新版本，請從 [App Service 環境簡介](intro.md)開始。
> 

Azure App Service 環境支援「自動調整」 。 您可以根據度量或排程自動調整個別的背景工作集區。

![背景工作集區的自動調整選項。][intro]

自動調整可將您的資源使用量最佳化，方法是透過自動增加或縮減 App Service 環境以符合您的預算和或負載設定檔。

## <a name="configure-worker-pool-autoscale"></a>設定背景工作集區自動調整
您可以從 [背景工作集區]  的 [設定] 索引標籤存取自動調整功能。

![背景工作集區的 [設定] 索引標籤。][settings-scale]

該處的介面應相當熟悉，因為這是和您調整 App Service 方案時所看到的相同的體驗。 

![手動調整設定。][scale-manual]

您也可以設定自動調整設定檔。

![自動調整設定。][scale-profile]

自動調整設定檔對於設定您的規模的限制很實用。 如此一來，您可以同時藉由設定下限調整值 (1) 獲得一致的效能體驗和藉由設定上限 (2) 獲得可預測的資本支出。

![設定檔中的調整設定。][scale-profile2]

在定義設定檔之後，您可以新增自動調整規則，以相應增加或減少設定檔所定義的範圍內背景工作集區中的執行個體數目。 自動調整規則是以度量為基礎。

![調整規則。][scale-rule]

 任何背景工作集區或前端度量均可用來定義自動調整規則。 這些度量是您可以在資源刀鋒視窗圖形中監視或設定警示的相同度量。

## <a name="autoscale-example"></a>自動調整範例
App Service 環境的自動調整可使用逐步解說一個案例來加以說明。

本文說明設定自動調整時所有必要的考量。 本文將引導您進行當您納入 App Service 環境所裝載的自動調整 App Service 環境時派上用場的所有互動。

### <a name="scenario-introduction"></a>案例簡介
Frank 是企業的系統管理員，他已將所負責的部分工作量移轉至 App Service 環境。

App Service 環境設為以如下方式手動調整：

* **前端：** 3
* **背景工作集區 1**：10
* **背景工作集區 2**：5
* **背景工作集區 3**：5

背景工作集區 1 是用於生產工作負載，而背景工作集區 2 和背景工作集區 3 用於品質保證 (QA) 及開發工作負載。

品管和開發人員的 App Service 方案設定為手動調整。 生產 App Service 方案則設為自動調整以處理負載和流量的變化。

Frank 非常熟悉應用程式。 他知道負載尖峰時間是上午 9:00 到下午 6:00，因為這是員工在辦公室時會使用的企業營運 (LOB) 應用程式。 之後當使用者下班時，使用量便會下降。 非尖峰時間仍會有一些負載，因為使用者可以使用其行動裝置或家用電腦從遠端存取應用程式。 生產 App Service 方案已利用下列規則設定為根據 CPU 使用量自動調整：

![LOB 應用程式的特定設定。][asp-scale]

| **自動調整設定檔 - 工作日 - App Service 方案** | **自動調整設定檔 - 週末 - App Service 方案** |
| --- | --- |
| **名稱：** 工作日設定檔 |**名稱：** 週末設定檔 |
| **調整規模：** 排程和效能規則 |**調整規模：** 排程和效能規則 |
| **設定檔：** 工作日 |**設定檔：** 週末 |
| **類型：** 循環 |**類型：** 循環 |
| **目標範圍：** 5 到 20 個執行個體 |**目標範圍：** 3 到 10 個執行個體 |
| **星期幾：** 星期一、星期二、星期三、星期四、星期五 |**星期幾：** 星期六、星期日 |
| **開始時間︰** 上午 9:00 |**開始時間︰** 上午 9:00 |
| **時區：** UTC-08 |**時區：** UTC-08 |
|  | |
| **自動調整規則 (相應增加)** |**自動調整規則 (相應增加)** |
| **資源：** 生產環境 (App Service 環境) |**資源：** 生產環境 (App Service 環境) |
| **度量：** CPU % |**度量：** CPU % |
| **作業：** 大於 60% |**作業：** 大於 80% |
| **持續時間：** 5 分鐘 |**持續時間：** 10 分鐘 |
| **時間彙總：** 平均 |**時間彙總：** 平均 |
| **動作：** 計數增加 2 |**動作：** 計數增加 1 |
| **冷卻時間 (分鐘)：** 15 |**冷卻時間 (分鐘)：** 20 |
|  | |
| **自動調整規則 (相應減少)** |**自動調整規則 (相應減少)** |
| **資源：** 生產環境 (App Service 環境) |**資源：** 生產環境 (App Service 環境) |
| **度量：** CPU % |**度量：** CPU % |
| **作業：** 少於 30 % |**作業：** 少於 20 % |
| **持續時間：** 10 分鐘 |**持續時間：** 15 分鐘 |
| **時間彙總：** 平均 |**時間彙總：** 平均 |
| **動作：** 計數減少 1 |**動作：** 計數減少 1 |
| **冷卻時間 (分鐘)：** 20 |**冷卻時間 (分鐘)：** 10 |

### <a name="app-service-plan-inflation-rate"></a>App Service 方案擴大率
設定為自動調整的 App Service 方案，就會以每小時最大的速率進行。 此速率可以根據自動調整規則上提供的值計算。

了解及計算「App Service 方案擴大率」  對 App Service 環境自動調整來說很重要，因為背景工作集區的調整變更並非瞬間完成。

App Service 方案擴大率的計算方式如下：

![App Service 方案擴大率計算。][ASP-Inflation]

根據生產 App Service 方案的工作日設定檔的「自動調整 - 相應增加」規則：

![根據「自動調整 - 相應增加」規則的工作日 App Service 方案擴大率。][Equation1]

如果是生產 App Service 方案的週末設定檔的「自動調整 - 相應增加」規則，公式會解析為：

![根據「自動調整 - 相應增加」規則的週末 App Service 方案擴大率。][Equation2]

此值也可以針對相應減少作業計算。

根據生產 App Service 方案的工作日設定檔的「自動調整 - 相應減少」規則，這看起來會是如下：

![根據「自動調整 - 相應減少」規則的工作日 App Service 方案擴大率。][Equation3]

如果是生產 App Service 方案的週末設定檔的「自動調整 - 相應減少」規則，公式會解析為：  

![根據「自動調整 - 相應減少」規則的週末 App Service 方案擴大率。][Equation4]

生產 App Service 方案能在週間以每小時 8 個執行個體，以及在週末期間以每小時 4 個執行個體的最大速率成長。 它能夠在週間以每小時 4 個執行個體，以及在週末期間以每小時 6 個執行個體的最大速率釋出執行個體。

如果在背景工作集區中裝載多個 App Service 方案，則必須將「總擴大率」  計算為裝載在該背景工作集區中的所有 App Service 方案的擴大率總和。

![裝載於背景工作集區中的多個 App Service 方案的總擴大率計算。][ASP-Total-Inflation]

### <a name="use-the-app-service-plan-inflation-rate-to-define-worker-pool-autoscale-rules"></a>使用 App Service 方案擴大率來定義背景工作集區自動調整規則
裝載 App Service 方案且設定為自動調整的背景工作集區必須配置容量緩衝區。 緩衝區可在需要時允許自動調整作業增加和縮減 App Service 方案。 最小緩衝區會計算為總計 App Service 方案擴大率。

由於 App Service 環境調整作業需要一些時間才能套用，任何變更應該考量當調整正在進行時可能會發生的進一步需求變更。 為了因應此延遲，建議您使用計算的總計 App Service 方案擴大率作為對每個自動調整作業加入的執行個體數目下限。

使用此資訊，Frank 可以定義下列的自動調整設定檔和規則：

![LOB 範例的自動調整設定檔規則。][Worker-Pool-Scale]

| **自動調整設定檔 - 工作日** | **自動調整設定檔 - 週末** |
| --- | --- |
| **名稱：** 工作日設定檔 |**名稱：** 週末設定檔 |
| **調整規模：** 排程和效能規則 |**調整規模：** 排程和效能規則 |
| **設定檔：** 工作日 |**設定檔：** 週末 |
| **類型：** 循環 |**類型：** 循環 |
| **目標範圍：** 13 到 25 個執行個體 |**目標範圍：** 6 到 15 個執行個體 |
| **星期幾：** 星期一、星期二、星期三、星期四、星期五 |**星期幾：** 星期六、星期日 |
| **開始時間︰** 上午 7:00 |**開始時間︰** 上午 9:00 |
| **時區：** UTC-08 |**時區：** UTC-08 |
|  | |
| **自動調整規則 (相應增加)** |**自動調整規則 (相應增加)** |
| **資源：** 背景工作集區 1 |**資源：** 背景工作集區 1 |
| **度量：** WorkersAvailable |**度量：** WorkersAvailable |
| **作業：** 少於 8 |**作業：** 少於 3 |
| **持續時間：** 20 分鐘 |**持續時間：** 30 分鐘 |
| **時間彙總：** 平均 |**時間彙總：** 平均 |
| **動作：** 計數增加 8 |**動作：** 計數增加 3 |
| **冷卻時間 (分鐘)：** 180 |**冷卻時間 (分鐘)：** 180 |
|  | |
| **自動調整規則 (相應減少)** |**自動調整規則 (相應減少)** |
| **資源：** 背景工作集區 1 |**資源：** 背景工作集區 1 |
| **度量：** WorkersAvailable |**度量：** WorkersAvailable |
| **作業：** 大於 8 |**作業：** 大於 3 |
| **持續時間：** 20 分鐘 |**持續時間：** 15 分鐘 |
| **時間彙總：** 平均 |**時間彙總：** 平均 |
| **動作：** 計數減少 2 |**動作：** 計數減少 3 |
| **冷卻時間 (分鐘)：** 120 |**冷卻時間 (分鐘)：** 120 |

設定檔中定義的目標範圍的計算方式是由設定檔中為 App Service 方案定義的最小執行個體 + 緩衝區。

最大值範圍是裝載於背景工作集區的所有 App Service 方案最大範圍的總和。

相應增加規則的增加計數應該設定為至少是 App Service 方案擴大率的一倍進行相應增加。

減少計數可以調整成介於 App Service 方案擴大率的一半或一倍之間進行相應減少。

### <a name="autoscale-for-front-end-pool"></a>前端集區的自動調整
前端自動調整規則比背景工作集區規則簡單。 您主要應該  
確定測量的持續時間，而冷卻計時器會將 App Service 方案的調整作業不是瞬間完成的事實納入考量。

在此案例中，Frank 知道當前端達到 80% 的 CPU 使用量時錯誤率會增加，他設定自動調整規則來增加執行個體，如下所示：

![前端集區的自動調整設定。][Front-End-Scale]

| **自動調整設定檔 - 前端** |
| --- |
| **名稱：** 自動調整 - 前端 |
| **調整規模：** 排程和效能規則 |
| **設定檔：** 每日 |
| **類型：** 循環 |
| **目標範圍：** 3 到 10 個執行個體 |
| **星期幾：** 每日 |
| **開始時間︰** 上午 9:00 |
| **時區：** UTC-08 |
|  |
| **自動調整規則 (相應增加)** |
| **資源：** 前端集區 |
| **度量：** CPU % |
| **作業：** 大於 60% |
| **持續時間：** 20 分鐘 |
| **時間彙總：** 平均 |
| **動作：** 計數增加 3 |
| **冷卻時間 (分鐘)：** 120 |
|  |
| **自動調整規則 (相應減少)** |
| **資源：** 背景工作集區 1 |
| **度量：** CPU % |
| **作業：** 少於 30 % |
| **持續時間：** 20 分鐘 |
| **時間彙總：** 平均 |
| **動作：** 計數減少 3 |
| **冷卻時間 (分鐘)：** 120 |

<!-- IMAGES -->
[intro]: ./media/app-service-environment-auto-scale/introduction.png
[settings-scale]: ./media/app-service-environment-auto-scale/settings-scale.png
[scale-manual]: ./media/app-service-environment-auto-scale/scale-manual.png
[scale-profile]: ./media/app-service-environment-auto-scale/scale-profile.png
[scale-profile2]: ./media/app-service-environment-auto-scale/scale-profile-2.png
[scale-rule]: ./media/app-service-environment-auto-scale/scale-rule.png
[asp-scale]: ./media/app-service-environment-auto-scale/asp-scale.png
[ASP-Inflation]: ./media/app-service-environment-auto-scale/asp-inflation-rate.png
[Equation1]: ./media/app-service-environment-auto-scale/equation1.png
[Equation2]: ./media/app-service-environment-auto-scale/equation2.png
[Equation3]: ./media/app-service-environment-auto-scale/equation3.png
[Equation4]: ./media/app-service-environment-auto-scale/equation4.png
[ASP-Total-Inflation]: ./media/app-service-environment-auto-scale/asp-total-inflation-rate.png
[Worker-Pool-Scale]: ./media/app-service-environment-auto-scale/wp-scale.png
[Front-End-Scale]: ./media/app-service-environment-auto-scale/fe-scale.png
