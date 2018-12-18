---
title: 自動調整 Azure Batch 集區中的計算節點 | Microsoft Docs
description: 在雲端集區上啟用自動調整，以動態調整集區中的計算節點數目。
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: c624cdfc-c5f2-4d13-a7d7-ae080833b779
ms.service: batch
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: multiple
ms.date: 06/20/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ab41211fb0b0b6360bdbc255e367d0492c2438ed
ms.sourcegitcommit: 7ad9db3d5f5fd35cfaa9f0735e8c0187b9c32ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39330641"
---
# <a name="create-an-automatic-scaling-formula-for-scaling-compute-nodes-in-a-batch-pool"></a>建立自動調整公式來調整 Batch 集區中的計算節點

Azure Batch 可以根據您定義的參數自動調整集區。 使用自動調整，Batch 會隨著工作需求增加，動態地將節點新增至集區，以及隨著工作需求減少而移除計算節點。 自動調整 Batch 應用程式所使用的計算節點數目，可讓您節省時間與金錢。 

您可讓您定義的「自動調整公式」與計算節點的集區產生關聯，以在該集區上啟用自動調整。 Batch 服務會使用自動調整公式來判斷要執行您的工作負載所需的計算節點數目。 計算節點可以是專用節點或[低優先順序節點](batch-low-pri-vms.md)。 Batch 會回應定期收集的服務計量資料。 使用此計量資料，Batch 會根據您的公式以可設定的間隔調整集區中的計算節點數目。

您可以在建立集區時或在現有的集區上啟用自動調整。 您也可以變更已針對自動調整設定之集區上的現有公式。 Batch 可讓您在將公式指派給集區之前評估公式，以及監視自動調整回合的狀態。

本文會討論構成自動調整公式的各種實體，包括變數、運算子、作業和函式。 我們討論如何取得 Batch 內的各種計算資源和工作計量。 您可以使用這些計量，根據資源使用量和工作狀態調整集區的節點計數。 然後，我們會說明如何藉由使用 Batch REST 和 .NET API，建立公式以及對集區啟用自動調整。 最後，我們會完成幾個範例公式。

> [!IMPORTANT]
> 當您建立 Batch 帳戶時，您可以指定[帳戶設定](batch-api-basics.md#account)，該設定會決定集區是配置在 Batch 服務訂用帳戶 (預設值)，或是您的使用者訂用帳戶。 如果您使用預設 Batch 服務設定來建立 Batch 帳戶，則您的帳戶受限為可用於處理的核心數目上限。 Batch 服務只會將計算節點調整為最多達到該核心限制。 基於這個理由，Batch 服務不會達到自動調整公式所指定的目標計算節點數目。 如需檢視和增加帳戶配額的相關資訊，請參閱 [Azure Batch 服務的配額和限制](batch-quota-limit.md) 。
>
>如果您以使用者訂用帳戶設定建立您的帳戶，則您的帳戶會共用訂用帳戶的核心配額。 如需詳細資訊，請參閱 [Azure 訂用帳戶和服務限制、配額與條件約束](../azure-subscription-service-limits.md)中的 [虛擬機器限制](../azure-subscription-service-limits.md#virtual-machines-limits)。
>
>

## <a name="automatic-scaling-formulas"></a>自動調整公式
自動調整公式是您定義的字串值，其中包含一或多個陳述式。 自動調整公式已指派給集區的 [autoScaleFormula][rest_autoscaleformula] 元素 (Batch REST) 或 [CloudPool.AutoScaleFormula][net_cloudpool_autoscaleformula] 屬性 (Batch .NET)。 Batch 集區會使用您的公式來決定集區中可供下一個間隔處理的目標計算節點數目。 公式字串不得超過 8 KB、最多只能包含 100 個陳述式 (以分號隔開)，而且可以包含換行和註解。

您可以將自動調整公式視為 Batch 自動調整「語言」。 公式陳述式是自由格式的運算式，可以包括服務定義的變數 (Batch 服務所定義的變數) 和使用者定義的變數 (您所定義的變數)。 它們可以使用內建類型、運算子和函式對這些值執行各種作業。 例如，陳述式可能會採用下列格式：

```
$myNewVariable = function($ServiceDefinedVariable, $myCustomVariable);
```

公式通常包含多個陳述式，其可對在先前陳述式中取得的值執行作業。 例如，首先我們會取得 `variable1` 的值，然後將它傳遞至函式以填入 `variable2`：

```
$variable1 = function1($ServiceDefinedVariable);
$variable2 = function2($OtherServiceDefinedVariable, $variable1);
```

在您的自動調整公式中包含這些陳述式，以達成目標計算節點數目。 專用節點與低優先順序節點各有自己的目標設定，因此您可以為每個節點類型定義目標。 自動調整公式可以包含專用節點目標值、低優先順序節點目標值，或同時包含兩者。

目標節點數目可能會更高、更低，或與集區中該類型目前的節點數目相同。 Batch 服務會在特定間隔評估集區的自動調整公式 (請參閱[自動調整間隔](#automatic-scaling-interval))。 Batch 會將集區中每個節點類型的目標數目調整為自動調整公式在評估時指定的數目。

### <a name="sample-autoscale-formula"></a>自動調整公式範例

以下是可加以調整以適用於大部分情況的自動調整公式範例。 您可以依照需求調整範例公式中的 `startingNumberOfVMs` 和 `maxNumberofVMs` 變數。 此公式會調整專用節點，但是可加以修改，套用來調整低優先順序節點。 

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicatedNodes=min(maxNumberofVMs, pendingTaskSamples);
```

使用此自動調整公式，一開始會建立包含單一 VM 的集區。 `$PendingTasks` 計量會定義執行中或已排入佇列的工作數目。 公式會尋找過去 180 秒內的平均擱置中工作數目，並據以設定 `$TargetDedicatedNodes` 變數。 公式會確保目標專用節點數目絕不會超出 25 部 VM。 集區會隨著新工作的提交而自動成長。 隨著工作完成，VM 會逐一變成可用，且自動調整公式會縮小集區。

## <a name="variables"></a>變數
您可以在自動調整公式中同時使用**服務定義**和**使用者定義**的變數。 服務定義的變數內建在 Batch 服務中。 有些服務定義的變數是讀寫，有些是唯讀。 使用者定義的變數是您定義的變數。 在上一節中所示的範例公式中，`$TargetDedicatedNodes` 和 `$PendingTasks` 是服務定義的變數。 `startingNumberOfVMs` 和 `maxNumberofVMs` 變數是使用者定義的變數。

> [!NOTE]
> 服務定義的變數前面一律會加上貨幣符號 ($)。 針對使用者定義的變數而言，貨幣符號是選擇性的。
>
>

下表顯示 Batch 服務所定義的讀寫和唯讀變數。

您可以取得並設定這些這些服務定義的變數值，以管理集區中的計算節點：

| 讀寫服務定義變數 | 說明 |
| --- | --- |
| $TargetDedicatedNodes |集區之專用計算節點的目標數目。 專用節點數目被指定作為目標，因為集區不一定會達到想要的節點數目。 例如，如果在集區達到初始目標之前，自動調整評估修改目標專用節點數目，則集區可能未達到目標。 <br /><br /> 如果目標超過 Batch 帳戶節點或核心配額，以 Batch 服務設定建立之帳戶中的集區可能未達到其目標。 如果目標超過訂用帳戶的共用核心配額，以使用者訂用帳戶設定建立之帳戶中的集區可能未達到其目標。|
| $TargetLowPriorityNodes |集區之低優先順序計算節點的目標數目。 低優先順序節點數目被指定作為目標，因為集區不一定會達到想要的節點數目。 例如，如果在集區達到初始目標之前，自動調整評估修改目標低優先順序節點數目，則集區可能未達到目標。 如果目標超過 Batch 帳戶節點或核心配額，則集區也可能未達到其目標。 <br /><br /> 如需有關低優先順序計算節點的詳細資訊，請參閱[搭配 Batch 使用低優先順序 VM (預覽)](batch-low-pri-vms.md)。 |
| $NodeDeallocationOption |計算節點從集區移除時所發生的動作。 可能的值包括：<ul><li>**requeue**：立即終止工作，並將這些工作放回工作佇列，為它們重新排程。<li>**terminate**：立即終止工作，並從作業佇列移除這些工作。<li>**taskcompletion**：等待目前執行中的工作完成，然後再從集區中移除該節點。<li>**retaineddata**：等待所有本機工作保留在節點上的資料先清除，再從集區移除節點。</ul> |

您可以取得這些服務定義的變數值，以根據 Batch 服務提供的計量進行調整：

| 唯讀服務定義變數 | 說明 |
| --- | --- |
| $CPUPercent |CPU 使用量的平均百分比。 |
| $WallClockSeconds |已耗用的秒數。 |
| $MemoryBytes |已使用的平均 MB 數目。 |
| $DiskBytes |已在本機磁碟上使用的平均 GB 數目。 |
| $DiskReadBytes |已讀取的位元組數目。 |
| $DiskWriteBytes |已寫入的位元組數目。 |
| $DiskReadOps |已執行的讀取磁碟作業計數。 |
| $DiskWriteOps |已執行的寫入磁碟作業計數。 |
| $NetworkInBytes |輸入位元組的數目。 |
| $NetworkOutBytes |輸出位元組的數目。 |
| $SampleNodeCount |計算節點的計數。 |
| $ActiveTasks |準備好執行但還未執行的工作數目。 $ActiveTasks 計數包含處於作用中狀態，而且已滿足其相依性的所有工作。 處於作用中狀態但未滿足其相依性的任何工作會從 $ActiveTasks 計數排除。|
| $RunningTasks |處於執行中狀態的工作數目。 |
| $PendingTasks |$ActiveTasks 和 $RunningTasks 的總和。 |
| $SucceededTasks |已成功完成的工作數目。 |
| $FailedTasks |失敗的工作數目。 |
| $CurrentDedicatedNodes |目前的專用計算節點數目。 |
| $CurrentLowPriorityNodes |低優先順序計算節點的目前數目，包括任何已被優先佔用的節點。 |
| $PreemptedNodeCount | 優先佔用狀態的集區中之節點數目。 |

> [!TIP]
> 上表中服務定義的唯讀變數是可提供各種方法來存取相關聯資料的物件。 如需詳細資訊，請參閱本文稍後的[取得範例資料](#getsampledata)。
>
>

## <a name="types"></a>類型
公式中支援以下類型：

* double
* doubleVec
* doubleVecList
* 字串
* timestamp：timestamp 是包含下列成員的複合結構：

  * 年
  * month (1-12)
  * day (1-31)
  * weekday (以數字格式表示，例如 1 代表星期一)
  * hour (以 24 小時制數字格式表示，例如 13 代表下午 1:00)
  * minute (00-59)
  * second (00-59)
* timeinterval

  * TimeInterval_Zero
  * TimeInterval_100ns
  * TimeInterval_Microsecond
  * TimeInterval_Millisecond
  * TimeInterval_Second
  * TimeInterval_Minute
  * TimeInterval_Hour
  * TimeInterval_Day
  * TimeInterval_Week
  * TimeInterval_Year

## <a name="operations"></a>作業
上一節列出的類型允許這些作業。

| 作業 | 支援的運算子 | 結果類型 |
| --- | --- | --- |
| double 運算子  double |+、-、*、/ |double |
| double 運算子  timeinterval |* |timeinterval |
| doubleVec 運算子  double |+、-、*、/ |doubleVec |
| doubleVec 運算子  doubleVec |+、-、*、/ |doubleVec |
| timeinterval 運算子  double |*、/ |timeinterval |
| timeinterval 運算子  timeinterval |+、- |timeinterval |
| timeinterval 運算子  timestamp |+ |timestamp |
| timestamp 運算子  timeinterval |+ |timestamp |
| timestamp  timestamp |- |timeinterval |
| double |-、! |double |
| timeinterval |- |timeinterval |
| double 運算子  double |<、<=、==、>=、>、!= |double |
| string  string |<、<=、==、>=、>、!= |double |
| timestamp  timestamp |<、<=、==、>=、>、!= |double |
| timeinterval 運算子  timeinterval |<、<=、==、>=、>、!= |double |
| double 運算子  double |&&, &#124;&#124; |double |

測試具有三元運算子的雙精準數 (`double ? statement1 : statement2`) 時，非零為 **true**，而零則為 **false**。

## <a name="functions"></a>Functions
這些預先定義的 **函式** 可供您用來定義自動調整公式。

| 函式 | 傳回類型 | 說明 |
| --- | --- | --- |
| avg(doubleVecList) |double |傳回 doubleVecList 中所有值的平均值。 |
| len(doubleVecList) |double |傳回 doubleVecList 建立的向量的長度。 |
| lg(double) |double |傳回 double 的對數底數 2。 |
| lg(doubleVecList) |doubleVec |傳回 doubleVecList 的全元件對數底數 2。 vec(double) 必須針對此參數明確傳遞。 否則會假設為 double lg(double) 版本。 |
| ln(double) |double |傳回 double 的自然底數。 |
| ln(doubleVecList) |doubleVec |傳回 doubleVecList 的全元件對數底數 2。 vec(double) 必須針對此參數明確傳遞。 否則會假設為 double lg(double) 版本。 |
| log(double) |double |傳回 double 的對數底數 10。 |
| log(doubleVecList) |doubleVec |傳回 doubleVecList 的全元件對數底數 10。 vec(double) 必須針對單一 double 參數明確傳遞。 否則，會假設為 double log (double) 版本。 |
| max(doubleVecList) |double |傳回 doubleVecList 中的最大值。 |
| min(doubleVecList) |double |傳回 doubleVecList 中的最小值。 |
| norm(doubleVecList) |double |傳回 doubleVecList 建立的向量的 2-norm。 |
| percentile(doubleVec v, double p) |double |傳回向量 v 的百分位數元素。 |
| rand() |double |傳回介於 0.0 到 1.0 之間的隨機值。 |
| range(doubleVecList) |double |傳回 doubleVecList 中最小和最大值之間的差異。 |
| std(doubleVecList) |double |傳回 doubleVecList 中值的標準差範例。 |
| stop() | |停止評估自動調整運算式。 |
| sum(doubleVecList) |double |傳回 doubleVecList 所有元件的總和。 |
| time(string dateTime="") |timestamp |如果未傳遞參數，則傳回目前時間的時間戳記，如果有傳遞參數，則為 dateTime 字串的時間戳記。 支援的 dateTime 格式為 W3C-DTF 和 RFC 1123。 |
| val(doubleVec v, double i) |double |傳回向量 v 中位置 i 的元素值，起始索引為零。 |

上表中所述的某些函式可以接受清單作為引數。 逗號分隔清單是 *double* 和 *doubleVec* 的任意組合。 例如︰

`doubleVecList := ( (double | doubleVec)+(, (double | doubleVec) )* )?`

評估之前，*doubleVecList* 值會轉換成單一的 *doubleVec*。 例如，如果 `v = [1,2,3]`，呼叫 `avg(v)` 就相當於呼叫 `avg(1,2,3)`。 呼叫 `avg(v, 7)` 就相當於呼叫 `avg(1,2,3,7)`。

## <a name="getsampledata"></a>取得樣本資料
自動調整公式會對 Batch 服務所提供的度量資料 (範例) 產生作用。 公式會根據從服務取得的值擴大或縮減集區大小。 上述服務定義的變數是可提供各種方法來存取與該物件相關聯資料的物件。 例如，下列運算式顯示取得最後五分鐘的 CPU 使用量的要求：

```
$CPUPercent.GetSample(TimeInterval_Minute * 5)
```

| 方法 | 說明 |
| --- | --- |
| GetSample() |`GetSample()` 方法會傳回資料樣本的向量。<br/><br/>一個樣本有 30 秒的度量資料。 換句話說，每隔 30 秒會取得範例。 如下所述，從收集樣本到可用於公式之間會延遲。 因此，並非一段指定時間內的所有樣本都可供公式評估。<ul><li>`doubleVec GetSample(double count)`<br/>指定要從最近收集的樣本中取得的樣本數。<br/><br/>`GetSample(1)` 會傳回最後一個可用的樣本。 不過，這不適用於 `$CPUPercent` 之類的度量，因為不可能知道「何時」收集到樣本。 此範例可能是最新的，也可能因為系統問題，是更舊的。 在此情況下，最好使用如下所示的時間間隔。<li>`doubleVec GetSample((timestamp or timeinterval) startTime [, double samplePercent])`<br/>指定收集樣本資料的時間範圍。 它也會選擇性地指定在要求的時間範圍內必須可用的樣本數百分比。<br/><br/>如果 CPUPercent 歷程記錄中有最後 10 分鐘的所有樣本，則 `$CPUPercent.GetSample(TimeInterval_Minute * 10)` 會傳回 20 個樣本。 不過，如果最後一分鐘的歷程記錄無法使用，則只會傳回 18 個樣本。 在此案例中：<br/><br/>`$CPUPercent.GetSample(TimeInterval_Minute * 10, 95)` 會失敗，因為只有 90% 的樣本可用。<br/><br/>`$CPUPercent.GetSample(TimeInterval_Minute * 10, 80)` 會成功。<li>`doubleVec GetSample((timestamp or timeinterval) startTime, (timestamp or timeinterval) endTime [, double samplePercent])`<br/>指定收集資料的時間範圍 (含開始時間和結束時間)。<br/><br/>如上所述，從收集樣本到可用於公式之間會延遲。 當您使用 `GetSample` 方法時，請考慮此延遲。 請參閱下文中的 `GetSamplePercent`。 |
| GetSamplePeriod() |傳回歷史範例資料集中取得範例的期間。 |
| Count() |傳回度量歷程記錄中的範例總數。 |
| HistoryBeginTime() |傳回度量的最舊可用資料範例的時間戳記。 |
| GetSamplePercent() |傳回指定的時間間隔內可用的樣本百分比。 例如︰<br/><br/>`doubleVec GetSamplePercent( (timestamp or timeinterval) startTime [, (timestamp or timeinterval) endTime] )`<br/><br/>因為 `GetSample` 方法在傳回樣本的百分比小於指定的 `samplePercent` 時會失敗，因此，您可以先使用 `GetSamplePercent` 方法進行檢查。 然後您可以在樣本不足時執行替代動作，而不暫停自動調整評估。 |

### <a name="samples-sample-percentage-and-the-getsample-method"></a>樣本、樣本百分比和 GetSample()  方法
自動調整公式的核心是要取得工作和資源度量資料，然後根據該資料調整集區大小。 因此，請務必清楚了解自動調整公式如何與計量資料 (樣本) 互動。

**範例**

Batch 服務會定期取得工作和資源計量的樣本，使其可供自動調整公式使用。 Batch 服務會每隔 30 秒記錄一次這些樣本。 不過，記錄樣本的時間與樣本可供自動調整公式使用 (與讀取) 的時間之間有所延遲。 此外，由於各種因素 (例如網路或其他基礎結構問題)，可能未在特定間隔內記錄樣本。

**樣本百分比**

將 `samplePercent` 傳遞至 `GetSample()` 方法，或呼叫 `GetSamplePercent()` 方法時，_percent_ 是指 Batch 服務可能記錄的樣本總數，與自動調整公式實際可用的樣本數目之間的比較。

讓我們以 10 分鐘的時間範圍為例。 因為會每隔 30 秒記錄一次樣本，所以在 10 分鐘的時間範圍內，Batch 服務所記錄的樣本總數就已達到 20 個樣本 (每分鐘 2 個)。 不過，由於回報機制固有的延遲，以及 Azure 中的其他問題，可能只有 15 個樣本可供自動調整公式讀取。 因此，舉例來說，在這 10 分鐘的期間內，記錄的樣本總數只有 75% 可供您的公式使用。

**GetSample() 和樣本範圍**

您的自動調整公式即將擴大和縮減您的集區 &mdash; 新增節點或移除節點。 因為節點為付費使用，您想要確保您的公式使用根據充足資料的明智的分析方法。 因此，建議您在公式中使用趨勢類型分析。 此類型會根據所收集樣本的範圍來擴大和縮減集區。

若要這樣做，請使用 `GetSample(interval look-back start, interval look-back end)` 傳回樣本的向量：

```
$runningTasksSample = $RunningTasks.GetSample(1 * TimeInterval_Minute, 6 * TimeInterval_Minute);
```

Batch 評估上述程式碼後，它會以值的向量形式傳回樣本範圍。 例如︰

```
$runningTasksSample=[1,1,1,1,1,1,1,1,1,1];
```

收集樣本向量後，您便可使用 `min()`、`max()` 和 `avg()` 等函式從所收集的範圍衍生出有意義的值。

若要增加安全性，如果特定一段時間可用的樣本小於特定百分比，您可以強制公式評估為失敗。 強制公式評估為失敗會指示 Batch 在指定的樣本百分比無法使用時停止進一步評估公式。 在此情況下，不會變更集區大小。 若要指定評估成功所需的樣本百分比，請將其指定為 `GetSample()`的第三個參數。 以下指定了 75% 的樣本需求：

```
$runningTasksSample = $RunningTasks.GetSample(60 * TimeInterval_Second, 120 * TimeInterval_Second, 75);
```

因為樣本可用性可能延遲，所以請務必記得指定回顧開始時間早於一分鐘的時間範圍。 樣本需要花大約一分鐘的時間才能傳播到整個系統，所以通常無法使用 `(0 * TimeInterval_Second, 60 * TimeInterval_Second)` 範圍中的樣本。 同樣地，您可以使用 `GetSample()` 的百分比參數來強制特定樣本百分比需求。

> [!IMPORTANT]
> 我們**強烈建議**您***避免「只」*`GetSample(1)`依賴自動調整公式中的** 。 這是因為 `GetSample(1)` 基本上會向 Batch 服務表示：「不論您多久以前擷取最後一個樣本，請將它提供給我」。 因為它只是單一樣本，而且可能是較舊的樣本，所以可能無法代表最近工作或資源狀態的全貌。 如果您使用 `GetSample(1)`，請確定它是較大的陳述式，而且不是您的公式所依賴的唯一資料點。
>
>

## <a name="metrics"></a>度量
您可以在定義公式時使用資源和工作計量。 您會根據您取得和評估的度量資料來調整集區中專用節點的目標數目。 如需每個度量的詳細資訊，請參閱上面的 [變數](#variables) 一節。

<table>
  <tr>
    <th>計量</th>
    <th>說明</th>
  </tr>
  <tr>
    <td><b>Resource</b></td>
    <td><p>資源計量是以計算節點的 CPU、頻寬和記憶體使用量以及節點數目為基礎。</p>
        <p> 這些服務定義的變數適合用於根據節點計數進行調整：</p>
    <p><ul>
            <li>$TargetDedicatedNodes</li>
            <li>$TargetLowPriorityNodes</li>
            <li>$CurrentDedicatedNodes</li>
            <li>$CurrentLowPriorityNodes</li>
            <li>$preemptedNodeCount</li>
            <li>$SampleNodeCount</li>
    </ul></p>
    <p>這些服務定義的變數適合用於根據節點資源使用量進行調整：</p>
    <p><ul>
      <li>$CPUPercent</li>
      <li>$WallClockSeconds</li>
      <li>$MemoryBytes</li>
      <li>$DiskBytes</li>
      <li>$DiskReadBytes</li>
      <li>$DiskWriteBytes</li>
      <li>$DiskReadOps</li>
      <li>$DiskWriteOps</li>
      <li>$NetworkInBytes</li>
      <li>$NetworkOutBytes</li></ul></p>
  </tr>
  <tr>
    <td><b>Task</b></td>
    <td><p>工作計量是根據工作的狀態，例如作用中、暫止和已完成。 下列服務定義的變數適合用於根據工作度量進行集區大小調整：</p>
    <p><ul>
      <li>$ActiveTasks</li>
      <li>$RunningTasks</li>
      <li>$PendingTasks</li>
      <li>$SucceededTasks</li>
            <li>$FailedTasks</li></ul></p>
        </td>
  </tr>
</table>

## <a name="write-an-autoscale-formula"></a>撰寫自動調整公式
您建置自動調整公式的方式是使用上述元件撰寫陳述式，再將這些陳述式結合成完整的公式。 在本節中，我們會建立可以執行一些真實世界調整決策的範例自動調整公式。

首先，讓我們定義新自動調整公式的需求。 公式應該︰

1. 如果 CPU 使用率偏高，則增加集區中專用計算節點的目標數目。
2. 如果 CPU 使用率偏低，則減少集區中專用計算節點的目標數目。
3. 一律以 400 為專用節點的數目上限。

為了在高 CPU 使用率期間增加節點數目，將陳述式定義為在使用者定義的變數 (`$totalDedicatedNodes`) 中填入 110% 的專用節點目前目標數目值，但僅限在過去 10 分鐘期間平均 CPU 使用率下限大於 70%。 否則，請使用專用節點目前數目值。

```
$totalDedicatedNodes =
    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
    ($CurrentDedicatedNodes * 1.1) : $CurrentDedicatedNodes;
```

為了減少低 CPU 使用率期間的專用節點數目，如果在過去 60 分鐘內平均 CPU 使用率低於 20%，公式中的下一個陳述式會將相同的 `$totalDedicatedNodes` 變數設定為 90% 的專用節點目前目標數目。 否則，使用我們在上述陳述式中填入的目前 `$totalDedicatedNodes` 值。

```
$totalDedicatedNodes =
    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
    ($CurrentDedicatedNodes * 0.9) : $totalDedicatedNodes;
```

現在將專用計算節點的目標數目設定為上限 400 個：

```
$TargetDedicatedNodes = min(400, $totalDedicatedNodes)
```

以下是完整的公式：

```
$totalDedicatedNodes =
    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
    ($CurrentDedicatedNodes * 1.1) : $CurrentDedicatedNodes;
$totalDedicatedNodes =
    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
    ($CurrentDedicatedNodes * 0.9) : $totalDedicatedNodes;
$TargetDedicatedNodes = min(400, $totalDedicatedNodes)
```

## <a name="create-an-autoscale-enabled-pool-with-net"></a>使用 .NET 建立已啟用自動調整的集區

若要在 .NET 中建立已啟用自動調整的集區，請遵循下列步驟：

1. 使用 [BatchClient.PoolOperations.CreatePool](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.pooloperations.createpool) 建立集區。
2. 將 [CloudPool.AutoScaleEnabled](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleenabled) 屬性設定為 `true`。
3. 使用自動調整公式來設定 [CloudPool.AutoScaleFormula](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleformula) 屬性。
4. (選擇性) 設定 [CloudPool.AutoScaleEvaluationInterval](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval) 屬性 (預設值為 15 分鐘)。
5. 使用 [CloudPool.Commit](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.commit) 或 [CommitAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.commitasync) 認可集區。

下列程式碼片段在 .NET 中建立已啟用自動調整的集區。 集區的自動調整公式會將星期一的專用節點目標數目設定為 5，而將一週其他各天的節點目標數目設定為 1。 [自動調整間隔](#automatic-scaling-interval) 設定為 30 分鐘。 在本文中的此部分與其他 C# 程式碼片段中，`myBatchClient` 是適當初始化的 [BatchClient][net_batchclient] 類別執行個體。

```csharp
CloudPool pool = myBatchClient.PoolOperations.CreatePool(
                    poolId: "mypool",
                    virtualMachineSize: "standard_d1_v2",
                    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));    
pool.AutoScaleEnabled = true;
pool.AutoScaleFormula = "$TargetDedicatedNodes = (time().weekday == 1 ? 5:1);";
pool.AutoScaleEvaluationInterval = TimeSpan.FromMinutes(30);
await pool.CommitAsync();
```

> [!IMPORTANT]
> 當您建立已啟用自動調整的集區時，請勿在對 **CreatePool** 的呼叫上指定 _targetDedicatedComputeNodes_ 參數或 _targetLowPriorityComputeNodes_ 參數。 請改為在集區上指定 **AutoScaleEnabled** 和 **AutoScaleFormula** 屬性。 這些屬性的值會判斷每個節點類型的目標數目。 此外，若要對已啟用自動調整的集區手動調整大小 (例如使用 [BatchClient.PoolOperations.ResizePoolAsync][net_poolops_resizepoolasync])，請先在集區**停用**自動調整，然後調整其大小。
>
>

除了 Batch .NET，您可以使用任何其他 [Batch SDK](batch-apis-tools.md#azure-accounts-for-batch-development)、[Batch REST](https://docs.microsoft.com/rest/api/batchservice/)、[Batch PowerShell Cmdlet](batch-powershell-cmdlets-get-started.md) 和 [Batch CLI](batch-cli-get-started.md) 來設定自動調整。


### <a name="automatic-scaling-interval"></a>自動調整間隔
依預設，Batch 服務會根據其自動調整公式每隔 15 分鐘調整集區的大小。 可使用下列的集區屬性設定此間隔：

* [CloudPool.AutoScaleEvaluationInterval][net_cloudpool_autoscaleevalinterval] (Batch .NET)
* [autoScaleEvaluationInterval][rest_autoscaleinterval] (REST API)

最小間隔為 5 分鐘，而最大間隔為 168 小時。 如果指定此範圍以外的時間間隔，則 Batch 服務會傳回「不正確的要求 (400)」錯誤。

> [!NOTE]
> 自動調整目前不適合作為低於一分鐘的變更回應，而是要在您執行工作負載時逐步調整您的集區大小。
>
>

## <a name="enable-autoscaling-on-an-existing-pool"></a>在現有集區啟用自動調整

每個 Batch SDK 會提供啟用自動調整的方法。 例如︰

* [BatchClient.PoolOperations.EnableAutoScaleAsync][net_enableautoscaleasync] (Batch .NET)
* [在自動調整中啟用集區][rest_enableautoscale] (REST API)

當您在現有集區啟用自動調整時，請記住下列幾點：

* 如果在您發出啟用自動調整要求時，集區的自動調整目前已停用，您必須在發出要求時指定有效的自動調整公式。 您可以選擇性地指定自動調整評估間隔。 如果您未指定間隔，則會使用預設值 15 分鐘。
* 如果集區的自動調整目前已啟用，您可以指定自動調整公式、評估間隔，或同時指定兩者。 您必須至少指定其中一個屬性。

  * 如果您指定新的自動調整評估間隔，則會停止現有的評估排程並啟動新的排程。 新排程的開始時間是發出啟用自動調整要求的時間。
  * 如果您省略自動調整公式或評估間隔，則 Batch 服務會繼續使用該設定目前的值。

> [!NOTE]
> 如果當您在 .NET 中建立集區時，指定 **CreatePool** 方法之 *targetDedicatedComputeNodes* 或 *targetLowPriorityComputeNodes* 參數的值，或在其他語言中指定相當的參數，則在評估自動調整公式時會略過這些值。
>
>

此 C# 程式碼片段使用 [Batch .NET][net_api] 程式庫，在現有的集區上啟用自動調整：

```csharp
// Define the autoscaling formula. This formula sets the target number of nodes
// to 5 on Mondays, and 1 on every other day of the week
string myAutoScaleFormula = "$TargetDedicatedNodes = (time().weekday == 1 ? 5:1);";

// Set the autoscale formula on the existing pool
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleFormula: myAutoScaleFormula);
```

### <a name="update-an-autoscale-formula"></a>更新自動調整公式

若要更新現有已啟用自動調整集區上的公式，呼叫作業以使用新的公式再次啟用自動調整。 例如，如果在執行下列 .NET 程式碼時，已在 `myexistingpool` 上啟用自動調整，則會以 `myNewFormula` 的內容取代其自動調整公式。

```csharp
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleFormula: myNewFormula);
```

### <a name="update-the-autoscale-interval"></a>更新自動調整間隔

若要更新現有已啟用自動調整集區的自動調整評估間隔，呼叫作業以使用新的間隔再次啟用自動調整。 例如，若要針對在 .NET 中已啟用自動調整的集區，將自動調整評估間隔設定為 60 分鐘︰

```csharp
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleEvaluationInterval: TimeSpan.FromMinutes(60));
```

## <a name="evaluate-an-autoscale-formula"></a>評估自動調整公式

您可以先評估公式，再將它套用至集區。 如此一來，您可以測試公式，以在將公式放入生產環境前，查看其陳述式的評估情況。

若要評估自動調整公式，您必須先使用有效的公式在集區啟用自動調整。 若要在尚未啟用自動調整的集區上測試公式，在第一次啟用自動調整時使用一行公式 `$TargetDedicatedNodes = 0`。 然後，使用下列其中之一來評估您想要測試的公式︰

* [BatchClient.PoolOperations.EvaluateAutoScale](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.pooloperations.evaluateautoscale) 或 [EvaluateAutoScaleAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.pooloperations.evaluateautoscaleasync)

    這些 Batch .NET 方法都需要現有集區的識別碼以及包含要評估之自動調整公式的字串。

* [評估自動調整公式](https://docs.microsoft.com/rest/api/batchservice/evaluate-an-automatic-scaling-formula)

    在這個 REST 要求中，於 URI 中指定集區識別碼，以及於要求主體的 *autoScaleFormula* 元素中指定自動調整公式。 作業的回應會包含可能與公式相關的任何錯誤資訊。

在這個 [Batch .NET][net_api] 程式碼片段中，我們會評估自動調整公式。 如果集區並未啟用自動調整，我們會先加以啟用。

```csharp
// First obtain a reference to an existing pool
CloudPool pool = await batchClient.PoolOperations.GetPoolAsync("myExistingPool");

// If autoscaling isn't already enabled on the pool, enable it.
// You can't evaluate an autoscale formula on non-autoscale-enabled pool.
if (pool.AutoScaleEnabled == false)
{
    // We need a valid autoscale formula to enable autoscaling on the
    // pool. This formula is valid, but won't resize the pool:
    await pool.EnableAutoScaleAsync(
        autoscaleFormula: "$TargetDedicatedNodes = $CurrentDedicatedNodes;",
        autoscaleEvaluationInterval: TimeSpan.FromMinutes(5));

    // Batch limits EnableAutoScaleAsync calls to once every 30 seconds.
    // Because we want to apply our new autoscale formula below if it
    // evaluates successfully, and we *just* enabled autoscaling on
    // this pool, we pause here to ensure we pass that threshold.
    Thread.Sleep(TimeSpan.FromSeconds(31));

    // Refresh the properties of the pool so that we've got the
    // latest value for AutoScaleEnabled
    await pool.RefreshAsync();
}

// We must ensure that autoscaling is enabled on the pool prior to
// evaluating a formula
if (pool.AutoScaleEnabled == true)
{
    // The formula to evaluate - adjusts target number of nodes based on
    // day of week and time of day
    string myFormula = @"
        $curTime = time();
        $workHours = $curTime.hour >= 8 && $curTime.hour < 18;
        $isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
        $isWorkingWeekdayHour = $workHours && $isWeekday;
        $TargetDedicatedNodes = $isWorkingWeekdayHour ? 20:10;
    ";

    // Perform the autoscale formula evaluation. Note that this code does not
    // actually apply the formula to the pool.
    AutoScaleRun eval =
        await batchClient.PoolOperations.EvaluateAutoScaleAsync(pool.Id, myFormula);

    if (eval.Error == null)
    {
        // Evaluation success - print the results of the AutoScaleRun.
        // This will display the values of each variable as evaluated by the
        // autoscale formula.
        Console.WriteLine("AutoScaleRun.Results: " +
            eval.Results.Replace("$", "\n    $"));

        // Apply the formula to the pool since it evaluated successfully
        await batchClient.PoolOperations.EnableAutoScaleAsync(pool.Id, myFormula);
    }
    else
    {
        // Evaluation failed, output the message associated with the error
        Console.WriteLine("AutoScaleRun.Error.Message: " +
            eval.Error.Message);
    }
}
```

成功評估此程式碼片段所顯示的公式會產生類似以下的結果：

```
AutoScaleRun.Results:
    $TargetDedicatedNodes=10;
    $NodeDeallocationOption=requeue;
    $curTime=2016-10-13T19:18:47.805Z;
    $isWeekday=1;
    $isWorkingWeekdayHour=0;
    $workHours=0
```

## <a name="get-information-about-autoscale-runs"></a>取得自動調整執行的相關資訊

若要確保您的公式如預期般執行，我們建議您定期檢查 Batch 對您的集區執行之自動調整執行的結果。 若要這麼做，請取得 (或重新整理) 集區的參考，並檢查其上次自動調整執行的內容。

在 Batch .NET 中，[CloudPool.AutoScaleRun](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.autoscalerun) 屬性有數個屬性，可提供最近在集區上執行之自動調整的相關資訊：

* [AutoScaleRun.Timestamp](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.autoscalerun.timestamp)
* [AutoScaleRun.Results](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.autoscalerun.results)
* [AutoScaleRun.Error](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.autoscalerun.error)

在 REST API 中，[取得集區的相關資訊](https://docs.microsoft.com/rest/api/batchservice/get-information-about-a-pool)要求會傳回集區的相關資訊，其中包含 [autoScaleRun](https://docs.microsoft.com/rest/api/batchservice/get-information-about-a-pool#bk_autrun) 屬性中最近執行的自動調整資訊。

下列 C# 程式碼片段會使用 Batch .NET 程式庫來列印集區 _myPool_ 上次自動調整執行的相關資訊︰

```csharp
await Cloud pool = myBatchClient.PoolOperations.GetPoolAsync("myPool");
Console.WriteLine("Last execution: " + pool.AutoScaleRun.Timestamp);
Console.WriteLine("Result:" + pool.AutoScaleRun.Results.Replace("$", "\n  $"));
Console.WriteLine("Error: " + pool.AutoScaleRun.Error);
```

上述程式碼片段的範例輸出︰

```
Last execution: 10/14/2016 18:36:43
Result:
  $TargetDedicatedNodes=10;
  $NodeDeallocationOption=requeue;
  $curTime=2016-10-14T18:36:43.282Z;
  $isWeekday=1;
  $isWorkingWeekdayHour=0;
  $workHours=0
Error:
```

## <a name="example-autoscale-formulas"></a>自動調整公式範例
讓我們看看下面幾個公式，其顯示調整集區中計算資源數量的不同方式。

### <a name="example-1-time-based-adjustment"></a>範例 1：以時間為基礎的調整
假設您想要根據一週的天數和一天的時間，調整集區大小。 這個範例示範如何據以增加或減少集區中的節點數目。

此公式會先取得目前的時間。 如果是工作日 (1-5) 且在上班時間內 (上午 8:00 - 下午 6:00)，則將目標集區大小設為 20 個節點。 否則，它會設定為 10 個節點。

```
$curTime = time();
$workHours = $curTime.hour >= 8 && $curTime.hour < 18;
$isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
$isWorkingWeekdayHour = $workHours && $isWeekday;
$TargetDedicatedNodes = $isWorkingWeekdayHour ? 20:10;
```

### <a name="example-2-task-based-adjustment"></a>範例 2：以工作為基礎的調整
在此範例中，是根據佇列中的工作數目調整集區大小。 公式字串中接受註解和換行。

```csharp
// Get pending tasks for the past 15 minutes.
$samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
// If we have fewer than 70 percent data points, we use the last sample point,
// otherwise we use the maximum of last sample point and the history average.
$tasks = $samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1), avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
// If number of pending tasks is not 0, set targetVM to pending tasks, otherwise
// half of current dedicated.
$targetVMs = $tasks > 0? $tasks:max(0, $TargetDedicatedNodes/2);
// The pool size is capped at 20, if target VM value is more than that, set it
// to 20. This value should be adjusted according to your use case.
$TargetDedicatedNodes = max(0, min($targetVMs, 20));
// Set node deallocation mode - keep nodes active only until tasks finish
$NodeDeallocationOption = taskcompletion;
```

### <a name="example-3-accounting-for-parallel-tasks"></a>範例 3：考量平行工作
此範例會根據工作數目調整集區大小。 此公式也會考慮集區已設定的 [MaxTasksPerComputeNode][net_maxtasks] 值。 已在集區上啟用[平行工作執行](batch-parallel-node-tasks.md)的情況下，這個方法很有用。

```csharp
// Determine whether 70 percent of the samples have been recorded in the past
// 15 minutes; if not, use last sample
$samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
$tasks = $samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1),avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
// Set the number of nodes to add to one-fourth the number of active tasks (the
// MaxTasksPerComputeNode property on this pool is set to 4, adjust this number
// for your use case)
$cores = $TargetDedicatedNodes * 4;
$extraVMs = (($tasks - $cores) + 3) / 4;
$targetVMs = ($TargetDedicatedNodes + $extraVMs);
// Attempt to grow the number of compute nodes to match the number of active
// tasks, with a maximum of 3
$TargetDedicatedNodes = max(0,min($targetVMs,3));
// Keep the nodes active until the tasks finish
$NodeDeallocationOption = taskcompletion;
```

### <a name="example-4-setting-an-initial-pool-size"></a>範例 4：設定初始集區大小
此範例顯示的 C# 程式碼片段具有自動調整公式，其在初始期間將集區大小設為指定的節點數目。 然後在初始期間經過之後，再根據執行中和作用中的工作數目來調整集區大小。

下列程式碼片段中的公式：

* 將初始的集區大小設為 4 個節點。
* 在集區生命週期的最初 10 分鐘內不調整集區大小。
* 10 分鐘後，取得過去 60 分鐘內執行中和作用中工作數目的最大值。
  * 如果這兩個值都是 0 (表示過去 60 分鐘沒有執行或作用中的工作)，集區大小就設為 0。
  * 如果其中一個值大於零，則不進行任何變更。

```csharp
string now = DateTime.UtcNow.ToString("r");
string formula = string.Format(@"
    $TargetDedicatedNodes = {1};
    lifespan         = time() - time(""{0}"");
    span             = TimeInterval_Minute * 60;
    startup          = TimeInterval_Minute * 10;
    ratio            = 50;

    $TargetDedicatedNodes = (lifespan > startup ? (max($RunningTasks.GetSample(span, ratio), $ActiveTasks.GetSample(span, ratio)) == 0 ? 0 : $TargetDedicatedNodes) : {1});
    ", now, 4);
```

## <a name="next-steps"></a>後續步驟
* [使用並行節點工作最大化 Azure Batch 計算資源使用量](batch-parallel-node-tasks.md) 包含有關如何對集區中的計算節點同時執行多項工作的詳細資料。 除了自動調整，這項功能有助於減少某些工作負載的作業持續時間，進而節省金錢。
* 另一種效率提升方式，則是確定您的 Batch 應用程式以最佳方式查詢 Batch 服務。 請參閱[有效率地查詢 Azure Batch 服務](batch-efficient-list-queries.md)，以了解如何在查詢可能數千個計算節點或工作的狀態時，限制越過網路的資料量。

[net_api]: https://docs.microsoft.com/dotnet/api/microsoft.azure.batch
[net_batchclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.batchclient
[net_cloudpool_autoscaleformula]: https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleformula
[net_cloudpool_autoscaleevalinterval]: https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval
[net_enableautoscaleasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.pooloperations.enableautoscaleasync
[net_maxtasks]: https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool.maxtaskspercomputenode
[net_poolops_resizepoolasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.pooloperations.resizepoolasync

[rest_api]: https://docs.microsoft.com/rest/api/batchservice/
[rest_autoscaleformula]: https://docs.microsoft.com/rest/api/batchservice/enable-automatic-scaling-on-a-pool
[rest_autoscaleinterval]: https://docs.microsoft.com/rest/api/batchservice/enable-automatic-scaling-on-a-pool
[rest_enableautoscale]: https://docs.microsoft.com/rest/api/batchservice/enable-automatic-scaling-on-a-pool
