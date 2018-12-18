---
title: 使用 Visual Studio 發佈 Azure 應用程式精靈 | Microsoft Docs
description: 了解如何在 Visual Studio 發佈 Azure 應用程式精靈中進行各種設定
services: visual-studio-online
author: ghogen
manager: douge
assetId: 7d8f1ac9-e439-47e0-a183-0642c4ea1920
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 03/21/2017
ms.author: ghogen
ms.openlocfilehash: 02d38f49a1bfe490acbcfee95a8a703cf0d7594f
ms.sourcegitcommit: 3c3488fb16a3c3287c3e1cd11435174711e92126
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "31795974"
---
# <a name="using-the-visual-studio-publish-azure-application-wizard"></a>使用 Visual Studio 發佈 Azure 應用程式精靈

在 Visual Studio 中開發 web 應用程式之後，您可以使用 [發佈 Azure 應用程式] 精靈，將應用程式發佈至 Azure 雲端服務。

> [!Note]
> 本文是關於部署到雲端服務，而不是網站。 如需部署到網站的相關資訊，請參閱 [如何部署到 Azure 網站](https://social.msdn.microsoft.com/Search/windowsazure?query=How%20to%20Deploy%20an%20Azure%20Web%20Site&Refinement=138&ac=4#refinementChanges=117&pageNumber=1&showMore=false)。

## <a name="accessing-the-publish-azure-application-wizard"></a>存取發佈 Azure 應用程式精靈

視您擁有的 Visual Studio 專案類型而定，您有兩種方式可存取 [發佈 Azure 應用程式] 精靈。

**如果您有 Azure 雲端服務專案︰**

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後從操作功能表中選取 [發佈]。

**如果您有未啟用 Azure 的 Web 應用程式專案︰**

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後從操作功能表中選取 [轉換] > [轉換為 Azure 雲端服務專案]。 

1. 在 [方案總管] 中，以滑鼠右鍵按一下新建立的 Azure 專案，然後從操作功能表中選取 [發佈]。

## <a name="sign-in-page"></a>登入頁面

![登入頁面](./media/vs-azure-tools-publish-azure-application-wizard/sign-in.png)

**帳戶** - 選取帳戶或選取帳戶下拉式清單中的 [新增帳戶]。

**選擇您的訂用帳戶** - 選擇要用於部署的訂用帳戶。

## <a name="settings-page---common-settings-tab"></a>設定頁面 - 一般設定索引標籤

![一般設定](./media/vs-azure-tools-publish-azure-application-wizard/settings-common-settings.png)

**雲端服務** - 使用下拉式清單中，選取現有的雲端服務，或選取 &lt;新建>，然後建立雲端服務。 資料中心會針對每項雲端服務顯示於括號內。 建議雲端服務的資料中心位置與儲存體帳戶的資料中心位置相同 (進階設定)。

**環境** - 選取 [生產] 或 [預備]。 如果您想要在測試環境中部署應用程式，請選擇預備環境。 

**建置組態** - 選取 [偵錯] 或 [發行]。

**服務組態** - 選取 [雲端] 或 [本機]。

**啟用所有角色的遠端桌面**：如果您想要從遠端連線到服務，請選取此選項。 此選項主要用於疑難排解。 如需詳細資訊，請參閱[使用 Visual Studio 啟用 Azure 雲端服務中角色的遠端桌面連線](cloud-services/cloud-services-role-enable-remote-desktop-visual-studio.md)。

**啟用所有 Web 角色的 Web Deploy**：選取此選項以啟用服務的 Web 部署。 您也必須選取 [啟用所有角色的遠端桌面] 選項，才能使用這項功能。 如需詳細資訊，請參閱[使用 Visual Studio 發佈雲端服務](vs-azure-tools-publishing-a-cloud-service.md)。

## <a name="settings-page---advanced-settings-tab"></a>設定頁面 - 進階設定索引標籤

![進階設定](./media/vs-azure-tools-publish-azure-application-wizard/settings-advanced-settings.png)

**部署標籤**：接受預設名稱，或輸入您所選擇的名稱。 若要將日期附加至部署標籤，請選取核取方塊。 

**儲存體帳戶**：選取想用於此部署的儲存體帳戶，**&lt;新建> 可建立儲存體帳戶。 資料中心會針對每個儲存體帳戶顯示於括號內。 建議使儲存體帳戶的資料中心位置與雲端服務的資料中心位置相同 (一般設定)。

Azure 儲存體帳戶會儲存應用程式部署的封裝。 部署應用程式之後，封裝會從儲存體帳戶中移除。

**失敗時刪除部署**- 選取此選項，可以在發佈期間遇到任何錯誤時刪除部署。 如果您想要針對您的雲端服務維護固定的虛擬 IP 位址，應該取消核取此選項。

**部署更新** - 如果您只想要部署已更新的元件，請選取此選項。 這種部署類型比完整部署更快速。 如果您想要針對您的雲端服務維護固定的虛擬 IP 位址，應該核取此選項。 

**部署更新 - 設定** - 此對話方塊用來進一步指定您希望更新角色的方式。 如果您選擇 [累加式更新]，就會逐一更新應用程式的每個執行個體，讓應用程式隨時可供使用。 如果您選擇 [同時更新]，即會同時更新應用程式的所有執行個體。 同時更新較為快速，但是您的服務可能無法在更新過程中使用。

![部署設定](./media/vs-azure-tools-publish-azure-application-wizard/deployment-settings.png)

**啟用 IntelliTrace** - 指定您是否要啟用 IntelliTrace。 有了 IntelliTrace，您可以於角色執行個體在 Azure 中執行時，記錄其廣泛的偵錯資訊。 如果您需要找出問題的原因，您可以從 Visual Studio 使用 IntelliTrace 記錄檔來瀏覽程式碼，如同它是在 Azure 中執行。 如需使用 IntelliTrace 的詳細資訊，請參閱[使用 Visual Studio 和 IntelliTrace 進行已發佈 Azure 雲端服務的偵錯](./vs-azure-tools-intellitrace-debug-published-cloud-services.md)。

**啟用分析** - 指定您是否要啟用效能分析。 Visual Studio 分析工具可讓您取得雲端服務執行情況在計算方面的深入分析。 如需有關如何使用 Visual Studio 分析工具的詳細資訊，請參閱[測試 Azure 雲端服務的效能](./vs-azure-tools-performance-profiling-cloud-services.md)。

**啟用所有角色的遠端偵錯工具** - 指定您是否要啟用遠端偵錯。 如需使用 Visual Studio 進行雲端服務偵錯的詳細資訊，請參閱[在 Visual Studio 中進行 Azure 雲端服務或虛擬機器的偵錯](./vs-azure-tools-debug-cloud-services-virtual-machines.md)。

## <a name="diagnostics-settings-page"></a>診斷設定頁面

![診斷設定](./media/vs-azure-tools-publish-azure-application-wizard/diagnostic-settings.png)

診斷可讓您針對 Azure 雲端服務 (或 Azure 虛擬機器) 進行疑難排解。 如需診斷的相關資訊，請參閱[為 Azure 雲端服務和虛擬機器設定診斷功能](./vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md)。 如需 Application Insights 的相關資訊，請參閱[什麼是 Application Insights？](./application-insights/app-insights-overview.md)。

## <a name="summary-page"></a>摘要頁面

![總結](./media/vs-azure-tools-publish-azure-application-wizard/summary.png)

**目標設定檔** - 您可以選擇從您所選擇的設定建立發行設定檔。 例如，您可能會建立一個設定檔用於測試環境，並建立另一個用於生產。 若要儲存這個設定檔，請選擇 **儲存** 圖示。 此精靈會建立設定檔並將它儲存在 Visual Studio 專案。 若要修改設定檔名稱，請開啟 [目標設定檔] 清單，然後選擇 [&lt;管理...&gt;]。

   > [!Note]
   > 發佈設定檔會出現在 Visual Studio 的 [方案總管] 中，而且設定檔設定會寫入至副檔名為.azurePubxml 的檔案。 設定會儲存為 XML 標記的屬性。

## <a name="publishing-your-application"></a>發佈您的應用程式

設定專案部署的所有設定後，請選取對話方塊底部的 [發佈]。 您可以在 Visual Studio 中監視 [輸出]  視窗中的處理序狀態。

## <a name="next-steps"></a>後續步驟

- [從 Visual Studio 將 Web 應用程式移轉並發佈至 Azure 雲端服務](./vs-azure-tools-migrate-publish-web-app-to-cloud-service.md)

- [了解如何使用 Visual Studio 來發佈 Azure 雲端服務](./vs-azure-tools-publishing-a-cloud-service.md)

- [使用 Visual Studio 和 IntelliTrace 進行已發佈 Azure 雲端服務的偵錯](./vs-azure-tools-intellitrace-debug-published-cloud-services.md)

- [測試 Azure 雲端服務的效能](./vs-azure-tools-performance-profiling-cloud-services.md)

- [為 Azure 雲端服務和虛擬機器設定診斷功能](./vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md)。

- [什麼是 Application Insights？](./application-insights/app-insights-overview.md)