---
title: Azure SDK for .NET 2.8 版本資訊
description: Azure SDK for .NET 2.8 版本資訊
services: app-service\web
documentationcenter: .net
author: chrissfanos
editor: ''
ms.assetid: de7207ff-ba4f-4008-9141-8742fcaa3254
ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 02/24/2017
ms.author: juliako
ms.openlocfilehash: 0b9f55d69c824e86245738a082f95fc529583f58
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "22985953"
---
# <a name="azure-sdk-for-net-28-281-and-282"></a>Azure SDK for .NET 2.8、2.8.1 和 2.8.2
## <a name="overview"></a>概觀
本文包含 Azure SDK for .NET 2.8、2.8.1 和 2.8.2 版本的版本資訊 (包含已知問題和重大變更)。 

如需這一版中的新功能和更新的完整清單，請參閱 [Azure SDK 2.8 for Visual Studio 2013 and Visual Studio 2015](https://azure.microsoft.com/blog/announcing-the-azure-sdk-2-8-for-net/) 公告。 

## <a name="azure-sdk-for-net-28"></a>Azure SDK for .NET 2.8
### <a name="download-azure-sdk-for-net-28"></a>下載 Azure SDK for .NET 2.8
[Azure SDK for .NET 2.8 for Visual Studio 2015](http://go.microsoft.com/fwlink/?LinkId=699285) 

[Azure SDK for .NET 2.8 for Visual Studio 2013](http://go.microsoft.com/fwlink/?LinkId=699287)

### <a name="net-452-support"></a>.NET 4.5.2 支援
#### <a name="known-issues"></a>已知問題
Azure.NET SDK 2.8 可讓您建立 .NET 4.5.2 雲端服務套件。 不過，.NET 4.5.2 Framework 要到 2016 年 1 月的客體作業系統版本，才會安裝在預設的客體作業系統映像上。 在這之前，可以透過獨立的客體作業系統版本 November 2015-02 取得 .NET 4.5.2 Framework。 請參閱 [Azure 客體 OS 版本與 SDK 相容性矩陣](../cloud-services/cloud-services-guestos-update-matrix.md) 頁面以追蹤該映像何時發行。  一旦 November 2015-02 映像發行，您便可選擇透過更新您的雲端服務組態檔 (.cscfg) 來使用該映像。 在服務組態檔中，將 ServiceConfiguration 項目的 osVersion 屬性設定為字串 "WA-GUEST-OS-4.26_201511-02"。 如果您選擇加入以使用此映像，那麼您將不會再收到客體 OS 的自動更新。 若要取得自動更新，osVersion 必須設為 “*”，且 .NET 4.5.2 將只能在 2016 年 1 月透過自動更新取得。

### <a name="azure-data-factory"></a>Azure Data Factory
#### <a name="known-issues"></a>已知問題
如果電腦上安裝的 Azure Power Shell 版本比 0.9.8 新，則在與範例資料相關的 **Data Factory 範本** 專案建立期間，Azure Power Shell 指令碼可能會失敗。

若要順利地建立這種類型的專案，您必須安裝 [Azure Power Shell 0.9.8 版](https://github.com/Azure/azure-powershell/releases/download/v0.9.8-September2015/azure-powershell.0.9.8.msi)。

### <a name="azure-resource-manager-tools"></a>Azure 資源管理員工具
#### <a name="breaking-changes"></a>重大變更
Azure 資源群組專案提供的 PowerShell 指令碼在這個版本中已更新，以搭配新的 Azure PowerShell Cmdlet 1.0 版使用。  使用 2.8 以前的 SDK 版本時，將無法從 Visual Studio 使用這個新的指令碼。  

使用 2.8 SDK 時，將無法從 Visual Studio 執行在舊版 SDK 建立的專案中的指令碼。  使用適當的 Azure PowerShell Cmdlet 版本，所有指令碼將可繼續在 Visual Studio 以外使用。  

2.8 SDK 需要 1.0 版的 Azure PowerShell Cmdlet。  所有其他版本的 SDK 則需要 0.9.8 版的 Azure PowerShell Cmdlet。  如需詳細資訊，請參閱 [此部落格](http://go.microsoft.com/fwlink/?LinkID=623011) 。

### <a name="web-tools-extensions"></a>Web 工具擴充功能
#### <a name="known-issues"></a>已知問題
下列版本將解決下列已知問題。

* 非生產環境 (例如 Azure China 或 Azure Stack 客戶) 的 App Service 相關雲端與伺服器總管筆勢無效。 對於這些受影響區域中的客戶，從 Azure 入口網站下載發行設定檔就能發行。 未來版本將為 Azure China 或 Azure Stack 客戶修復筆勢，例如「連結偵錯工具」和「檢視資料流記錄」。 
* 客戶在建立 App Service 時，若要部署的目標 App Insights 執行個體位於美國東部以外的區域，他們可能會看到錯誤。 在這些情況下，在入口網站建立 App Service 和下載發行設定檔就能發行。 

### <a name="azure-hdinsight-tools"></a>Azure HDInsight 工具
#### <a name="new-updates"></a>新的更新
* 您可以透過 HiveServer2 在叢集中執行 Hive 查詢，而且幾乎不需要額外支出，並可即時查看工作記錄檔。
* 使用新的 Hive 工作執行檢視，您可以深入探究您的工作、尋找更多詳細資料和識別潛在的問題。

如需詳細資訊，請參閱 [適用於 Visual Studio 2013 和 Visual Studio 2015 的 Azure SDK 2.8](https://azure.microsoft.com/blog/announcing-the-azure-sdk-2-8-for-net/)。 

## <a name="azure-sdk-for-net-281"></a>Azure SDK for .NET 2.8.1
### <a name="known-issues-for-visual-studio-2013-and-visual-studio-2015"></a>Visual Studio 2013 和 Visual Studio 2015 的已知問題
1. 以位置為目標的觸發 WebJob 發佈會顯示錯誤，並且不會設定排程，不過它會將 WebJob 推送到 Azure。 需要已排程工作的客戶可以使用 Azure 入口網站來設定 WebJob 的排程。 
2. Python 客戶可能會遭遇到偵錯工具問題。 服務小組即將推出此問題的修正，不過如果客戶受到影響，請透過論壇、公告部落格或版本資訊意見區段知會 Microsoft。 
3. 某些區域 (如印度南部) 的客戶將遭遇 App Service 佈建錯誤。 這個情況與入口網站相符，遭遇這個問題的客戶可以使用 Azure 入口網站來要求這些地理區域的發佈權限。 一旦使用 Azure 入口網站來要求這些區域的存取權限後，佈建應該能正常運作。 

## <a name="azure-sdk-for-net-282"></a>Azure SDK for .NET 2.8.2
在安裝 2.8.2 工具之後，客戶可能會遇到下列問題。         

* 如果您使用 Windows 10，且尚未安裝 Internet Explorer，您可能會收到「找不到 Internet Explorer」的錯誤。
  若要解決此問題，請使用 [新增/移除 Windows 元件] 對話方塊來安裝 Internet Explorer。

如果您發現這個問題，請使用 [傳送笑臉] 功能來報告它。

如需詳細資訊，請參閱 [這篇](https://azure.microsoft.com/blog/announcing-azure-sdk-2-8-2-for-net/) 文章。

## <a name="other-updates"></a>其他更新
如需其他更新，請參閱 [Azure SDK 2.8 公告文章](https://azure.microsoft.com/blog/announcing-the-azure-sdk-2-8-for-net/)。

## <a name="also-see"></a>另請參閱
[Azure SDK 2.8 公告文章](https://azure.microsoft.com/blog/announcing-the-azure-sdk-2-8-for-net/)

[Azure SDK for .NET 和 API 的支援和停用資訊](https://msdn.microsoft.com/library/azure/dn479282.aspx)

