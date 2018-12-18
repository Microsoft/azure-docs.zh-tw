---
title: Azure Application Insights OpenCensus 分散式追蹤本機轉送工具 | Microsoft docs
description: 將 OpenCensus 分散式追蹤 (trace) 和跨度 (span) 從 Python 和 Go 等語言轉送給 Azure Application Insights
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/18/2018
ms.reviewer: nimolnar
ms.author: mbullwin
ms.openlocfilehash: ddfcb90090d82d8fe947292737163a81c715b32d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972822"
---
# <a name="local-forwarder"></a>本機轉送工具

本機轉送工具是會從各種 SDK 收集 Application Insights 或 [OpenCensus](https://opencensus.io/) 遙測資料的代理程式，並且會將這些資料路由至 Application Insights。 此工具能夠在 Windows 和 Linux 下執行。 您也可以在 macOS 下執行，但目前尚未正式支援。

## <a name="running-local-forwarder"></a>執行本機轉送工具

本機轉送工具是 [GitHub 上的開放原始碼專案](https://github.com/Microsoft/ApplicationInsights-LocalForwarder/releases)。 在多個平台上執行本機轉送工具的方式有很多種。

### <a name="windows"></a>Windows

#### <a name="windows-service"></a>Windows 服務

在 Windows 下執行本機轉送工具最簡單的方式是將其安裝為 Windows 服務。 該版本隨附可輕鬆向作業系統註冊的 Windows 服務可執行檔 (WindowsServiceHost/Microsoft.LocalForwarder.WindowsServiceHost.exe)。

> [!NOTE]
> 本機轉送工具服務至少需要 .NET Framework 4.7。 如果您沒有 .NET Framework 4.7，服務還是會安裝，但不會啟動。 若要存取最新版本的 .NET Framework，**[請瀏覽 .NET Framework 下載頁面](
https://www.microsoft.com/net/download/dotnet-framework-runtime/net472?utm_source=getdotnet&utm_medium=referral)**。

1. 在 GitHub 上的[本機轉送工具版本頁面](https://github.com/Microsoft/ApplicationInsights-LocalForwarder/releases)中，下載 LF.WindowsServiceHost.zip 檔案。

    ![本機轉送工具版本下載頁面的螢幕擷取畫面](.\media\opencensus-local-forwarder\001-local-forwarder-windows-service-host-zip.png)

2. 為了方便示範，在此範例中我們會將 .zip 檔案直接擷取到此路徑：`C:\LF-WindowsServiceHost`。

    若要註冊此服務，並將其設定為在系統開機時啟動，請以系統管理員身分從命令列中執行下列命令：

    ```
    sc create "Local Forwarder" binpath="C:\LF-WindowsServiceHost\Microsoft.LocalForwarder.WindowsServiceHost.exe" start=auto
    ```
    
    您應該會收到如下所示的回應：
    
    `[SC] CreateService SUCCESS`
    
    若要透過服務 GUI 檢查您的新服務，請輸入 ``services.msc``
        
     ![本機轉送工具服務的螢幕擷取畫面](.\media\opencensus-local-forwarder\002-services.png)

3. **以滑鼠右鍵按一下**新的本機轉送工具，然後選取 [啟動]。 您的服務現在會進入執行狀態。

4. 服務建立時並沒有預設任何復原動作。 您可以**按一下滑鼠右鍵**並選取 [屬性] > [復原]，以設定服務錯誤的自動回應。

    或是，如果您想要以程式設計方式設定發生錯誤時的自動復原選項，您可以使用：

    ```
    sc failure "Local Forwarder" reset= 432000 actions= restart/1000/restart/1000/restart/1000
    ```

5. 在 ``Microsoft.LocalForwarder.WindowsServiceHost.exe`` 檔案所在的位置中 (在此範例中是 ``C:\LF-WindowsServiceHost``) 有名為 ``LocalForwarder.config`` 的檔案。 這是一個 xml 架構的檔案，可讓您調整 localforwader 的組態，並指定 Application Insights 資源 (分散式追蹤資料會轉送到此) 的檢測金鑰。 

    在編輯 ``LocalForwarder.config`` 檔案以新增檢測金鑰之後，必須重新啟動**本機轉送工具服務**，您的變更才會生效。
    
6. 若要確認您所需的設定是否已就緒，以及本機轉送工具是否正如預期般接聽追蹤資料，請檢查 ``LocalForwarder.log`` 檔案。 您應該會在檔案底部看到類似下列的影像：

    ![LocalForwarder.log 檔案的螢幕擷取畫面](.\media\opencensus-local-forwarder\003-log-file.png)

#### <a name="console-application"></a>主控台應用程式

針對特定使用案例，將本機轉送工具當作主控台應用程式來執行可能會有些助益。 該版本隨附下列主控台主機的可執行檔版本：
* 架構相依的 .NET Core 二進位檔：/ConsoleHost/publish/Microsoft.LocalForwarder.ConsoleHost.dll。 執行此二進位檔需要安裝 .NET Core 執行階段；請參閱此下載[網頁](https://www.microsoft.com/net/download/dotnet-core/2.1)，以取得詳細資訊。
```batchfile
E:\uncdrop\ConsoleHost\publish>dotnet Microsoft.LocalForwarder.ConsoleHost.dll
```
* 適用於 x86 和 x64 平台的一組獨立式 .NET Core 二進位檔。 這些不需要執行 .NET Core 執行階段。 /ConsoleHost/win-x86/publish/Microsoft.LocalForwarder.ConsoleHost.exe、/ConsoleHost/win-x64/publish/Microsoft.LocalForwarder.ConsoleHost.exe。
```batchfile
E:\uncdrop\ConsoleHost\win-x86\publish>Microsoft.LocalForwarder.ConsoleHost.exe
E:\uncdrop\ConsoleHost\win-x64\publish>Microsoft.LocalForwarder.ConsoleHost.exe
```

### <a name="linux"></a>Linux

如同 Windows，該版本隨附下列主控台主機的可執行檔版本：
* 架構相依的 .NET Core 二進位檔：/ConsoleHost/publish/Microsoft.LocalForwarder.ConsoleHost.dll。 執行此二進位檔需要安裝 .NET Core 執行階段；請參閱此下載[網頁](https://www.microsoft.com/net/download/dotnet-core/2.1)，以取得詳細資訊。

```batchfile
dotnet Microsoft.LocalForwarder.ConsoleHost.dll
```

* 適用於 linux-64 平台的一組獨立式 .NET Core 二進位檔。 此版本不需要執行 .NET Core 執行階段。 /ConsoleHost/linux-x64/publish/Microsoft.LocalForwarder.ConsoleHost。

```batchfile
user@machine:~/ConsoleHost/linux-x64/publish$ sudo chmod +x Microsoft.LocalForwarder.ConsoleHost
user@machine:~/ConsoleHost/linux-x64/publish$ ./Microsoft.LocalForwarder.ConsoleHost
```

許多 Linux 使用者想以精靈形式來執行本機轉送工具。 Linux 系統隨附許多服務管理解決方案，例如 Upstart、sysv 或 systemd。 無論您的特定版本為何，您都能夠以最適合您案例的方式使用解決方案來執行本機轉送工具。

例如，讓我們使用 systemd 來建立精靈服務。 我們將使用架構相依版本，但您也可以對獨立式版本使用相同步驟。

* 建立下列名為 localforwarder.service 的服務檔案，並將它放入 /lib/systemd/system。
此範例假設您的使用者名稱是 SAMPLE_USER，而且您已經將本機轉送工具的架構相依二進位檔 (在 */ConsoleHost/publish* 中) 複製到 /home/SAMPLE_USER/LOCALFORWARDER_DIR。

```
# localforwarder.service
# Place this file into /lib/systemd/system/
# Use 'systemctl enable localforwarder' to start the service automatically on each boot
# Use 'systemctl start localforwarder' to start the service immediately

[Unit]
Description=Local Forwarder service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=SAMPLE_USER
WorkingDirectory=/home/SAMPLE_USER/LOCALFORWARDER_DIR
ExecStart=/usr/bin/env dotnet /home/SAMPLE_USER/LOCALFORWARDER_DIR/Microsoft.LocalForwarder.ConsoleHost.dll noninteractive

[Install]
WantedBy=multi-user.target
```

* 執行下列命令以指示 systemd 在每次開機時啟動本機轉送工具

```
systemctl enable localforwarder
```

* 執行下列命令以指示 systemd 立即啟動本機轉送工具

```
systemctl start localforwarder
```

* 若要監視服務，可檢查 /home/SAMPLE_USER/LOCALFORWARDER_DIR 目錄中的 *.log 檔案。

### <a name="mac"></a>Mac
本機轉送工具可在 macOS 上執行，但目前尚未正式支援。

### <a name="self-hosting"></a>自我裝載
本機轉送工具也會散發為 .NET Standard NuGet 套件，可讓您將其裝載在自己的 .NET 應用程式中。

```C#
using Library;
...
Host host = new Host();

// see section below on configuring local forwarder
string configuration = ...;
    
host.Run(config, TimeSpan.FromSeconds(5));
...
host.Stop();
```

## <a name="configuring-local-forwarder"></a>設定本機轉送工具

* 當您執行其中一個本機轉送工具自有的主機時 (主控台主機或 Windows 服務主機)，您會發現 **LocalForwarder.config** 位在二進位檔旁邊。
* 自我裝載本機轉送工具 NuGet 時，必須在程式碼中提供相同格式的組態 (請參閱自我裝載區段)。 如需組態語法，請查看 GitHub 存放庫中的 [LocalForwarder.config](https://github.com/Microsoft/ApplicationInsights-LocalForwarder/blob/master/src/ConsoleHost/LocalForwarder.config)。 

> [!NOTE]
> 組態可能會隨著版本變更，因此請注意您使用的版本。

## <a name="monitoring-local-forwarder"></a>監視本機轉送工具

追蹤會寫出至可執行檔 (用於執行本機轉送工具) 旁的檔案系統 (尋找 *.log 檔)。 您可以將名為 NLog.config 的檔案放在可執行檔旁邊，以您自己的組態取代預設組態。 請參閱[文件](https://github.com/NLog/NLog/wiki/Configuration-file#configuration-file-format)以了解格式描述。

如果未提供任何組態檔案 (也就是預設狀態)，本機轉送工具會使用預設組態，您可以[在此處](https://github.com/Microsoft/ApplicationInsights-LocalForwarder/blob/master/src/Common/NLog.config)找到該組態。

## <a name="next-steps"></a>後續步驟

* [Open Census](https://opencensus.io/)
