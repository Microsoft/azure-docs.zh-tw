---
title: 雲端服務角色回收的常見原因 | Microsoft Docs
description: 突然回收的雲端服務角色可能會造成顯著的停機時間。 以下是導致角色回收的一些常見問題，或許有助於縮短停機時間。
services: cloud-services
documentationcenter: ''
author: simonxjx
manager: felixwu
editor: ''
tags: top-support-issue
ms.assetid: 533930d1-8035-4402-b16a-cf887b2c4f85
ms.service: cloud-services
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: 20b98fa9656c9c66a81ff98a70fcdbfb29d4dad6
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35767019"
---
# <a name="common-issues-that-cause-roles-to-recycle"></a>導致角色回收的常見問題
本文討論部署問題的常見原因，和可協助您解決這些問題的疑難排解秘訣。 應用程式出現問題的徵候之一，是角色執行個體無法啟動，或是在初始化中、忙碌和停止中狀態之間循環。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="missing-runtime-dependencies"></a>遺失執行階段相依性
如果應用程式中的角色依賴任何不屬於 .NET Framework 或 Azure 受控程式庫的組件，您必須明確地在應用程式封裝中包含該組件。 請記住，其他 Microsoft 架構依預設並未在 Azure 上提供。 如果您的角色依賴這類架構，您必須將這些組件新增至應用程式封裝。

在您建置及封裝應用程式之前，請確認下列項目：

* 如果使用 Visual studio，請確定您不屬於 Azure SDK 或 .NET Framework 的專案中每個參考的組件，[複製到本機] 屬性都設定為 [True]。
* 確定 web.config 檔案未參考 compilation 元素中任何未使用的組件。
* 每個 .cshtml 檔案的 [建置動作] 都設為 [內容]。 這可確保檔案會正確出現在封裝中，並且讓其他參考的檔案出現在封裝中。

## <a name="assembly-targets-wrong-platform"></a>組件以錯誤的平台作為目標
Azure 是 64 位元環境。 因此，針對 32 位元目標編譯的 .NET 組件無法在 Azure 上運作。

## <a name="role-throws-unhandled-exceptions-while-initializing-or-stopping"></a>角色在初始化或停止時會擲回未處理的例外狀況
[RoleEntryPoint] 類別的方法 (包括 [OnStart]、[OnStop] 和 [Run]) 方法所擲回的任何例外狀況，都是未處理的例外狀況。 如果其中有一個方法發生未處理的例外狀況，角色就會進行回收。 如果角色重複回收，它可能會在每次嘗試啟動時擲回未處理的例外狀況。

## <a name="role-returns-from-run-method"></a>角色因 Run 方法而回收
[Run] 方法應該無限期地執行。 如果您的程式碼覆寫 [Run] 方法，此方法應無限期地停用。 如果 [Run] 方法回復，角色即會回收。

## <a name="incorrect-diagnosticsconnectionstring-setting"></a>不正確的 DiagnosticsConnectionString 設定
如果應用程式使用 Azure 診斷，則您的服務組態檔必須指定 `DiagnosticsConnectionString` 組態設定。 此設定應指定您在 Azure 中的儲存體帳戶的 HTTPS 連線。

為了在您將應用程式封裝部署至 Azure 之前確保 `DiagnosticsConnectionString` 設定正確無誤，請確認下列事項：  

* `DiagnosticsConnectionString` 設定指向 Azure 中的有效儲存體帳戶。  
  根據預設，此設定會指向模擬儲存體帳戶，因此您必須在部署應用程式封裝之前明確變更這項設定。 若未變更此設定，當角色執行個體嘗試啟動診斷監視器時，將會擲回例外狀況。 這可能會導致角色執行個體無限期地回收。
* 連接字串是以下列 [格式](../storage/common/storage-configure-connection-string.md)指定。 (通訊協定必須指定為 HTTPS。)以您的儲存體帳戶名稱取代 MyAccountName，並以您的存取金鑰取代 MyAccountKey：    

        DefaultEndpointsProtocol=https;AccountName=MyAccountName;AccountKey=MyAccountKey

  如果您使用 Azure Tools for Microsoft Visual Studio 開發應用程式，您可以使用屬性頁面來設定此值。

## <a name="exported-certificate-does-not-include-private-key"></a>匯出的憑證未包含私密金鑰
若要在 SSL 下執行 Web 角色，您必須確保匯出的管理憑證包含私密金鑰。 如果您使用「Windows 憑證管理員」匯出憑證，請務必針對 [匯出私密金鑰] 選項選取 [是]。 憑證必須匯出為 PFX 格式，這是目前唯一支援的格式。

## <a name="next-steps"></a>後續步驟
檢視更多雲端服務的 [疑難排解文章](https://azure.microsoft.com/documentation/articles/?tag=top-support-issue&product=cloud-services) 。

在以下位置檢視多個角色回收案例： [Kevin Williamson 的部落格系列](http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx)。

[RoleEntryPoint]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx
[OnStart]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx
[OnStop]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[Run]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx
