---
title: Azure 診斷擴充功能 1.2 版設定結構描述
description: 如果您是搭配 Azure 虛擬機器、虛擬機器擴展集、Service Fabric 或雲端服務使用 Azure SDK 2.5 才相關。
services: azure-monitor
author: rboucher
ms.service: azure-monitor
ms.devlang: dotnet
ms.topic: reference
ms.date: 05/15/2017
ms.author: robb
ms.component: diagnostic-extension
ms.openlocfilehash: 8c3980231404e5c8068dbd011d20759f207d7fff
ms.sourcegitcommit: 828d8ef0ec47767d251355c2002ade13d1c162af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36937949"
---
# <a name="azure-diagnostics-12-configuration-schema"></a>Azure 診斷 1.2 組態結構描述
> [!NOTE]
> Azure 診斷是一種元件，可針對 Azure 虛擬機器、虛擬機器擴展集，Service Fabric 以及雲端服務，收集相關效能計數器和其他統計資料。  此頁面只在您使用其中一個服務時才相關。
>

Azure 診斷要與 Azure 監視器、Application Insights 和 Log Analytics 等其他 Microsoft 診斷產品搭配使用。

此結構描述定義當診斷監視器啟動，您可以用來初始化診斷組態設定的可能值。  


 執行下列 PowerShell 命令，以下載公用組態檔結構描述定義：  

```PowerShell  
(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File –Encoding utf8 -FilePath 'C:\temp\WadConfig.xsd'  
```  

 如需 Azure 診斷的詳細資訊，請參閱[在 Azure 雲端服務中啟用診斷](http://azure.microsoft.com/documentation/articles/cloud-services-dotnet-diagnostics/)。  

## <a name="example-of-the-diagnostics-configuration-file"></a>診斷組態檔的範例  
 下列範例顯示一般的診斷組態檔：  

```xml
<?xml version="1.0" encoding="utf-8"?>  
<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">  
  <WadCfg>  
    <DiagnosticMonitorConfiguration overallQuotaInMB="10000">  
      <PerformanceCounters scheduledTransferPeriod="PT1M">  
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />  
      </PerformanceCounters>  
      <Directories scheduledTransferPeriod="PT5M">  
        <IISLogs containerName="iislogs" />  
        <FailedRequestLogs containerName="iisfailed" />  
        <DataSources>  
          <DirectoryConfiguration containerName="mynewprocess">  
            <Absolute path="C:\MyNewProcess" expandEnvironment="false" />  
          </DirectoryConfiguration>  
          <DirectoryConfiguration containerName="badapp">  
            <Absolute path="%SYSTEMDRIVE%\BadApp" expandEnvironment="true" />  
          </DirectoryConfiguration>  
          <DirectoryConfiguration containerName="goodapp">  
            <LocalResource name="Skippy" relativePath="..\PeanutButter"/>  
          </DirectoryConfiguration>  
        </DataSources>  
      </Directories>  
      <EtwProviders>  
        <EtwEventSourceProviderConfiguration provider="MyProviderClass" scheduledTransferPeriod="PT5M">  
          <Event id="0"/>  
          <Event id="1" eventDestination="errorTable"/>  
          <DefaultEvents />  
        </EtwEventSourceProviderConfiguration>  
        <EtwManifestProviderConfiguration provider="5974b00b-84c2-44bc-9e58-3a2451b4e3ad" scheduledTransferLogLevelFilter="Information" scheduledTransferPeriod="PT2M">  
          <Event id="0"/>  
          <DefaultEvents eventDestination="defaultTable"/>  
        </EtwManifestProviderConfiguration>  
      </EtwProviders>  
      <WindowsEventLog scheduledTransferPeriod="PT5M">  
        <DataSource name="System!*[System[Provider[@Name='Microsoft Antimalware']]]"/>  
        <DataSource name="System!*[System[Provider[@Name='NTFS'] and (EventID=55)]]" />  
        <DataSource name="System!*[System[Provider[@Name='disk'] and (EventID=7 or EventID=52 or EventID=55)]]" />  
      </WindowsEventLog>  
      <CrashDumps containerName="wad-crashdumps" directoryQuotaPercentage="30" dumpType="Mini">  
        <CrashDumpConfiguration processName="mynewprocess.exe" />  
        <CrashDumpConfiguration processName="badapp.exe"/>  
      </CrashDumps>  
    </DiagnosticMonitorConfiguration>  
  </WadCfg>  
</PublicConfig>  

```  

## <a name="diagnostics-configuration-namespace"></a>診斷組態命名空間  
 適用於診斷組態檔的 XML 命名空間如下：  

```  
http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration  
```  

## <a name="publicconfig-element"></a>PublicConfig 元素  
 診斷組態檔的最上層元素。 下表說明組態檔的元素。  

|元素名稱|說明|  
|------------------|-----------------|  
|**WadCfg**|必要。 要收集之遙測資料的組態設定。|  
|**StorageAccount**|要儲存資料的 Azure 儲存體帳戶名稱。 這可能也會在執行 Set-AzureServiceDiagnosticsExtension Cmdlet 時指定為參數。|  
|**LocalResourceDirectory**|虛擬機器上監視代理程式用來儲存事件資料的目錄。 如果沒有設定，會使用預設的目錄：<br /><br /> 針對背景工作/web 角色：`C:\Resources\<guid>\directory\<guid>.<RoleName.DiagnosticStore\`<br /><br /> 針對虛擬機器：`C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<WADVersion>\WAD<WADVersion>`<br /><br /> 必要屬性包括：<br /><br /> -                      **path** - 系統上 Azure 診斷所使用的目錄。<br /><br /> -                      **expandEnvironment** - 控制是否要展開路徑名稱中的環境變數。|  

## <a name="wadcfg-element"></a>WadCFG 元素  
定義要收集之遙測資料的組態設定。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**DiagnosticMonitorConfiguration**|必要。 選用屬性包括：<br /><br /> -                     **overallQuotaInMB** - 可供 Azure 診斷所收集的各種類型診斷資料取用的本機磁碟空間量上限。 預設設定為 5120 MB。<br /><br /> -                     **useProxyServer** - 設定 Azure 診斷來使用 Proxy 伺服器設定，如 IE 設定中所設定。|  
|**CrashDumps**|啟用收集損毀傾印。 選用屬性包括：<br /><br /> -                     **containerName** - 在您的 Azure 儲存體帳戶中用來儲存損毀傾印的 Blob 容器名稱。<br /><br /> -                     **crashDumpType** - 設定 Azure 診斷來收集迷你或完整的損毀傾印。<br /><br /> -                     **directoryQuotaPercentage**- 設定要在 VM 上保留以供損毀傾印使用的 **overallQuotaInMB** 百分比。|  
|**DiagnosticInfrastructureLogs**|啟用收集 Azure 診斷所產生的記錄。 診斷基礎結構記錄適用於疑難排解診斷系統本身。 選用屬性包括：<br /><br /> -                     **scheduledTransferLogLevelFilter** - 設定所收集之記錄的最低嚴重性層級。<br /><br /> -                     **scheduledTransferPeriod** - 排程傳輸至儲存體之間的間隔，無條件進位到最接近的分鐘數。 值是 [XML「持續時間資料類型」(英文)](http://www.w3schools.com/xml/schema_dtypes_date.asp)。|  
|**Directories**|啟用收集目錄、IIS 失敗的存取要求記錄和/或 IIS 記錄的內容。 選用屬性：<br /><br /> **scheduledTransferPeriod** - 排程傳輸至儲存體之間的間隔，無條件進位到最接近的分鐘數。 值是 [XML「持續時間資料類型」(英文)](http://www.w3schools.com/xml/schema_dtypes_date.asp)。|  
|**EtwProviders**|設定要收集來自 EventSource 和/或以 ETW 資訊清單為基礎之提供者的 ETW 事件。|  
|**計量**|此元素讓您能夠產生已最佳化的效能計數器資料表來進行快速查詢。 除了效能計數器資料表之外，**PerformanceCounters** 元素中所定義的每個效能計數器都會儲存於 Metrics 資料表中。 必要屬性：<br /><br /> **resourceId** - 這是您要部署 Azure 診斷的虛擬機器資源識別碼。 從 [Azure 入口網站](https://portal.azure.com)取得 **resourceID**。 選取 [瀏覽][資源群組] ->  -> **<Name\>**。 按一下 [屬性] 圖格，並複製 [識別碼] 欄位的值。|  
|**PerformanceCounters**|啟用收集效能計數器。 選用屬性：<br /><br /> **scheduledTransferPeriod** - 排程傳輸至儲存體之間的間隔，無條件進位到最接近的分鐘數。 值是 [XML「持續時間資料類型」(英文)](http://www.w3schools.com/xml/schema_dtypes_date.asp)。|  
|**WindowsEventLog**|啟用收集 Windows 事件記錄檔。 選用屬性：<br /><br /> **scheduledTransferPeriod** - 排程傳輸至儲存體之間的間隔，無條件進位到最接近的分鐘數。 值是 [XML「持續時間資料類型」(英文)](http://www.w3schools.com/xml/schema_dtypes_date.asp)。|  

## <a name="crashdumps-element"></a>CrashDumps 元素  
 啟用收集損毀傾印。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**CrashDumpConfiguration**|必要。 必要屬性：<br /><br /> **processName** - 您希望 Azure 診斷收集損毀傾印的處理序名稱。|  
|**crashDumpType**|設定 Azure 診斷來收集迷你或完整的損毀傾印。|  
|**directoryQuotaPercentage**|設定要在 VM 上保留以供損毀傾印使用的 **overallQuotaInMB** 百分比。|  

## <a name="directories-element"></a>Directories 元素  
 啟用收集目錄、IIS 失敗的存取要求記錄和/或 IIS 記錄的內容。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**DataSources**|要監視的目錄清單。|  
|**FailedRequestLogs**|在組態中包含此元素，就能夠收集對於 IIS 站台或應用程式之失敗要求的相關記錄。 您也必須在 **Web.config** 的 **system.WebServer** 下啟用追蹤選項。|  
|**IISLogs**|在組態中包含此元素，就能收集 IIS 記錄：<br /><br /> **containerName** - 在您的 Azure 儲存體帳戶中用來儲存 IIS 記錄的 Blob 容器名稱。|  

## <a name="datasources-element"></a>DataSources 元素  
 要監視的目錄清單。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**DirectoryConfiguration**|必要。 必要屬性：<br /><br /> **containerName** - 在您的 Azure 儲存體帳戶中用來儲存記錄檔的 Blob 容器名稱。|  

## <a name="directoryconfiguration-element"></a>DirectoryConfiguration 元素  
 **DirectoryConfiguration** 可能包括 **Absolute** 或 **LocalResource** 元素，但非兩者。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**Absolute**|要監視之目錄的絕對路徑。 以下為必要屬性：<br /><br /> -                     **Path** - 要監視之目錄的絕對路徑。<br /><br /> -                      **expandEnvironment** - 設定是否要展開 Path 中的環境變數。|  
|**LocalResource**|相對於要監視之本機資源的路徑。 必要屬性包括：<br /><br /> -                     **Name** - 包含要監視之目錄的本機資源<br /><br /> -                     **relativePath** - 包含要監視目錄之 Name 的相對路徑|  

## <a name="etwproviders-element"></a>EtwProviders 元素  
 設定要收集來自 EventSource 和/或以 ETW 資訊清單為基礎之提供者的 ETW 事件。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**EtwEventSourceProviderConfiguration**|設定要收集從 [EventSource 類別](http://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource\(v=vs.110\).aspx)產生的事件。 必要屬性：<br /><br /> **provider** - EventSource 事件的類別名稱。<br /><br /> 選用屬性包括：<br /><br /> -                     **scheduledTransferLogLevelFilter** - 要傳輸至儲存體帳戶的最低嚴重性層級。<br /><br /> -                     **scheduledTransferPeriod** - 排程傳輸至儲存體之間的間隔，無條件進位到最接近的分鐘數。 值是 [XML「持續時間資料類型」(英文)](http://www.w3schools.com/xml/schema_dtypes_date.asp)。|  
|**EtwManifestProviderConfiguration**|必要屬性：<br /><br /> **provider** - 事件提供者的 GUID<br /><br /> 選用屬性包括：<br /><br /> - **scheduledTransferLogLevelFilter** - 要傳輸至儲存體帳戶的最低嚴重性層級。<br /><br /> -                     **scheduledTransferPeriod** - 排程傳輸至儲存體之間的間隔，無條件進位到最接近的分鐘數。 值是 [XML「持續時間資料類型」(英文)](http://www.w3schools.com/xml/schema_dtypes_date.asp)。|  

## <a name="etweventsourceproviderconfiguration-element"></a>EtwEventSourceProviderConfiguration 元素  
 設定要收集從 [EventSource 類別](http://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource\(v=vs.110\).aspx)產生的事件。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**DefaultEvents**|選用屬性：<br /><br /> **eventDestination** - 要儲存事件的資料表名稱|  
|**Event**|必要屬性：<br /><br /> **id** - 事件的識別碼。<br /><br /> 選用屬性：<br /><br /> **eventDestination** - 要儲存事件的資料表名稱|  

## <a name="etwmanifestproviderconfiguration-element"></a>EtwManifestProviderConfiguration 元素  
 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**DefaultEvents**|選用屬性：<br /><br /> **eventDestination** - 要儲存事件的資料表名稱|  
|**Event**|必要屬性：<br /><br /> **id** - 事件的識別碼。<br /><br /> 選用屬性：<br /><br /> **eventDestination** - 要儲存事件的資料表名稱|  

## <a name="metrics-element"></a>Metrics 元素  
 讓您能夠產生已最佳化的效能計數器資料表來進行快速查詢。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**MetricAggregation**|必要屬性：<br /><br /> **scheduledTransferPeriod** - 排程傳輸至儲存體之間的間隔，無條件進位到最接近的分鐘數。 值是 [XML「持續時間資料類型」(英文)](http://www.w3schools.com/xml/schema_dtypes_date.asp)。|  

## <a name="performancecounters-element"></a>PerformanceCounters 元素  
 啟用收集效能計數器。 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**PerformanceCounterConfiguration**|以下為必要屬性：<br /><br /> -                     **counterSpecifier** - 效能計數器的名稱。 例如： `\Processor(_Total)\% Processor Time`。 若要在主機上取得效能計數器清單，請執行 `typeperf` 命令。<br /><br /> -                     **sampleRate** - 應針對計數器進行取樣的頻率。<br /><br /> 選用屬性：<br /><br /> **unit** - 計數器的測量單位。|  

## <a name="performancecounterconfiguration-element"></a>PerformanceCounterConfiguration 元素  
 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**annotation**|必要屬性：<br /><br /> **displayName** - 計數器的顯示名稱<br /><br /> 選用屬性：<br /><br /> **locale** - 顯示計數器名稱時所使用的地區設定|  

## <a name="windowseventlog-element"></a>WindowsEventLog 元素  
 下表說明子元素：  

|元素名稱|說明|  
|------------------|-----------------|  
|**DataSource**|要收集的 Windows 事件記錄檔。 必要屬性：<br /><br /> **name** - 說明要收集之 Windows 事件的 XPath 查詢。 例如︰<br /><br /> `Application!*[System[(Level >= 3)]], System!*[System[(Level <=3)]], System!*[System[Provider[@Name='Microsoft Antimalware']]], Security!*[System[(Level >= 3]]`<br /><br /> 若要收集所有事件，請指定 "*"。|
