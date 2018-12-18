---
title: Azure 自動化圖形化 Runbook SDK 概觀
description: 本文說明如何使用 Azure 自動化圖形化 Runbook SDK
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 07/20/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 702af8311887afc94e7127704d3377e944503324
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39240383"
---
# <a name="use-the-azure-automation-graphical-runbook-sdk"></a>使用 Azure 自動化圖形化 Runbook SDK

[圖形化 Runbook](automation-graphical-authoring-intro.md)是協助您管理基礎 Windows PowerShell 或 PowerShell 工作流程程式碼複雜性的 Runbook。 Microsoft Azure 自動化圖形化撰寫 SDK 可讓開發人員建立和編輯圖形化 Runbook，以便與 Azure 自動化服務搭配使用。 下列程式碼片段顯示從您的程式碼建立圖形化 Runbook 的基本流程。

## <a name="pre-requisites"></a>先決條件

若要開始，請將 `Microsoft.Azure.Automation.GraphicalRunbook.Model` 套件匯入您的專案。

## <a name="create-a-runbook-object-instance"></a>建立 Runbook 物件執行個體

請參考 `Orchestrator.GraphRunbook.Model` 組件，並建立 `Orchestrator.GraphRunbook.Model.GraphRunbook` 類別的執行個體：

```csharp
using Orchestrator.GraphRunbook.Model;
using Orchestrator.GraphRunbook.Model.ExecutableView;

var runbook = new GraphRunbook();
```

## <a name="add-runbook-parameters"></a>新增 Runbook 參數

具現化 `Orchestrator.GraphRunbook.Model.Parameter` 物件，並將其新增至 Runbook：

```csharp
runbook.AddParameter(
 new Parameter("YourName") {
  TypeName = typeof(string).FullName,
   DefaultValue = "World",
   Optional = true
 });

runbook.AddParameter(
 new Parameter("AnotherParameter") {
  TypeName = typeof(int).FullName, ...
 });
```

## <a name="add-activities-and-links"></a>新增活動和連結

將適當類型的活動具現化，並將其新增至 Runbook：

```csharp
var writeOutputActivityType = new CommandActivityType {
 CommandName = "Write-Output",
  ModuleName = "Microsoft.PowerShell.Utility",
 InputParameterSets = new [] {
  new ParameterSet {
   Name = "Default",
    new [] {
     new Parameter("InputObject"), ...
    }
  },
  ...
 }
};

var outputName = runbook.AddActivity(
 new CommandActivity("Output name", writeOutputActivityType) {
  ParameterSetName = "Default",
   Parameters = new ActivityParameters {
    {
     "InputObject",
     new RunbookParameterValueDescriptor("YourName")
    }
   },
   PositionX = 0,
   PositionY = 0
 });

var initializeRunbookVariable = runbook.AddActivity(
 new WorkflowScriptActivity("Initialize variable") {
  Process = "$a = 0",
   PositionX = 0,
   PositionY = 100
 });
```

活動會在 `Orchestrator.GraphRunbook.Model` 命名空間中由下列類別實作：

|類別  |活動  |
|---------|---------|
|CommandActivity     | 叫用 PowerShell 命令 (cmdlet、函式等)。        |
|InvokeRunbookActivity     | 叫用另一個 Runbook 內嵌。        |
|JunctionActivity     | 等候所有內送分支完成。        |
|WorkflowScriptActivity     | 在 Runbook 的內容中執行 PowerShell 或 PowerShell 工作流程程式碼 (取決於 Runbook 類型) 的區塊。 這是功能強大的工具，但請勿過度使用：UI 會將這個指令碼區塊顯示為文字；執行引擎會將提供的區塊視為黑箱，且不會嘗試分析其內容，除了基本的語法檢查之外。 如果您只需要叫用單一 PowerShell 命令，建議使用 CommandActivity。        |

> [!NOTE]
> 請勿從提供的類別衍生您自己的活動：Azure 自動化將無法使用 Runbook 搭配自訂活動類型。

必須提供 CommandActivity 和 InvokeRunbookActivity 參數作為值描述項，而非直接值。 值描述項會指定實際參數值的產生方式。 目前提供下列值描述項：


|描述項  |定義  |
|---------|---------|
|ConstantValueDescriptor     | 是指硬式編碼的常數值。        |
|RunbookParameterValueDescriptor     | 是指依名稱而定的 Runbook 參數。        |
|ActivityOutputValueDescriptor     | 是指上游活動輸出，可讓一個活動「訂閱」另一個活動所產生的資料。        |
|AutomationVariableValueDescriptor     | 是指依名稱而定的自動化變數資產。         |
|AutomationCredentialValueDescriptor     | 是指依名稱而定的自動化憑證資產。        |
|AutomationConnectionValueDescriptor     | 是指依名稱而定的自動化連線資產。        |
|PowerShellExpressionValueDescriptor     | 指定自由格式 PowerShell 運算式，在即將叫用活動之前才會加以評估。  <br/>這是功能強大的工具，但請勿過度使用：UI 會將這個運算式顯示為文字；執行引擎會將提供的區塊視為黑箱，且不會嘗試分析其內容，除了基本的語法檢查之外。 如果可能的話，最好使用更明確的值描述項。      |

> [!NOTE]
> 請勿從提供的類別衍生您自己的值描述項：Azure 自動化將無法使用 Runbook 搭配自訂值描述項類型。

將連線活動的連結具現化，並將其新增至 Runbook：

```csharp
runbook.AddLink(new Link(activityA, activityB, LinkType.Sequence));

runbook.AddLink(
 new Link(activityB, activityC, LinkType.Pipeline) {
  Condition = string.Format("$ActivityOutput['{0}'] -gt 0", activityB.Name)
 });
```

## <a name="save-the-runbook-to-a-file"></a>將 Runbook 儲存到檔案

使用 `Orchestrator.GraphRunbook.Model.Serialization.RunbookSerializer` 將 Runbook 序列化為字串：

```csharp
var serialized = RunbookSerializer.Serialize(runbook);
```

此字串可以儲存至副檔名為 **.graphrunbook** 的檔案，且此檔案可以匯入 Azure 自動化中。
序列化格式在未來的 `Orchestrator.GraphRunbook.Model.dll` 版本中可能會有所變更。 我們保證回溯相容性：利用較舊版 `Orchestrator.GraphRunbook.Model.dll` 序列化的任何 Runbook 都可由新版本還原序列化。 不保證往後相容性：以較新版本序列化的 Runbook 可能無法由較舊版本還原序列化。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 自動化中的圖形化 Runbook，請參閱[圖形化撰寫簡介](automation-graphical-authoring-intro.md)
