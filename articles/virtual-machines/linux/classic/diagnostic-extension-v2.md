---
title: 用 VM 擴充功能監視 Linux VM | Microsoft Docs
description: 了解如何使用 Linux 診斷延伸模組監視 Azure 中 Linux VM 的效能和診斷資料。
services: virtual-machines-linux
author: NingKuang
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: f54a11c5-5a0e-40ff-af6c-e60bd464058b
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 12/15/2015
ms.author: Ning
ms.openlocfilehash: 13d7594c15959661f3f9c3ab2165739719beac07
ms.sourcegitcommit: cfff72e240193b5a802532de12651162c31778b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39308216"
---
# <a name="use-the-linux-diagnostic-extension-to-monitor-the-performance-and-diagnostic-data-of-a-linux-vm"></a>使用 Linux 診斷擴充功能監視 Linux VM 的效能和診斷資料

本文說明 Linux 診斷擴充功能 2.3 版。

> [!IMPORTANT]
> 此版本已被取代，且可能會在 2018 年 6 月 30 日之後取消發佈。 3.0 版已取代該版本。 如需詳細資訊，請參閱 [Linux 診斷擴充功能3.0 版的文件](../diagnostic-extension.md)。

## <a name="introduction"></a>簡介

(**注意**：Linux 診斷擴充功能是 [GitHub(英文)](https://github.com/Azure/azure-linux-extensions/tree/master/Diagnostic) 上的開放原始碼，GitHub 是擴充功能的最新資訊第一次發佈之處。 您應先查看 [GitHub 頁面](https://github.com/Azure/azure-linux-extensions/tree/master/Diagnostic)。)

Linux 診斷延伸模組可協助使用者監視在 Microsoft Azure 上執行的 Linux VM。 它可提供下列功能：

* 從 Linux VM 收集系統效能資訊並上傳至使用者的儲存體資料表，包括診斷和 syslog 資訊。
* 讓使用者自訂將要收集並上傳的資料計量。
* 讓使用者將指定的記錄檔上傳至指定的儲存體資料表。

在目前版本 2.3 版中，資料包括：

* 所有 Linux Rsyslog 記錄檔，包括系統、安全性及應用程式記錄檔。
* 在 [System Center 跨平台解決方案網站](https://scx.codeplex.com/wikipage?title=xplatproviders)上指定的所有系統資料。
* 使用者指定的記錄檔。

此延伸模組可搭配傳統和 Resource Manager 部署模型使用。

### <a name="current-version-of-the-extension-and-deprecation-of-old-versions"></a>擴充功能的目前版本和已被取代的舊版本

擴充功能的最新版本是 **2.3**，**所有舊版本 (2.0、2.1 及 2.2) 都將在今年 (2017 年) 底被取代及取消發佈**。 如果您已經安裝停用自動升級次要版本的 Linux 診斷擴充功能，強烈建議您解除安裝該擴充功能，再重新安裝啟用自動升級次要版本的擴充功能。 在傳統 (ASM) VM 上，如果是透過 Azure XPLAT CLI 或 Powershell 安裝擴充功能，您可以藉由指定 '2.*' 為版本來達到此目的。 在 ARM VM 上，您可以藉由在 VM 部署範本中加入 '"autoUpgradeMinorVersion": true' 來達到此目的。 此外，擴充功能的任何新安裝應將自動升級次要版本選項開啟。

## <a name="enable-the-extension"></a>啟用延伸模組

使用 [Azure 入口網站](https://portal.azure.com/#)、Azure PowerShell 或 Azure CLI 指令碼，可以啟用此擴充功能。

若要直接從 Azure 入口網站檢視和設定系統和效能資料，請遵循 [Azure 部落格上的這些步驟](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/)進行。

本文著重於如何使用 Azure CLI 命令來啟用及設定延伸模組。 這可讓您直接從儲存體資料表讀取和檢視資料。

請注意，此處描述的設定方法無法用於 Azure 入口網站。 若要直接從 Azure 入口網站檢視和設定系統及效能資料，就必須透過 Azure 入口網站啟用此延伸模組。

## <a name="prerequisites"></a>必要條件

* **Azure Linux Agent 2.0.6 版或更新版本**。

  請注意，大部分的 Azure VM Linux 資源庫映像包含版本 2.0.6 或更新版本。 您可以執行 **WAAgent -version** 來確認 VM 上安裝的版本。 如果執行的 VM 版本早於 2.0.6，您可以依照 [GitHub 上的這些指示](https://github.com/Azure/WALinuxAgent "指示") 更新它。
* **Azure CLI**。 遵循[本指引來安裝 CLI](../../../cli-install-nodejs.md)，以在電腦上設定 Azure CLI 環境。 安裝 Azure CLI 之後，您可以在命令列介面 (Bash、終端機或命令提示字元) 使用 **azure** 命令來存取 Azure CLI 命令。 例如︰

  * 執行 **azure vm extension set --help** 取得詳細的說明資訊。
  * 執行 **azure login** 登入 Azure。
  * 執行 **azure vm list** 列出您在 Azure 上的所有虛擬機器。
* 儲存資料的儲存體帳戶。 您需要先前建立的儲存體帳戶名稱和存取金鑰，才能將資料上傳至您的儲存體。

## <a name="use-the-azure-cli-command-to-enable-the-linux-diagnostic-extension"></a>使用 Azure CLI 命令啟動 Linux 診斷延伸模組

### <a name="scenario-1-enable-the-extension-with-the-default-data-set"></a>案例 1. 以預設的資料集啟動延伸模組

2.3 版或更新版本，預設將收集的資料包括︰

* 所有 Rsyslog 資訊 (包括系統、安全性及應用程式記錄檔)。  
* 一組核心基礎系統資料。 請注意，[System Center 跨平台解決方案網站](https://scx.codeplex.com/wikipage?title=xplatproviders)上會說明完整資料集。
  如果您想要啟用額外的資料，請繼續進行案例 2 和 3 的步驟。

步驟 1. 使用下列內容建立名為 PrivateConfig.json 的檔案：

    {
        "storageAccountName" : "the storage account to receive data",
        "storageAccountKey" : "the key of the account"
    }

步驟 2. 執行 **azure vm extension set vm_name LinuxDiagnostic Microsoft.OSTCExtensions 2.\* --private-config-path PrivateConfig.json**。

### <a name="scenario-2-customize-the-performance-monitor-metrics"></a>案例 2. 自訂效能監視器計量

本節描述如何自訂效能和診斷資料的資料表。

步驟 1. 使用案例 1 描述的內容建立名為 PrivateConfig.json 的檔案。 同時也建立名為 PublicConfig.json 的檔案。 指定您想要收集的特定資料。

如需所有受支援的提供者和變數，請參考 [System Center 跨平台解決方案網站](https://scx.codeplex.com/wikipage?title=xplatproviders)。 您可以有多個查詢，並藉由將更多查詢附加至指令碼，將它們儲存成多個資料表。

根據預設，一律會收集 Rsyslog 資料。

    {
          "perfCfg":
          [
              {
                  "query" : "SELECT PercentAvailableMemory, AvailableMemory, UsedMemory ,PercentUsedSwap FROM SCX_MemoryStatisticalInformation",
                  "table" : "LinuxMemory"
              }
          ]
    }


步驟 2. 執行 **azure vm extension set vm_name LinuxDiagnostic Microsoft.OSTCExtensions '2.*' --private-config-path PrivateConfig.json --public-config-path PublicConfig.json**。

### <a name="scenario-3-upload-your-own-log-files"></a>案例 3. 上傳您自己的記錄檔

本節說明如何收集特定記錄檔並上傳至儲存體帳戶。 您必須指定記錄檔的路徑，以及要儲存記錄檔的資料表名稱。 您可以將多個檔案/資料表項目加入至指令碼，以建立多個記錄檔。

步驟 1. 使用案例 1 描述的內容建立名為 PrivateConfig.json 的檔案。 然後使用下列內容建立另一個名為 PublicConfig.json 的檔案：

```json
{
    "fileCfg" :
    [
        {
            "file" : "/var/log/mysql.err",
            "table" : "mysqlerr"
            }
    ]
}
```

步驟 2. 執行 `azure vm extension set vm_name LinuxDiagnostic Microsoft.OSTCExtensions '2.*' --private-config-path PrivateConfig.json --public-config-path PublicConfig.json`。

請注意，若使用這項設定，在 2.3 版以前的擴充功能版本上，所有寫入 `/var/log/mysql.err` 的記錄可能也會複製到 `/var/log/syslog` (或 `/var/log/messages`，視 Linux 散發版本而定)。 如果您想要避免此重複的記錄，可以在您的 rsyslog 設定中排除 `local6` 設備記錄檔的記錄功能。 這依 Linux 散發版本而定，但在 Ubuntu 14.04 系統上，要修改的檔案是 `/etc/rsyslog.d/50-default.conf`，您可以將 `*.*;auth,authpriv.none -/var/log/syslog` 這一行取代為 `*.*;auth,authpriv,local6.none -/var/log/syslog`。 在最新的 Hotfix 版本 2.3 (2.3.9007) 中已修正此問題，因此如果您有擴充功能版本 2.3，應該就不會發生此問題。 如果在重新啟動 VM 之後仍然發生此問題，請與我們連絡，並協助我們針對無法自動安裝最新 Hotfix 版本的原因進行疑難排解。

### <a name="scenario-4-stop-the-extension-from-collecting-any-logs"></a>案例 4. 停止擴充收集任何記錄檔

本節說明如何阻止延伸模組收集記錄檔。 請注意，即使使用這項重新設定，監視代理程式處理序仍會啟動並執行。 如果您想要完全停止監視代理程式處理序，您可以停用延伸模組來達到此目的。 停用擴充功能的命令是 `azure vm extension set --disable <vm_name> LinuxDiagnostic Microsoft.OSTCExtensions '2.*'`。

步驟 1. 使用案例 1 描述的內容建立名為 PrivateConfig.json 的檔案。 使用下列內容建立另一個名為 PublicConfig.json 的檔案：

    {
        "perfCfg" : [],
        "enableSyslog" : "false"
    }


步驟 2. 執行 **azure vm extension set vm_name LinuxDiagnostic Microsoft.OSTCExtensions '2.*' --private-config-path PrivateConfig.json --public-config-path PublicConfig.json**。

## <a name="review-your-data"></a>檢閱資料

效能和診斷資料會儲存在 Azure 儲存體資料表中。 檢閱[如何從 Ruby 使用 Azure 表格儲存體](../../../cosmos-db/table-storage-how-to-use-ruby.md)，以了解如何使用 Azure CLI 指令碼存取儲存體資料表中的資料。

此外，您可以使用下列 UI 工具來存取資料：

1. Visual Studio 伺服器總管。 移至您的儲存體帳戶。 VM 執行約 5 分鐘後，您應該會看到四個預設資料表：“LinuxCpu”, ”LinuxDisk”, ”LinuxMemory” 和 ”Linuxsyslog”。 按兩下資料表名稱以檢視資料。
1. [Azure 儲存體總管](https://azurestorageexplorer.codeplex.com/ "Azure 儲存體總管")。

![映像](./media/diagnostic-extension/no1.png)

如果您已啟用 fileCfg 或 perfCfg (如案例 2 和 3 所述)，您可以使用 Visual Studio 伺服器總管和 Azure 儲存體總管來檢視非預設的資料。

## <a name="known-issues"></a>已知問題

* 您只能透過指令碼存取 Rsyslog 資訊和客戶指定記錄檔。
