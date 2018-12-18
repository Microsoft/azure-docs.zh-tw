---
title: 在 Visual Studio 中進行 Azure 雲端服務或虛擬機器的偵錯 | Microsoft Docs
description: 在 Visual Studio 中進行雲端服務或虛擬機器的偵錯
services: visual-studio-online
documentationcenter: na
author: mikejo
manager: douge
editor: ''
ms.assetid: 945e06e0-2100-41af-b218-72347367ddab
ms.service: visual-studio-online
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.custom: vs-azure
ms.workload: azure-vs
ms.date: 11/11/2016
ms.author: mikejo
ms.openlocfilehash: af961533648ce3a939c25fca32173684255a52a8
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42444685"
---
# <a name="debugging-an-azure-cloud-service-or-virtual-machine-in-visual-studio"></a>在 Visual Studio 中進行 Azure 雲端服務或虛擬機器的偵錯

Visual Studio 提供您偵錯 Azure 雲端服務和虛擬機器的不同選項。

## <a name="debug-your-cloud-service-on-your-local-computer"></a>在您的本機電腦上偵錯您的雲端服務

使用 Azure 計算模擬器在本機電腦上偵錯您的雲端服務可以節省時間和金錢。 透過在部署服務之前於本機偵錯服務，您可以改善可靠性和效能，而不需支付運算時間。 不過，某些錯誤可能只有當您在 Azure 中執行雲端服務本身時才會發生。 如果在發佈您的服務時啟用遠端偵錯，接著將偵錯工具附加至角色執行個體，則可以偵錯這些錯誤。

模擬器會模擬 Azure 運算服務，並在您的本機環境中執行，使得您可以在部署雲端服務之前對其測試和偵錯。 模擬器會處理您的角色執行個體的生命週期，並可讓您存取模擬的資源，例如本機儲存體。 當您從 Visual Studio 偵錯或執行您的服務時，它會自動啟動模擬器做為背景應用程式，然後將服務部署至模擬器。 當服務在本機環境中執行時，您可以使用模擬器來檢視服矛。 您可以執行完整版或精簡版的模擬器。 (從 Azure 2.3 起，精簡版的模擬器是預設值。)請參閱[使用 Emulator Express 在本機執行雲端服務並進行偵錯](vs-azure-tools-emulator-express-debug-run.md)。

### <a name="to-debug-your-cloud-service-on-your-local-computer"></a>在您的本機電腦上偵錯您的雲端服務

1. 在功能表列上，選擇 [偵錯]、[開始偵錯] 來執行您的 Azure 雲端服務專案。 或者，您可以按 F5。 您會看到正在啟動計算模擬器的訊息。 當模擬器啟動時，系統匣圖示可加以確認。

    ![系統匣中的 Azure 模擬器](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC783828.png)

2. 開啟通知區域中 Azure 圖示的捷徑功能表，然後選取 [顯示計算模擬器 UI] ，來顯示計算模擬器的使用者介面。

    UI 的左方窗格會顯示目前已部署至計算模擬器的服務，以及每個服務正在執行的角色執行個體。 您可以選擇將在右窗格中顯示生命週期、記錄和診斷資訊的服務或角色。 如果您將焦點放在包含視窗的上邊界時，它會展開以填滿右方窗格。

3. 選取 [偵錯] 功能表中的命令，並在您的程式碼中設定中斷點，來逐步執行應用程式。 在偵錯工具中逐步執行應用程式時，窗格會隨著應用程式的目前狀態而更新。 當您停止偵錯時，應用程式部署會遭到刪除。 如果您的應用程式包含 Web 角色，而且您已設定 [啟動] 動作屬性來啟動 Web 瀏覽器，Visual Studio 會在瀏覽器中啟動您的 Web 應用程式。 如果您變更服務組態中角色的執行個體數目，您必須停止雲端服務，然後重新啟動偵錯，以便對此角色的新執行個體進行偵錯。

    **附註：** 停止執行或偵錯您的服務時，不會停止本機計算模擬器和儲存體模擬器。 您必須從通知區域明確停止它們。

## <a name="debug-a-cloud-service-in-azure"></a>在 Azure 中偵錯雲端服務

若要從遠端電腦偵錯雲端服務，您必須在部署雲端服務時明確啟用該功能，使得需要的服務 (例如 msvsmon.exe) 會安裝在執行您的角色執行個體的虛擬機器上。 發佈服務時如果您未啟用遠端偵錯，您必須在啟用遠端偵錯的情況鑑重新發佈服務。

如果您啟用雲端服務的遠端偵錯，它不會出現效能降低或產生其他費用。 不要對生產環境服務使用遠端偵錯，因為這可能會對正在使用該服務的用戶端造成不良影響。

> [!NOTE]
> 從 Visual Studio 發佈雲端服務時，您可以為該服務中以 .NET Framework 4 或 .NET Framework 4.5 為目標的任何角色啟用 **IntelliTrace** 。 藉由使用 **IntelliTrace**，您可以檢查角色執行個體在過去發生的事件，並重現當時的情況。 請參閱[使用 IntelliTrace 和 Visual Studio 偵錯發佈的雲端服務](http://go.microsoft.com/fwlink/?LinkID=623016)和[使用 IntelliTrace](https://msdn.microsoft.com/library/dd264915.aspx)。

### <a name="to-enable-remote-debugging-for-a-cloud-service"></a>啟用雲端服務的遠端偵錯

1. 開啟 Azure 專案的捷徑功能表，然後選取 [發佈] 。

2. 選取 [預備] 環境和 [偵錯] 組態。

    這只是指導方針。 您可以選擇在生產環境中執行測試環境。 不過，如果您在生產環境上啟用遠端偵錯，可能會對使用者造成不良影響。 您可以選擇 [發行] 組態，但 [偵錯] 組態會讓偵錯更容易。

    ![選擇偵錯組態](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746717.gif)

3. 遵循往常的步驟，但在 [進階設定] 索引標籤上選取 [啟用所有角色的遠端偵錯工具] 核取方塊。

    ![偵錯組態](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746718.gif)

### <a name="to-attach-the-debugger-to-a-cloud-service-in-azure"></a>將偵錯工具附加至在 Azure 中的雲端服務

1. 在伺服器總管中，展開雲端服務的節點。

2. 開啟您要附加的角色或角色執行個體的捷徑功能表，然後選取 [附加偵錯工具] 。

    如果您偵錯角色，Visual Studio 偵錯工具會附加至該角色的每個執行個體。 偵錯工具會在執行該行程式碼並符合中斷點任何條件的第一個角色執行個體的中斷點上中斷。 如果您偵錯某個執行個體，偵錯工具僅會附加至該執行個體，並只有在該特定執行個體執行該行程式碼並符合中斷點的條件時在某個中斷點上中斷。

    ![附加偵錯工具](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746719.gif)

3. 將偵錯工具附加至執行個體之後，如往常般進行偵錯。 偵錯工具會自動附加至符合角色之適當的主機處理序。 根據角色，偵錯工具會附加至 w3wp.exe、WaWorkerHost.exe 或 WaIISHost.exe。 若要確認偵錯工具附加的程序，請展開 [伺服器總管] 中的執行個體節點。 如需有關 Azure 程序的詳細資訊，請參閱[ Azure 角色架構](http://blogs.msdn.com/b/kwill/archive/2011/05/05/windows-azure-role-architecture.aspx)。

    ![選取程式碼類型對話方塊](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC718346.png)

4. 若要識別附加偵錯工具的處理序，請開啟 [處理序] 對話方塊，在功能表列上依序選擇 [偵錯] > [Windows]、[處理序]。 (鍵盤：Ctrl+Alt+Z) 若要中斷特定處理序的連結，請開啟其捷徑功能表，然後選取 [中斷處理序連結]。 或者，在「伺服器總管」中找出執行個體節點、尋找處理序、開啟其捷徑功能表，然後選取 [中斷處理序連結] 。

    ![偵錯處理序](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC690787.gif)

> [!WARNING]
> 進行遠端偵錯時，避免長時間在中斷點停止運作。 Azure 會將停止超過幾分鐘時間的程序視為沒有回應，並停止將流量傳送到該執行個體。 如果您停止太久，msvsmon.exe 會從處理序中斷連結。

若要將偵錯工具與您執行個體或角色中的所有處理序中斷連結，請開啟您要偵錯的角色或執行個體的捷徑功能表，然後選取 [中斷連結偵錯工具] 。

## <a name="limitations-of-remote-debugging-in-azure"></a>在 Azure 中遠端偵錯的限制

自 Azure SDK 2.3 起，遠端偵錯具有下列限制：

* 若啟用遠端偵錯，您無法在具有超過 25 個執行個體的任何角色中發佈雲端服務。
* 偵錯工具會使用連接埠 30400 至 30424、31400 至 31424，以及 32400 至 32424。 如果您嘗試使用這些任一連接埠，將無法發佈您的服務，Azure 的活動記錄檔中將會出現下列其中一個錯誤訊息：

  * 根據 .csdef 檔案驗證 .cscfg 檔案時發生錯誤。
    角色 'role' 端點 Microsoft.WindowsAzure.Plugins.RemoteDebugger.Connector 保留的連接埠範圍 'range' 與已定義的連接埠或範圍重疊。
  * 配置失敗。 請稍後重試、試著減少 VM 大小或角色執行個體的數目，或試著部署到不同的區域。

## <a name="debugging-azure-virtual-machines"></a>偵錯 Azure 虛擬機器

您可以在 Visual Studio 中使用伺服器總管偵錯 Azure 虛擬機器上執行的程式。 當您在 Azure 虛擬機器上啟用遠端偵錯時，Azure 會在虛擬機器上安裝遠端偵錯擴充功能。 然後，您可以附加至虛擬機器上的處理序，並像平常一樣偵錯。

> [!NOTE]
> 透過 Azure 資源管理員堆疊建立的虛擬機器可以使用 Visual Studio 2015 中的雲端總管進行遠端偵錯。 如需詳細資訊，請參閱 [使用雲端總管管理 Azure 資源](http://go.microsoft.com/fwlink/?LinkId=623031)。

### <a name="to-debug-an-azure-virtual-machine"></a>偵錯 Azure 虛擬機器

1. 在伺服器總管中，展開 [虛擬機器] 節點並選取您要偵錯的虛擬機器節點。

2. 開啟操作功能表，然後選取 [啟用偵錯功能] 。 當詢問您是否確定要在虛擬機器上啟用偵錯時，請按一下 [是] 。

    Azure 會將遠端偵錯擴充功能安裝到虛擬機器以啟用偵錯。

    ![虛擬機器啟用偵錯命令](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746720.png)

    ![Azure 活動記錄檔](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746721.png)

3. 遠端偵錯擴充功能安裝完成後，開啟虛擬機器的操作功能表，然後選取 [附加偵錯工具...] 

    Azure 會取得虛擬機器上處理序的清單，並將清單顯示在 [附加至處理序] 對話方塊。

    ![附加偵錯工具命令](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746722.png)

4. 在 [附加至處理序] 對話方塊中，選取 [選取] 以將結果清單限制為只顯示您要偵錯的程式碼類型。 您可以對 32 位元或 64 位元受控碼、機器碼或兩者進行偵錯。

    ![選取程式碼類型對話方塊](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC718346.png)

5. 選取您想要在虛擬機器上偵錯的處理序，然後選取 [附加] 。 例如，如果您想要在虛擬機器上偵錯 Web 應用程式，可以選擇 w3wp.exe 處理序。 如需詳細資訊，請參閱[在 Visual Studio 中偵錯一或多個處理序](https://msdn.microsoft.com/library/jj919165.aspx)和 [Azure 角色架構](http://blogs.msdn.com/b/kwill/archive/2011/05/05/windows-azure-role-architecture.aspx)。

## <a name="create-a-web-project-and-a-virtual-machine-for-debugging"></a>建立 Web 專案和虛擬機器進行偵錯

發佈 Azure 專案之前，您可能會發現，在支援偵錯和測試案例，以及您可以在其中安裝測試和監視程式的受控制環境中測試它很有用。 執行這類測試的方式之一是在虛擬機器上遠端偵錯您的應用程式。

Visual Studio ASP.NET 專案提供選項，讓您建立可用於測試應用程式的實用虛擬機器。 虛擬機器包含經常需要的端點，例如 PowerShell、遠端桌面和 WebDeploy。

### <a name="to-create-a-web-project-and-a-virtual-machine-for-debugging"></a>建立 Web 專案和虛擬機器進行偵錯

1. 在 Visual Studio 中，建立新的 ASP.NET Web 應用程式。

2. 在 [新增 ASP.NET 專案] 對話方塊中，於 [Azure] 區段的下拉式清單方塊中選擇 [虛擬機器]  。 維持選取 [建立遠端資源]  核取方塊。 選取 [確定]  以繼續操作。

    [在 Azure 上建立虛擬機器]  對話方塊隨即出現。

    ![建立 ASP.NET Web 專案對話方塊](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746723.png)

    **附註：** 如果您還沒登入，將會要求您登入 Azure 帳戶。

3. 選取虛擬機器的各種設定，然後選取 [確定] 。 如需詳細資訊，請參閱 [虛擬機器](http://go.microsoft.com/fwlink/?LinkId=623033) 。

    您輸入的 DNS 名稱會成為虛擬機器的名稱。

    ![在 Azure 上建立虛擬機器對話方塊](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746724.png)

    Azure 會建立虛擬機器，然後佈建及設定端點，例如遠端桌面和 Web Deploy

4. 虛擬機器完全設定好之後，請在伺服器總管中選取虛擬機器的節點。

5. 開啟操作功能表，然後選取 [啟用偵錯功能] 。 當詢問您是否確定要在虛擬機器上啟用偵錯時，請按一下 [是] 。

    Azure 會將遠端偵錯擴充功能安裝到虛擬機器以啟用偵錯。

    ![虛擬機器啟用偵錯命令](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746720.png)

    ![Azure 活動記錄檔](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746721.png)

6. 如 [HOW TO：在 Visual Studio 中使用單鍵發佈來部署 Web 專案](https://msdn.microsoft.com/library/dd465337.aspx)所述，發佈您的專案。 因為您要在虛擬機器上偵錯，請在 [發佈 Web] 精靈的 [設定] 頁面上，選取 [偵錯] 做為組態。 這樣可確保程式碼符號在偵錯時可供使用。

    ![發佈設定](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC718349.png)

7. 如果稍早已部署專案，請在 [檔案發佈選項] 中，選取 [移除目的地的額外檔案]。

8. 專案發佈之後，在「伺服器總管」中的虛擬機器操作功能表上，選取 [附加偵錯工具...] 

    Azure 會取得虛擬機器上處理序的清單，並將清單顯示在 [附加至處理序] 對話方塊。

    ![附加偵錯工具命令](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC746722.png)

9. 在 [附加至處理序] 對話方塊中，選取 [選取] 以將結果清單限制為只顯示您要偵錯的程式碼類型。 您可以對 32 位元或 64 位元受控碼、機器碼或兩者進行偵錯。

    ![選取程式碼類型對話方塊](./media/vs-azure-tools-debug-cloud-services-virtual-machines/IC718346.png)

10. 選取您想要在虛擬機器上偵錯的處理序，然後選取 [附加] 。 例如，如果您想要在虛擬機器上偵錯 Web 應用程式，可以選擇 w3wp.exe 處理序。 如需詳細資訊，請參閱 [在 Visual Studio 中偵錯一或多個處理序](https://msdn.microsoft.com/library/jj919165.aspx) 。

## <a name="next-steps"></a>後續步驟

* 使用 **Intellitrace** 從發行伺服器收集呼叫和事件的記錄檔。 請參閱 [使用 IntelliTrace 和 Visual Studio 偵錯發佈的雲端服務](http://go.microsoft.com/fwlink/?LinkID=623016)。

* 不論角色是在開發環境中或在 Azure 中執行，請使用 **Azure 診斷** 來記錄在角色內執行的程式碼的詳細資訊。 請參閱 [使用 Azure 診斷收集記錄資料](http://go.microsoft.com/fwlink/p/?LinkId=400450)。
