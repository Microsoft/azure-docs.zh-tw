---
title: 將客體 OS 計量傳送到 Windows 虛擬機器的 Azure 監視器資料存放區 (傳統)
description: 將客體 OS 計量傳送到 Windows 虛擬機器的 Azure 監視器資料存放區 (傳統)
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ancav
ms.component: ''
ms.openlocfilehash: cb803450f7765ae62292ff3afb7f32209b437f78
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978911"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-data-store-for-a-windows-virtual-machine-classic"></a>將客體 OS 計量傳送到 Windows 虛擬機器的 Azure 監視器資料存放區 (傳統)

Azure 監視器 [Windows Azure 診斷擴充功能](https://docs.microsoft.com/azure/monitoring-and-diagnostics/azure-diagnostics) (WAD) 可讓您從當作虛擬機器、雲端服務或 Service Fabric 叢集一部分執行的客體作業系統 (客體 OS)，收集計量與記錄。 擴充功能可以將遙測資料傳送到先前連結的文章中所列的許多不同位置。

本文說明將 Windows 虛擬機器的客體 OS 效能計量傳送至 Azure 監視器資料存放區的流程。 從 WAD 1.11 版開始，您可以直接將計量寫入已收集標準平台計量的 Azure 監視器計量存放區。 儲存在此位置可讓您存取平台計量可用的相同動作。  動作包括近乎即時的警示、圖表、路由、從 REST API 存取以及更多功能。  過去的 WAD 擴充功能會寫入 Azure 儲存體，而不是 Azure 監視器資料存放區。 

本文所述的流程僅適用於執行 Windows 作業系統的傳統虛擬機器。

## <a name="pre-requisites"></a>先決條件

- 您必須是 Azure 訂用帳戶的[服務管理員或共同管理員](https://docs.microsoft.com/azure/billing/billing-add-change-azure-subscription-administrator.md) 

- 您必須先向 [Microsoft.Insights](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1) 註冊您的訂用帳戶 

- 您需要安裝 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1)，或可以使用 [Azure CloudShell](https://docs.microsoft.com/azure/cloud-shell/overview.md) 

## <a name="create-a-classic-virtual-machine-and-storage-account"></a>建立傳統虛擬機器和儲存體帳戶

1. 使用 Azure 入口網站![建立傳統 VM](./media/metrics-store-custom-guestos-classic-vm/create-classic-vm.png)來建立傳統 VM

1. 建立此 VM 時，請選擇建立新的傳統儲存體帳戶。 我們會在稍後步驟中使用此儲存體帳戶。

1. 在 Azure 入口網站中，瀏覽到儲存體帳戶資源刀鋒視窗，然後選擇 [金鑰] 並記下儲存體帳戶名稱和儲存體帳戶金鑰。 您在稍後步驟![儲存體存取金鑰](./media/metrics-store-custom-guestos-classic-vm/storage-access-keys.png)中會需要這些金鑰

## <a name="create-a-service-principal"></a>建立服務主體

使用[建立服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)中找到的指示，在您的 Azure Active Directory 租用戶中建立服務主體。 進行此流程時，請注意下列事項： 
- 為此應用程式建立新用戶端密碼  
- 儲存金鑰和用戶端識別碼以便在稍後步驟中使用。

將您希望發出計量的資源的權限授予此應用程式「監視計量發行者」。 您可以使用資源群組或整個訂用帳戶。  

> [!NOTE]
> 診斷擴充功能將使用服務主體，對 Azure 監視器驗證，並發出您傳統 VM 的計量。

## <a name="author-diagnostics-extension-configuration"></a>製作診斷擴充功能設定

1. 準備 WAD 診斷擴充功能組態檔。 此檔案指定診斷擴充功能應為您傳統 VM 收集的記錄與效能計數器。 範例如下。

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="applicationInsights.errors">
            <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" />
            <Directories scheduledTransferPeriod="PT1M">
                <IISLogs containerName="wad-iis-logfiles" />
                <FailedRequestLogs containerName="wad-failedrequestlogs" />
            </Directories>
            <PerformanceCounters scheduledTransferPeriod="PT1M">
                <PerformanceCounterConfiguration counterSpecifier="\Processor(*)\% Processor Time" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(*)\Disk Read Bytes/sec" sampleRate="PT15S" />
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
            <Metrics resourceId="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicCompute/virtualMachines/MyClassicVM">
                <MetricAggregation scheduledTransferPeriod="PT1M" />
                <MetricAggregation scheduledTransferPeriod="PT1H" />
            </Metrics>
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
1. 在診斷檔案的 “SinksConfig” 區段中，定義新的 Azure 監視器接收：

    ```xml
    <SinksConfig>
        <Sink name="AzMonSink">
            <AzureMonitor>
                <ResourceId>Provide your Classic VM’s Resource ID </ResourceId>
                <Region>Region your VM is deployed in</Region>
            </AzureMonitor>
        </Sink>
    </SinksConfig>
    ```

1. 在組態檔列出所要收集效能計數器的區段中，將效能計數器路由傳送到 Azure 監視器接收 "AzMonSink"。

    ```xml
    <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzMonSink">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" />
    ...
    </PerformanceCounters>
    ```

1. 在私用組態中定義 Azure 監視器帳戶，並新增用來發出計量的服務主體資訊。

    ```xml
    <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <StorageAccount name="" endpoint="" />
        <AzureMonitorAccount>
            <ServicePrincipalMeta>
                <PrincipalId>clientId for your service principal</PrincipalId>
                <Secret>client secret of your service principal</Secret>
            </ServicePrincipalMeta>
        </AzureMonitorAccount>
    </PrivateConfig>
    ```

1. 將此檔案儲存在本機。

## <a name="deploy-diagnostics-extension-to-your-cloud-service"></a>將診斷擴充功能部署到雲端服務

1. 啟動 PowerShell 並登入

    ```powershell
    Login-AzureRmAccount
    ```

1. 首先將內容設定至您的傳統 VM

    ```powershell
    $VM = Get-AzureVM -ServiceName <VM’s Service_Name> -Name <VM Name>
    ```

1. 設定以 VM 建立的傳統儲存體帳戶的內容。

    ```powershell
    $StorageContext = New-AzureStorageContext -StorageAccountName <name of your storage account from earlier steps> -storageaccountkey "<storage account key from earlier steps>"
    ```

1.  使用下列命令，將診斷檔案路徑設定為變數。

    ```powershell
    $diagconfig = “<path of the diagnostics configuration file with the Azure Monitor sink configured>”
    ```

1.  使用診斷檔案和設定好的 Azure 監視器接收來為您的傳統 VM 準備更新

    ```powershell
    $VM_Update = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $diagconfig -VM $VM -StorageContext $Storage_Context
    ```

1.  執行下列命令，將更新部署至您的 VM

    ```powershell
    Update-AzureVM -ServiceName "ClassicVMWAD7216" -Name "ClassicVMWAD" -VM $VM_Update.VM
    ```

> [!NOTE]
> 在安裝診斷擴充功能期間，仍會強制您提供儲存體帳戶。 診斷組態檔中指定的任何記錄和/或效能計數器，都會寫入指定的儲存體帳戶。

## <a name="plot-the-metrics-in-the-azure-portal"></a>在 Azure 入口網站中繪製計量

1.  瀏覽至 Azure 入口網站

1.  在左側功能表中，按一下 [監視]

1.  在 [監視] 刀鋒視窗上，按一下 [計量]
   ![瀏覽計量](./media/metrics-store-custom-guestos-classic-vm/navigate-metrics.png)

1. 在 [資源] 下拉式清單中，選取您的傳統 VM

1. 在 [命名空間] 下拉式清單中，選取 **azure.vm.windows.guest**

1. 在 [計量] 下拉式清單中，選取 **Memory\Committed Bytes in Use**
   ![繪製計量](./media/metrics-store-custom-guestos-classic-vm/plot-metrics.png)


## <a name="next-steps"></a>後續步驟
- 深入了解[自訂計量](metrics-custom-overview.md)。
