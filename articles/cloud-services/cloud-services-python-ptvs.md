---
title: 開始使用 Python 和 Azure 雲端服務 | Microsoft Docs
description: 使用 Python Tools for Visual Studio 建立 Azure 雲端服務的概觀，包括 Web 角色和背景工作角色。
services: cloud-services
documentationcenter: python
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 5489405d-6fa9-4b11-a161-609103cbdc18
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: python
ms.topic: hero-article
ms.date: 07/18/2017
ms.author: jeconnoc
ms.openlocfilehash: d9871b399ff06fb08d817c0d892b667b4ef0242d
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41918732"
---
# <a name="python-web-and-worker-roles-with-python-tools-for-visual-studio"></a>採用 Python Tools for Visual Studio 的 Python Web 和背景工作角色

本文提供在 [Python Tools for Visual Studio][Python Tools for Visual Studio] 中使用 Python Web 和背景工作角色的概觀。 學習如何使用 Visual Studio 來建立和部署使用 Python 的基本雲端服務。

## <a name="prerequisites"></a>必要條件
* [Visual Studio 2013、2015 或 2017](https://www.visualstudio.com/)
* [Python Tools for Visual Studio][Python Tools for Visual Studio] (PTVS)
* [Azure SDK Tools for VS 2013][Azure SDK Tools for VS 2013] 或  
[Azure SDK Tools for VS 2015][Azure SDK Tools for VS 2015] 或  
[Azure SDK Tools for VS 2017][Azure SDK Tools for VS 2017]
* [Python 2.7 32 位元][Python 2.7 32-bit]或 [Python 3.5 32 位元][Python 3.5 32-bit]

[!INCLUDE [create-account-and-websites-note](../../includes/create-account-and-websites-note.md)]

## <a name="what-are-python-web-and-worker-roles"></a>什麼是 Python Web 和背景工作角色？
Azure 提供三種運算模型來執行應用程式：[Azure App Service 中的 Web Apps 功能][execution model-web sites]、[Azure 虛擬機器][execution model-vms]和 [Azure 雲端服務][execution model-cloud services]。 這三種模型都支援 Python。 雲端服務 (包含 Web 和背景工作角色) 可提供*平台即服務 (PaaS)*。 在雲端服務中，Web 角色應用程式會提供專用的 Internet Information Services (IIS) Web 伺服器，用以代管前端 Web 應用程式，而背景工作角色則可執行獨立於使用者互動或輸入以外的非同步、長時間執行或持續性工作。

如需詳細資訊，請參閱[什麼是雲端服務？] (英文)。

> [!NOTE]
> *尋求建置簡單的網站？*
> 如果您的案例只需要簡單的網站前端，請考慮使用 Azure App Service 中的輕量型 Web Apps 功能。 隨著網站擴大，以及需求改變，您可以很輕易地升級到雲端服務。 請參閱 [Python 開發人員中心](https://azure.microsoft.com/develop/python/)，尋找 Azure App Service 中的 Web Apps 功能開發的相關文章。
> <br />
> 
> 

## <a name="project-creation"></a>建立專案
在 Visual Studio 中，您可以在 [Python] 下選取 [新增專案] 對話方塊中的 [Azure 雲端服務]。

![New Project Dialog](./media/cloud-services-python-ptvs/new-project-cloud-service.png)

在 [Azure 雲端服務] 精靈中，您可以建立新的 Web 和背景工作角色。

![Azure Cloud Service Dialog](./media/cloud-services-python-ptvs/new-service-wizard.png)

背景工作角色範本隨附未定案程式碼，可連接至 Azure 儲存體帳戶或 Azure 服務匯流排。

![Cloud Service Solution](./media/cloud-services-python-ptvs/worker.png)

您可以隨時將 Web 或背景工作角色加入至現有的雲端服務。  您可以選擇加入方案中現有的專案，或建立新專案。

![Add Role Command](./media/cloud-services-python-ptvs/add-new-or-existing-role.png)

雲端服務可以包含以不同語言實作的角色。  例如，您可以有使用 Django 實作的 Python Web 角色，以及 Python 或 C# 背景工作角色。  您可以使用服務匯流排佇列或儲存體佇列，輕鬆地與角色進行通訊。

## <a name="install-python-on-the-cloud-service"></a>在雲端服務上安裝 Python
> [!WARNING]
> 與 Visual Studio 一起安裝的安裝指令碼 (在這篇文章上次更新時) 無法運作。 本節說明因應措施。
> 
> 

安裝指令碼的主要問題是其未安裝 python。 首先，在 [ServiceDefinition.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef) 檔案中定義兩個[啟動工作](cloud-services-startup-tasks.md)。 第一個工作 (**PrepPython.ps1**) 會下載並安裝 Python 執行階段。 第二個工作 (**PipInstaller.ps1**) 會執行 pip 以安裝您可能具有的任何相依項目。

下列指令碼是針對 Python 3.5 而撰寫。 如果您想要使用 2.x 版的 python，請針對兩個啟動工作以及執行階段工作將 **PYTHON2** 變數檔案設定為 [on]︰`<Variable name="PYTHON2" value="<mark>on</mark>" />`。

```xml
<Startup>

  <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PrepPython.ps1">
    <Environment>
      <Variable name="EMULATED">
        <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
      </Variable>
      <Variable name="PYTHON2" value="off" />
    </Environment>
  </Task>

  <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PipInstaller.ps1">
    <Environment>
      <Variable name="EMULATED">
        <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
      </Variable>
      <Variable name="PYTHON2" value="off" />
    </Environment>

  </Task>

</Startup>
```

**PYTHON2** 和 **PYPATH** 變數必須新增至背景工作啟動工作。 只有在 **PYTHON2** 變數設定為 [on] 時，才會使用 **PYPATH** 變數。

```xml
<Runtime>
  <Environment>
    <Variable name="EMULATED">
      <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
    </Variable>
    <Variable name="PYTHON2" value="off" />
    <Variable name="PYPATH" value="%SystemDrive%\Python27" />
  </Environment>
  <EntryPoint>
    <ProgramEntryPoint commandLine="bin\ps.cmd LaunchWorker.ps1" setReadyOnProcessStart="true" />
  </EntryPoint>
</Runtime>
```

#### <a name="sample-servicedefinitioncsdef"></a>範例 ServiceDefinition.csdef
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="AzureCloudServicePython" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRole1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
      <Setting name="Python2" />
    </ConfigurationSettings>
    <Startup>
      <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PrepPython.ps1">
        <Environment>
          <Variable name="EMULATED">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
          <Variable name="PYTHON2" value="off" />
        </Environment>
      </Task>
      <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PipInstaller.ps1">
        <Environment>
          <Variable name="EMULATED">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
          <Variable name="PYTHON2" value="off" />
        </Environment>
      </Task>
    </Startup>
    <Runtime>
      <Environment>
        <Variable name="EMULATED">
          <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
        </Variable>
        <Variable name="PYTHON2" value="off" />
        <Variable name="PYPATH" value="%SystemDrive%\Python27" />
      </Environment>
      <EntryPoint>
        <ProgramEntryPoint commandLine="bin\ps.cmd LaunchWorker.ps1" setReadyOnProcessStart="true" />
      </EntryPoint>
    </Runtime>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
  </WorkerRole>
</ServiceDefinition>
```



接下來，在角色的 **./bin** 資料夾中建立 Next, create the **PrepPython.ps1** 和 **PipInstaller.ps1** 檔案。

#### <a name="preppythonps1"></a>PrepPython.ps1
此指令碼會安裝 python。 如果 **PYTHON2** 環境變數設定為 [on]，則會安裝 Python 2.7，否則會安裝 Python 3.5。

```powershell
[Net.ServicePointManager]::SecurityProtocol = "tls12, tls11, tls"
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated){
    Write-Output "Checking if python is installed...$nl"
    if ($is_python2) {
        & "${env:SystemDrive}\Python27\python.exe"  -V | Out-Null
    }
    else {
        py -V | Out-Null
    }

    if (-not $?) {

        $url = "https://www.python.org/ftp/python/3.5.2/python-3.5.2-amd64.exe"
        $outFile = "${env:TEMP}\python-3.5.2-amd64.exe"

        if ($is_python2) {
            $url = "https://www.python.org/ftp/python/2.7.12/python-2.7.12.amd64.msi"
            $outFile = "${env:TEMP}\python-2.7.12.amd64.msi"
        }

        Write-Output "Not found, downloading $url to $outFile$nl"
        Invoke-WebRequest $url -OutFile $outFile
        Write-Output "Installing$nl"

        if ($is_python2) {
            Start-Process msiexec.exe -ArgumentList "/q", "/i", "$outFile", "ALLUSERS=1" -Wait
        }
        else {
            Start-Process "$outFile" -ArgumentList "/quiet", "InstallAllUsers=1" -Wait
        }

        Write-Output "Done$nl"
    }
    else {
        Write-Output "Already installed"
    }
}
```

#### <a name="pipinstallerps1"></a>PipInstaller.ps1
此指令碼會叫出 pip 並安裝 **requirements.txt** 檔案中的所有相依項目。 如果 **PYTHON2** 環境變數設定為 [on]，則會使用 Python 2.7，否則會使用 Python 3.5。

```powershell
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated){
    Write-Output "Checking if requirements.txt exists$nl"
    if (Test-Path ..\requirements.txt) {
        Write-Output "Found. Processing pip$nl"

        if ($is_python2) {
            & "${env:SystemDrive}\Python27\python.exe" -m pip install -r ..\requirements.txt
        }
        else {
            py -m pip install -r ..\requirements.txt
        }

        Write-Output "Done$nl"
    }
    else {
        Write-Output "Not found$nl"
    }
}
```

#### <a name="modify-launchworkerps1"></a>修改 LaunchWorker.ps1
> [!NOTE]
> 如果是**背景工作角色**專案，則需要 **LauncherWorker.ps1** 檔案才能執行啟動檔。 在 **Web 角色**專案中，啟動檔已定義於專案屬性中。
> 
> 

最初建立 **bin\LaunchWorker.ps1**是為了進行許多準備工作，但它未真正發生作用。 以下列指令碼取代該檔案中的內容。

此指令碼會從 python 專案呼叫 **worker.py** 檔。 如果 **PYTHON2** 環境變數設定為 [on]，則會使用 Python 2.7，否則會使用 Python 3.5。

```powershell
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated)
{
    Write-Output "Running worker.py$nl"

    if ($is_python2) {
        cd..
        iex "$env:PYPATH\python.exe worker.py"
    }
    else {
        cd..
        iex "py worker.py"
    }
}
else
{
    Write-Output "Running (EMULATED) worker.py$nl"

    # Customize to your local dev environment

    if ($is_python2) {
        cd..
        iex "$env:PYPATH\python.exe worker.py"
    }
    else {
        cd..
        iex "py worker.py"
    }
}
```

#### <a name="pscmd"></a>ps.cmd
Visual Studio 範本應已在 **./bin** 資料夾中建立 **ps.cmd** 檔案。 此 shell 指令碼會呼叫上述 PowerShell 包裝函式指令碼，並根據所呼叫 PowerShell 包裝函式的名稱提供記錄。 如果未建立此檔案，以下是該檔案所應包含的內容。 

```bat
@echo off

cd /D %~dp0

if not exist "%DiagnosticStore%\LogFiles" mkdir "%DiagnosticStore%\LogFiles"
%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy Unrestricted -File %* >> "%DiagnosticStore%\LogFiles\%~n1.txt" 2>> "%DiagnosticStore%\LogFiles\%~n1.err.txt"
```



## <a name="run-locally"></a>在本機執行
如果您將雲端服務專案設為啟始專案並按下 F5，雲端服務會在本機 Azure 模擬器中執行。

雖然 PTVS 支援在模擬器中啟動，但偵錯無法運作 (例如，中斷點)。

若要對 Web 和背景工作角色進行偵錯，您可以將角色專案設為啟始專案，然後偵錯。  您也可以設定多個啟始專案。  以滑鼠右鍵按一下方案，然後選取 [設定啟始專案]。

![Solution Startup Project Properties](./media/cloud-services-python-ptvs/startup.png)

## <a name="publish-to-azure"></a>發佈至 Azure
若要發佈，請以滑鼠右鍵按一下方案中的雲端服務專案，然後選取 [發佈]。

![Microsoft Azure Publish Sign In](./media/cloud-services-python-ptvs/publish-sign-in.png)

遵循精靈的指示。 如有需要，請啟用遠端桌面。 當您需要進行偵錯時，遠端桌面很實用。

組態設定完成之後，按一下 [發行]。

輸出視窗中會顯示一些進度，接著會出現 [Microsoft Azure 活動記錄] 視窗。

![Microsoft Azure Activity Log Window](./media/cloud-services-python-ptvs/publish-activity-log.png)

部署需要幾分鐘才能完成，然後 Web 及/或背景工作角色就會在 Azure 上運作！

### <a name="investigate-logs"></a>調查記錄檔
雲端服務虛擬機器啟動並安裝 Python 之後，您可以查看記錄檔，以找出任何失敗訊息。 這些記錄檔位於 **C:\Resources\Directory\\{role}\LogFiles** 資料夾中。 當指令碼嘗試偵測是否已安裝 Python 時，**PrepPython.err.txt** 中至少會有一個錯誤，而 **PipInstaller.err.txt** 可能會抱怨 pip 的版本過時。

## <a name="next-steps"></a>後續步驟
如需在 Python Tools for Visual Studio 中使用 Web 和背景工作角色的相關詳細資訊，請參閱 PTVS 文件：

* [雲端服務專案][Cloud Service Projects]

如需有關從 Web 和背景工作角色使用 Azure 服務的詳細資訊，例如使用 Azure 儲存體或服務匯流排，請參閱下列文章：

* [Blob 服務][Blob Service]
* [表格服務][Table Service]
* [佇列服務][Queue Service]
* [服務匯流排佇列][Service Bus Queues]
* [服務匯流排主題][Service Bus Topics]

<!--Link references-->

[什麼是雲端服務？]: cloud-services-choose-me.md
[execution model-web sites]: ../app-service/app-service-web-overview.md
[execution model-vms]:../virtual-machines/windows/overview.md
[execution model-cloud services]: cloud-services-choose-me.md
[Python Developer Center]: /develop/python/

[Blob Service]:../storage/blobs/storage-python-how-to-use-blob-storage.md
[Queue Service]: ../storage/queues/storage-python-how-to-use-queue-storage.md
[Table Service]:../cosmos-db/table-storage-how-to-use-python.md
[Service Bus Queues]: ../service-bus-messaging/service-bus-python-how-to-use-queues.md
[Service Bus Topics]: ../service-bus-messaging/service-bus-python-how-to-use-topics-subscriptions.md


<!--External Link references-->

[Python Tools for Visual Studio]: http://aka.ms/ptvs
[Python Tools for Visual Studio Documentation]: http://aka.ms/ptvsdocs
[Cloud Service Projects]: https://docs.microsoft.com/visualstudio/python/python-azure-cloud-service-project-template
[Azure SDK Tools for VS 2013]: http://go.microsoft.com/fwlink/?LinkId=746482
[Azure SDK Tools for VS 2015]: http://go.microsoft.com/fwlink/?LinkId=746481
[Azure SDK Tools for VS 2017]: http://go.microsoft.com/fwlink/?LinkId=746483
[Python 2.7 32-bit]: https://www.python.org/downloads/
[Python 3.5 32-bit]: https://www.python.org/downloads/
