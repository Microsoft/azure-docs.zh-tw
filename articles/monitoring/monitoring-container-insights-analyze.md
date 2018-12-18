---
title: 使用適用於容器的 Azure 監視器來監視 AKS 叢集效能 | Microsoft Docs
description: 本文說明如何使用適用於容器的 Azure 監視器，來檢視和分析效能與記錄資料。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/14/2018
ms.author: magoedte
ms.openlocfilehash: 6df7d42bc291713a815cac9f719f53136ed35b19
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956650"
---
## <a name="understand-aks-cluster-performance-with-azure-monitor-for-containers"></a>使用適用於容器的 Azure 監視器來了解 AKS 叢集效能
檢視 Azure Kubernetes Service (AKS) 叢集的效能，可以使用適用於容器的 Azure 監視器從兩個檢視方塊中觀測到：直接從 AKS 叢集，或者從 Azure 監視器檢視訂用帳戶中的所有 AKS 叢集。 

本文將協助您了解這兩個檢視方塊之間的體驗，以及如何快速地評估、調查和解決所偵測到的問題。

如需啟用適用於容器的 Azure 監視器相關資訊，請參閱[將適用於容器的 Azure 監視器上線](monitoring-container-insights-onboard.md)。

Azure 監視器提供多叢集檢視，來顯示您訂用帳戶中跨資源群組部署的所有受監視 AKS 叢集的健康狀態，以及未受到解決方案監視之 AKS 叢集的健康狀態。  您可以立即了解叢集健康情況，並從這裡向下切入到節點和控制器效能頁面，或者您可以瀏覽以查看叢集的效能圖表。  針對已探索到且識別為未受監視的 AKS 叢集，您隨時都可選擇針對該叢集啟用監視。  

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站
登入 [Azure 入口網站](https://portal.azure.com)。 

## <a name="multi-cluster-view-from-azure-monitor"></a>Azure 監視器中的多叢集檢視 
若要檢視已部署的所有 AKS 叢集健康狀態，請從 Azure 入口網站的左側窗格選取 [監視]。  在 [Insights] 區段下方，選取 [容器 (預覽)]。  

![Azure 監視器多叢集儀表板範例](./media/monitoring-container-insights-analyze/azmon-containers-multiview.png)

在 [受監視的叢集] 索引標籤上，您可以了解下列內容：

1. 有多少個叢集處於重大或狀況不良狀態，以及有多少個叢集的狀況良好或未報告 (稱為未知狀態)？
2. 每個叢集中可部署多少個節點、使用者和系統 Pod。  

以下為已定義的健康狀態： 

1. **狀況良好**：針對 VM 並未偵測到任何問題，而其會視需要運作。 
2. **重大**：已偵測到一或多個重大問題，您必須解決這類問題，才能如預期般地還原正常的作業狀態。
3. **警告**：已偵測到一或多個問題，您必須解決這類問題，或者健康情況可能會變成「重大」。
4. **未知**：如果服務無法建立與節點或 Pod 的連線，狀態就會變更為未知狀態。

健康狀態會將整體叢集狀態計算為這三個狀態中「最差」的狀態，但有一個例外狀況：如果這三個狀態中有任一個為「未知」，則整體叢集狀態將會顯示**未知**。  

下表提供計算明細，其會在多叢集檢視上控制受監視叢集的健康狀態。

| |狀態 |可用性 |  
|-------|-------|-----------------|  
|**使用者 Pod**| | |  
| |Healthy |100% |  
| |警告 |90 - 99% |  
| |重要 |<90% |  
| |不明 |如果未在過去 30 分鐘內報告 |  
|**系統 Pod**| | |  
| |Healthy |100% |
| |警告 |N/A |
| |重要 |<100% |
| |不明 |如果未在過去 30 分鐘內報告 |
|**Node** | | |
| |Healthy |>85% |
| |警告 |60 - 84% |
| |重要 |<60% |
| |不明 |如果未在過去 30 分鐘內報告 |

從叢集清單中，您可按一下叢集名稱以向下切入到 [叢集] 頁面、在該特定叢集的 [節點] 資料行中按一下節點的彙總以向下切入到 [節點] 效能頁面，或者按一下 [使用者 Pod] 或 [系統 Pod] 資料行的彙總以向下切入到 [控制器] 效能頁面。   

## <a name="view-performance-directly-from-an-aks-cluster"></a>直接從 AKS 叢集檢視效能
若要存取適用於容器的 Azure 監視器，可從左側窗格中選取 [Insights (預覽)]，直接從 AKS 叢集存取。 關於 AKS 叢集的檢視資訊，會組織成四個檢視方塊：

- 叢集
- 節點 
- Controllers  
- 容器

當您按一下 [Insights (預覽)] 時所開啟的預設頁面為 [叢集]，其包含四個效能折線圖來顯示您叢集的關鍵效能計量。 

![[叢集] 索引標籤上的範例效能圖表](./media/monitoring-container-insights-analyze/containers-cluster-perfview.png)

效能圖表會顯示四個效能計量：

- **節點 CPU 使用率&nbsp;%**：整個叢集中 CPU 使用率的彙總檢視方塊。 您可以從圖表上方的百分位數選取器中，個別或合併選取 [Avg]、[Min]、[Max]、[50th]、[90th] 和 [95th]，來篩選時間範圍的結果。 
- **節點記憶體使用率&nbsp;%**：整個叢集中記憶體使用率的彙總檢視方塊。 您可以從圖表上方的百分位數選取器中，個別或合併選取 [Avg]、[Min]、[Max]、[50th]、[90th] 和 [95th]，來篩選時間範圍的結果。 
- **節點計數**：Kubernetes 中的節點計數和狀態。 所表示的叢集節點狀態為 [所有]、[就緒] 和 [尚未就緒]，且可以在圖表上方的選取器中進行個別篩選或結合。 
- **活動 Pod 計數**：Kubernetes 中的 Pod 計數和狀態。 所表示的叢集節點狀態為 [所有]、[擱置]、[執行中] 和 [未知]，且可以在圖表上方的選取器中進行個別篩選或結合。 

當您切換至 [節點]、[控制站] 和 [容器] 索引標籤時，頁面右邊會自動顯示屬性窗格。  它會顯示所選項目的屬性，包括您為了組織 Kubernetes 物件而定義的標籤。 按一下窗格中的 **>>** 連結，以檢視\隱藏此窗格。  

![範例 Kubernetes 觀點屬性窗格](./media/monitoring-container-insights-analyze/perspectives-preview-pane-01.png)

當您展開階層中的物件時，屬性窗格會根據所選的物件更新。 在此窗格中，您也可以按一下窗格頂端的 [檢視 Kubernetes 事件記錄] 連結，以檢視已預先定義記錄搜尋的 Kubernetes 事件。 如需有關檢視 Kubernetes 記錄資料的詳細資訊，請參閱[搜尋記錄來分析資料](#search-logs-to-analyze-data)。

使用頁面頂端的 [+ 新增篩選器] 選項，依**服務**、**節點**或**命名空間**來篩選檢視的結果，當您選取篩選範圍之後，接著可選取 [選取值] 欄位中所示的其中一個值。  設定篩選器之後，就會在檢視 AKS 叢集的任一個檢視方塊時全域套用。  此公式僅支援等號。  您可以在第一個篩選器上方新增其他篩選器，進一步縮小您的結果。  例如，如果您依**節點**指定篩選器，則您的第二個篩選器只允許您選取 [服務] 或 [命名空間]。  

![使用篩選器來縮小結果的範例](./media/monitoring-container-insights-analyze/add-filter-option-01.png)

在一個索引標籤中指定篩選器會持續在您選取另一個篩選器時套用，並在您按一下指定篩選器旁邊的 **x** 符號之後加以刪除。   

切換至 [節點] 索引標籤，而資料列階層會依循 Kubernetes 物件模型，從您叢集中的節點開始。 展開節點後，您便可以檢視一或多個在節點上執行的 Pod。 若有超過一個容器被分組至 Pod，它們會以最後一個資料列的形式顯示於階層中。 當主機有處理器或記憶體的壓力時，您也可檢視主機上有多少非 Pod 相關的工作負載正在執行。

![效能檢視中的範例 Kubernetes 節點階層](./media/monitoring-container-insights-analyze/containers-nodes-view.png)

您可以從頁面頂端選取控制器或容器，然後檢閱那些物件的狀態與資源使用率。  如果您想要改為檢閱記憶體使用率，請在 [計量] 下拉式清單中選取 [記憶體 RSS] 或 [記憶體工作集]。 只有 Kubernetes 1.8 版和更新版本才支援**記憶體 RSS**。 否則，您會以 *NaN&nbsp;%* 的形式檢視 **Min&nbsp;%** 的值，這是代表未定義或無法顯示之值的數值資料類型值。 

![容器節點效能檢視](./media/monitoring-container-insights-analyze/containers-node-metric-dropdown.png)

根據預設，效能資料是以過去六小時的資料為基礎，但您可以使用左上方的 [時間範圍] 選項來變更時間長度。 您也可以在百分位數選取器中選取 [Avg]、[Min]、[Max]、[50th]、[90th] 和 [95th]，來篩選時間範圍內的結果。 

![資料篩選的百分位數選取範圍](./media/monitoring-container-insights-analyze/containers-metric-percentile-filter.png)

在下列範例中，請注意節點 *aks-nodepool1-*，**容器**的值為 14，這是已部署容器總數的彙總。

![每個節點的彙總容器數範例](./media/monitoring-container-insights-analyze/containers-nodes-containerstotal.png)

這可幫助您快速識別叢集中節點之間的容器是否出現不平衡的情況。 

當您檢視節點時所顯示的資訊已詳述於下表：

| 欄 | 說明 | 
|--------|-------------|
| 名稱 | 主機的名稱。 |
| 狀態 | 節點狀態的 Kubernetes 檢視。 |
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 根據選取期間內百分位數的平均節點百分比。 |
| Avg、Min、Max、50th、90th | 根據所選取時間的期間內百分位數的平均節點實際值。 從為節點設定的 CPU/記憶體限制測量所得的平均值；對於 Pod 與容器而言，則是主機所報告的平均值。 |
| 容器 | 容器的數目。 |
| Uptime | 呈現自節點啟動或重新啟動以來經過的時間。 |
| Controllers | 僅適用於容器與 Pod。 顯示它位在哪個控制器之中。 並非所有 Pod 都位在控制器之中，因此有些可能會顯示 **N/A**。 | 
| Trend Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 代表控制器百分位數計量百分比的橫條圖趨勢。 |

在選取器中，選取 [控制器]。

![選取控制器檢視](./media/monitoring-container-insights-analyze/containers-controllers-tab.png)

您可在這裡檢視控制器的效能健康情況。

![<Name> 控制器效能檢視](./media/monitoring-container-insights-analyze/containers-controllers-view.png)

資料列階層會以控制器開始並展開控制器。 您會檢視一或多個容器。 展開 Pod，最後一個資料夾會顯示分組至 Pod 的容器。  

在您檢視控制器時所顯示的資訊已詳述於下表：

| 欄 | 說明 | 
|--------|-------------|
| 名稱 | 控制器的名稱。|
| 狀態 | 容器的彙總狀態，就是當其完成執行後的狀態，例如 [確定]、[終止]、[失敗]、[停止] 或 [暫停]。 如果容器在執行中，但狀態卻未正確呈現或未由代理程式擷取，而且超過 30 分鐘都沒有回應時，則狀態為 [未知]。 下表中提供狀態圖示的其他詳細資料。|
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 針對選取計量和百分位數之每個實體的平均百分比彙總平均值。 |
| Avg、Min、Max、50th、90th  | 容器針對選取百分位數的平均 CPU millicore 或記憶體效能彙總。 平均值是從為 Pod 設定的 CPU/記憶體限制測量所得。 |
| 容器 | 該控制器或 Pod 的容器總數。 |
| Restarts | 容器重新啟動計數的匯總。 |
| Uptime | 代表自容器啟動以來經過的時間。 |
| 節點 | 僅適用於容器與 Pod。 顯示它位在哪個控制器之中。 | 
| Trend Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;%| 代表控制器百分位數計量百分比的橫條圖趨勢。 |

[狀態] 欄位中的圖示會指出容器的線上狀態：
 
| 圖示 | 狀態 | 
|--------|-------------|
| ![準備好執行的狀態圖示](./media/monitoring-container-insights-analyze/containers-ready-icon.png) | 執行中 (就緒)|
| ![等候中或已暫停狀態圖示](./media/monitoring-container-insights-analyze/containers-waiting-icon.png) | 等候中或已暫停|
| ![上次回報的執行狀態圖示](./media/monitoring-container-insights-analyze/containers-grey-icon.png) | 上次回報的執行，但尚未回應超過 30 分鐘|
| ![成功狀態圖示](./media/monitoring-container-insights-analyze/containers-green-icon.png) | 已成功停止或無法停止|

狀態圖示會根據 Pod 所提供的功能顯示計數。 它會顯示較差的兩個狀態，且當您停留在狀態上時，會顯示容器中所有 Pod 的彙總狀態。 如果沒有就緒狀態，則狀態值會顯示 **(0)**。 

在選取器中，選取 [容器]。

![選取容器檢視](./media/monitoring-container-insights-analyze/containers-containers-tab.png)

您可在這裡檢視 Azure Kubernetes 容器的效能健康情況。  

![<Name> 控制器效能檢視](./media/monitoring-container-insights-analyze/containers-containers-view.png)

在您檢視容器時所顯示的資訊已詳述於下表：

| 欄 | 說明 | 
|--------|-------------|
| 名稱 | 控制器的名稱。|
| 狀態 | 容器的狀態，若有的話。 下一個表格會提供狀態圖示的其他詳細資料。|
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 針對選取計量和百分位數之每個實體的平均百分比彙總。 |
| Avg、Min、Max、50th、90th  | 彙總容器所選百分位數的平均 CPU millicore 或記憶體效能。 平均值是從為 Pod 設定的 CPU/記憶體限制測量所得。 |
| Pod | Pod 所在的容器。| 
| 節點 |  容器所在的節點。 | 
| Restarts | 代表自容器啟動以來經過的時間。 |
| Uptime | 代表自容器啟動或重新啟動以來經過的時間。 |
| Trend Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 代表容器平均計量百分比的橫條圖趨勢。 |

[狀態] 欄位中的圖示會指出 Pod 的線上狀態，如下表所述：
 
| 圖示 | 狀態 |  
|--------|-------------|  
| ![準備好執行的狀態圖示](./media/monitoring-container-insights-analyze/containers-ready-icon.png) | 執行中 (就緒)|  
| ![等候中或已暫停狀態圖示](./media/monitoring-container-insights-analyze/containers-waiting-icon.png) | 等候中或已暫停|  
| ![上次回報的執行狀態圖示](./media/monitoring-container-insights-analyze/containers-grey-icon.png) | 上次回報的執行，但未回應的時間超過 30 分鐘|  
| ![終止的狀態圖示](./media/monitoring-container-insights-analyze/containers-terminated-icon.png) | 已成功停止或無法停止|  
| ![失敗狀態圖示](./media/monitoring-container-insights-analyze/containers-failed-icon.png) | 失敗狀態 |  

## <a name="container-data-collection-details"></a>容器資料收集詳細資料
容器深入解析會從容器主機和容器收集各種效能計量和記錄資料。 每隔三分鐘會收集一次資料。

### <a name="container-records"></a>容器資料列

下表顯示適用於容器的 Azure 監視器所收集的記錄範例，以及記錄搜尋結果中所顯示的資料類型：

| 資料類型 | 記錄檔搜尋中的資料類型 | 欄位 |
| --- | --- | --- |
| 主機和容器的效能 | `Perf` | Computer、ObjectName、CounterName &#40;%Processor Time、Disk Reads MB、Disk Writes MB、Memory Usage MB、Network Receive Bytes、Network Send Bytes、Processor Usage sec、Network&#41;、CounterValue、TimeGenerated、CounterPath、SourceSystem |
| 容器清查 | `ContainerInventory` | TimeGenerated、Computer、container name、ContainerHostname、Image、ImageTag、ContainerState、ExitCode、EnvironmentVar、Command、CreatedTime、StartedTime、FinishedTime、SourceSystem、ContainerID、ImageID |
| 容器映像清查 | `ContainerImageInventory` | TimeGenerated、Computer、Image、ImageTag、ImageSize、VirtualSize、Running、Paused、Stopped、Failed、SourceSystem、ImageID、TotalContainer |
| 容器記錄檔 | `ContainerLog` | TimeGenerated、Computer、image ID、container name、LogEntrySource、LogEntry、SourceSystem、ContainerID |
| 容器服務記錄檔 | `ContainerServiceLog`  | TimeGenerated、Computer、TimeOfCommand、Image、Command、SourceSystem、ContainerID |
| 容器節點清查 | `ContainerNodeInventory_CL`| TimeGenerated、Computer、ClassName_s、DockerVersion_s、OperatingSystem_s、Volume_s、Network_s、NodeRole_s、OrchestratorType_s、InstanceID_g、SourceSystem|
| 容器流程 | `ContainerProcess_CL` | TimeGenerated、Computer、Pod_s、Namespace_s、ClassName_s、InstanceID_s、Uid_s、PID_s、PPID_s、C_s、STIME_s、Tty_s、TIME_s、Cmd_s、Id_s、Name_s、SourceSystem |
| 清查 Kubernetes 叢集中的 Pod | `KubePodInventory` | TimeGenerated、Computer、ClusterId、ContainerCreationTimeStamp、PodUid、PodCreationTimeStamp、ContainerRestartCount、PodRestartCount、PodStartTime、ContainerStartTime、ServiceName、ControllerKind、ControllerName、ContainerStatus、ContainerID、ContainerName、Name、PodLabel、Namespace、PodStatus、ClusterName、PodIp、SourceSystem |
| 清查 Kubernetes 叢集中的節點部分 | `KubeNodeInventory` | TimeGenerated、Computer、ClusterName、ClusterId、LastTransitionTimeReady、Labels、Status、KubeletVersion、KubeProxyVersion、CreationTimeStamp、SourceSystem | 
| Kubernetes 事件 | `KubeEvents_CL` | TimeGenerated、Computer、ClusterId_s、FirstSeen_t、LastSeen_t、Count_d、ObjectKind_s、Namespace_s、Name_s、Reason_s、Type_s、TimeGenerated_s、SourceComponent_s、ClusterName_s、Message、SourceSystem | 
| Kubernetes 叢集中的服務 | `KubeServices_CL` | TimeGenerated、ServiceName_s、Namespace_s、SelectorLabels_s、ClusterId_s、ClusterName_s、ClusterIP_s、ServiceType_s、SourceSystem | 
| Kubernetes 叢集節點部分的效能計量 | Perf &#124; where ObjectName == “K8SNode” | Computer、ObjectName、CounterName &#40;cpuUsageNanoCores、memoryWorkingSetBytes、memoryRssBytes、networkRxBytes、networkTxBytes、restartTimeEpoch、networkRxBytesPerSec、networkTxBytesPerSec、cpuAllocatableNanoCores、memoryAllocatableBytes、cpuCapacityNanoCores、memoryCapacityBytes&#41;、CounterValue、TimeGenerated、CounterPath、SourceSystem | 
| Kubernetes 叢集容器部分的效能計量 | Perf &#124; where ObjectName == “K8SContainer” | CounterName &#40;cpuUsageNanoCores、memoryWorkingSetBytes、memoryRssBytes、restartTimeEpoch、cpuRequestNanoCores、memoryRequestBytes、cpuLimitNanoCores、memoryLimitBytes&#41;、CounterValue、TimeGenerated、CounterPath、SourceSystem | 

## <a name="search-logs-to-analyze-data"></a>搜尋記錄來分析資料
Log Analytics 可協助您找出趨勢、診斷瓶頸、預測或讓將資料相互關聯，協助您判斷目前叢集設定是否以最佳方式運作。 已為您提供可立即開始使用，或自訂以您想要的方式傳回資訊的預先定義記錄搜尋。 

您可以在預覽窗格中選取 [檢視 Kubernetes 事件記錄] 或 [檢視容器記錄] 選項，以在工作區中對資料執行互動式分析。 [記錄搜尋] 頁面會出現在您所在 Azure 入口網站頁面的右邊。

![在 Log Analytics 中分析資料](./media/monitoring-container-insights-analyze/container-health-log-search-example.png)   

轉送至 Log Analytics 的容器記錄輸出為 STDOUT 與 STDERR。 因為 Azure 監視器會監視 Azure 受控 Kubernetes (AKS)，所以，由於所產生的資料量很大，因此今天不會收集 Kube 系統。 

### <a name="example-log-search-queries"></a>範例記錄檔搜尋查詢
從一或兩個範例開始建置查詢，然後加以修改以滿足您的需求，通常很實用。 若要取得建置更進階查詢的協助，您可以試驗下列範例查詢：

| 查詢 | 說明 | 
|-------|-------------|
| ContainerInventory<br> &#124; project Computer, Name, Image, ImageTag, ContainerState, CreatedTime, StartedTime, FinishedTime<br> &#124; render table | 列出所有容器的生命週期資訊| 
| KubeEvents_CL<br> &#124; where not(isempty(Namespace_s))<br> &#124; sort by TimeGenerated desc<br> &#124; render table | Kubernetes 事件|
| ContainerImageInventory<br> &#124; summarize AggregatedValue = count() by Image, ImageTag, Running | 映像清查 | 
| **在進階分析中，選取折線圖**：<br> Perf<br> &#124; where ObjectName == "Container" and CounterName == "% Processor Time"<br> &#124; summarize AvgCPUPercent = avg(CounterValue) by bin(TimeGenerated, 30m), InstanceName | 容器 CPU | 
| **在進階分析中，選取折線圖**：<br> Perf &#124; where ObjectName == "Container" and CounterName == "Memory Usage MB"<br> &#124; summarize AvgUsedMemory = avg(CounterValue) by bin(TimeGenerated, 30m), InstanceName | 容器記憶體 |
