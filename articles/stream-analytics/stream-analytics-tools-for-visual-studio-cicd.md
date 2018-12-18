---
title: 使用串流分析工具持續進行整合及開發
description: 本文說明如何使用 Azure 串流分析的 Visual Studio 工具設定持續整合及部署程序。
services: stream-analytics
author: su-jie
ms.author: sujie
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 9/27/2017
ms.openlocfilehash: e4e831c602255df66f4c86ffa17336f51d2b52f7
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30906268"
---
# <a name="continuously-integrate-and-develop-with-stream-analytics-tools"></a>使用串流分析工具持續進行整合及開發
本文說明如何使用適用於 Visual Studio 的 Azure 串流分析工具來設定持續整合及部署程序。

使用 2.3.0000.0 版或更新版本的[適用於 Visual Studio 的串流分析工具](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-tools-for-visual-studio)來取得 MSBuild 支援。

Nuget 套件可供使用：[Microsoft.Azure.Stream Analytics.CICD](https://www.nuget.org/packages/Microsoft.Azure.StreamAnalytics.CICD/)。 它提供的本機執行與部署工具 MSBuild，可針對串流分析 Visual Studio 專案，支援連續整合及部署程序。 
> [!NOTE] 
NuGet 套件僅能搭配 2.3.0000.0 或更新版本之適用於 Visual Studio 的串流分析工具使用。 如果您有使用舊版 Visual Studio 工具建立的專案，請直接使用 2.3.0000.0 或更新版本加以開啟並儲存。 接著啟用新功能。 

如需詳細資訊，請參閱[適用於 Visual Studio 的串流分析工具](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-tools-for-visual-studio)。

## <a name="msbuild"></a>MSBuild
與標準 Visual Studio MSBuild 體驗類似，若要建置專案，您有兩個選項。 您可以使用滑鼠右鍵按一下專案，然後選擇 [建置]。 您也可以從命令列中使用 NuGet 套件中的 **MSBuild**。
```
./build/msbuild /t:build [Your Project Full Path] /p:CompilerTaskAssemblyFile=Microsoft.WindowsAzure.StreamAnalytics.Common.CompileService.dll  /p:ASATargetsFilePath="[NuGet Package Local Path]\build\StreamAnalytics.targets"

```

串流分析 Visual Studio 專案建置成功時，會在 **bin/[Debug/Retail]/Deploy** 資料夾下方產生下列兩個 Azure Resource Manager 範本檔案： 

*  Resource Manager 範本檔案

       [ProjectName].JobTemplate.json 

*  Resource Manager 參數檔案

       [ProjectName].JobTemplate.parameters.json   

parameters.json 檔案中的預設參數來自 Visual Studio 專案中的設定。 如果您想要部署到其他環境，請據以取代參數。

> [!NOTE] 
對於所有認證，預設值都會設為 null。 部署至雲端之前，「必須」設定這些值。

```json
"Input_EntryStream_sharedAccessPolicyKey": {
      "value": null
    },
```
深入了解如何[使用 Resource Manager 範本檔案和 Azure PowerShell 部署](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)。 深入了解如何[在 Resource Manager 範本中使用物件作為參數](https://docs.microsoft.com/azure/architecture/building-blocks/extending-templates/objects-as-parameters)。


## <a name="command-line-tool"></a>命令列工具

### <a name="build-the-project"></a>建置專案
NuGet 套件有一個稱為 **SA.exe** 的命令列工具。 它支援在任意電腦上進行專案建置和本機測試，讓您可以用於持續整合和持續傳遞程序。 

部署檔案預設會放置在目前的目錄底下。 您可以使用下列 -OutputPath 參數指定輸出路徑：

```
./tools/SA.exe build -Project [Your Project Full Path] [-OutputPath <outputPath>] 
```

### <a name="test-the-script-locally"></a>在本機測試指令碼

如果您的專案已經在 Visual Studio 中指定本機輸入檔案，您可以使用 *localrun* 命令來執行自動化指令碼測試。 輸出結果會放在目前的目錄底下。
 
```
localrun -Project [ProjectFullPath]
```

### <a name="generate-a-job-definition-file-to-use-with-the-stream-analytics-powershell-api"></a>產生可搭配串流分析 PowerShell API 使用的作業定義檔案

*arm* 命令會採用透過組建產生的作業範本和作業範本參數檔案作為輸入。 然後，將它們結合到可搭配串流分析 PowerShell API 使用的作業定義 JSON 檔案中。

```
arm -JobTemplate <templateFilePath> -JobParameterFile <jobParameterFilePath> [-OutputFile <asaArmFilePath>]
```
範例：
```
./tools/SA.exe arm -JobTemplate "ProjectA.JobTemplate.json" -JobParameterFile "ProjectA.JobTemplate.parameters.json" -OutputFile "JobDefinition.json" 
```


