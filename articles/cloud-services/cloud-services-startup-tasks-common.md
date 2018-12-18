---
title: 雲端服務的常見啟動工作 | Microsoft Docs
description: 提供一些常見的啟動工作範例，做為您在雲端服務 Web 角色或背景工作角色中執行的參考。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: a7095dad-1ee7-4141-bc6a-ef19a6e570f1
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.author: jeconnoc
ms.openlocfilehash: c9f0707f6d24ba899c89bf19066994ae860a69d5
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39620982"
---
# <a name="common-cloud-service-startup-tasks"></a>常見的雲端服務啟動工作
本文提供一些常見的啟動工作範例，做為您在雲端服務中執行的參考。 您可以利用啟動工作，在角色啟動之前執行作業。 您可能想要執行的作業包括安裝元件、註冊 COM 元件、設定登錄機碼，或啟動長時間執行的處理序。 

請參閱 [這篇文章](cloud-services-startup-tasks.md) ，了解啟動工作的運作方式，特別是該如何建立定義啟動工作的項目。

> [!NOTE]
> 啟動工作不適用於虛擬機器，只適用於雲端服務 Web 和背景工作角色。
> 

## <a name="define-environment-variables-before-a-role-starts"></a>定義角色啟動前的環境變數
如果您需要針對特定工作定義的環境變數，可以使用 [Task] 項目中的 [Task] 項目。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
                <Environment>
                    <Variable name="MyEnvironmentVariable" value="MyVariableValue" />
                </Environment>
            </Task>
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

變數也可以使用[有效的 Azure XPath 值](cloud-services-role-config-xpath.md)，以參考部署的相關項目。 而不是使用 `value` 屬性定義 [RoleInstanceValue] 子項目。

```xml
<Variable name="PathToStartupStorage">
    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
</Variable>
```


## <a name="configure-iis-startup-with-appcmdexe"></a>使用 AppCmd.exe 設定 IIS 啟動
[AppCmd.exe](https://technet.microsoft.com/library/jj635852.aspx) 命令列工具可用來管理在 Azure 上啟動時的 IIS 設定。 *AppCmd.exe* 提供方便的命令列，可存取 Azure 上啟動工作所使用的組態設定。 使用 *AppCmd.exe*，即可針對應用程式和網站，新增、修改或移除網站設定。

不過，使用 *AppCmd.exe* 做為啟動工作時，需要注意幾件事情：

* 啟動工作可以在重新開機之間執行多次。 例如，角色回收時。
* 如果 *AppCmd.exe* 動作執行一次以上，可能會產生錯誤。 例如，嘗試將區段新增至 *Web.config* 兩次，就可能會產生錯誤。
* 若啟動工作傳回非零的結束代碼或 **errorlevel**，就會執行失敗。 例如，*AppCmd.exe* 產生錯誤時。

呼叫 *AppCmd.exe* 之後，最好檢查 **errorlevel**，如果您是使用 *.cmd* 檔案包裝 *AppCmd.exe* 的呼叫，可讓這項檢查作業更容易進行。 如果偵測到已知的 **errorlevel** 回應，可以予以忽略，或是回傳。

*AppCmd.exe* 傳回的 errorlevel 會列在 winerror.h 檔案中，[MSDN](https://msdn.microsoft.com/library/windows/desktop/ms681382.aspx) 中也會顯示。

### <a name="example-of-managing-the-error-level"></a>管理錯誤層級的範例
此範例會在 *Web.config* 檔案中，加入 JSON 的 compression 區段和壓縮項目，以及錯誤處理和記錄。

這裡會顯示 [EndPoints] 檔案的相關區段，其中包括將 [executionContext](https://msdn.microsoft.com/library/azure/gg557552.aspx#Task) 屬性設為 `elevated`，藉此提供 *AppCmd.exe* 足夠的權限，以變更 *Web.config* 檔案中的設定：

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

*Startup.cmd* 批次檔會使用 *AppCmd.exe*，在 *Web.config* 檔案中新增 JSON 的 compression 區段和 compression 項目。 預期的 **errorlevel** = 183 會使用 VERIFY.EXE 命令列程式設為零。 非預期的 errorlevel 會記錄至 StartupErrorLog.txt。

```cmd
REM   *** Add a compression section to the Web.config file. ***
%windir%\system32\inetsrv\appcmd set config /section:urlCompression /doDynamicCompression:True /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1

REM   ERRORLEVEL 183 occurs when trying to add a section that already exists. This error is expected if this
REM   batch file were executed twice. This can occur and must be accounted for in a Azure startup
REM   task. To handle this situation, set the ERRORLEVEL to zero by using the Verify command. The Verify
REM   command will safely set the ERRORLEVEL to zero.
IF %ERRORLEVEL% EQU 183 DO VERIFY > NUL

REM   If the ERRORLEVEL is not zero at this point, some other error occurred.
IF %ERRORLEVEL% NEQ 0 (
    ECHO Error adding a compression section to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
    GOTO ErrorExit
)

REM   *** Add compression for json. ***
%windir%\system32\inetsrv\appcmd set config  -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1
IF %ERRORLEVEL% EQU 183 VERIFY > NUL
IF %ERRORLEVEL% NEQ 0 (
    ECHO Error adding the JSON compression type to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
    GOTO ErrorExit
)

REM   *** Exit batch file. ***
EXIT /b 0

REM   *** Log error and exit ***
:ErrorExit
REM   Report the date, time, and ERRORLEVEL of the error.
DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
ECHO An error occurred during startup. ERRORLEVEL = %ERRORLEVEL% >> "%TEMP%\StartupLog.txt" 2>&1
EXIT %ERRORLEVEL%
```

## <a name="add-firewall-rules"></a>新增防火牆規則
Azure 實際上擁有兩個防火牆。 第一道防火牆會控制虛擬機器與外界之間的連結。 此防火牆是由 [EndPoints] 檔案中的 [EndPoints] 項目所控制。

第二道防火牆會控制虛擬機器與該虛擬機器中處理序之間的連結。 可以透過 `netsh advfirewall firewall` 命令列工具來控制此防火牆。

Azure 會針對在角色內啟動的處理序建立防火牆規則。 例如，在您啟動服務或程式時，Azure 會自動建立必要的防火牆規則，藉此允許該服務與網際網路通訊。 不過，如果您建立的服務是由角色外部的處理序啟動 (像是 COM+ 服務，或是 Windows 排程器工作)，您就必須手動建立防火牆規則以允許存取該服務。 您可以使用啟動工作建立這些防火牆規則。

建立防火牆規則的啟動工作必須具有 [executionContext][task] (提高權限) 的 **executionContext**，就會執行失敗。 將以下啟動工作加入 [EndPoints] 檔案。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="AddFirewallRules.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

若要加入防火牆規則，您必須在啟動批次檔中使用適當的 `netsh advfirewall firewall` 命令。 在此範例中，啟動工作的 TCP 連接埠 80 需要具備高安全性與加密功能。

```cmd
REM   Add a firewall rule in a startup task.

REM   Add an inbound rule requiring security and encryption for TCP port 80 traffic.
netsh advfirewall firewall add rule name="Require Encryption for Inbound TCP/80" protocol=TCP dir=in localport=80 security=authdynenc action=allow >> "%TEMP%\StartupLog.txt" 2>&1

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

## <a name="block-a-specific-ip-address"></a>封鎖特定的 IP 位址
您可以透過修改 IIS **web.config** 檔來限制 Azure Web 角色只能存取一組指定的 IP 位址。 您也必須使用命令檔來解除鎖定 **ApplicationHost.config** 檔案的 **ipSecurity** 區段。

若要解除鎖定 **ApplicationHost.config** 檔案的 **ipSecurity** 區段，請建立會在角色啟動時執行的命令檔。 在 Web 角色的根層級建立名為 **startup** 的資料夾，然後在此資料夾中建立名為 **startup.cmd** 的批次檔。 將這個檔案新增至 Visual Studio 專案，並將屬性設為 [一律複製]，以確保將它納入套件中。

將以下啟動工作加入 [EndPoints] 檔案。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="startup.cmd" executionContext="elevated" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

將此命令加入 **startup.cmd** 檔案:

```cmd
@echo off
@echo Installing "IPv4 Address and Domain Restrictions" feature 
powershell -ExecutionPolicy Unrestricted -command "Install-WindowsFeature Web-IP-Security"
@echo Unlocking configuration for "IPv4 Address and Domain Restrictions" feature 
%windir%\system32\inetsrv\AppCmd.exe unlock config -section:system.webServer/security/ipSecurity
```

此工作可讓 **startup.cmd** 批次檔在 Web 角色每次初始化時一併執行，以確保將必要的 **ipSecurity** 區段解除鎖定。

最後，修改 Web 角色 [web.config](http://www.iis.net/configreference/system.webserver/security/ipsecurity#005) 檔案中的 **system.webServer** 區段，新增具有存取權的 IP 位址清單，如以下範例所示：

此範例組態 **允許** 所有 IP 存取伺服器 (另外定義的 2 個 IP 除外)。

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are granted access-->
    <ipSecurity>
        <!--The following IP addresses are denied access-->
        <add allowed="false" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="false" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

此範例組態 **拒絕** 所有 IP 存取伺服器 (另外定義的 2 個 IP 除外)。

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are denied access-->
    <ipSecurity allowUnlisted="false">
        <!--The following IP addresses are granted access-->
        <add allowed="true" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="true" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

## <a name="create-a-powershell-startup-task"></a>建立 PowerShell 啟動工作
Windows PowerShell 指令碼不能從 [EndPoints] 檔案直接呼叫，但可以從啟動批次檔內叫用。

PowerShell (依預設) 不會執行未簽署的指令碼。 除非簽署指令碼，否則需要將 PowerShell 設為執行未簽署的指令碼。 若要執行未簽署的指令碼，**ExecutionPolicy** 必須設為 **Unrestricted**。 您使用的 **ExecutionPolicy** 設定需取決於 Windows PowerShell 的版本。

```cmd
REM   Run an unsigned PowerShell script and log the output
PowerShell -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

如果您使用的客體 OS 是執行 PowerShell 2.0 或 1.0，可以強制執行第 2 版，如果無法這麼做，請改用第 1 版。

```cmd
REM   Attempt to set the execution policy by using PowerShell version 2.0 syntax.
PowerShell -Version 2.0 -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   If PowerShell version 2.0 isn't available. Set the execution policy by using the PowerShell
IF %ERRORLEVEL% EQU -393216 (
   PowerShell -Command "Set-ExecutionPolicy Unrestricted" >> "%TEMP%\StartupLog.txt" 2>&1
   PowerShell .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1
)

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

## <a name="create-files-in-local-storage-from-a-startup-task"></a>透過啟動工作在本機儲存體中建立檔案
您可以使用本機儲存資源，儲存啟動工作所建立的檔案，以便日後供您的應用程式存取。

若要建立本機儲存資源，請將 [LocalResources] 區段新增至 [EndPoints] 檔案，然後新增 [LocalStorage] 子項目。 將本機儲存體資源命名為不重複的名稱，然後為啟動工作提供適當的大小。

若要在啟動工作中使用本機儲存體資源，您需要建立環境變數來參考本機儲存體資源的位置。 接著，啟動工作和應用程式就能對本機儲存資源讀取及寫入檔案。

**ServiceDefinition.csdef** 檔案的相關區段如下：

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WorkerRole name="WorkerRole1">
    ...

    <LocalResources>
      <LocalStorage name="StartupLocalStorage" sizeInMB="5"/>
    </LocalResources>

    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>
          <Variable name="PathToStartupStorage">
            <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
  </WorkerRole>
</ServiceDefinition>
```

在範例中，此 **Startup.cmd** 批次檔使用 **PathToStartupStorage** 環境變數，在本機儲存體位置上建立 **MyTest.txt** 檔案。

```cmd
REM   Create a simple text file.

ECHO This text will go into the MyTest.txt file which will be in the    >  "%PathToStartupStorage%\MyTest.txt"
ECHO path pointed to by the PathToStartupStorage environment variable.  >> "%PathToStartupStorage%\MyTest.txt"
ECHO The contents of the PathToStartupStorage environment variable is   >> "%PathToStartupStorage%\MyTest.txt"
ECHO "%PathToStartupStorage%".                                          >> "%PathToStartupStorage%\MyTest.txt"

REM   Exit the batch file with ERRORLEVEL 0.

EXIT /b 0
```

您可以使用 [GetLocalResource](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.getlocalresource.aspx) 方法，從 Azure SDK 存取本機儲存體資料夾。

```csharp
string localStoragePath = Microsoft.WindowsAzure.ServiceRuntime.RoleEnvironment.GetLocalResource("StartupLocalStorage").RootPath;

string fileContent = System.IO.File.ReadAllText(System.IO.Path.Combine(localStoragePath, "MyTestFile.txt"));
```

## <a name="run-in-the-emulator-or-cloud"></a>在模擬器或雲端中執行
當啟動工作在雲端中運作時，您可以讓啟動工作執行有別於在計算模擬器中運作時的步驟。 例如，在模擬器中執行時，您可能只想使用 SQL 資料的全新複本。 或者，您可能想針對雲端執行一些效能最佳化作業，而這是不需要的作業。

在 [EndPoints] 檔案中建立環境變數，即可在計算模擬器和雲端上完成執行不同動作的作業。 然後，您會在啟動工作中測試該環境變數的值。

若要建立環境變數，請新增 [Variable]/[RoleInstanceValue] 項目，並建立值為 `/RoleEnvironment/Deployment/@emulated` 的 XPath。 在計算模擬器上執行時，**%ComputeEmulatorRunning%** 環境變數的值會是 `true`；在雲端上執行時則為 `false`。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WorkerRole name="WorkerRole1">

    ...

    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>
          <Variable name="ComputeEmulatorRunning">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>

  </WorkerRole>
</ServiceDefinition>
```

工作現在可以檢查 **%ComputeEmulatorRunning%** 環境變數，並根據角色是在雲端或模擬器中執行，而執行不同的動作。 以下是檢查該環境變數的 .cmd 命令介面指令碼。

```cmd
REM   Check if this task is running on the compute emulator.

IF "%ComputeEmulatorRunning%" == "true" (
    REM   This task is running on the compute emulator. Perform tasks that must be run only in the compute emulator.
) ELSE (
    REM   This task is running on the cloud. Perform tasks that must be run only in the cloud.
)
```


## <a name="detect-that-your-task-has-already-run"></a>偵測工作是否已在執行
角色可能會未經重新開機便進行回收，導致您的啟動工作再執行一次。 沒有標幟可指出工作是否已在裝載的 VM 上執行。 可能有些工作即便多次執行也沒關係。 不過，在某些情況下，您必須避免工作執行一次以上。

若要偵測工作是否已在執行，最簡單的方式，是在工作成功執行時於 **%TEMP%** 資料夾中建立檔案，並在工作開始時尋找這個檔案。 以下是可為您執行此作業的範例 cmd Shell 指令碼。

```cmd
REM   If Task1_Success.txt exists, then Application 1 is already installed.
IF EXIST "%PathToApp1Install%\Task1_Success.txt" (
  ECHO Application 1 is already installed. Exiting. >> "%TEMP%\StartupLog.txt" 2>&1
  GOTO Finish
)

REM   Run your real exe task
ECHO Running XYZ >> "%TEMP%\StartupLog.txt" 2>&1
"%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

IF %ERRORLEVEL% EQU 0 (
  REM   The application installed without error. Create a file to indicate that the task
  REM   does not need to be run again.

  ECHO This line will create a file to indicate that Application 1 installed correctly. > "%PathToApp1Install%\Task1_Success.txt"

) ELSE (
  REM   An error occurred. Log the error and exit with the error code.

  DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
  TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
  ECHO  An error occurred running task 1. Errorlevel = %ERRORLEVEL%. >> "%TEMP%\StartupLog.txt" 2>&1

  EXIT %ERRORLEVEL%
)

:Finish

REM   Exit normally.
EXIT /B 0
```

## <a name="task-best-practices"></a>工作最佳作法
為 Web 或背景工作角色設定工作時，應該遵循的最佳作法如下。

### <a name="always-log-startup-activities"></a>務必記錄啟動活動
Visual Studio 並未提供可逐步執行批次檔的偵錯工具，因此最好盡可能取得越多批次檔作業上的資料。 記錄批次檔的輸出 (包括 **stdout** 和 **stderr**)，在您嘗試偵錯及修正批次檔時，這些資料可以提供重要資訊。 若要將 **stdout** 和 **stderr** 記錄到 **%TEMP%** 環境變數所指目錄中的 StartupLog.txt 檔案，請在您想要記錄的特定行結尾加上 `>>  "%TEMP%\\StartupLog.txt" 2>&1` 文字。 例如，執行 **%PathToApp1Install%** 目錄中的 setup.exe 時：

    "%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

若要簡化您的 XML，您可以建立包裝函式 *cmd* 檔案，該檔案會呼叫您的所有啟動工作與記錄，並確保每項子工作都共用相同的環境變數。

不過，您可能會覺得在每個啟動工作的一端使用 `>> "%TEMP%\StartupLog.txt" 2>&1` 很麻煩。 您可以建立會為您處理記錄的包裝函式，以強制執行工作記錄。 此包裝函式會呼叫您要執行的實際批次檔。 來自目標批次檔的任何輸出會被重新導向至 *Startuplog.txt* 檔案。

以下範例示範如何從啟動批次檔重新導向所有的輸出。 在此範例中，ServerDefinition.csdef 檔案會建立啟動工作，以呼叫 *logwrap.cmd*。 *logwrap.cmd* 會呼叫 *Startup2.cmd*，並將所有輸出重新導向至 **%TEMP%\\StartupLog.txt**。

ServiceDefinition.cmd：

```xml
<Startup>
    <Task commandLine="logwrap.cmd startup2.cmd" executionContext="limited" taskType="simple" />
</Startup>
```

**logwrap.cmd：**

```cmd
@ECHO OFF

REM   logwrap.cmd calls passed in batch file, redirecting all output to the StartupLog.txt log file.

ECHO [%date% %time%] == START logwrap.cmd ============================================== >> "%TEMP%\StartupLog.txt" 2>&1
ECHO [%date% %time%] Running %1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   Call the child command batch file, redirecting all output to the StartupLog.txt log file.
START /B /WAIT %1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   Log the completion of child command.
ECHO [%date% %time%] Done >> "%TEMP%\StartupLog.txt" 2>&1

IF %ERRORLEVEL% EQU 0 (

   REM   No errors occurred. Exit logwrap.cmd normally.
   ECHO [%date% %time%] == END logwrap.cmd ================================================ >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO.  >> "%TEMP%\StartupLog.txt" 2>&1
   EXIT /B 0

) ELSE (

   REM   Log the error.
   ECHO [%date% %time%] An error occurred. The ERRORLEVEL = %ERRORLEVEL%.  >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO [%date% %time%] == END logwrap.cmd ================================================ >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO.  >> "%TEMP%\StartupLog.txt" 2>&1
   EXIT /B %ERRORLEVEL%

)
```

**Startup2.cmd：**

```cmd
@ECHO OFF

REM   This is the batch file where the startup steps should be performed. Because of the
REM   way Startup2.cmd was called, all commands and their outputs will be stored in the
REM   StartupLog.txt file in the directory pointed to by the TEMP environment variable.

REM   If an error occurs, the following command will pass the ERRORLEVEL back to the
REM   calling batch file.

ECHO [%date% %time%] Some log information about this task
ECHO [%date% %time%] Some more log information about this task

EXIT %ERRORLEVEL%
```

**StartupLog.txt** 檔案中的範例輸出：

```txt
[Mon 10/17/2016 20:24:46.75] == START logwrap.cmd ============================================== 
[Mon 10/17/2016 20:24:46.75] Running command1.cmd 
[Mon 10/17/2016 20:24:46.77] Some log information about this task
[Mon 10/17/2016 20:24:46.77] Some more log information about this task
[Mon 10/17/2016 20:24:46.77] Done 
[Mon 10/17/2016 20:24:46.77] == END logwrap.cmd ================================================ 
```

> [!TIP]
> **StartupLog.txt** 檔案位於 *C:\Resources\temp\\{角色識別碼} \RoleTemp* 資料夾。
> 
> 

### <a name="set-executioncontext-appropriately-for-startup-tasks"></a>正確設定啟動工作的 executionContext
正確設定啟動工作的權限。 有時候，即使角色是以正常的權限執行，啟動工作也必須以較高的權限執行。

[elevated][task] 屬性會設定啟動工作的權限等級。 使用 `executionContext="limited"` 表示啟動工作有和角色相同的權限等級。 使用 `executionContext="elevated"` 表示啟動工作有系統管理員權限，您不用將系統管理員權限提供給您的角色，便可讓啟動工作執行系統管理員工作。

需要提高權限的啟動工作範例，是使用 **AppCmd.exe** 設定 IIS 的啟動工作。 **AppCmd.exe** 需使用 `executionContext="elevated"`。

### <a name="use-the-appropriate-tasktype"></a>使用正確的 taskType
[taskType][Task] 屬性會決定執行啟動工作的方式。 此屬性有三個值：**simple**、**background** 和 **foreground**。 background 和 foreground 工作會以非同步方式啟動，而 simple 工作會以同步方式執行，一次一個。

利用 **simple** 啟動工作後，您就可以設定工作發生的順序，亦即工作在 ServiceDefinition.csdef 檔案中列出的順序。 如果 **simple** 工作以非零的結束代碼結束，則啟動程序會隨即停止，而角色也將不會啟動。

**background** 啟動工作和 **foreground** 啟動工作之間的差異，在於 **foreground** 工作會讓角色不斷執行，直到 **foreground** 工作結束為止。 換句話說，如果 **foreground** 工作停止回應或當機，那麼在 **foreground** 工作強制關閉前，都不會回收角色。 基於這個理由，除非需要使用 **foreground** 工作的功能，否則建議您將 **background** 工作用於非同步的啟動工作。

### <a name="end-batch-files-with-exit-b-0"></a>以 EXIT /B 0 結束批次檔
如果每個 simple 啟動工作的 **errorlevel** 是零，角色才會啟動。 並非所有程式都會正確設定 **errorlevel** (結束代碼)，因此如果一切要正確執行，批次檔應該要以 `EXIT /B 0` 結束。

啟動批次檔的結尾遺漏 `EXIT /B 0` ，是角色無法啟動的常見原因。

> [!NOTE]
> 我注意到使用 `/B` 參數時，巢狀批次檔有時會停滯。 您可能想要確定另一個批次檔呼叫目前的批次檔時，不會發生此停滯問題，就像是使用[記錄包裝函式](#always-log-startup-activities)一般。 在此案例中，您可以省略 `/B` 參數。
> 
> 

### <a name="expect-startup-tasks-to-run-more-than-once"></a>啟動工作預計會執行一次以上
並非所有角色回收都會重新開機，但所有角色回收都會執行全部的啟動工作。 換句話說，啟動工作必須能夠在重新開機之間，順利地執行多次。 這在[上一節](#detect-that-your-task-has-already-run)中討論。

### <a name="use-local-storage-to-store-files-that-must-be-accessed-in-the-role"></a>使用本機儲存體儲存必須在角色中存取的檔案
如果您想要在啟動工作期間複製或建立檔案，然後供您的角色存取，則該檔案必須放在本機儲存體中。 請參閱[上一節](#create-files-in-local-storage-from-a-startup-task)。

## <a name="next-steps"></a>後續步驟
檢視雲端 [服務模型和封裝](cloud-services-model-and-package.md)

深入了解 [工作](cloud-services-startup-tasks.md) 如何運作。

[建立和部署](cloud-services-how-to-create-deploy-portal.md) 雲端服務封裝。

[EndPoints]: cloud-services-model-and-package.md#csdef
[Task]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Task
[Startup]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Startup
[Runtime]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Runtime
[Task]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Environment
[Variable]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Variable
[RoleInstanceValue]: https://msdn.microsoft.com/library/azure/gg557552.aspx#RoleInstanceValue
[RoleEnvironment]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx
[EndPoints]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Endpoints
[LocalStorage]: https://msdn.microsoft.com/library/azure/gg557552.aspx#LocalStorage
[LocalResources]: https://msdn.microsoft.com/library/azure/gg557552.aspx#LocalResources
[RoleInstanceValue]: https://msdn.microsoft.com/library/azure/gg557552.aspx#RoleInstanceValue
