---
title: Azure 診斷 1.0 組態結構描述
description: 如果您是搭配 Azure 虛擬機器、虛擬機器擴展集、Service Fabric 或雲端服務使用 Azure SDK 2.4 和以下版本才相關。
services: azure-monitor
author: rboucher
ms.service: azure-monitor
ms.devlang: dotnet
ms.topic: reference
ms.date: 05/15/2017
ms.author: robb
ms.component: diagnostic-extension
ms.openlocfilehash: 916e2123262402e23f35778e66683ecce2cec4b7
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35262580"
---
# <a name="azure-diagnostics-10-configuration-schema"></a>Azure 診斷 1.0 組態結構描述
> [!NOTE]
> Azure 診斷是一種元件，可針對 Azure 虛擬機器、虛擬機器擴展集，Service Fabric 以及雲端服務，收集相關效能計數器和其他統計資料。  此頁面只在您使用其中一個服務時才相關。
>

Azure 診斷要與 Azure 監視器、Application Insights 和 Log Analytics 等其他 Microsoft 診斷產品搭配使用。

Azure 診斷組態檔會定義用來初始化診斷監視器的值。 在診斷監視器啟動時，會使用此檔案來初始化診斷組態設定。  

 根據預設，Azure 診斷組態結構描述檔案是安裝於 `C:\Program Files\Microsoft SDKs\Azure\.NET SDK\<version>\schemas` 目錄中。 請使用安裝的 [Azure SDK](http://www.windowsazure.com/develop/downloads/) 版本來取代 `<version>`。  

> [!NOTE]
>  診斷組態檔通常會與啟動工作搭配使用，這類啟動工作需要在啟動程序前期收集診斷資料。 如需使用 Azure 診斷的詳細資訊，請參閱[使用 Azure 診斷收集記錄資料](assetId:///83a91c23-5ca2-4fc9-8df3-62036c37a3d7)。  

## <a name="example-of-the-diagnostics-configuration-file"></a>診斷組態檔的範例  
 下列範例顯示一般的診斷組態檔：  

```xml  
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticMonitorConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"  
      configurationChangePollInterval="PT1M"  
      overallQuotaInMB="4096">  
   <DiagnosticInfrastructureLogs bufferQuotaInMB="1024"  
      scheduledTransferLogLevelFilter="Verbose"  
      scheduledTransferPeriod="PT1M" />  
   <Logs bufferQuotaInMB="1024"  
      scheduledTransferLogLevelFilter="Verbose"  
      scheduledTransferPeriod="PT1M" />  

   <Directories bufferQuotaInMB="1024"   
      scheduledTransferPeriod="PT1M">  

      <!-- These three elements specify the special directories   
           that are set up for the log types -->  
      <CrashDumps container="wad-crash-dumps" directoryQuotaInMB="256" />  
      <FailedRequestLogs container="wad-frq" directoryQuotaInMB="256" />  
      <IISLogs container="wad-iis" directoryQuotaInMB="256" />  

      <!-- For regular directories the DataSources element is used -->  
      <DataSources>  
         <DirectoryConfiguration container="wad-panther" directoryQuotaInMB="128">  
            <!-- Absolute specifies an absolute path with optional environment expansion -->  
            <Absolute expandEnvironment="true" path="%SystemRoot%\system32\sysprep\Panther" />  
         </DirectoryConfiguration>  
         <DirectoryConfiguration container="wad-custom" directoryQuotaInMB="128">  
            <!-- LocalResource specifies a path relative to a local   
                 resource defined in the service definition -->  
            <LocalResource name="MyLoggingLocalResource" relativePath="logs" />  
         </DirectoryConfiguration>  
      </DataSources>  
   </Directories>  

   <PerformanceCounters bufferQuotaInMB="512" scheduledTransferPeriod="PT1M">  
      <!-- The counter specifier is in the same format as the imperative   
           diagnostics configuration API -->  
      <PerformanceCounterConfiguration   
         counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT5S" />  
   </PerformanceCounters>  

   <WindowsEventLog bufferQuotaInMB="512"  
      scheduledTransferLogLevelFilter="Verbose"  
      scheduledTransferPeriod="PT1M">  
      <!-- The event log name is in the same format as the imperative   
           diagnostics configuration API -->  
      <DataSource name="System!*" />  
   </WindowsEventLog>  
</DiagnosticMonitorConfiguration>  
```  

## <a name="diagnosticsconfiguration-namespace"></a>DiagnosticsConfiguration 命名空間  
 適用於診斷組態檔的 XML 命名空間如下：  

```  
http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration  
```  

## <a name="schema-elements"></a>結構描述元素  
 診斷組態檔包含下列元素。


## <a name="diagnosticmonitorconfiguration-element"></a>DiagnosticMonitorConfiguration 元素  
診斷組態檔的最上層元素。  

屬性：

|屬性  |類型   |必要| 預設值 | 說明|  
|-----------|-------|--------|---------|------------|  
|**configurationChangePollInterval**|duration|選用 | PT1M| 指定診斷監視器輪詢診斷組態變更的間隔。|  
|**overallQuotaInMB**|unsignedInt|選用| 4000 MB。 如果您提供值，該值不能超過此數量 |針對所有記錄緩衝區配置的檔案系統儲存體總數量。|  

## <a name="diagnosticinfrastructurelogs-element"></a>DiagnosticInfrastructureLogs 元素  
針對基本診斷基礎結構所產生的記錄定義緩衝區組態。

父元素：[DiagnosticMonitorConfiguration 元素](#DiagnosticMonitorConfiguration)。  

屬性：

|屬性|類型|說明|  
|---------|----|-----------------|  
|**bufferQuotaInMB**|unsignedInt|選用。 指定適用於所指定資料的檔案系統儲存體數量上限。<br /><br /> 預設值為 0。|  
|**scheduledTransferLogLevelFilter**|字串|選用。 指定所傳輸記錄項目的最低嚴重性層級。 預設值為 **Undefined**。 其他可能的值為 **Verbose**、**Information**、**Warning**、**Error** 及 **Critical**。|  
|**scheduledTransferPeriod**|duration|選用。 指定排程傳輸資料之間的間隔，無條件進位到最接近的分鐘數。<br /><br /> 預設值為 PT0S。|  

## <a name="logs-element"></a>Logs 元素  
 定義基本 Azure 記錄的緩衝區組態。

 父元素：[DiagnosticMonitorConfiguration 元素](#DiagnosticMonitorConfiguration)。  

屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**bufferQuotaInMB**|unsignedInt|選用。 指定適用於所指定資料的檔案系統儲存體數量上限。<br /><br /> 預設值為 0。|  
|**scheduledTransferLogLevelFilter**|字串|選用。 指定所傳輸記錄項目的最低嚴重性層級。 預設值為 **Undefined**。 其他可能的值為 **Verbose**、**Information**、**Warning**、**Error** 及 **Critical**。|  
|**scheduledTransferPeriod**|duration|選用。 指定排程傳輸資料之間的間隔，無條件進位到最接近的分鐘數。<br /><br /> 預設值為 PT0S。|  

## <a name="directories-element"></a>Directories 元素  
定義您可以定義之檔案式記錄的緩衝區組態。

父元素：[DiagnosticMonitorConfiguration 元素](#DiagnosticMonitorConfiguration)。  


屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**bufferQuotaInMB**|unsignedInt|選用。 指定適用於所指定資料的檔案系統儲存體數量上限。<br /><br /> 預設值為 0。|  
|**scheduledTransferPeriod**|duration|選用。 指定排程傳輸資料之間的間隔，無條件進位到最接近的分鐘數。<br /><br /> 預設值為 PT0S。|  

## <a name="crashdumps-element"></a>CrashDumps 元素  
 定義損毀傾印目錄。

 父元素︰[Directories 元素](#Directories)。  

屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**container**|字串|要傳輸目錄內容的容器名稱。|  
|**directoryQuotaInMB**|unsignedInt|選用。 指定目錄的大小上限，以 MB 為單位。<br /><br /> 預設值為 0。|  

## <a name="failedrequestlogs-element"></a>FailedRequestLogs 元素  
 定義失敗的要求記錄目錄。

 父元素︰[Directories 元素](#Directories)。  

屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**container**|字串|要傳輸目錄內容的容器名稱。|  
|**directoryQuotaInMB**|unsignedInt|選用。 指定目錄的大小上限，以 MB 為單位。<br /><br /> 預設值為 0。|  

##  <a name="iislogs-element"></a>IISLogs 元素  
 定義 IIS 記錄目錄。

 父元素︰[Directories 元素](#Directories)。  

屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**container**|字串|要傳輸目錄內容的容器名稱。|  
|**directoryQuotaInMB**|unsignedInt|選用。 指定目錄的大小上限，以 MB 為單位。<br /><br /> 預設值為 0。|  

## <a name="datasources-element"></a>DataSources 元素  
 定義零或多個額外的記錄目錄。

 父元素︰[Directories 元素](#Directories)。

## <a name="directoryconfiguration-element"></a>DirectoryConfiguration 元素  
 定義要監視的記錄檔目錄。

 父元素︰[DataSources 元素](#DataSources)。

屬性：

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**container**|字串|要傳輸目錄內容的容器名稱。|  
|**directoryQuotaInMB**|unsignedInt|選用。 指定目錄的大小上限，以 MB 為單位。<br /><br /> 預設值為 0。|  

## <a name="absolute-element"></a>Absolute 元素  
 搭配選擇性環境變數展開來定義要監視之目錄的絕對路徑。

 父元素：[DirectoryConfiguration 元素](#DirectoryConfiguration)。  

屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**路徑**|字串|必要。 要監視之目錄的絕對路徑。|  
|**expandEnvironment**|布林值|必要。 如果設定為 **true**，就會展開路徑中的環境變數。|  

## <a name="localresource-element"></a>LocalResource 元素  
 定義相對於服務定義中所定義之本機資源的路徑。

 父元素：[DirectoryConfiguration 元素](#DirectoryConfiguration)。  

屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**name**|字串|必要。 包含要監視之目錄的本機資源名稱。|  
|**relativePath**|字串|必要。 相對於要監視之本機資源的路徑。|  

## <a name="performancecounters-element"></a>PerformanceCounters 元素  
 定義要收集之效能計數器的路徑。

 父元素：[DiagnosticMonitorConfiguration 元素](#DiagnosticMonitorConfiguration)。


 屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**bufferQuotaInMB**|unsignedInt|選用。 指定適用於所指定資料的檔案系統儲存體數量上限。<br /><br /> 預設值為 0。|  
|**scheduledTransferPeriod**|duration|選用。 指定排程傳輸資料之間的間隔，無條件進位到最接近的分鐘數。<br /><br /> 預設值為 PT0S。|  

## <a name="performancecounterconfiguration-element"></a>PerformanceCounterConfiguration 元素  
 定義要收集的效能計數器。

 父元素︰[PerformanceCounters 元素](#PerformanceCounters)。  

 屬性：  

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**counterSpecifier**|字串|必要。 要收集之效能計數器的路徑。|  
|**sampleRate**|duration|必要。 應收集效能計數器的頻率。|  

## <a name="windowseventlog-element"></a>WindowsEventLog 元素  
 定義要監視的事件記錄。

 父元素：[DiagnosticMonitorConfiguration 元素](#DiagnosticMonitorConfiguration)。

  屬性：

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**bufferQuotaInMB**|unsignedInt|選用。 指定適用於所指定資料的檔案系統儲存體數量上限。<br /><br /> 預設值為 0。|  
|**scheduledTransferLogLevelFilter**|字串|選用。 指定所傳輸記錄項目的最低嚴重性層級。 預設值為 **Undefined**。 其他可能的值為 **Verbose**、**Information**、**Warning**、**Error** 及 **Critical**。|  
|**scheduledTransferPeriod**|duration|選用。 指定排程傳輸資料之間的間隔，無條件進位到最接近的分鐘數。<br /><br /> 預設值為 PT0S。|  

## <a name="datasource-element"></a>DataSource 元素  
 定義要監視的事件記錄。

 父元素︰[WindowsEventLog 元素](#windowsEventLog)。  

 屬性：

|屬性|類型|說明|  
|---------------|----------|-----------------|  
|**name**|字串|必要。 指定要收集之記錄的 XPath 運算式。|  
