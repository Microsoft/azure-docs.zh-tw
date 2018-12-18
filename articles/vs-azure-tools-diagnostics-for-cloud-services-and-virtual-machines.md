---
title: 為 Azure 雲端服務和虛擬機器設定診斷 | Microsoft Docs
description: 了解如何在 Visual Studio 中為 Azure 雲端服務和虛擬機器 (VM) 設定診斷。
services: visual-studio-online
documentationcenter: na
author: mikejo
manager: douge
editor: ''
ms.assetid: e70cd7b4-6298-43aa-adea-6fd618414c26
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 06/28/2018
ms.author: mikejo
ms.openlocfilehash: 2ff2a619dabd7dfabf89361172557efa4884ba12
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37110482"
---
# <a name="set-up-diagnostics-for-azure-cloud-services-and-virtual-machines"></a>為 Azure 雲端服務和虛擬機器設定診斷
需要針對 Azure 雲端服務或 Azure 虛擬機器進行疑難排解時，您可以使用 Visual Studio 更輕鬆地設定 Azure 診斷。 診斷會在執行雲端服務的虛擬機器和虛擬機器執行個體上擷取系統資料和記錄資料。 診斷資料會傳輸到您選擇的儲存體帳戶。 如需 Azure 中診斷記錄的詳細資訊，請參閱[在 Azure App Service 中針對 Web 應用程式啟用診斷記錄](app-service/web-sites-enable-diagnostic-log.md)。

在本文中，我們示範如何在部署前後使用 Visual Studio 來開啟並設定 Azure 診斷。 了解如何在 Azure 虛擬機器上設定診斷、如何選取要收集的診斷資訊類型，以及如何在收集之後檢視資訊。

您可以使用下列其中一個選項來設定 Azure 診斷：

* 在 Visual Studio 的 [診斷設定]  對話方塊中變更診斷設定。 這些設定會儲存在稱為 diagnostics.wadcfgx 的檔案 (在 Azure SDK 2.4 或更早版本中，檔案稱為 diagnostics.wadcfg)。 您也可以直接修改設定檔。 如果您手動更新檔案，設定變更會在您下一次將雲端服務部署至 Azure 或在模擬器中執行服務時生效。
* 使用 Visual Studio 中的 [雲端總管] 或 [伺服器總管]，為執行中的雲端服務或虛擬機器變更診斷設定。

## <a name="azure-sdk-26-diagnostics-changes"></a>Azure SDK 2.6 診斷變更
下列變更適用於 Visual Studio 中的 Azure SDK 2.6 和更新版本專案：

* 本機模擬器現在支援診斷。 這表示您可以收集診斷資料，並確保您在 Visual Studio 中進行開發及測試時，您的應用程式會建立正確的追蹤。 當您在 Visual Studio 中使用 Azure 儲存體模擬器來執行您的雲端服務專案時，連接字串 `UseDevelopmentStorage=true` 會開啟診斷資料收集。 所有的診斷資料都會收集到「開發儲存體」儲存體帳戶中。
* 診斷儲存體帳戶連接字串 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 會儲存在服務設定 (.cscfg) 檔案中。 在 Azure SDK 2.5 中，診斷儲存體帳戶會在 diagnostics.wadcfgx 檔案中指定。

在 Azure SDK 2.6 和更新版本與 Azure SDK 2.4 及更舊版本中，連接字串的運作方式不同於其中某些索引鍵的運作方式：

* 在 Azure SDK 2.4 及更舊版本中，階段診斷外掛程式使用連接字串會作為執行階段，以取得用於傳輸診斷記錄的儲存體帳戶資訊。
* 在 Azure SDK 2.6 及更新版本中，Visual studio 會使用診斷連接字串，在發佈期間設定內含適當儲存體帳戶資訊的 Azure 診斷延伸模組。 您可以使用連接字串，為 Visual Studio 在發佈期間使用的不同服務設定，定義不同的儲存體帳戶。 不過，因為診斷外掛程式在 Azure SDK 2.5 之後無法使用，所以 .cscfg 檔案本身無法設定診斷延伸模組。 您必須使用 Visual Studio 或 PowerShell 等工具，個別設定延伸模組。
* 為了使用 PowerShell 簡化診斷延伸模組的設定程序，從 Visual Studio 的封裝輸出包含每個角色之診斷延伸模組的公用設定 XML。 Visual Studio 使用診斷連接字串，填入公用設定中的儲存體帳戶資訊。 公用設定檔是在 Extensions 資料夾中建立的。 公用設定檔會使用命名模式 PaaSDiagnostics.&lt;role name\>.PubConfig.xml。 任何以 PowerShell 為基礎的部署都可以使用此模式，將每個設定對應至角色。
* [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)會使用 .cscfg 檔案中的連接字串來存取診斷資料。 資料會出現在 [監視] 索引標籤上。需有連接字串，才能設定服務以在入口網站中顯示詳細監視資料。

## <a name="migrate-projects-to-azure-sdk-26-and-later"></a>將專案移轉至 Azure SDK 2.6 及更新版本
從 Azure SDK 2.5 移轉至 Azure SDK 2.6 或更新版本時，如果您已在 .wadcfgx 檔案中指定診斷儲存體帳戶，儲存體帳戶就會留在該檔案中。 若要針對不同儲存體組態充分利用不同儲存體帳戶的靈活性，請手動將連接字串加入專案。 如果您將專案從 Azure SDK 2.4 或更舊版本移轉至 Azure SDK 2.6，系統會保留診斷連接字串。 不過，請注意 Azure SDK 2.6 中連接字串處理方式的變更，如上一節所述。

### <a name="how-visual-studio-determines-the-diagnostics-storage-account"></a>Visual Studio 如何決定診斷儲存體帳戶
* 如果在 .cscfg 檔案中指定診斷連接字串，Visual Studio 會在發佈期間，以及在封裝期間產生公用設定 XML 檔案時，使用它來設定診斷延伸模組。
* 如果未在 .cscfg 檔案中指定診斷連接字串，Visual Studio 會回復到使用 .wadcfgx 檔案中指定的儲存體帳戶來設定診斷延伸模組，以進行發佈以及在封裝期間產生公用設定 XML 檔案。
* 在 .cscfg 檔案中的診斷連接字串的優先順序高於 .wadcfgx 檔案中的儲存體帳戶。 如果在 .cscfg 檔案中指定診斷連接字串，Visual Studio 會使用該連接字串，並忽略 .wadcfgx 中的儲存體帳戶。

### <a name="what-does-the-update-development-storage-connection-strings-check-box-do"></a>「更新開發儲存體連接字串...」核取方塊的功用？
[在發佈至 Microsoft Azure 時使用 Microsoft Azure 儲存體帳戶認證更新診斷和快取的開發儲存體連接字串]  核取方塊是一種便利方式，可使用發佈期間指定的 Azure 儲存體帳戶更新任何開發儲存體帳戶連接字串。

例如，如果您選取此核取方塊，而且診斷連接字串指定 `UseDevelopmentStorage=true`，則在您將專案發佈至 Azure 時，Visual Studio 會使用您在發佈精靈中指定的儲存體帳戶，自動更新診斷連接字串。 不過，如果將實際的儲存體帳戶指定為診斷連接字串，則會改用該帳戶。

## <a name="diagnostics-functionality-differences-in-azure-sdk-24-and-earlier-vs-azure-sdk-25-and-later"></a>Azure SDK 2.4 及更舊版本與 Azure SDK 2.5 及更新版本之間的診斷功能差異
如果您要將專案從 Azure SDK 2.4 及更舊版本更新為 Azure SDK 2.5 或更新版本，請謹記下列診斷功能差異：

* **設定 API 已被取代**。 診斷的程式設計設定可在 Azure SDK 2.4 或更舊版本中使用，但在 Azure SDK 2.5 及更新版本中已被取代。 如果診斷設定目前以程式碼定義，您必須在移轉專案中從頭開始進行這些設定才能讓診斷保持運作。 Azure SDK 2.4 的診斷設定檔是 diagnostics.wadcfg。 Azure SDK 2.5 及更新版本的診斷設定檔是 diagnostics.wadcfgx。
* **雲端服務應用程式的診斷只能在角色層級設定**。 在 Azure SDK 2.5 及更新版本中，您無法在執行個體層級設定雲端服務應用程式的診斷。
* **每次部署您的應用程式時，都會更新診斷設定**。 如果您從 Visual Studio [伺服器總管] 變更診斷設定，然後重新部署您的應用程式，則會導致同位檢查的問題。
* **在 Azure SDK 2.5 及更新版本中，損毀傾印是在診斷設定檔中設定，而不是在程式碼中設定**。 如果已在程式碼中設定損毀傾印，您必須手動將程式碼中的設定傳輸至設定檔。 在移轉至 Azure SDK 2.6 期間不會傳輸損毀傾印。

## <a name="turn-on-diagnostics-in-cloud-service-projects-before-you-deploy-them"></a>部署雲端服務專案中的診斷之前先將其開啟
在 Visual Studio 中，當您在部署之前執行模擬器中的服務時，就可以收集在 Azure 中執行之角色的診斷資料。 在 Visual Studio 中對診斷設定進行的所有變更都會儲存在 diagnostics.wadcfgx 組態檔中。 這些設定會在您部署雲端服務時指定儲存診斷資料的儲存體帳戶。

[!INCLUDE [cloud-services-wad-warning](../includes/cloud-services-wad-warning.md)]

### <a name="to-turn-on-diagnostics-in-visual-studio-before-deployment"></a>在部署之前開啟 Visual Studio 中的診斷

1. 在角色的捷徑功能表上，選取 [屬性]。 在角色的 [屬性] 對話方塊中，選取 [設定] 索引標籤。
2. 在 [診斷] 區段中，確定已選取 [啟用診斷] 核取方塊。
   
    ![存取啟用診斷選項](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796660.png)
3. 若要指定診斷資料的儲存體帳戶，請選擇省略符號 (...) 按鈕。
   
    ![指定要使用的儲存體帳戶](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796661.png)
4. 在 [建立儲存體連接字串]  對話方塊中，指定要使用 Azure 儲存體模擬器、Azure 訂用帳戶或手動輸入的認證來連接。
   
    ![儲存體帳戶對話方塊](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796662.png)
   
   * 如果您選取**Microsoft Azure 儲存體模擬器**，則連接字串會設定為 `UseDevelopmentStorage=true`。
   * 如果選取 [您的訂用帳戶]，您可以選取要使用的 Azure 訂用帳戶，並輸入帳戶名稱。 若要管理您的 Azure 訂用帳戶，請選取 [管理帳戶]。
   * 如果您選取 [手動輸入的認證]，請輸入您想要使用之 Azure 帳戶的名稱和金鑰。
5. 若要檢視 [診斷設定] 對話方塊，請選取 [設定] 對話方塊。 除了 [一般] 和 [記錄目錄] 之外，每個索引標籤都代表您可以收集的診斷資料來源。 預設 [一般] 索引標籤提供下列診斷資料收集選項：[只記錄錯誤]、[所有資訊] 和 [自訂計劃]。 預設 [只記錄錯誤] 選項會佔用最少的儲存體，因為它不會傳輸警告或追蹤訊息。 [所有資訊]  選項會傳輸大部分的資訊、使用最多的儲存體，因此它是最昂貴的選項。

   > [!NOTE]
   > 針對「磁碟配額 (以 MB 為單位)」支援的大小下限為 4GB。 不過，如果您有收集記憶體傾印，請將這個值增加為較高的值，例如 10GB。
   >
  
    ![啟用 Azure 診斷和組態](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC758144.png)
6. 在此範例中，請選取 [自訂計劃]  選項，您就可以自訂收集的資料。
7. 在 [以 MB 為單位的磁碟配額]  方塊中，您可以設定要在儲存體帳戶中配置診斷資料的空間大小。 您可以變更或接受預設值。
8. 在您想要收集之診斷資料的每個索引標籤上，選取 [啟用 \<log type\> 的傳輸] 核取方塊。 例如，如果您想要收集應用程式記錄檔，請在 [應用程式記錄檔] 索引標籤上選取 [啟用應用程式記錄檔的傳輸] 核取方塊。 此外，請指定每個診斷資料類型所需的其他資訊。 如需每個索引標籤的設定資訊，請參閱本文稍後的 **設定診斷資料來源**一節。 
9. 啟用所有您想要的診斷資料收集之後，請選取 [確定]。
10. 如往常一般在 Visual Studio 中建立 Azure 雲端服務專案。 當您使用您的應用程式，您已啟用的記錄檔資訊會儲存到您指定的 Azure 儲存體帳戶。

## <a name="turn-on-diagnostics-on-azure-virtual-machines"></a>開啟 Azure 虛擬機器上的診斷
在 Visual Studio 中，您可以收集 Azure 虛擬機器的診斷資料。

### <a name="to-turn-on-diagnostics-on-azure-virtual-machines"></a>開啟 Azure 虛擬機器上的診斷

1. 在 [伺服器總管] 中，選取 Azure 節點，然後連線至您的 Azure 訂用帳戶 (如果您尚未連線)。
2. 展開 **虛擬機器** 節點。 您可以建立新的虛擬機器，或選取現有的節點。
3. 在您所需之虛擬機器的捷徑功能表上，選取 [設定]。 [虛擬機器設定] 對話方塊隨即出現。
   
    ![設定 Azure 虛擬機器](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796663.png)
4. 如果尚未安裝，請新增 Microsoft Monitoring Agent 診斷延伸模組。 您可以使用此延伸模組，收集 Azure 虛擬機器的診斷資料。 在 [已安裝的延伸模組] 下，於 [選取可用的延伸模組] 下拉式功能表中，選取 [Microsoft Monitoring Agent 診斷]。
   
    ![安裝 Azure 虛擬機器延伸模組](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC766024.png)
   
    > [!NOTE]
   > 其他可供您的虛擬機器使用的診斷延伸模組。 如需詳細資訊，請參閱[適用於 Windows 的虛擬機器擴充功能和功能](https://docs.microsoft.com/azure/virtual-machines/windows/extensions-features)。
   > 
   > 
5. 若要新增延伸模組及檢視其 [診斷設定] 對話方塊，請選取 [新增]。
6. 若要指定儲存體帳戶，請選取 [設定]，然後選取 [確定]。
   
    每個索引標籤 (除了 [一般] 和 [記錄目錄] 之外) 都代表您可以收集的診斷資料來源。
   
    ![啟用 Azure 診斷和組態](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC758144.png)
   
    預設索引標籤 [一般] 提供下列診斷資料收集選項：[只記錄錯誤]、[所有資訊] 和 [自訂計劃]。 預設選項 [只記錄錯誤] 會佔用最少的儲存體，因為它不會傳輸警告或追蹤訊息。 [所有資訊]  選項會傳輸大部分的資訊，因此它是對儲存體而言最昂貴的選項。
7. 在此範例中，請選取 [自訂計劃]  選項，您就可以自訂收集的資料。
8. [以 MB 為單位的磁碟配額]  方塊可指定您想要在儲存體帳戶中配置診斷資料的空間大小。 您可以任意變更預設值。
9. 在您想要收集之診斷資料的每個索引標籤上，選取其 [啟用 \<log type\> 的傳輸] 核取方塊。
   
    例如，如果您想要收集應用程式記錄檔，請選取 [應用程式記錄檔] 索引標籤上的 [啟用應用程式記錄檔的傳輸] 核取方塊。此外，請指定每個診斷資料類型所需的其他資訊。 如需每個索引標籤的設定資訊，請參閱本文稍後的 **設定診斷資料來源**一節。
10. 啟用所有您想要的診斷資料收集之後，請選取 [確定]。
11. 儲存更新的專案。
    
    [Microsoft Azure 活動記錄]  視窗中的訊息指出虛擬機器已更新。

## <a name="set-up-diagnostics-data-sources"></a>設定診斷資料來源
啟用診斷資料收集之後，您可以準確選擇您要收集的資料來源和所收集的資訊。 以下幾節描述 [診斷設定]  對話方塊中的索引標籤，以及每個設定選項的意義。

### <a name="application-logs"></a>應用程式記錄檔
應用程式記錄具有 Web 應用程式所產生的診斷資訊。 如果您想要擷取應用程式記錄檔，請選取 [啟用應用程式記錄檔傳輸] 核取方塊。 若要增加或減少將應用程式記錄傳輸至儲存體帳戶的間隔，請變更 [傳輸期間 (分鐘)] 值。 您也可以藉由設定**記錄層級**值變更記錄檔中擷取的資訊量。 例如，選取 [Verbose] 以取得詳細資訊，或選取 [嚴重] 只擷取嚴重的錯誤。 如果您有特定的診斷提供者會發出應用程式記錄，您可以藉由將提供者的 GUID 新增至 [提供者 GUID] 方塊來擷取記錄。

  ![應用程式記錄檔](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC758145.png)

如需應用程式記錄的詳細資訊，請參閱[在 Azure App Service 中針對 Web 應用程式啟用診斷記錄](app-service/web-sites-enable-diagnostic-log.md)。

### <a name="windows-event-logs"></a>Windows 事件記錄檔
若要擷取 Windows 事件記錄，請選取 [啟用 Windows 事件記錄傳輸] 核取方塊。 若要增加或減少將事件記錄傳輸至儲存體帳戶的間隔，請變更 [傳輸期間 (分鐘)] 值。 選取您想要追蹤之事件類型的核取方塊。

![事件記錄檔](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796664.png)

如果您正在使用 Azure SDK 2.6 或更新版本，並想要指定自訂資料來源，請在 [\<資料來源名稱\>] 文字方塊中將其輸入，然後選取 [新增]。 資料來源會新增至 diagnostics.cfcfg 檔案。

如果您正在使用 Azure SDK 2.5 且想要指定自訂資料來源，您可以將它新增至 diagnostics.wadcfgx 檔案的 `WindowsEventLog` 區段，如下列範例所示：

```
<WindowsEventLog scheduledTransferPeriod="PT1M">
   <DataSource name="Application!*" />
   <DataSource name="CustomDataSource!*" />
</WindowsEventLog>
```
### <a name="performance-counters"></a>效能計數器
效能計數器資訊可協助您找出系統瓶頸並微調系統和應用程式效能。 如需詳細資訊，請參閱[在 Azure 應用程式中建立及使用效能計數器](https://msdn.microsoft.com/library/azure/hh411542.aspx)。 若要擷取效能計數器，請選取 [啟用效能計數器的傳輸] 核取方塊。 若要增加或減少將事件記錄傳輸至儲存體帳戶的間隔，請變更 [傳輸期間 (分鐘)] 值。 選取您想要追蹤之效能計數器的核取方塊。

![效能計數器](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC758147.png)

若要追蹤未列出的效能計數器，請使用建議的語法輸入效能計數器。 然後，選取 [新增]。 虛擬機器上的作業系統會決定您可以追蹤的效能計數器。如需語法的詳細資訊，請參閱[指定計數器路徑](https://msdn.microsoft.com/library/windows/desktop/aa373193.aspx)。

### <a name="infrastructure-logs"></a>基礎結構記錄檔
基礎結構記錄具有 Azure 診斷基礎結構、RemoteAccess 模組和 RemoteForwarder 模組的相關資訊。 若要收集基礎結構記錄的相關資訊，請選取 [啟用基礎結構記錄的傳輸] 核取方塊。 若要增加或減少將基礎結構記錄傳輸至儲存體帳戶的間隔，請變更 [傳輸期間 (分鐘)] 值。

![診斷基礎結構記錄檔](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC758148.png)

如需詳細資訊，請參閱 [使用 Azure 診斷收集記錄資料](https://msdn.microsoft.com/library/azure/gg433048.aspx)。

### <a name="log-directories"></a>記錄檔目錄
記錄目錄具有從 Internet Information Services (IIS) 要求、失敗要求，或您選擇之資料夾的記錄目錄中收集的資料。 若要擷取記錄目錄，請選取 [啟用記錄目錄的傳輸] 核取方塊。 若要增加或減少將記錄傳輸至儲存體帳戶的間隔，請變更 [傳輸期間 (分鐘)] 值。

選取您想要收集之記錄的核取方塊，例如 [IIS 記錄] 和 [失敗要求] 記錄。 提供預設儲存體容器名稱，但您可以變更名稱。

您可以從任何資料夾擷取記錄。 在 [絕對目錄中的記錄] 區段中指定路徑，然後選取 [新增目錄]。 記錄會擷取至指定的容器。

![記錄檔目錄](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796665.png)

### <a name="etw-logs"></a>ETW 記錄檔
如果您使用 [Windows 事件追蹤](https://msdn.microsoft.com/library/windows/desktop/bb968803\(v=vs.85\).aspx) (ETW) 而且想要擷取 ETW 記錄檔，請選取 [啟用 ETW 記錄檔的傳輸] 核取方塊。 若要增加或減少將記錄傳輸至儲存體帳戶的間隔，請變更 [傳輸期間 (分鐘)] 值。

從事件來源和您指定的事件資訊清單擷取事件。 若要指定事件來源，請在 [事件來源] 區段中輸入名稱，然後選取 [新增事件來源]。 同樣地，您可以指定 [事件資訊清單] 區段中的事件資訊清單，然後選取 [新增事件資訊清單]。

![ETW 記錄檔](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC766025.png)

透過 [System.Diagnostics.aspx](https://msdn.microsoft.com/library/system.diagnostics(v=vs.110)) 命名空間中的類別，支援 ASP.NET 中的 ETW 架構。 Microsoft.WindowsAzure.Diagnostics 命名空間 (繼承自並擴充標準 [System.Diagnostics.aspx](https://msdn.microsoft.com/library/system.diagnostics(v=vs.110)) 類別) 可讓您使用 [System.Diagnostics.aspx](https://msdn.microsoft.com/library/system.diagnostics(v=vs.110)) 作為 Azure 環境中的記錄架構。 如需詳細資訊，請參閱[在 Microsoft Azure 中控制記錄和追蹤](https://msdn.microsoft.com/magazine/ff714589.aspx)和[在 Azure 雲端服務和虛擬機器中啟用診斷](cloud-services/cloud-services-dotnet-diagnostics.md)。

### <a name="crash-dumps"></a>損毀傾印
若要擷取角色執行個體何時損毀的相關資訊，請選取 [啟用損毀傾印的傳輸] 核取方塊。 (由於 ASP.NET 處理大多數例外狀況，這通常只對背景工作角色才有用。)若要增加或減少專用於損毀傾印的儲存空間百分比，您可以變更**目錄配額 (%)** 值。 您可以變更儲存損毀傾印的儲存體容器，也可以選取您想要擷取**完整**或**最小**傾印。

下一個螢幕擷取畫面列出目前正在追蹤的處理序。 選取您想要擷取之處理序的核取方塊。 若要將另一個處理序新增至清單，請輸入處理序名稱，然後選擇 [新增處理序]。

![損毀傾印](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC766026.png)

如需詳細資訊，請參閱[在 Microsoft Azure 中控制記錄和追蹤](https://msdn.microsoft.com/magazine/ff714589.aspx)和 [Microsoft Azure 診斷第 4 部分：自訂記錄元件和 Azure 診斷 1.3 變更](http://justazure.com/microsoft-azure-diagnostics-part-4-custom-logging-components-azure-diagnostics-1-3-changes/)。

## <a name="view-the-diagnostics-data"></a>檢視診斷資料
收集到雲端服務或虛擬機器的診斷資料之後，您可以檢視它。

### <a name="to-view-cloud-service-diagnostics-data"></a>檢視雲端服務診斷資料
1. 如往常般部署雲端服務並加以執行。
2. 您可以在 Visual Studio 產生的報告或儲存體帳戶的資料表中檢視診斷資料。 若要在報告中檢視資料，請開啟 [雲端總管] 或 [伺服器總管]，開啟您所需之角色節點的捷徑功能表，然後選取 [檢視診斷資料]。
   
    ![檢視診斷資料](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC748912.png)
   
    顯示可用資料的報告隨即出現。
   
    ![Visual Studio 中的 Microsoft Azure 診斷報告](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796666.png)
   
    如果未顯示最新的資料，您可能必須等到傳輸週期過後。
   
    若要立即更新資料，請選取 [重新整理] 連結。 若要自動更新資料，請在 [自動重新整理] 下拉式清單中選取一個間隔。 若要匯出錯誤資料，請選取 [匯出至 CSV]  按鈕，以建立您可以在 Excel 工作表中開啟的逗號分隔值檔案。
   
    在 [雲端總管] 或 [伺服器總管] 中，開啟與部署相關聯的儲存體帳戶。
3. 在資料表檢視器中開啟診斷資料表，然後檢閱您所收集的資料。 在 IIS 記錄檔和自訂記錄檔中，您可以開啟 blob 容器。 下表列出資料表或 blob 容器，其中包含不同記錄檔的資料。 除了該記錄檔的資料之外，資料表項目包含 **EventTickCount**、**DeploymentId**、**Role** 和 **RoleInstance**，以協助您識別哪些虛擬機器與角色產生了資料及其時機。 
   
   | 診斷資料 | 說明 | 位置 |
   | --- | --- | --- |
   | 應用程式記錄檔 |您的程式碼藉由呼叫 **System.Diagnostics.Trace** 類別之方法所產生的記錄。 |WADLogsTable |
   | 事件記錄檔 |資料來自虛擬機器上的 Windows 事件記錄。 Windows 會將資訊儲存在這些記錄中，但應用程式和服務也會使用記錄來報告錯誤或記錄資訊。 |WADWindowsEventLogsTable |
   | 效能計數器 |您可以在可供虛擬機器使用的任何效能計數器上收集資料。 作業系統會提供效能計數器，其包括記憶體使用狀況和處理器時間等多種統計資料。 |WADPerformanceCountersTable |
   | 基礎結構記錄檔 |從診斷基礎結構本身產生的記錄。 |WADDiagnosticInfrastructureLogsTable |
   | IIS 記錄檔 |記錄 Web 要求的記錄。 如果您的雲端服務取得大量的流量，這些記錄可能冗長。 最好只在需要時才收集和儲存此資料。 |您可以在部署、角色及執行個體之路徑下的 wad-iis-failedreqlogs 下的 blob 容器中找到失敗要求記錄。 您可以在 wad-iis-logfiles 下找到完整的記錄檔。 每個檔案的項目都建立於 WADDirectories 資料表中。 |
   | 損毀傾印 |提供雲端服務處理序的二進位映像 (通常是背景工作角色)。 |wad-crush-dumps blob 容器 |
   | 自訂記錄檔 |您預先定義的資料記錄檔。 |您可以在儲存體帳戶中以程式碼指定自訂記錄檔的位置。 例如，您可以指定自訂 blob 容器。 |
4. 如果任何類型的資料遭到截斷，您可以嘗試增加該資料類型的緩衝區，或是縮短資料從虛擬機器傳輸至儲存體帳戶之間的間隔。
5. (選用) 偶爾會從儲存體帳戶清除資料以降低整體儲存成本。
6. 當您執行完整部署時，Azure 中可支援 diagnostics.cscfg 檔案 (在 Azure SDK 2.5 中支援 .wadcfgx)，且您的雲端服務會取得對診斷組態進行的任何變更。 如果您改為更新現有的部署，.cscfg 檔案就不會在 Azure 中更新。 您仍然可以遵循下一節中的步驟變更診斷設定。 如需有關執行完整部署和更新現有部署的詳細資訊，請參閱 [發佈 Azure 應用程式精靈](vs-azure-tools-publish-azure-application-wizard.md)。

### <a name="to-view-virtual-machine-diagnostics-data"></a>檢視虛擬機器診斷資料
1. 在虛擬機器的捷徑功能表上，選取 [檢視診斷資料] 。
   
    ![在 Azure 虛擬機器中檢視診斷資料](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC766027.png)
   
    [診斷摘要] 對話方塊隨即出現。
   
    ![Azure 虛擬機器診斷摘要](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC796667.png)  
   
    如果未顯示最新的資料，您可能必須等到傳輸週期過後。
   
    若要立即更新資料，請選取 [重新整理] 連結。 若要自動更新資料，請在 [自動重新整理] 下拉式清單中選取一個間隔。 若要匯出錯誤資料，請選取 [匯出至 CSV]  按鈕，以建立您可以在 Excel 工作表中開啟的逗號分隔值檔案。

## <a name="set-up-cloud-service-diagnostics-after-deployment"></a>在部署之後設定雲端服務診斷
如果您要調查已執行之雲端服務的相關問題，您可能會想要收集您原本在部署角色之前並未指定的資料。 在此情況下，您可以變更 [伺服器總管] 中的設定，開始收集這類資料。 您可以在角色中設定單一執行個體或所有執行個體的診斷，取決於您是否從執行個體或角色的捷徑功能表開啟 [診斷設定] 對話方塊。 如果您設定角色節點，任何您所做的變更都將套用至所有執行個體。 如果您設定執行個體節點，任何您所做的變更只會套用至該執行個體。

### <a name="to-set-up-diagnostics-for-a-running-cloud-service"></a>設定執行中雲端服務的診斷
1. 在 [伺服器總管] 中，展開 **雲端服務** 節點，然後展開節點清單，以找出您想要調查的角色或執行個體 (或兩者)。
   
    ![設定診斷](./media/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/IC748913.png)
2. 在執行個體節點或角色節點的捷徑功能表上，選取 [更新診斷設定] ，然後選取您想要收集的診斷設定。
   
    如需設定的相關資訊，請參閱本文中的**設定診斷資料來源**一節。 如需如何檢視診斷資料的相關資訊，請參閱本文中的**檢視診斷資料**一節。
   
    如果您變更 [伺服器總管]中的資料收集，這些變更會持續生效，直到您完全重新部署您的雲端服務為止。 如果使用預設發佈設定，不會覆寫變更。 預設發佈設定是更新現有的部署，而不是執行完整部署。 若要在部署階段確定清除設定，請移至 [發佈] 精靈中的 [進階設定] 索引標籤，然後清除 [部署更新] 核取方塊。 當您在清除該核取方塊時重新部署，設定將還原為.wadcfgx (或.wadcfg) 檔案中透過角色的**屬性**編輯器進行的設定。 如果您更新您的部署，Azure 會保留先前的設定。

## <a name="troubleshoot-azure-cloud-service-issues"></a>疑難排解 Azure 雲端服務問題
如果您遇到雲端服務專案的相關問題，例如陷入「忙碌」狀態的角色、重複回收，或擲回內部伺服器錯誤，您都可以使用工具和技術診斷和修正問題。 如需常見問題和解決方案的特定範例，以及您可以用來診斷和修正這些錯誤的概念和工具概觀，請參閱 [Azure PaaS 運算診斷資料](http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx)。

## <a name="q--a"></a>問答集
**何謂緩衝區大小和適當大小？**

在每個虛擬機器執行個體上，配額會限制可以儲存在本機檔案系統上的診斷資料數量。 此外，您可以為每種診斷資料類型指定可用的緩衝區大小。 這個緩衝區大小就像是該資料類型的個別配額。 若要判斷整體配額和剩餘的記憶體數量，請查看對話方塊底部，以取得診斷資料類型。 如果您指定較大的緩衝區或更多類型的資料，您會愈趨近整體配額。 您可以藉由修改 diagnostics.wadcfg 或 .wadcfgx 設定檔變更整體配額。 診斷資料會與您的應用程式資料儲存在同一個檔案系統中。 如果您的應用程式使用大量磁碟空間，您不應該增加整體診斷配額。

**何謂傳輸週期和適當時間長度？**

傳輸週期是資料擷取之間經過的時間量。 每個傳輸週期之後，資料會從虛擬機器上的本機檔案系統上移至儲存體帳戶中的資料表。 如果收集的資料量在傳輸週期結束前超過配額，則會捨棄較舊的資料。 如果您因為資料超過緩衝區大小或整體配額而遺失資料，您可能會想要縮短傳輸週期。

**時間戳記位於哪個時區？**

時間戳記位於裝載雲端服務之資料中心的當地時區。 使用下列記錄資料表中的三個時間戳記資料行：

* **PreciseTimeStamp**：事件的 ETW 時間戳記。 也就是從用戶端記錄事件的時間。
* **TIMESTAMP**：**PreciseTimeStamp** 無條件捨去到上傳頻率界限的值。 例如，如果您的上傳頻率為 5 分鐘，而事件時間為 00:17:12，則 TIMESTAMP 是 00:15:00。
* **Timestamp**：Azure 資料表中建立實體的時間戳記。

**收集診斷資訊時如何管理成本？**

預設設定 (將 [記錄層級] 設為 [錯誤] 並將 [傳輸週期] 設為 [1 分鐘]) 的設計目的是將成本降至最低。 當收集更多診斷資料時，或如果縮短傳輸週期，會增加運算成本。 請勿收集超過需求量的資料，也別忘了在您不需要時停用資料收集。 您可以一律重新啟用它，即使在執行階段也一樣，如本文稍早所述。

**如何從 IIS 收集失敗要求記錄檔？**

根據預設，IIS 不會收集失敗要求記錄檔。 您可以編輯 Web 角色的 web.config 檔案，設定 IIS 來收集失敗要求記錄。

**我無法從 OnStart 之類的 RoleEntryPoint 方法取得追蹤資訊。問題在哪裡？**

**RoleEntryPoint** 的方法是在 WAIISHost.exe 的內容中呼叫，而不在 IIS 中。 在通常會啟用追蹤的 web.config 中，設定資訊並不適用。 若要解決這個問題，請將.config 檔案新增至 web 角色專案，並將檔案命名以符合包含 **RoleEntryPoint** 程式碼的輸出組件。 在預設 web 角色專案中，.config 檔案的名稱應該是 WAIISHost.exe.config。將下列幾行新增至此檔案：

```
<system.diagnostics>
  <trace>
      <listeners>
          <add name “AzureDiagnostics” type=”Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitorTraceListener”>
              <filter type=”” />
          </add>
      </listeners>
  </trace>
</system.diagnostics>
```

在 [屬性] 視窗中，將 [複製到輸出目錄] 屬性設為 [永遠複製]。

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure 中的診斷記錄，請參閱[在 Azure 雲端服務](cloud-services/cloud-services-dotnet-diagnostics.md)和[虛擬機器中啟用診斷和在 Azure App Service 中啟用 Web 應用程式的診斷記錄](app-service/web-sites-enable-diagnostic-log.md)。

