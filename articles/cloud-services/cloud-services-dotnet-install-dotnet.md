---
title: 在 Azure 雲端服務 (傳統) 角色上安裝 .NET |Microsoft Docs
description: 本文說明如何在雲端服務 Web 和背景工作角色上手動安裝 .NET Framework
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 18665fabad079a8759f26be8834b2fe029ab5f49
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98742772"
---
# <a name="install-net-on-azure-cloud-services-classic-roles"></a>在 Azure 雲端服務 (傳統) 角色上安裝 .NET

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

本文說明如何安裝未隨附於 Azure 客體 OS 的 .NET Framework 版本。 若要設定雲端服務 web 和背景工作角色，您可以在客體 OS 上使用 .NET。

例如，您可以在來賓 OS 系列4上安裝 .NET Framework 4.6.2，而不會隨附任何 .NET Framework 4.6 版本。  (來賓 OS 系列5隨附 .NET Framework 4.6 ) 。如需 Azure 虛擬作業系統版本的最新資訊，請參閱 Azure 虛擬作業系統 [版本新聞](cloud-services-guestos-update-matrix.md)。 

>[!IMPORTANT]
>Azure SDK 2.9 包含在來賓 OS 系列4或更早版本上部署 .NET Framework 4.6 的限制。 可在 [Microsoft Docs](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9) 網站上取得限制的修正程式。

若要在您的 web 和背景工作角色上安裝 .NET，請包括 .NET web 安裝程式作為雲端服務專案的一部分。 啟動安裝程式作為角色啟動工作的一部分。 

## <a name="add-the-net-installer-to-your-project"></a>將 .NET 安裝程式加入至專案
若要下載 .NET framework 的 Web 安裝程式，請選擇您需要安裝的版本：

* [.NET Framework 4.8 web 安裝程式](https://dotnet.microsoft.com/download/thank-you/net48)
* [.NET Framework 4.7.2 web 安裝程式](https://go.microsoft.com/fwlink/?LinkId=863262)
* [.NET Framework 4.6.2 web 安裝程式](https://www.microsoft.com/download/details.aspx?id=53345)

若要新增 web 角色的安裝程式：
  1. 在 [方案總管] 中，於雲端服務專案中的 [角色] 下，以滑鼠右鍵按一下您的 web角色，然後依序選取 [新增] > [新增資料夾]。 建立名為 **bin** 的資料夾。
  2. 以滑鼠右鍵按一下 [bin] 資料夾，然後選取 [**加入**  >  **現有專案**]。 選取 .NET 安裝程式，並將它加入至 bin 資料夾。
  
若要新增 worker 角色的安裝程式：
* 以滑鼠右鍵按一下您的 worker 角色，然後依序選取 [新增] > [現有項目]。 選取 .NET 安裝程式，並將它加入至角色。 

以這個方式將檔案新增至角色內容資料夾時，檔案就會自動新增至雲端服務套件。 然後檔案就會部署到虛擬機器上的一致位置。 為雲端服務中的每個 Web 和背景工作角色重複此程序，以便所有角色都有安裝程式副本。

> [!NOTE]
> 您應該在雲端服務角色上安裝 .NET Framework 4.6.2，即使您的應用程式是以 .NET Framework 4.6 為目標。 客體 OS 包含知識庫[更新 3098779](https://support.microsoft.com/kb/3098779) 和[更新 3097997](https://support.microsoft.com/kb/3097997)。 當您執行 .NET 應用程式時，如果 .NET Framework 4.6 安裝在知識庫更新之上，則可能會發生問題。 若要避免這些問題，請安裝 .NET Framework 4.6.2，而不是4.6 版。 如需詳細資訊，請參閱[知識庫文章 3118750](https://support.microsoft.com/kb/3118750) 和 [4340191](https://support.microsoft.com/kb/4340191)。
> 
> 

![安裝程式檔案的角色內容][1]

## <a name="define-startup-tasks-for-your-roles"></a>定義角色的啟動工作
您可以利用啟動工作，在角色啟動之前執行作業。 將 .NET Framework 安裝為啟動工作的一部分，可確保在執行任何應用程式程式碼之前就已安裝好 Framework。 如需啟動工作的詳細資訊，請參閱 [在 Azure 中執行啟動](cloud-services-startup-tasks.md)工作。 

1. 將下列內容新增至所有角色的 **WebRole** 或 **WorkerRole** 節點底下的 ServiceDefinition.csdef 檔案：
   
    ```xml
    <LocalResources>
      <LocalStorage name="NETFXInstall" sizeInMB="1024" cleanOnRoleRecycle="false" />
    </LocalResources>    
    <Startup>
      <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
        <Environment>
          <Variable name="PathToNETFXInstall">
            <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='NETFXInstall']/@path" />
          </Variable>
          <Variable name="ComputeEmulatorRunning">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
    ```
   
    上述組態會使用系統管理員權限來執行主控台命令 `install.cmd`，以便安裝 .NET Framework。 此設定也會建立名為 **NETFXInstall** 的 **LocalStorage** 元素。 啟動指令碼會將暫存資料夾設定為使用這個本機儲存資源。 
    
    > [!IMPORTANT]
    > 若要確保正確安裝架構，請將此資源的大小設定為至少 1,024 MB。
    
    如需有關啟動工作的詳細資訊，請參閱[常見的 Azure 雲端服務啟動工作](cloud-services-startup-tasks-common.md)。

2. 建立名為 **install.cmd** 的檔案，並將下列安裝指令碼新增至檔案。

   指令碼會透過查詢登錄來檢查指定的 .NET Framework 版本是否已在電腦上安裝。 如果未安裝 .NET Framework 版本，則會開啟 .NET Framework 的 web 安裝程式。 為協助針對任何問題進行疑難排解，該指令碼會將所有活動記錄到 startuptasklog-(目前日期和時間).txt 的檔案 (儲存於 **InstallLogs** 本機儲存體)。
   
   > [!IMPORTANT]
   > 使用 Windows 記事本之類的基本文字編輯器來建立 install.cmd 檔案。 如果您是使用 Visual Studio 來建立文字檔案，然後將副檔名變更為 .cmd，檔案可能仍會包含 UTF-8 位元組順序標記。 執行指令碼的第一行時，此標記可能會導致錯誤。 若要避免這個錯誤，請在指令碼的第一行使用 REM 陳述式，位元組順序處理就可以跳過。 
   > 
   >
   
   ```cmd
   REM Set the value of netfx to install appropriate .NET Framework. 
   REM ***** To install .NET 4.5.2 set the variable netfx to "NDP452" *****
   REM ***** To install .NET 4.6 set the variable netfx to "NDP46" *****
   REM ***** To install .NET 4.6.1 set the variable netfx to "NDP461" ***** https://go.microsoft.com/fwlink/?LinkId=671729
   REM ***** To install .NET 4.6.2 set the variable netfx to "NDP462" ***** https://www.microsoft.com/download/details.aspx?id=53345
   REM ***** To install .NET 4.7 set the variable netfx to "NDP47" ***** 
   REM ***** To install .NET 4.7.1 set the variable netfx to "NDP471" ***** https://go.microsoft.com/fwlink/?LinkId=852095
   REM ***** To install .NET 4.7.2 set the variable netfx to "NDP472" ***** https://go.microsoft.com/fwlink/?LinkId=863262
   set netfx="NDP472"
   REM ***** To install .NET 4.8 set the variable netfx to "NDP48" ***** https://dotnet.microsoft.com/download/thank-you/net48
      
   REM ***** Set script start timestamp *****
   set timehour=%time:~0,2%
   set timestamp=%date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2%
   set "log=install.cmd started %timestamp%."
   
   REM ***** Exit script if running in Emulator *****
   if "%ComputeEmulatorRunning%"=="true" goto exit
   
   REM ***** Needed to correctly install .NET 4.6.1, otherwise you may see an out of disk space error *****
   set TMP=%PathToNETFXInstall%
   set TEMP=%PathToNETFXInstall%
   
   REM ***** Setup .NET filenames and registry keys *****
   if %netfx%=="NDP472" goto NDP472
   if %netfx%=="NDP471" goto NDP471
   if %netfx%=="NDP47" goto NDP47
   if %netfx%=="NDP462" goto NDP462
   if %netfx%=="NDP461" goto NDP461
   if %netfx%=="NDP46" goto NDP46
   
   set "netfxinstallfile=NDP452-KB2901954-Web.exe"
   set netfxregkey="0x5cbf5"
   goto logtimestamp
   
   :NDP46
   set "netfxinstallfile=NDP46-KB3045560-Web.exe"
   set netfxregkey="0x6004f"
   goto logtimestamp
   
   :NDP461
   set "netfxinstallfile=NDP461-KB3102438-Web.exe"
   set netfxregkey="0x6040e"
   goto logtimestamp
   
   :NDP462
   set "netfxinstallfile=NDP462-KB3151802-Web.exe"
   set netfxregkey="0x60632"
   goto logtimestamp
   
   :NDP47
   set "netfxinstallfile=NDP47-KB3186500-Web.exe"
   set netfxregkey="0x707FE"
   goto logtimestamp
   
   :NDP471
   set "netfxinstallfile=NDP471-KB4033344-Web.exe"
   set netfxregkey="0x709fc"
   goto logtimestamp
   
   :NDP472
   set "netfxinstallfile=NDP472-KB4054531-Web.exe"
   set netfxregkey="0x70BF6"
   goto logtimestamp
   
   :logtimestamp
   REM ***** Setup LogFile with timestamp *****
   md "%PathToNETFXInstall%\log"
   set startuptasklog="%PathToNETFXInstall%log\startuptasklog-%timestamp%.txt"
   set netfxinstallerlog="%PathToNETFXInstall%log\NetFXInstallerLog-%timestamp%"
   echo %log% >> %startuptasklog%
   echo Logfile generated at: %startuptasklog% >> %startuptasklog%
   echo TMP set to: %TMP% >> %startuptasklog%
   echo TEMP set to: %TEMP% >> %startuptasklog%
   
   REM ***** Check if .NET is installed *****
   echo Checking if .NET (%netfx%) is installed >> %startuptasklog%
   set /A netfxregkeydecimal=%netfxregkey%
   set foundkey=0
   FOR /F "usebackq skip=2 tokens=1,2*" %%A in (`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v Release 2^>nul`) do @set /A foundkey=%%C
   echo Minimum required key: %netfxregkeydecimal% -- found key: %foundkey% >> %startuptasklog%
   if %foundkey% GEQ %netfxregkeydecimal% goto installed
   
   REM ***** Installing .NET *****
   echo Installing .NET with commandline: start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog%  /chainingpackage "CloudService Startup Task" >> %startuptasklog%
   start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog% /chainingpackage "CloudService Startup Task" >> %startuptasklog% 2>>&1
   if %ERRORLEVEL%== 0 goto installed
       echo .NET installer exited with code %ERRORLEVEL% >> %startuptasklog%    
       if %ERRORLEVEL%== 3010 goto restart
       if %ERRORLEVEL%== 1641 goto restart
       echo .NET (%netfx%) install failed with Error Code %ERRORLEVEL%. Further logs can be found in %netfxinstallerlog% >> %startuptasklog%
       goto exit
       
   :restart
   echo Restarting to complete .NET (%netfx%) installation >> %startuptasklog%
   shutdown.exe /r /t 5 /c "Installed .NET framework" /f /d p:2:4
   
   :installed
   echo .NET (%netfx%) is installed >> %startuptasklog%
   
   :end
   echo install.cmd completed: %date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2% >> %startuptasklog%
   
   :exit
   EXIT /B 0
   ```

3. 使用方案總管中的 [**加入** 現有專案]，將安裝 .cmd 檔案新增至每個角色，  >  如本主題稍早所述。  

    完成此步驟之後，所有角色應該都有 .NET 安裝程式檔案，以及 install.cmd 檔案。

   ![所有檔案的角色內容][2]

## <a name="configure-diagnostics-to-transfer-startup-logs-to-blob-storage"></a>設定診斷以將啟動記錄傳輸到 Blob 儲存體
如要簡化針對安裝問題進行疑難排解，您可以設定 Azure 診斷，來將啟動工作指令碼或 .NET 安裝程式所產生的所有記錄檔傳輸到 Azure Blob 儲存體。 您可以使用這種方法，從 Blob 儲存體下載記錄，而無需遠端桌面到角色，即可檢視記錄。


若要設定診斷，請開啟 diagnostics.wadcfgx 檔案，並在 [目錄] 節點下新增下列內容： 

```xml 
<DataSources>
 <DirectoryConfiguration containerName="netfx-install">
  <LocalResource name="NETFXInstall" relativePath="log"/>
 </DirectoryConfiguration>
</DataSources>
```

這個 XML 會將診斷設定為將 NETFXInstall 資源下 log 目錄中的檔案，傳輸到 **netfx-install** Blob 容器中的診斷儲存體帳戶。

## <a name="deploy-your-cloud-service"></a>部署您的雲端服務
部署雲端服務時，啟動工作會安裝 .NET Framework (如果尚未安裝)。 安裝架構時，雲端服務角色會處於忙碌狀態。 如果架構安裝需要重新啟動，服務角色可能也會重新啟動。 

## <a name="additional-resources"></a>其他資源
* [安裝 .NET Framework][Installing the .NET Framework]
* [判斷安裝的 .NET Framework 版本][How to: Determine Which .NET Framework Versions Are Installed]
* [疑難排解 .NET Framework 安裝][Troubleshooting .NET Framework Installations]

[How to: Determine Which .NET Framework Versions Are Installed]: /dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed
[Installing the .NET Framework]: /dotnet/framework/install/guide-for-developers
[Troubleshooting .NET Framework Installations]: /dotnet/framework/install/troubleshoot-blocked-installations-and-uninstallations

<!--Image references-->
[1]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithinstallerfiles.png
[2]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithallfiles.png



