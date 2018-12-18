---
title: 對 IIS 使用 Blob 儲存體，對 Azure Log Analytics 中的事件使用表格儲存體 | Microsoft Docs
description: Log Analytics 可以讀取 Azure 服務 (將診斷寫入表格儲存體) 的記錄，或讀取寫入 Blob 儲存體的 IIS 記錄。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: bf444752-ecc1-4306-9489-c29cb37d6045
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/12/2017
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 8f923cc081ea652c8e32d4109225044c70c8767d
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37128736"
---
# <a name="use-azure-blob-storage-for-iis-and-azure-table-storage-for-events-with-log-analytics"></a>針對 Log Analytics 的事件，使用 IIS 和 Azure 表格儲存體的 Azure blob 儲存體

Log Analytics 可以讀取下列服務 (將診斷寫入表格儲存體) 的記錄，或讀取寫入 Blob 儲存體的 IIS 記錄：

* Service Fabric 叢集 (預覽)
* 虛擬機器
* Web/背景工作角色

您必須先啟用 Azure 診斷，Log Analytics 才能收集這些資源的資料。

啟用診斷之後，您可以使用 Azure 入口網站或 PowerShell，設定 Log Analytics 來收集記錄檔。

Azure 診斷是 Azure 的擴充功能，可讓您從背景工作角色、Web 角色或 Azure 中執行的虛擬機器收集診斷資料。 資料會儲存在 Azure 儲存體帳戶中，接著可由 Log Analytics 收集。

若要讓 Log Analytics 收集這些 Azure 診斷記錄，記錄檔必須位於下列位置：

| 記錄類型 | 資源類型 | 位置 |
| --- | --- | --- |
| IIS 記錄檔 |虛擬機器 <br> Web 角色 <br> 背景工作角色 |wad-iis-logfiles (Blob 儲存體) |
| syslog |虛擬機器 |LinuxsyslogVer2v0 (表格儲存體) |
| Service Fabric 運作事件 |Service Fabric 節點 |WADServiceFabricSystemEventTable |
| Service Fabric Reliable Actor 事件 |Service Fabric 節點 |WADServiceFabricReliableActorEventTable |
| Service Fabric Reliable Service 事件 |Service Fabric 節點 |WADServiceFabricReliableServiceEventTable |
| Windows 事件記錄檔 |Service Fabric 節點 <br> 虛擬機器 <br> Web 角色 <br> 背景工作角色 |WADWindowsEventLogsTable (表格儲存體) |
| Windows ETW 記錄檔 |Service Fabric 節點 <br> 虛擬機器 <br> Web 角色 <br> 背景工作角色 |WADETWEventTable (表格儲存體) |

> [!NOTE]
> 的 IIS 記錄檔，以啟用額外的見解。
>
>

若為虛擬機器，您可以選擇將 [Log Analytics agent](log-analytics-azure-vm-extension.md) 安裝到虛擬機器中以啟用其他見解。 除了分析 IIS 記錄檔和事件記錄檔之外，您也可以執行其他分析，包括組態變更追蹤、SQL 評估及更新評估。

## <a name="enable-azure-diagnostics-in-a-virtual-machine-for-event-log-and-iis-log-collection"></a>為事件記錄檔和 IIS 記錄檔集合啟用虛擬機器中的 Azure 診斷
您可以搭配使用下列程序與 Microsoft Azure 入口網站，為事件記錄檔和 IIS 記錄檔集合啟用虛擬機器中的 Azure 診斷。

### <a name="to-enable-azure-diagnostics-in-a-virtual-machine-with-the-azure-portal"></a>使用 Azure 入口網站啟用虛擬機器中的 Azure 診斷
1. 建立虛擬機器時安裝 VM 代理程式。 如果虛擬機器已經存在，請確認已安裝 VM 代理程式。

   * 在 Azure 入口網站中，瀏覽至虛擬機器，依序選取 [選擇性組態] 及 [診斷]，並將 [狀態] 設為 [開啟]。

     完成時，VM 中就會安裝好 Azure 診斷擴充功能並執行。 此擴充功能會負責收集您的診斷資料。
2. 在現有的 VM 上啟用監視和設定事件記錄。 您可以在 VM 層級上啟用診斷。 若要啟用診斷然後設定事件記錄，請執行下列步驟：

   1. 選取 VM。
   2. 按一下 [監視] 。
   3. 按一下 [診斷] 。
   4. 將 [狀態] 設為 [開啟]。
   5. 選取您想要收集的每個診斷記錄。
   6. 按一下 [確定]。

## <a name="enable-azure-diagnostics-in-a-web-role-for-iis-log-and-event-collection"></a>在 Web 角色中針對 IIS 記錄檔和事件集合啟用 Azure 診斷
請參閱[如何在雲端服務中啟用診斷](../cloud-services/cloud-services-dotnet-diagnostics.md)瞭解啟用 Azure 診斷的一般步驟。 下面的指示會使用此資訊並自訂它來與 Log Analytics 搭配使用。

啟用 Azure 診斷時：

* 預設會儲存 IIS 記錄檔，並在 scheduledTransferPeriod 傳輸間隔傳輸記錄檔資料。
* 預設不會傳輸 Windows 事件記錄檔。

### <a name="to-enable-diagnostics"></a>啟用診斷
若要啟用 Windows 事件記錄檔，或是變更 scheduledTransferPeriod，請使用 XML 組態檔 (diagnostics.wadcfg) 設定 Azure 診斷，如[步驟 4：建立您的診斷組態檔並安裝擴充功能](../cloud-services/cloud-services-dotnet-diagnostics.md)所示

下列範例組態檔會從應用程式和系統記錄檔中收集 IIS 記錄檔和所有事件：

```
    <?xml version="1.0" encoding="utf-8" ?>
    <DiagnosticMonitorConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"
          configurationChangePollInterval="PT1M"
          overallQuotaInMB="4096">

      <Directories bufferQuotaInMB="0"
         scheduledTransferPeriod="PT10M">  
        <!-- IISLogs are only relevant to Web roles -->
        <IISLogs container="wad-iis" directoryQuotaInMB="0" />
      </Directories>

      <WindowsEventLog bufferQuotaInMB="0"
         scheduledTransferLogLevelFilter="Verbose"
         scheduledTransferPeriod="PT10M">
        <DataSource name="Application!*" />
        <DataSource name="System!*" />
      </WindowsEventLog>

    </DiagnosticMonitorConfiguration>
```

請確定 ConfigurationSettings 指定儲存體帳戶，如下列範例所示︰

```
    <ConfigurationSettings>
       <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="DefaultEndpointsProtocol=https;AccountName=<AccountName>;AccountKey=<AccountKey>"/>
    </ConfigurationSettings>
```

**AccountName** 和 **AccountKey** 值可在 Azure 入口網站中儲存體帳戶儀表板的 [管理存取金鑰] 底下找到。 連接字串的通訊協定必須為 **https**。

將更新的診斷組態套用至雲端服務，且該服務已開始將診斷寫入 Azure 儲存體時，您即可設定 Log Analytics。

## <a name="use-the-azure-portal-to-collect-logs-from-azure-storage"></a>使用 Azure 入口網站從 Azure 儲存體收集記錄
您可以使用 Azure 入口網站，設定 Log Analytics 來收集下列 Azure 服務的記錄︰

* Service Fabric 叢集
* 虛擬機器
* Web/背景工作角色

在 Azure 入口網站中，瀏覽至您的 Log Analytics 工作區並執行下列工作︰

1. 按一下 [儲存體帳戶記錄]
2. 按一下 [新增] 工作
3. 選取包含診斷記錄的儲存體帳戶
   * 這可以是傳統儲存體帳戶或 Azure Resource Manager 儲存體帳戶
4. 選取您想要收集記錄的資料類型
   * 可以選擇的項目為 IIS 記錄、事件、Syslog (Linux)、ETW 記錄檔、Service Fabric 事件
5. 「來源」的值會根據資料類型自動填入且無法變更
6. 按一下 [確定] 以儲存組態

針對您想要 Log Analytics 收集的其他儲存體帳戶及資料類型，重複步驟 2-6。

大約 30 分鐘內，您就能夠在 Log Analytics 中看到來自儲存體帳戶的資料。 您只會看到套用組態之後寫入儲存體的資料。 Log Analytics 不會從儲存體帳戶讀取預先存在的資料。

> [!NOTE]
> 入口網站不會驗證「來源」是否存在於儲存體帳戶，或是否已寫入新的資料。
>
>

## <a name="enable-azure-diagnostics-in-a-virtual-machine-for-event-log-and-iis-log-collection-using-powershell"></a>在虛擬機器中使用 PowerShell 來啟用 Azure 診斷以收集事件記錄檔和 IIS 記錄檔
使用[設定 Log Analytics 來為 Azure 診斷編製索引](log-analytics-powershell-workspace-configuration.md#configuring-log-analytics-to-index-azure-diagnostics) 中的步驟，以使用 PowerShell 從寫入表格儲存體的 Azure 診斷讀取資料。

您可以使用 Azure PowerShell，更精確地指定寫入至 Azure 儲存體的事件。
如需詳細資訊，請參閱[在 Azure 虛擬機器中啟用診斷](../virtual-machines-dotnet-diagnostics.md)。

您可以使用下列 PowerShell 指令碼啟用和更新 Azure 診斷。
您也可以搭配使用此指令碼與自訂記錄組態。
請修改指令碼來設定儲存體帳戶、服務名稱和虛擬機器名稱。
指令碼會使用適用於傳統虛擬機器的 Cmdlet。

檢閱下列指令碼範例、複製它、視需要修改它、將範例儲存為 PowerShell 指令碼檔案，然後再執行指令碼。

```
    #Connect to Azure
    Add-AzureAccount

    # settings to change:
    $wad_storage_account_name = "myStorageAccount"
    $service_name = "myService"
    $vm_name = "myVM"

    #Construct Azure Diagnostics public config and convert to config format

    # Collect just system error events:
    $wad_xml_config = "<WadCfg><DiagnosticMonitorConfiguration><WindowsEventLog scheduledTransferPeriod=""PT1M""><DataSource name=""System!* "" /></WindowsEventLog></DiagnosticMonitorConfiguration></WadCfg>"

    $wad_b64_config = [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($wad_xml_config))
    $wad_public_config = [string]::Format("{{""xmlCfg"":""{0}""}}",$wad_b64_config)

    #Construct Azure diagnostics private config

    $wad_storage_account_key = (Get-AzureStorageKey $wad_storage_account_name).Primary
    $wad_private_config = [string]::Format("{{""storageAccountName"":""{0}"",""storageAccountKey"":""{1}""}}",$wad_storage_account_name,$wad_storage_account_key)

    #Enable Diagnostics Extension for Virtual Machine

    $wad_extension_name = "IaaSDiagnostics"
    $wad_publisher = "Microsoft.Azure.Diagnostics"
    $wad_version = (Get-AzureVMAvailableExtension -Publisher $wad_publisher -ExtensionName $wad_extension_name).Version # Gets latest version of the extension

    (Get-AzureVM -ServiceName $service_name -Name $vm_name) | Set-AzureVMExtension -ExtensionName $wad_extension_name -Publisher $wad_publisher -PublicConfiguration $wad_public_config -PrivateConfiguration $wad_private_config -Version $wad_version | Update-AzureVM
```


## <a name="next-steps"></a>後續步驟
* 針對支援的 Azure 服務[收集 Azure 服務的記錄檔與計量](log-analytics-azure-storage.md)。
* [啟用解決方案](log-analytics-add-solutions.md) 以提供資料的深入見解。
* [使用搜尋查詢](log-analytics-log-searches.md) 以分析資料。
