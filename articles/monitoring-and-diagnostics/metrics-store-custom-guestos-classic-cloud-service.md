---
title: 將客體作業系統計量傳送至 Azure 監視器計量存放區的傳統雲端服務
description: 將客體作業系統計量傳送至 Azure 監視器計量存放區的傳統雲端服務
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: howto
ms.date: 09/24/2018
ms.author: ancav
ms.component: metrics
ms.openlocfilehash: be27ff3f8dda3209a011c3ad79d1a7a1f1d259fe
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986909"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-classic-cloud-service"></a>將客體作業系統計量傳送至 Azure 監視器計量存放區的傳統雲端服務

Azure 監視器 [Windows Azure 診斷擴充功能](azure-diagnostics.md) (WAD) 可讓您從當作虛擬機器、雲端服務或 Service Fabric 叢集一部分執行的客體作業系統 (guestOS) 收集計量與記錄。  擴充功能可以將遙測資料傳送到先前連結的文章中所列的許多不同位置。  

此文章說明將 Azure 傳統雲端服務的客體 OS 效能計量傳送至 Azure 監視器計量存放區的程序。 從 WAD 1.11 版開始，您可以直接將計量寫入到已收集標準平台計量的 Azure 監視器計量存放區。 將它們儲存在此位置可讓您存取與平台計量可用之動作相同的動作。  動作包括近乎即時的警示、圖表、路由、從 REST API 存取以及更多功能。  在過去，WAD 擴充功能會寫入到 Azure 儲存體，而不是 Azure 監視器資料存放區。  

此文章中所述的程序僅適用於 Azure 雲端服務上的效能計數器。 不適合用於其他自訂計量。 
   

## <a name="pre-requisites"></a>先決條件

- 您必須是 Azure 訂用帳戶的[服務管理員或共同管理員](https://docs.microsoft.com/azure/billing/billing-add-change-azure-subscription-administrator.md) 

- 您必須先向 [Microsoft.Insights](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1) 註冊您的訂用帳戶 

- 您需要安裝 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1)，或可以使用 [Azure CloudShell](https://docs.microsoft.com/azure/cloud-shell/overview.md) 


## <a name="provision-cloud-service-and-storage-account"></a>佈建雲端服務與儲存體帳戶 

1. 建立及部署傳統雲端服務 (您可在[這裡](../cloud-services/cloud-services-dotnet-get-started.md)找到範例傳統雲端服務應用程式和部署)。 

2. 您可以使用現有的儲存體帳戶或部署新的儲存體帳戶。 儲存體帳戶最好能與您剛才建立的傳統雲端服務位於相同的區域中。 在 Azure 入口網站中，瀏覽至 [儲存體帳戶資源] 刀鋒視窗，然後選擇 [金鑰]。 記下您在稍後步驟中需要的儲存體帳戶名稱與儲存體帳戶金鑰。

   ![儲存體帳戶金鑰](./media/metrics-store-custom-guestos-classic-cloud-service/storage-keys.png)



## <a name="create-a-service-principal"></a>建立服務主體 

使用在以下位置找到的指示，在您的 Azure Active Directory 租用戶中建立服務主體：../azure/azure-resource-manager/resource-group-create-service-principal-portal。 進行此程序時，請注意下列事項： 
  - 您可以將任何 URL 填入為登入 URL。  
  - 為此應用程式建立新用戶端密碼  
  - 儲存金鑰與用戶端識別碼，以便在稍後的步驟中使用。  

針對您希望對其發出計量的資源，將其「監視計量發行者」權限授與您在上一個步驟中建立的應用程式。 如果您計劃使用該應用程式對許多資源發出自訂計量，您可以在資源群組或訂用帳戶層級上授與這些權限。  

> [!NOTE]
> 診斷擴充功能會使用服務主體，對 Azure 監視器驗證，並發出雲端服務的計量 

## <a name="author-diagnostics-extension-configuration"></a>製作診斷擴充功能設定 

準備 WAD 診斷擴充功能設定檔。 此檔案指定診斷擴充功能應為雲端服務收集的記錄與效能計數器。 下面是範例診斷設定檔。  

```XML
<?xml version="1.0" encoding="utf-8"?> 
<DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
  <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
    <WadCfg> 
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096"> 
        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" /> 
        <Directories scheduledTransferPeriod="PT1M"> 
          <IISLogs containerName="wad-iis-logfiles" /> 
          <FailedRequestLogs containerName="wad-failedrequestlogs" /> 
        </Directories> 
        <PerformanceCounters scheduledTransferPeriod="PT1M"> 
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Page Faults/sec" sampleRate="PT15S" /> 
        </PerformanceCounters> 
        <WindowsEventLog scheduledTransferPeriod="PT1M"> 
          <DataSource name="Application!*[System[(Level=1 or Level=2 or Level=3)]]" /> 
          <DataSource name="Windows Azure!*[System[(Level=1 or Level=2 or Level=3 or Level=4)]]" /> 
        </WindowsEventLog> 
        <CrashDumps> 
          <CrashDumpConfiguration processName="WaIISHost.exe" /> 
          <CrashDumpConfiguration processName="WaWorkerHost.exe" /> 
          <CrashDumpConfiguration processName="w3wp.exe" /> 
        </CrashDumps> 
        <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Error" /> 
      </DiagnosticMonitorConfiguration> 
      <SinksConfig> 
      </SinksConfig> 
    </WadCfg> 
    <StorageAccount /> 
  </PublicConfig> 
  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
    <StorageAccount name="" endpoint="" /> 
</PrivateConfig> 
  <IsEnabled>true</IsEnabled> 
</DiagnosticsConfiguration> 
```

在診斷檔案的 "SinksConfig" 區段中，定義新的 Azure 監視器接收： 

```XML
  <SinksConfig> 
    <Sink name="AzMonSink"> 
    <AzureMonitor> 
      <ResourceId>-Provide ClassicCloudService’s Resource ID-</ResourceId> 
      <Region>-Azure Region your Cloud Service is deployed in-</Region> 
    </AzureMonitor> 
    </Sink> 
  </SinksConfig> 
```

在設定檔列出所要收集之效能計數器的區段中，新增 Azure 監視器接收。 此項目可確保將指定的所有效能計數器作為計量，路由傳送到 Azure 監視器。 根據您的需求新增/移除效能計數器。 

```XML
<PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzMonSink"> 
 <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" /> 
  … 
</PerformanceCounters> 
```
最後，在私人設定中，新增 [Azure 監視器帳戶] 區段。 輸入服務主體用戶端識別碼及在先前步驟中建立的密碼。 

```XML
<PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
  <StorageAccount name="" endpoint="" /> 
    <AzureMonitorAccount> 
      <ServicePrincipalMeta> 
        <PrincipalId>clientId from step 3</PrincipalId> 
        <Secret>client secret from step 3</Secret> 
      </ServicePrincipalMeta> 
    </AzureMonitorAccount> 
</PrivateConfig> 
```
 
在本機儲存此診斷檔案。  

## <a name="deploy-the-diagnostics-extension-to-your-cloud-service"></a>將診斷擴充功能部署到您的雲端服務 

啟動 PowerShell 並登入 Azure 

```PowerShell
Login-AzureRmAccount 
```

使用下列命令，將您在先前步驟中建立的儲存體帳戶詳細資料存放在變數中。 

```PowerShell
$storage_account = <name of your storage account from step 3> 
$storage_keys = <storage account key from step 3> 
```
 
同樣地，使用下列命令將診斷檔案路徑設定到變數中。 

```PowerShell
$diagconfig = “<path of the diagnostics configuration file with the Azure Monitor sink configured>” 
```
 
透過使用下列命令設定的 Azure 監視器接收，使用診斷檔案將診斷擴充功能部署到您的雲端服務 

```PowerShell
Set-AzureServiceDiagnosticsExtension -ServiceName <classicCloudServiceName> -StorageAccountName $storage_account -StorageAccountKey $storage_keys -DiagnosticsConfigurationPath $diagconfig 
```
 
> [!NOTE] 
> 在安裝診斷擴充功能期間，仍會強制您提供儲存體帳戶。 診斷設定檔中指定的任何記錄和/或效能計數器，都會寫入到指定的儲存體帳戶。  

## <a name="plot-metrics-in-the-azure-portal"></a>在 Azure 入口網站中繪製計量 

瀏覽至 Azure 入口網站 

 ![計量 Azure 入口網站](./media/metrics-store-custom-guestos-classic-cloud-service/navigate-metrics.png)

1. 在左側功能表中，按一下 [監視] 

1. 在 [監視] 刀鋒視窗上，按一下 [計量預覽] 索引標籤 

1. 在 [資源] 下拉式清單中，選取您的傳統雲端服務 

1. 在 [命名空間] 下拉式清單中，選取 **azure.vm.windows.guest** 

1. 在 [計量] 下拉式清單中，選取 [記憶體\認可的位元組 (使用中)] 

您可以使用維度篩選與分割功能，選擇檢視特定角色與每個角色執行個體所使用的記憶體總數。 

 ![計量 Azure 入口網站](./media/metrics-store-custom-guestos-classic-cloud-service/metrics-graph.png)

## <a name="next-steps"></a>後續步驟
- 深入了解[自訂計量](metrics-custom-overview.md)。



