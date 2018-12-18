---
title: 如何設定 Azure Data Lake Analytics 的 CI/CD 管線
description: 了解如何為 Azure Data Lake Analytics 設定持續整合和持續部署。
services: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: conceptual
ms.workload: big-data
ms.date: 07/03/2018
ms.openlocfilehash: 49ac9f9603a1b8043b19c327d5a66015959b9dd1
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045869"
---
# <a name="how-to-set-up-a-cicd-pipeline-for-azure-data-lake-analytics"></a>如何設定 Azure Data Lake Analytics 的 CI/CD 管線  

在本文中，您會了解如何針對 U-SQL 作業和 U-SQL 資料庫來設定持續整合和部署 (CI/CD) 管線。  

## <a name="use-cicd-for-u-sql-jobs"></a>使用 U-SQL 作業的 CI/CD

Azure Data Lake Tools for Visual Studio 提供 U-SQL 專案類型，有助於編排 U-SQL 指令碼。 使用 U-SQL 專案來管理 U-SQL 程式碼，有利於實現進階 CI/CD 案例。

## <a name="build-a-u-sql-project"></a>建置 U-SQL 專案

您可以使用 Microsoft Build Engine (MSBuild)，藉由傳遞對應的參數來建置 U-SQL 專案。 請遵循本文中的步驟，來設定 U-SQL 專案的建置程序。

### <a name="project-migration"></a>專案移轉

設定 U-SQL 專案的建置工作之前，請確定您擁有最新版本的 U-SQL 專案。 在編輯器中開啟 U-SQL 專案檔，並確認您擁有下列匯入項目：

```   
<!-- check for SDK Build target in current path then in USQLSDKPath-->
<Import Project="UsqlSDKBuild.targets" Condition="Exists('UsqlSDKBuild.targets')" />
<Import Project="$(USQLSDKPath)\UsqlSDKBuild.targets" Condition="!Exists('UsqlSDKBuild.targets') And '$(USQLSDKPath)' != '' And Exists('$(USQLSDKPath)\UsqlSDKBuild.targets')" />
``` 

如果沒有，則有兩種專案移轉選項：

- 選項 1：將舊有的匯入項目變更為前述項目。
- 選項 2：在 Azure Data Lake Tools for Visual Studio 中開啟舊專案。 使用比 2.3.3000.0 還新的版本。 舊有的專案範本將會自動升級為最新版本。 使用比 2.3.3000.0 還新的版本所建立的新專案會使用新的範本。

### <a name="get-nuget"></a>取得 NuGet

MSBuild 並未內建對 U-SQL 專案的支援。 若要獲得這項支援，您需要為解決方案新增對於 [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) NuGet 套件的參考，以新增所需的語言服務。

若要新增 NuGet 套件參考，請在 Visual Studio 方案總管中以滑鼠右鍵按一下解決方案，然後選擇 [管理 NuGet 套件]。 您也可以在解決方案資料夾內新增名為 `packages.config` 的檔案，然後於檔案中新增以下內容：

```xml 
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180620" targetFramework="net452" />
</packages>
``` 

### <a name="manage-u-sql-database-references"></a>管理 U-SQL 資料庫參考

U-SQL 專案中的 U-SQL 指令碼可能有 U-SQL 資料庫物件的查詢陳述式。 在此情況下，您需要先參考對應的 U-SQL 資料庫專案 (其中包含物件的定義)，才能建置 U-SQL 專案。 當您查詢 U-SQL 資料表或參考組件時，便是其中一例。 

深入了解 [U-SQL 資料庫專案](data-lake-analytics-data-lake-tools-develop-usql-database.md)。

>[!NOTE]
>U-SQL 資料庫專案目前處於公開預覽狀態。 如果專案中有 DROP 陳述式，則建置會失敗。 即將允許 DROP 陳述式。
>

### <a name="build-a-u-sql-project-with-the-msbuild-command-line"></a>使用 MSBuild 命令列建置 U-SQL 專案

先遷移專案，並取得 NuGet 套件。 然後，使用以下額外引數呼叫標準的 MSBuild 命令列，以建置 U-SQL 專案： 

``` 
msbuild USQLBuild.usqlproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime;USQLTargetType=SyntaxCheck;DataRoot=datarootfolder;/p:EnableDeployment=true
``` 

引數的定義和值如下：

* **USQLSDKPath=<U-SQL Nuget package>\build\runtime**。 此參數代表 U-SQL 語言服務的 NuGet 套件安裝路徑。
* **USQLTargetType=Merge 或 SyntaxCheck**：
    * **Merge**。 Merge 模式會編譯程式碼後置檔案。 範例為 **.cs**、**.py** 和 **.r** 檔案。 其會將產生的使用者定義程式碼程式庫內嵌到 U-SQL 指令碼中。 範圍為 dll 二進位、Python 或 R 程式碼。
    * **SyntaxCheck**。 SyntaxCheck 模式會先將程式碼後置檔案合併到 U-SQL 指令碼中。 然後會編譯 U-SQL 指令碼來驗證程式碼。
* **DataRoot=<DataRoot path>**。 唯有 SyntaxCheck 模式需要 DataRoot。 MSBuild 在使用 SyntaxCheck 模式建置指令碼時，會檢查指令碼中對於資料庫物件的參考。 請在建置之前，於組建電腦的 DataRoot 資料夾內設定相符的本機環境，加入來自 U-SQL 資料庫的參考物件。 若要管理這些資料庫相依性，您也可以[參考 U-SQL 資料庫專案](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project)。 MSBuild 只會檢查資料庫物件參考，不會檢查檔案。
* **EnableDeployment=true** 或 **false**。 EnableDeployment 指出它是否在建置過程中，允許部署參考的 U-SQL 資料庫。 如果您參考 U-SQL 資料庫專案，並在您的 U-SQL 指令碼中使用資料庫物件，請將這個參數設定為 **true**。

### <a name="continuous-integration-with-visual-studio-team-services"></a>持續與 Visual Studio Team Services 整合

除了命令列之外，您還可以使用 Visual Studio Build 或 MSBuild 工作，在 Visual Studio Team Services (VSTS) 中建置 U-SQL 專案。 若要設定組建管線，請務必在組建管線中新增兩個工作：NuGet 還原工作，以及 MSBuild 工作。

![U-SQL 專案的 MSBuild 工作](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png) 

1.  新增 NuGet 還原工作以取得解決方案參考的 NuGet 套件 (包括 `Azure.DataLake.USQL.SDK`)，讓 MSBuild 找得到 U-SQL 語言目標。 如果您想要直接在步驟 2 中使用 MSBuild 引數範例，可以將 [進階] > [目的地目錄] 設定為 `$(Build.SourcesDirectory)/packages`。

    ![U-SQL 專案的 NuGet 還原工作](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2.  在 Visual Studio Build Tools 中或 MSBuild 工作中設定 MSBuild 引數，如下列範例所示。 或者，您也可以在 VSTS 組建定義中，定義這些引數的變數。

    ![定義 U-SQL 專案的 CI/CD MSBuild 變數](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables.png) 

    ```
    /p:USQLSDKPath=/p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime /p:USQLTargetType=SyntaxCheck /p:DataRoot=$(Build.SourcesDirectory) /p:EnableDeployment=true
    ```

### <a name="u-sql-project-build-output"></a>U-SQL 專案建置輸出

執行建置後，將會編譯 U-SQL 專案中的所有指令碼，並輸出到名為 `USQLProjectName.usqlpack` 的 ZIP 檔案。 壓縮的建置輸出會保留專案中的資料夾結構。

> [!NOTE]
>
> 每個 U-SQL 指令碼的程式碼後置檔案會以內嵌陳述式的形式合併為指令碼建置的輸出。
>

## <a name="test-u-sql-scripts"></a>測試 U-SQL 指令碼

Azure Data Lake 提供適用於 U-SQL 指令碼和 C# UDO/UDAG/UDF 的測試專案：
* 了解如何[新增 U-SQL 指令碼的測試案例及擴充 C# 指令碼](data-lake-analytics-cicd-test.md#test-u-sql-scripts)。
* 了解如何[在 Visual Studio Team Services 中執行測試案例](data-lake-analytics-cicd-test.md#run-test-cases-in-visual-studio-team-service)。

## <a name="deploy-a-u-sql-job"></a>部署 U-SQL 作業

在透過組建和測試程序驗證程式碼之後，你可以透過 Azure PowerShell 工作直接從 Visual Studio Team Services 提交 U-SQL 作業。 也可以將指令碼部署到 Azure Data Lake Store 或 Azure Blob 儲存體，再[透過 Azure Data Factory 執行排程作業](https://docs.microsoft.com/azure/data-factory/transform-data-using-data-lake-analytics)。

### <a name="submit-u-sql-jobs-through-visual-studio-team-services"></a>透過 Visual Studio Team Services 提交 U-SQL 作業

U-SQL 專案的建置輸出是一個名為 **USQLProjectName.usqlpack** 的 ZIP 檔案。 此 ZIP 檔案包含專案中的所有 U-SQL 指令碼。 您可以在 Visual Studio Team Services 中使用 [Azure PowserShell 工作](https://docs.microsoft.com/vsts/pipelines/tasks/deploy/azure-powershell?view=vsts)，並搭配以下範例 PowserShell 指令碼，直接從 Visual Studio Team Services 組建或發行管線提交 U-SQL 作業。

```powershell
<#
    This script can be used to submit U-SQL Jobs with given U-SQL project build output(.usqlpack file).
    This will unzip the U-SQL project build output, and submit all scripts one-by-one.

    Note: the code behind file for each U-SQL script will be merged into the built U-SQL script in build output.
          
    Example :
        USQLJobSubmission.ps1 -ADLAAccountName "myadlaaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\" -DegreeOfParallelism 2
#>

param(
    [Parameter(Mandatory=$true)][string]$ADLAAccountName, # ADLA account name to submit U-SQL jobs
    [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
    [Parameter(Mandatory=$false)][string]$DegreeOfParallelism = 1
)

function Unzip($USQLPackfile, $UnzipOutput)
{
    $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
    Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
    Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

## Get U-SQL scripts in U-SQL project build output(.usqlpack file)
Function GetUsqlFiles()
{

    $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

    $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

    foreach ($USQLPackfile in $USQLPackfiles)
    {
        Unzip $USQLPackfile $UnzipOutput
    }

    $USQLFiles = Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue

    return $USQLFiles
}

## Submit U-SQL scripts to ADLA account one-by-one
Function SubmitAnalyticsJob()
{
    $usqlFiles = GetUsqlFiles

    Write-Output "$($usqlFiles.Count) jobs to be submitted..."

    # Submit each usql script and wait for completion before moving ahead.
    foreach ($usqlFile in $usqlFiles)
    {
        $scriptName = "[Release].[$([System.IO.Path]::GetFileNameWithoutExtension($usqlFile.fullname))]"

        Write-Output "Submitting job for '{$usqlFile}'"

        $jobToSubmit = Submit-AzureRmDataLakeAnalyticsJob -Account $ADLAAccountName -Name $scriptName -ScriptPath $usqlFile -DegreeOfParallelism $DegreeOfParallelism
        
        LogJobInformation $jobToSubmit
        
        Write-Output "Waiting for job to complete. Job ID:'{$($jobToSubmit.JobId)}', Name: '$($jobToSubmit.Name)' "
        $jobResult = Wait-AzureRmDataLakeAnalyticsJob -Account $ADLAAccountName -JobId $jobToSubmit.JobId  
        LogJobInformation $jobResult
    }
}

Function LogJobInformation($jobInfo)
{
    Write-Output "************************************************************************"
    Write-Output ([string]::Format("Job Id: {0}", $(DefaultIfNull $jobInfo.JobId)))
    Write-Output ([string]::Format("Job Name: {0}", $(DefaultIfNull $jobInfo.Name)))
    Write-Output ([string]::Format("Job State: {0}", $(DefaultIfNull $jobInfo.State)))
    Write-Output ([string]::Format("Job Started at: {0}", $(DefaultIfNull  $jobInfo.StartTime)))
    Write-Output ([string]::Format("Job Ended at: {0}", $(DefaultIfNull  $jobInfo.EndTime)))
    Write-Output ([string]::Format("Job Result: {0}", $(DefaultIfNull $jobInfo.Result)))
    Write-Output "************************************************************************"
}

Function DefaultIfNull($item)
{
    if ($item -ne $null)
    {
        return $item
    }
    return ""
}

Function Main()
{
    Write-Output ([string]::Format("ADLA account: {0}", $ADLAAccountName))
    Write-Output ([string]::Format("Root folde for usqlpack: {0}", $ArtifactsRoot))
    Write-Output ([string]::Format("AU count: {0}", $DegreeOfParallelism))

    Write-Output "Starting USQL script deployment..."
    
    SubmitAnalyticsJob

    Write-Output "Finished deployment..."
}

Main
```

### <a name="deploy-u-sql-jobs-through-azure-data-factory"></a>透過 Azure Data Factory 部署 U-SQL 作業

您可以直接從 Visual Studio Team Services 提交 U-SQL 作業。 您也可以將所建置的指令碼上傳到 Azure Data Lake Store 或 Azure Blob 儲存體，再[透過 Azure Data Factory 執行排程作業](https://docs.microsoft.com/azure/data-factory/transform-data-using-data-lake-analytics)。

在 Visual Studio Team Services 中使用 [Azure PowerShell 工作](https://docs.microsoft.com/vsts/pipelines/tasks/deploy/azure-powershell?view=vsts)，並搭配以下範例 PowerShell 指令碼，將 U-SQL 指令碼上傳到 Azure Data Lake Store 帳戶：

```powershell
<#
    This script can be used to upload U-SQL files to ADLS with given U-SQL project build output(.usqlpack file).
    This will unzip the U-SQL project build output, and upload all scripts to ADLS one-by-one.
          
    Example :
        FileUpload.ps1 -ADLSName "myadlsaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\"
#>

param(
    [Parameter(Mandatory=$true)][string]$ADLSName, # ADLS account name to upload U-SQL scripts
    [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
    [Parameter(Mandatory=$false)][string]$DesitinationFolder = "USQLScriptSource" # Desitination folder in ADLS
)

Function UploadResources()
{
    Write-Host "************************************************************************"
    Write-Host "Uploading files to $ADLSName"
    Write-Host "***********************************************************************"

    $usqlScripts = GetUsqlFiles

    $files = @(get-childitem $usqlScripts -recurse)
    foreach($file in $files)
    {
        Write-Host "Uploading file: $($file.Name)"
        Import-AzureRmDataLakeStoreItem -AccountName $ADLSName -Path $file.FullName -Destination "/$(Join-Path $DesitinationFolder $file)" -Force
    }
}

function Unzip($USQLPackfile, $UnzipOutput)
{
    $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
    Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
    Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

Function GetUsqlFiles()
{

    $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

    $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

    foreach ($USQLPackfile in $USQLPackfiles)
    {
        Unzip $USQLPackfile $UnzipOutput
    }

    return Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue
}

UploadResources
```

## <a name="cicd-for-a-u-sql-database"></a>U-SQL 資料庫的 CI/CD

Azure Data Lake Tools for Visual Studio 會提供 U-SQL 資料庫專案範本，以協助您開發、管理及部署 U-SQL 資料庫。 深入了解 [U-SQL 資料庫專案](data-lake-analytics-data-lake-tools-develop-usql-database.md)。

## <a name="build-u-sql-database-project"></a>建置 U-SQL 資料庫專案

### <a name="get-the-nuget-package"></a>取得 NuGet 套件

MSBuild 未提供內建的 U-SQL 資料庫專案支援。 若要獲得這項功能，您需要為解決方案新增對於 [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) NuGet 套件的參考，以新增所需的語言服務。

若要新增 NuGet 套件參考，請在 Visual Studio 方案總管中以滑鼠右鍵按一下解決方案。 選擇 [管理 NuGet 套件]。 然後搜尋並安裝 NuGet 套件。 您也可以在解決方案資料夾內新增名為 **packages.config** 的檔案，然後將以下內容放入檔案中：

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180615" targetFramework="net452" />
</packages>
```

### <a name="build-u-sql-a-database-project-with-the-msbuild-command-line"></a>使用 MSBuild 命令列建置 U-SQL 資料庫專案

若要建置 U-SQL 資料庫專案，請呼叫標準 MSBuild 命令列，再以額外引數的形式傳遞 U-SQL SDK NuGet 套件參考。 請參閱下列範例： 

```
msbuild DatabaseProject.usqldbproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime
```

引數 `USQLSDKPath=<U-SQL Nuget package>\build\runtime` 代表 U-SQL 語言服務的 NuGet 套件安裝路徑。

### <a name="continuous-integration-with-visual-studio-team-services"></a>持續與 Visual Studio Team Services 整合

除了命令列之外，您還可以使用 Visual Studio Build 或 MSBuild 工作，在 Visual Studio Team Services 中建置 U-SQL 資料庫專案。 若要設定組建工作，請務必在組建管線中新增兩個工作：NuGet 還原工作，以及 MSBuild 工作。

   ![U-SQL 專案的 CI/CD MSBuild 工作](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png) 


1.  新增 NuGet 還原工作以取得解決方案參考的 NuGet 套件 (包括 `Azure.DataLake.USQL.SDK`)，讓 MSBuild 找得到 U-SQL 語言目標。 如果您想要直接在步驟 2 中使用 MSBuild 引數範例，可以將 [進階] > [目的地目錄] 設定為 `$(Build.SourcesDirectory)/packages`。

    ![U-SQL 專案的 CI/CD NuGet 工作](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2.  在 Visual Studio Build Tools 中或 MSBuild 工作中設定 MSBuild 引數，如下列範例所示。 或者，您也可以在 VSTS 組建定義中，定義這些引數的變數。

   ![定義 U-SQL 資料庫專案的 CI/CD MSBuild 變數](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables-database-project.png) 

    ```
    /p:USQLSDKPath=/p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime
    ```
 
### <a name="u-sql-database-project-build-output"></a>U-SQL 資料庫專案建置輸出

U-SQL 資料庫專案的建置輸出，是一個名稱加上 `.usqldbpack` 尾碼的 U-SQL 資料庫部署套件。 `.usqldbpack` 套件是一個 ZIP 檔案，其中的 DDL 資料夾含有單一 U-SQL 指令碼中的所有 DDL 陳述式。 其含有 Temp 資料夾內組件的所有 **.dll** 和額外檔案。

## <a name="test-table-valued-functions-and-stored-procedures"></a>測試資料表值函式和預存程序

目前並不支援直接新增資料表值函式和預存程序的測試案例。 為了因應這種情況，您可以建立 U-SQL 專案，加入呼叫這些函式的 U-SQL 指令碼，然後再為它們撰寫測試案例。 請採取以下步驟，針對在 U-SQL 資料庫專案中定義的資料表值函式和預存程序設定測試案例：

1.  建立測試用的 U-SQL 專案，再撰寫呼叫資料表值函式和預存程序的 U-SQL 指令碼。
2.  將資料庫參考新增到該 U-SQL 專案。 為了取得資料表值函式和預存程序定義，您需要參考含有 DDL 陳述式的資料庫專案。 深入了解[資料庫參考](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project)。
3.  新增用以呼叫資料表值函式和預存程序的 U-SQL 指令碼測試案例。 [了解如何新增 U-SQL 指令碼的測試案例](data-lake-analytics-cicd-test.md#test-u-sql-scripts)。

## <a name="deploy-u-sql-database-through-visual-studio-team-service"></a>透過 Visual Studio Team Service 部署 U-SQL 資料庫

`PackageDeploymentTool.exe` 提供程式設計和命令列介面，協助您部署 U-SQL 資料庫部署套件 **.usqldbpack**。 SDK 隨附於 [U-SQL SDK NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/)中，位於 **build/runtime/PackageDeploymentTool.exe**。 只要使用 `PackageDeploymentTool.exe`，您就可以將 U-SQL 資料庫部署到 Azure Data Lake Analytics 和本機帳戶。

> [!NOTE]
>
> U-SQL 資料庫部署的 PowerShell 命令列支援和 Visual Studio Team Service 發行工作支援目前已擱置。
>

請採取以下步驟，在 Visual Studio Team Service 中設定資料庫部署工作：

1. 在組建或發行管線中新增 PowerShell 指令碼工作，並執行以下 PowerShell 指令碼。 此工作有助於取得 `PackageDeploymentTool.exe` 和 `PackageDeploymentTool.exe` 的 Azure SDK 相依性。 您可以設定 **-AzureSDK** 和 **-DBDeploymentTool** 參數，以將相依性和部署工具載入特定資料夾。 將 **-AzureSDK** 路徑傳遞至 `PackageDeploymentTool.exe`，當作步驟 2 的 **-AzureSDKPath** 參數。 

    ```powershell
    <#
        This script is used for getting dependencies and SDKs for U-SQL database deployment.
        PowerShell command line support for deploying U-SQL database package(.usqldbpack file) will come soon.
        
        Example :
            GetUSQLDBDeploymentSDK.ps1 -AzureSDK "AzureSDKFolderPath" -DBDeploymentTool "DBDeploymentToolFolderPath"
    #>

    param (
        [string]$AzureSDK = "AzureSDK", # Folder to cache Azure SDK dependencies
        [string]$DBDeploymentTool = "DBDeploymentTool", # Folder to cache U-SQL database deployment tool
        [string]$workingfolder = "" # Folder to execute these command lines
    )

    if ([string]::IsNullOrEmpty($workingfolder))
    {
        $scriptpath = $MyInvocation.MyCommand.Path
        $workingfolder = Split-Path $scriptpath
    }
    cd $workingfolder

    echo "workingfolder=$workingfolder, outputfolder=$outputfolder"
    echo "Downloading required packages..."

    iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Analytics/3.2.3-preview -outf Microsoft.Azure.Management.DataLake.Analytics.3.2.3-preview.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Store/2.3.3-preview -outf Microsoft.Azure.Management.DataLake.Store.2.3.3-preview.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.IdentityModel.Clients.ActiveDirectory/2.28.3 -outf Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime/2.3.11 -outf Microsoft.Rest.ClientRuntime.2.3.11.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure/3.3.7 -outf Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure.Authentication/2.3.3 -outf Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip
    iwr https://www.nuget.org/api/v2/package/Newtonsoft.Json/6.0.8 -outf Newtonsoft.Json.6.0.8.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.DataLake.USQL.SDK/ -outf USQLSDK.zip

    echo "Extracting packages..."

    Expand-Archive Microsoft.Azure.Management.DataLake.Analytics.3.2.3-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Analytics.3.2.3-preview -Force
    Expand-Archive Microsoft.Azure.Management.DataLake.Store.2.3.3-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Store.2.3.3-preview -Force
    Expand-Archive Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip -DestinationPath Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3 -Force
    Expand-Archive Microsoft.Rest.ClientRuntime.2.3.11.zip -DestinationPath Microsoft.Rest.ClientRuntime.2.3.11 -Force
    Expand-Archive Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.3.3.7 -Force
    Expand-Archive Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3 -Force
    Expand-Archive Newtonsoft.Json.6.0.8.zip -DestinationPath Newtonsoft.Json.6.0.8 -Force
    Expand-Archive USQLSDK.zip -DestinationPath USQLSDK -Force

    echo "Copy required DLLs to output folder..."

    mkdir $AzureSDK -Force
    mkdir $DBDeploymentTool -Force
    copy Microsoft.Azure.Management.DataLake.Analytics.3.2.3-preview\lib\net452\*.dll $AzureSDK
    copy Microsoft.Azure.Management.DataLake.Store.2.3.3-preview\lib\net452\*.dll $AzureSDK
    copy Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\*.dll $AzureSDK
    copy Microsoft.Rest.ClientRuntime.2.3.11\lib\net452\*.dll $AzureSDK
    copy Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\*.dll $AzureSDK
    copy Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3\lib\net452\*.dll $AzureSDK
    copy Newtonsoft.Json.6.0.8\lib\net45\*.dll $AzureSDK
    copy USQLSDK\build\runtime\*.* $DBDeploymentTool
    ```

2. 在組建或發行管線中新增**命令列工作**，然後藉由呼叫 `PackageDeploymentTool.exe` 來填入指令碼。 `PackageDeploymentTool.exe` 位於所定義的 **$DBDeploymentTool** 資料夾底下。 範例指令碼如下所示︰ 

    * 本機部署 U-SQL 資料庫：

        ```
        PackageDeploymentTool.exe deploylocal -Package <package path> -Database <database name> -DataRoot <data root path>
        ```

    * 使用互動式驗證模式將 U-SQL 資料庫部署到 Azure Data Lake Analytics 帳戶：

        ```
        PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -AzureSDKPath <azure sdk path> -Interactive
        ```

    * 使用**祕密**驗證將 U-SQL 資料庫部署到 Azure Data Lake Analytics 帳戶：

        ```
        PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete>
        ```

    * 使用 **certFile** 驗證將 U-SQL 資料庫部署到 Azure Data Lake Analytics 帳戶：

        ```
        PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete> -CertFile <certFile>
        ```

### <a name="packagedeploymenttoolexe-parameter-descriptions"></a>PackageDeploymentTool.exe h0 參數說明

#### <a name="common-parameters"></a>一般參數

| 參數 | 說明 | 預設值 | 必要 |
|---------|-----------|-------------|--------|
|Package|待部署的 U-SQL 資料庫部署套件路徑。|null|true|
|資料庫|要作為部署目標或要建立的資料庫名稱。|master|false|
|LogFile|記錄檔的路徑。 預設為標準輸出 (主控台)。|null|false|
|LogLevel|記錄層級：詳細資訊、一般、警告或錯誤。|LogLevel.Normal|false|

#### <a name="parameter-for-local-deployment"></a>本機部署參數

|參數|說明|預設值|必要|
|---------|-----------|-------------|--------|
|DataRoot|本機資料根資料夾的路徑。|null|true|

#### <a name="parameters-for-azure-data-lake-analytics-deployment"></a>Azure Data Lake Analytics 部署參數

|參數|說明|預設值|必要|
|---------|-----------|-------------|--------|
|帳戶|依照帳戶名稱指定部署的目的地 Azure Data Lake Analytics 帳戶。|null|true|
|ResourceGroup|Azure Data Lake Analytics 帳戶的 Azure 資源群組名稱。|null|true|
|SubscriptionId|Azure Data Lake Analytics 帳戶的 Azure 訂用帳戶識別碼。|null|true|
|租用戶|租用戶名稱是 Azure Active Directory (Azure AD) 網域名稱。 可於 Azure 入口網站的訂用帳戶管理頁面中找到。|null|true|
|AzureSDKPath|在 Azure SDK 中搜尋相依組件的路徑。|null|true|
|互動式|是否要使用互動模式來進行驗證。|false|false|
|ClientId|非互動式驗證所需的 Azure AD 應用程式識別碼。|null|非互動式驗證的必要項目。|
|Secrete|非互動式驗證的祕密或密碼。 只應在受信任且安全的環境中使用它。|null|若是非互動式驗證則為必要項目，否則請使用 SecreteFile。|
|SecreteFile|此檔案會儲存非互動式驗證的祕密或密碼。 請確保只有目前的使用者可讀取它。|null|若是非互動式驗證則為必要項目，否則請使用祕密。|
|CertFile|此檔案會儲存非互動式驗證的 X.509 憑證。 預設值是使用用戶端祕密驗證。|null|false|
| JobPrefix | U-SQL DDL 作業的資料庫部署前置詞。 | Deploy_ + DateTime.Now | false |

## <a name="next-steps"></a>後續步驟

- [如何測試 Azure Data Lake Analytics 程式碼](data-lake-analytics-cicd-test.md)。
- [在本機電腦上執行 U-SQL 指令碼](data-lake-analytics-data-lake-tools-local-run.md)。
- [使用 U-SQL 資料庫專案來開發 U-SQL 資料庫](data-lake-analytics-data-lake-tools-develop-usql-database.md)。
