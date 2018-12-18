---
title: Data Factory 的資料管理閘道 | Microsoft Docs
description: 設定資料閘道器以在內部部署與雲端之間移動資料。 使用 Azure Data Factory 中的資料管理閘道移動資料。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.assetid: b9084537-2e1c-4e96-b5bc-0e2044388ffd
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 988c264ef6052b4b41de493944ac8d39a197a083
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43698752"
---
# <a name="data-management-gateway"></a>資料管理閘道
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [自我裝載整合執行階段](../create-self-hosted-integration-runtime.md)。 

> [!NOTE]
> 資料管理閘道現已更名為「自我裝載整合執行階段」。  

資料管理閘道是一個用戶端代理程式，您必須在內部部署環境中部署此代理程式，才能在雲端與內部部署資料存放區之間複製資料。 如需 Data Factory 所支援的內部部署資料存放區，請參閱 [支援的資料來源](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 一節。

本文是用來補充 [在內部部署和雲端資料存放區之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文中的逐步解說。 在該逐步解說中，您會建立一個使用閘道將資料從內部部署 SQL Server 資料庫移到 Azure Blob 的管線。 這篇文章提供有關資料管理閘道的詳細深入資訊。 

您可以將多個內部部署機器關聯到閘道以相應放大資料管理閘道。 您可以增加可在節點上同時執行的資料移動作業數目來進行相應增加。 這項功能也適用於具有單一節點的邏輯閘道。 如需得詳細資料，請參閱[在 Azure Data Factory 中調整資料管理閘道](data-factory-data-management-gateway-high-availability-scalability.md)一文。

> [!NOTE]
> 目前在 Data Factory 中，閘道器只支援複製活動和預存程序活動。 您不能使用自訂活動中的閘道器來存取內部部署資料來源。      

## <a name="overview"></a>概觀
### <a name="capabilities-of-data-management-gateway"></a>資料管理閘道功能
資料管理閘道提供下列功能：

* 在相同 Data Factory 內建立內部部署資料來源和雲端資料來源的模型及移動資料。
* 具有用於監視和管理的單一窗格，可從 [Data Factory] 頁面看見閘道狀態。
* 安全地管理內部部署資料來源的存取權。
  * 不需要變更公司防火牆。 閘道器只會使輸出 HTTP 連線開啟網際網路。
  * 利用您的憑證加密內部部署資料存放區的認證。
* 有效率地移動資料 - 資料會以平行方式傳輸，且系統會採用自動重試邏輯，修復間歇性網路問題。

### <a name="command-flow-and-data-flow"></a>命令流程和資料流程
當您使用複製活動在內部部署與雲端之間複製資料時，該活動會使用閘道將資料從內部部署資料來源轉移到雲端，以及反向操作。

利用資料閘道來進行複製的高階資料流程和步驟摘要如下：![使用閘道的資料流程](./media/data-factory-data-management-gateway/data-flow-using-gateway.png)

1. 資料開發人員會使用 [Azure 入口網站](https://portal.azure.com)或 [PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.datafactories/)，為 Azure Data Factory 建立閘道器。
2. 資料開發人員會藉由指定閘道，建立內部部署資料存放區的連結服務。 在設定連結服務資料的過程中，開發人員會使用 [設定認證] 應用程式來指定驗證類型和認證。  [設定認證] 應用程式對話方塊將會與資料存放區進行通訊，以測試要儲存認證的連線與閘道。
3. 在雲端中儲存認證之前，閘道會利用與閘道 (由資料開發人員提供) 相關聯的憑證加密認證。
4. Data Factory 服務會和閘道進行通訊，以透過使用共用 Azure 服務匯流排佇列的控制通道，進行工作的排程和管理。 必須開始複製活動作業時，Data Factory 會將要求和認證資訊一起排入佇列。 輪詢佇列之後，閘道器隨即啟動。
5. 閘道會利用相同的憑證解密認證，然後利用適當的驗證類型和認證連接到內部部署資料存放區。
6. 閘道會根據「複製活動」在資料管線中的設定方式，將資料從內部部署存放區複製到雲端儲存體，或反向操作。 針對這個步驟，閘道會透過安全的 (HTTPS) 通道，直接與雲端式儲存體服務 (例如 Azure Blob 儲存體) 進行通訊。

### <a name="considerations-for-using-gateway"></a>使用閘道的考量
* 您可以將單一資料管理閘道執行個體用於多個內部部署資料來源。 不過， **單一閘道執行個體只會繫結至一個 Azure Data Factory** ，不能與另一個 Data Factory 共用。
* 單一電腦上**只能安裝一個資料管理閘道的執行個體**。 假設您有兩個需要存取內部部署資料來源的 Data Factory，您就需要在兩部內部部署電腦上安裝閘道。 換句話說，閘道會繫結至特定的 Data Factory
* **閘道不一定要在與資料來源相同的電腦上**。 不過，讓閘道較靠近資料來源可縮短閘道連線到資料來源的時間。 建議您將閘道安裝在與裝載內部部署資料來源的機器不同的機器上。 當閘道和資料來源位於不同的機器上時，閘道才不會與資料來源爭奪資源。
* 您可以有「多個閘道器在不同電腦上，但連接至相同的內部部署資料來源」 。 例如，您可能有兩個閘道器用於服務兩個 Data Factory，但相同的內部部署資料來源都向這兩個 Data Factory 註冊。
* 若您已在電腦上安裝用於 **Power BI** 案例的閘道器，請於另一台電腦上安裝**另一個用於 Azure Data Factory 的閘道器**。
* 即使您使用 **ExpressRoute**，也必須使用閘道。
* 即使您使用 **ExpressRoute**，也應該將資料來源視為內部部署資料來源 (亦即在防火牆後面)。 請使用閘道來建立服務與資料來源之間的連線。
* 您必須**使用閘道**，即使資料存放區位於 **Azure IaaS VM** 上的雲端中。

## <a name="installation"></a>安裝
### <a name="prerequisites"></a>必要條件
* 支援的 **作業系統** 版本包括 Windows 7、Windows 8/8.1、Windows 10、Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2。 目前不支援在網域控制站上安裝資料管理閘道。
* 必須有 .NET Framework 4.5.1 或更新版本。 如果您要在 Windows 7 電腦上安裝閘道，請安裝 .NET Framework 4.5 或更新版本。 如需詳細資訊，請參閱 [.NET Framework 系統需求](https://msdn.microsoft.com/library/8z6watww.aspx) 。
* 建議的閘道機器「組態」  為至少 2 GHz、4 核心、8 GB RAM 和 80 GB 磁碟。
* 如果主機電腦休眠，閘道器不會回應資料要求。 因此，安裝閘道器之前，請先在電腦上設定適當的「電源計劃」  。 如果電腦已設定為休眠，安裝閘道時會提示訊息。
* 您必須是電腦上的系統管理員，才能成功安裝和設定資料管理閘道。 您可以將其他使用者新增至**資料管理閘道使用者**本機 Windows 群組。 此群組的成員可以使用**資料管理閘道組態管理員**工具來設定閘道器。

因為複製活動執行會以特定的頻率發生，在電腦上的資源使用量 (CPU、記憶體) 也會遵循與尖峰和閒置時間相同的模式。 資源使用率也仰賴要移動的資料量。 如果有多個複製作業正在進行，您會看到資源使用量在尖峰時段增加。

### <a name="installation-options"></a>安裝選項
可以用下列方式安裝資料管理閘道：

* 從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=39717)下載 MSI 安裝套件。  MSI 也可用來將現有的資料管理閘道升級至最新的版本，並保留所有設定。
* 按一下 [手動設定] 底下的 [下載並安裝資料閘道] 連結，或 [快速安裝] 之下的 [直接安裝在此電腦上]。 如需使用快速安裝的逐步指示，請參閱 [在內部部署與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文。 手動步驟會帶您前往下載中心。  下一節會提供從下載中心下載並安裝閘道的指示。

### <a name="installation-best-practices"></a>安裝最佳作法：
1. 為閘道器設定主機電腦上的電源計劃，使電腦不休眠。 如果主機電腦休眠，閘道器不會回應資料要求。
2. 請備份與閘道器相關聯的憑證。

### <a name="install-the-gateway-from-download-center"></a>從下載中心安裝閘道
1. 瀏覽至 [Microsoft 資料管理閘道下載頁面](https://www.microsoft.com/download/details.aspx?id=39717)。
2. 按一下 [下載]，選取適當的版本 (**32 位元**與**64 位元**)，然後按 [下一步]。
3. 直接執行 **MSI** 或將它儲存至您的硬碟並執行。
4. 在 [歡迎] 頁面上，選取一個**語言**，然後按 [下一步]。
5. **接受**使用者授權合約，然後按 [下一步]。
6. 選取要安裝閘道的**資料夾**，然後按 [下一步]。
7. 在 [準備安裝] 頁面上，按一下 [安裝]。
8. 按一下 [完成]  來完成安裝。
9. 從 Azure 入口網站取得金鑰。 如需逐步指示，請參閱下一節。
10. 在您機器上執行的**資料管理閘道組態管理員**中的 [註冊閘道器] 頁面上，執行下列步驟：
    1. 將金鑰貼在文字中。
    2. (選擇性) 按一下 [顯示閘道器金鑰]  以查看金鑰文字。
    3. 按一下 [註冊] 。

### <a name="register-gateway-using-key"></a>使用金鑰註冊閘道
#### <a name="if-you-havent-already-created-a-logical-gateway-in-the-portal"></a>如果您尚未在入口網站中建立邏輯閘道
若要在入口網站中建立閘道並從 [設定] 頁面取得金鑰，請依照[在內部部署和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)一文中的逐步解說步驟操作。    

#### <a name="if-you-have-already-created-the-logical-gateway-in-the-portal"></a>如果您已經在入口網站中建立邏輯閘道
1. 在 Azure 入口網站中，瀏覽至 [Data Factory] 頁面，然後按一下 [連結服務] 圖格。

    ![Data Factory 頁面](media/data-factory-data-management-gateway/data-factory-blade.png)
2. 在 [已連結的服務] 頁面中，選取您在入口網站中建立的邏輯**閘道**。

    ![邏輯閘道](media/data-factory-data-management-gateway/data-factory-select-gateway.png)  
3. 在 [資料閘道] 頁面中，按一下 [下載並安裝資料閘道]。

    ![入口網站中的下載連結](media/data-factory-data-management-gateway/download-and-install-link-on-portal.png)   
4. 在 [設定] 頁面中，按一下 [重新建立金鑰]。 在仔細閱讀警告訊息後，請按一下 [是]。

    ![重新建立索引鍵](media/data-factory-data-management-gateway/recreate-key-button.png)
5. 按一下金鑰旁的 [複製] 按鈕。 金鑰會複製到剪貼簿中。

    ![複製金鑰](media/data-factory-data-management-gateway/copy-gateway-key.png)

### <a name="system-tray-icons-notifications"></a>系統匣圖示/通知
下圖顯示您會看到的某些系統匣圖示。

![系統匣圖示](./media/data-factory-data-management-gateway/gateway-tray-icons.png)

如果您將游標放在系統匣圖示/通知訊息上，您會在快顯視窗中看到閘道/更新作業的狀態詳細資料。

### <a name="ports-and-firewall"></a>連接埠和防火牆
有兩個您需要考量的防火牆：在組織的中央路由器上執行的**公司防火牆**，以及在已安裝閘道的本機電腦上設定為精靈的 **Windows 防火牆**。  

![防火牆](./media/data-factory-data-management-gateway/firewalls2.png)

在公司防火牆層級，您需要設定下列網域和輸出連接埠：

| 網域名稱 | 連接埠 | 說明 |
| --- | --- | --- |
| *.servicebus.windows.net |443 |用於與「資料移動服務」後端進行通訊 |
| *.core.windows.net |443 |用於使用 Azure Blob 的分段複製 (如果已設定)|
| *.frontend.clouddatahub.net |443 |用於與「資料移動服務」後端進行通訊 |
| *.servicebus.windows.net |9350-9354, 5671 |透過 TCP 的選擇性服務匯流排轉送，由複製精靈所使用 |


Windows 防火牆層級通常會啟用這些輸出連接埠。 如果沒有，您可以在閘道電腦上相應地設定網域和連接埠。

> [!NOTE]
> 1. 視您的來源/接收器而定，您可能需要將額外的網域和輸出連接埠加到您公司/Windows 防火牆的白名單中。
> 2. 對於某些雲端資料庫 (例如：[Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-configure-firewall-settings)、[Azure Data Lake](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-secure-data#set-ip-address-range-for-data-access) 等)，您可能需要將閘道電腦的 IP 位址加到其防火牆組態的白名單中。
>
>


#### <a name="copy-data-from-a-source-data-store-to-a-sink-data-store"></a>將資料從來源資料存放區複製到接收資料存放區
請確定在公司防火牆、閘道機器上的 Windows 防火牆及資料存放區本身都已正確啟用防火牆規則。 啟用這些規則可讓閘道成功連接到來源和接收器。 請為複製作業所涉及的每個資料存放區啟用規則。

例如，若要 **從內部部署資料存放區複製到 Azure SQL Database 接收器或 Azure SQL 資料倉儲接收器**，執行下列步驟︰

* 在 Windows 防火牆和公司防火牆的通訊埠 **1433** 上都允許進行輸出 **TCP** 通訊。
* 設定 Azure SQL 伺服器的防火牆設定，將閘道機器的 IP 位址新增到允許的 IP 位址清單中。

> [!NOTE]
> 如果您的防火牆不允許使用輸出連接埠 1433，閘道將無法直接存取 Azure SQL。 在此情況下，您可以使用[分段複製](https://docs.microsoft.com/azure/data-factory/data-factory-copy-activity-performance#staged-copy)來移到 SQL Azure Database/SQL Azure DW。 在此案例中，您只需要 HTTPS (連接埠 443) 即可進行資料移動。
>
>


### <a name="proxy-server-considerations"></a>Proxy 伺服器考量
如果您的公司網路環境使用 Proxy 伺服器來存取網際網路，請將資料管理閘道設定為使用適當的 Proxy 設定。 您可以在初始註冊階段期間設定 Proxy。

![在註冊期間設定 Proxy](media/data-factory-data-management-gateway/SetProxyDuringRegistration.png)

閘道會使用 Proxy 伺服器來連線到雲端服務。 進行初始設定時，按一下 [變更]  連結。 您會看到 [Proxy 設定]  對話方塊。

![使用組態管理員來設定 Proxy](media/data-factory-data-management-gateway/SetProxySettings.png)

有三個組態選項：

* **不使用 Proxy**︰閘道不會明確地使用任何 Proxy 來連線到雲端服務。
* **使用系統 Proxy**：閘道會使用 diahost.exe.config 和 diawp.exe.config 中設定的 Proxy 設定。如果 diahost.exe.config 和 diawp.exe.config 中未設定任何 Proxy，閘道就會直接連線到雲端服務而不經由 Proxy。
* **使用自訂 Proxy**：設定要用於閘道的 HTTP Proxy 設定，而不使用 diahost.exe.config 和 diawp.exe.config 中的組態。必須指定「IP 位址」和「連接埠」。  「使用者名稱」和「密碼」則為選擇性，需視您的 Proxy 驗證設定而定。  所有設定都會由閘道的認證憑證予以加密，並儲存在閘道主機機器的本機。

在您儲存已更新的 Proxy 設定之後，資料管理閘道主機服務會自動重新啟動。

在成功註冊閘道之後，如果您想要檢視或更新 Proxy 設定，請使用「資料管理閘道組態管理員」。

1. 啟動 **資料管理閘道器組態管理員**。
2. 切換到 [設定]  索引標籤。
3. 按一下 [HTTP Proxy] 區段中的 [變更] 連結，以啟動 [設定 HTTP Proxy] 對話方塊。  
4. 按 [下一步]  按鈕之後，您會看到一個警告對話方塊，此對話方塊會向您請求權限來儲存 Proxy 設定及重新啟動「閘道主機服務」。

您可以使用「組態管理員」工具來更新 HTTP Proxy。

![使用組態管理員來設定 Proxy](media/data-factory-data-management-gateway/SetProxyConfigManager.png)

> [!NOTE]
> 如果您為 Proxy 伺服器設定了 NTLM 驗證，「閘道主機服務」就會以網域帳戶執行。 如果您稍後變更網域帳戶的密碼，請記得更新服務的組態設定並相應地將它重新啟動。 基於這項需求，建議您使用不需要經常更新密碼的專用網域帳戶來存取 Proxy 伺服器。
>
>

### <a name="configure-proxy-server-settings"></a>設定 Proxy 伺服器設定
如果您為 HTTP Proxy 選取 [使用系統 Proxy] 設定，閘道就會使用 diahost.exe.config 和 diawp.exe.config 中的 Proxy 設定。如果 diahost.exe.config 和 diawp.exe.config 中未指定任何 Proxy，閘道就會直接連線到雲端服務而不經由 Proxy。 下列程序說明如何更新 diahost.exe.config 檔案。  

1. 在「檔案總管」中，建立一份 C:\Program Files\Microsoft Data Management Gateway\2.0\Shared\diahost.exe.config 的安全複本來備份原始檔案。
2. 以系統管理員身分啟動 Notepad.exe，並開啟文字檔 C:\Program Files\Microsoft Data Management Gateway\2.0\Shared\diahost.exe.config。您會在以下程式碼中看見 system.net 的預設標籤：

         <system.net>
             <defaultProxy useDefaultCredentials="true" />
         </system.net>    

   接著，您可以新增 Proxy 伺服器詳細資料，如以下範例所示：

         <system.net>
               <defaultProxy enabled="true">
                     <proxy bypassonlocal="true" proxyaddress="http://proxy.domain.org:8888/" />
               </defaultProxy>
         </system.net>

   在 Proxy 標記內可以有其他屬性，用以指定必要的設定，例如 scriptLocation。 請參閱 [Proxy 項目 (網路設定)](https://msdn.microsoft.com/library/sa91de1e.aspx) 以了解語法。

         <proxy autoDetect="true|false|unspecified" bypassonlocal="true|false|unspecified" proxyaddress="uriString" scriptLocation="uriString" usesystemdefault="true|false|unspecified "/>
3. 將組態檔儲存到原始位置中，然後重新啟動「資料管理閘道主機服務」以套用變更。 重新啟動服務：使用 [控制台] 中的 [服務] 小程式，或是從 [資料管理閘道組態管理員] > 按一下 [停止服務] 按鈕，然後按一下 [啟動服務]。 如果服務未啟動，可能因為在已編輯的應用程式組態檔中加入了不正確的 XML 標記語法。

> [!IMPORTANT]
> 別忘了「同時」更新 diahost.exe.config 和 diawp.exe.config。  


除了這幾點以外，您也必須確定 Microsoft Azure 包含在公司的允許清單中。 如需有效的 Microsoft Azure IP 位址清單，可從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=41653)下載。

#### <a name="possible-symptoms-for-firewall-and-proxy-server-related-issues"></a>防火牆和 Proxy 伺服器相關問題的可能徵兆
如果發生類似以下的錯誤，有可能是因為防火牆或 Proxy 伺服器的組態不正確，使得閘道無法連線到 Data Factory 來進行自我驗證。 請參閱上一節，以確保您的防火牆和 Proxy 伺服器的設定皆正確。

1. 當您嘗試註冊閘道器時，您會收到下列錯誤：「無法註冊閘道器金鑰。 再次嘗試註冊閘道器金鑰之前，請確認資料管理閘道已處於連線狀態，且已啟動資料管理閘道主機服務。」
2. 當您開啟「組態管理員」時，您會看到「已中斷連線」或「正在連線」狀態。 檢視 Windows 事件記錄時，在 [事件檢視器] > [應用程式和服務記錄] > [資料管理閘道] 下，您會看到如以下的錯誤訊息：`Unable to connect to the remote server`
   `A component of Data Management Gateway has become unresponsive and restarts automatically. Component name: Gateway.`

### <a name="open-port-8050-for-credential-encryption"></a>開啟用於認證加密的連接埠 8050
當您在 Azure 入口網站中設定了內部部署連結服務時，**設定認證**應用程式會使用輸入連接埠 **8050** 將認證轉送到閘道。 閘道設定期間，閘道安裝預設會在閘道電腦上開啟此連接埠。

如果您使用協力廠商的防火牆，則可以手動開啟連接埠 8050。 如果您在設定閘道時遇到防火牆問題，您可以嘗試使用下列命令來安裝閘道，而不設定防火牆。

    msiexec /q /i DataManagementGateway.msi NOFIREWALL=1

如果您選擇不開啟閘道機器上的連接埠 8050，則請使用「設定認證」  應用程式以外的機制來設定資料存放區認證。 例如，您可以使用 [New-AzureRmDataFactoryEncryptValue](https://docs.microsoft.com/powershell/module/azurerm.datafactories/new-azurermdatafactoryencryptvalue) PowerShell Cmdlet。 如需了解如何設定資料存放區認證，請參閱 [設定認證和安全性](#set-credentials-and-securityy) 一節。

## <a name="update"></a>更新
根據預設，資料管理閘道會在有更新版本的閘道時自動進行更新。 在所有排定的工作完成前，閘道不會進行更新。 更新作業完成後，閘道才會處理後續的工作。 如果更新失敗，閘道會回復為舊版本。

您會在下列位置看到已排定的更新時間︰

* Azure 入口網站中的 [閘道屬性] 頁面。
* 「資料管理閘道組態管理員」的 [首頁]。
* 系統匣通知訊息。

「資料管理閘道組態管理員」的 [首頁] 索引標籤會顯示更新排程，以及上次安裝/更新閘道的時間。

![更新排程](media/data-factory-data-management-gateway/UpdateSection.png)

您可以立即安裝更新，或等候閘道在排定時間自動更新。 例如，下圖顯示的是「閘道組態管理員」中所顯示的通知訊息以及 [更新] 按鈕，按一下此按鈕即可立即安裝更新。

![DMG 組態管理員中的更新](./media/data-factory-data-management-gateway/gateway-auto-update-config-manager.png)

系統匣中的通知訊息看起來就像下圖：

![系統匣訊息](./media/data-factory-data-management-gateway/gateway-auto-update-tray-message.png)

您會在系統匣中看到更新作業 (手動或自動) 的狀態。 下次啟動「閘道組態管理員」時，您會在通知列上看到指出閘道已更新的訊息，以及一個連到 [新功能主題](data-factory-gateway-release-notes.md)的連結。

### <a name="to-disableenable-auto-update-feature"></a>停用/啟用自動更新功能
您可以執行下列步驟來停用/啟用自動更新功能：

[適用於單一節點閘道]
1. 在閘道電腦上啟動 Windows PowerShell。
2. 切換至 C:\Program Files\Microsoft Integration Runtime\3.0\PowerShellScript\ 資料夾。
3. 執行下列命令，將自動更新功能關閉 (停用)。   

    ```PowerShell
    .\IntegrationRuntimeAutoUpdateToggle.ps1  -off
    ```
4. 若要將它重新開啟：

    ```PowerShell
    .\IntegrationRuntimeAutoUpdateToggle.ps1 -on  
    ```
[適用於多節點高度可用且可調整的閘道](data-factory-data-management-gateway-high-availability-scalability.md)
1. 在閘道電腦上啟動 Windows PowerShell。
2. 切換至 C:\Program Files\Microsoft Integration Runtime\3.0\PowerShellScript\ 資料夾。
3. 執行下列命令，將自動更新功能關閉 (停用)。   

    對於具備高可用性功能的閘道，需要額外的 AuthKey 參數。
    ```PowerShell
    .\IntegrationRuntimeAutoUpdateToggle.ps1  -off -AuthKey <your auth key>
    ```
4. 若要將它重新開啟：

    ```PowerShell
    .\IntegrationRuntimeAutoUpdateToggle.ps1  -on -AuthKey <your auth key> 
    ```

## <a name="configuration-manager"></a>組態管理員
安裝閘道後，您可以用下列方式啟動 [資料管理閘道組態管理員]：

1. 在 [搜尋] 視窗中，輸入**資料管理閘道**以存取這個公用程式。
2. 執行下列資料夾中的 **ConfigManager.exe** 可執行檔：**C:\Program Files\Microsoft Data Management Gateway\2.0\Shared**

### <a name="home-page"></a>首頁
首頁可讓您執行下列動作︰

* 檢視閘道的狀態 (連接至雲端服務等)。
*  。
* 在閘道電腦上**停止**後啟動 [資料管理閘道主機服務]。
* **更新排程** 。
* 檢視閘道 **上次更新**時的日期。

### <a name="settings-page"></a>設定頁面
[設定] 頁面可讓您執行下列動作︰

* 檢視、變更及匯出閘道所使用的 **憑證** 。 此憑證用來加密資料來源認證。
* 變更端點的 **HTTPS 連接埠** 。 閘道會開啟一個連接埠，以便設定資料來源認證。
* **狀態** 
* 檢視 [SSL 憑證]  用於入口網站與閘道之間的 SSL 通訊，以設定資料來源的認證。  

### <a name="remote-access-from-intranet"></a>來自內部網路的遠端存取  
未來將會啟用此功能。 在即將推出的更新 (v3.4 或更新版本) 中，我們將讓您啟用/停用任何目前使用連接埠 8050 來進行的遠端連線 (請參閱上一節)，同時使用 PowerShell 或「認證管理員」應用程式來加密認證。 

### <a name="diagnostics-page"></a>診斷頁面
[診斷] 頁面可讓您執行下列動作︰

* 啟用詳細資訊 **記錄**、在事件檢視器中檢視記錄檔，以及有失敗時將記錄檔傳送給 Microsoft。
* **測試連線** (對資料來源的連線)。  

### <a name="help-page"></a>Help page
[說明] 頁面會顯示以下資訊：  

* 閘道的簡短說明。
* 版本號碼
* 線上說明、隱私權聲明及授權合約的連結。  

## <a name="monitor-gateway-in-the-portal"></a>在入口網站中監視閘道
在 Azure 入口網站中，您可以檢視閘道機器近乎即時的資源使用率 (CPU、記憶體、網路 (輸入/輸出) 等) 快照集。  

1. 在 Azure 入口網站中，瀏覽至您資料處理站的首頁，然後按一下 [已連結的服務] 圖格。 

    ![Data Factory 首頁](./media/data-factory-data-management-gateway/monitor-data-factory-home-page.png) 
2. 在 [已連結的服務] 頁面中選取**閘道**。

    ![[已連結的服務] 頁面](./media/data-factory-data-management-gateway/monitor-linked-services-blade.png)
3. 在 [閘道] 頁面中，您可以看到閘道的記憶體和 CPU 使用量。

    ![閘道的 CPU 和記憶體使用量](./media/data-factory-data-management-gateway/gateway-simple-monitoring.png) 
4. 啟用 [進階設定] 可查看更多詳細資料，例如網路使用量。
    
    ![閘道的進階監視](./media/data-factory-data-management-gateway/gateway-advanced-monitoring.png)

下表說明 [閘道節點] 清單中的資料行：  

監視屬性 | 說明
:------------------ | :---------- 
名稱 | 邏輯閘道和閘道相關聯節點的名稱。 節點是安裝了閘道的內部部署 Windows 機器。 若要了解如何在單一邏輯閘道中擁有一個以上的節點 (最多四個節點)，請參閱[資料管理閘道 - 高可用性和延展性](data-factory-data-management-gateway-high-availability-scalability.md)。    
狀態 | 邏輯閘道和閘道節點的狀態。 範例：線上/離線/受限制/等等。如需這些狀態的相關資訊，請參閱[閘道狀態](#gateway-status)一節。 
版本 | 顯示邏輯閘道和每個閘道節點的版本。 邏輯閘道的版本取決於群組中大多數節點的版本。 如果邏輯閘道設定中有不同版本的節點，則只有版本號碼和邏輯閘道相同的節點會正常運作。 其他節點會進入受限制模式，並需要加以手動更新 (如果自動更新失敗才需要這麼做)。 
可用的記憶體 | 閘道節點上可用的記憶體。 這個值是近乎即時的快照集。 
CPU 使用率 | 閘道節點的 CPU 使用率。 這個值是近乎即時的快照集。 
網路功能 (輸入/輸出) | 閘道節點的網路使用率。 這個值是近乎即時的快照集。 
並行作業 (執行中/限制) | 每個節點上執行的作業或工作數目。 這個值是近乎即時的快照集。 限制表示每個節點的最大並行作業數。 這個值會根據機器大小來定義。 您可以提高限制以在進階案例 (CPU/記憶體/網路並未充分使用，但活動會逾時的案例) 中相應增加並行作業執行能力。單一節點的閘道也能擁有這樣的能力 (即使該閘道尚未啟用延展性和可用性功能)。  
角色 | 多節點閘道中的角色有兩種 - 發送器和背景工作角色。 所有節點都是背景工作角色，這表示它們全都能用來執行作業。 發送器節點只有一個，可用來提取雲端服務中的工作/作業，並發送到不同的背景工作節點 (包括發送器節點本身)。

當閘道中有兩個以上的節點 (相應放大案例) 時，您會在此頁面中看到某些更合理的設定。 如需設定多節點閘道的詳細資料，請參閱[資料管理閘道 - 高可用性和延展性](data-factory-data-management-gateway-high-availability-scalability.md)。

### <a name="gateway-status"></a>閘道狀態
下表提供**閘道節點**的可能狀態： 

狀態  | 註解/案例
:------- | :------------------
線上 | 節點已連線至 Data Factory 服務。
離線 | 節點已離線。
升級中 | 節點正在自動更新。
限制 | 由於連線問題。 可能是因為 HTTP 連接埠 8050 問題、服務匯流排連線問題或認證同步問題。 
非使用中 | 節點所在的組態不同於其他大多數節點的組態。<br/><br/> 節點無法連線至其他節點時，便會處於非使用中狀態。 


下表提供**邏輯閘道**的可能狀態。 閘道的狀態取決於閘道節點的狀態。 

狀態 | 註解
:----- | :-------
需要註冊 | 此邏輯閘道還沒有已註冊的節點
線上 | 閘道節點已連線
離線 | 沒有處於線上狀態的節點。
限制 | 此閘道中的節點並非全都處於健康情況良好的狀態。 這個狀態是某些節點可能會關閉的警告！ <br/><br/>可能是因為發送器/背景工作節點有認證同步問題。 

## <a name="scale-up-gateway"></a>相應增加閘道
您可以設定節點中可以執行的**並行資料移動作業**數目，以相應增加在內部部署機器與雲端資料存放區之間移動資料的能力。 

當可用的記憶體和 CPU 並未充分使用，但閒置容量已為 0，就應該增加節點上可執行的並行作業數目來進行相應增加。 當閘道超載而導致活動逾時，您也可以相應增加。 在閘道節點的進階設定中，您可以提高節點的容量上限。 
  

## <a name="troubleshooting-gateway-issues"></a>閘道問題疑難排解
如需對使用資料管理閘道的問題進行疑難排解的資訊/提示，請參閱[閘道問題疑難排解](data-factory-troubleshoot-gateway-issues.md)一文。  

## <a name="move-gateway-from-one-machine-to-another"></a>在機器之間移動閘道
本節提供將閘道器用戶端從一台電腦移至另一台電腦的步驟。

1. 在入口網站中，瀏覽至 **Data Factory 首頁**，然後按一下 [連結服務] 圖格。

    ![資料閘道連結](./media/data-factory-data-management-gateway/DataGatewaysLink.png)
2. 在 [連結服務] 頁面的 [資料閘道器] 區段中選取您的閘道器。

    ![[已連結的服務] 頁面與所選取的閘道](./media/data-factory-data-management-gateway/LinkedServiceBladeWithGateway.png)
3. 在 [資料閘道] 頁面中，按一下 [下載並安裝資料閘道]。

    ![下載閘道器連結](./media/data-factory-data-management-gateway/DownloadGatewayLink.png)
4. 在 [設定] 頁面中，按一下 [下載並安裝資料閘道]，然後依照指示在機器上安裝資料閘道。

    ![[設定] 頁面](./media/data-factory-data-management-gateway/ConfigureBlade.png)
5. 讓 [Microsoft 資料管理閘道組態管理員]  保持開啟。

    ![組態管理員](./media/data-factory-data-management-gateway/ConfigurationManager.png)    
6. 在入口網站的 [設定] 頁面中，按一下命令列上的 [重新建立金鑰]，然後按一下警告訊息中的 [是]。 按一下金鑰文字旁的 [複製]  按鈕，以將金鑰複製到剪貼簿。 一旦重新建立索引鍵，舊電腦上的閘道器便會停止運作。  

    ![重新建立索引鍵](./media/data-factory-data-management-gateway/RecreateKey.png)
7. 在您的電腦上，將**索引鍵**貼入**資料管理閘道組態管理員**之 [註冊閘道器] 頁面上的文字方塊。 (選擇性) 按一下 [顯示閘道器金鑰]  核取方塊以查看金鑰文字。

    ![複製金鑰與註冊](./media/data-factory-data-management-gateway/CopyKeyAndRegister.png)
8. 按一下 [註冊]  透過雲端服務註冊閘道器。
9. 在 [設定] 索引標籤上，按一下 [變更] 以選取舊閘道所使用的憑證，輸入**密碼**，然後按一下 [完成]。

   ![指定憑證](./media/data-factory-data-management-gateway/SpecifyCertificate.png)

   您可以執行下列步驟，從舊閘道器中匯出憑證：啟動舊電腦上的 [資料管理閘道組態管理員]，切換到 [憑證] 索引標籤，按一下 [匯出] 按鈕，然後遵循指示進行。
10. 成功註冊閘道器後，閘道器組態管理員首頁上的 [註冊] 應會設定為 [已註冊]，[狀態] 會設定為 [已啟動]。

## <a name="encrypting-credentials"></a>加密認證
若要在 Data Factory 編輯器中加密認證，請執行下列步驟︰

1. 在「閘道機器」 上啟動網頁瀏覽器，瀏覽至 [Azure 入口網站](http://portal.azure.com)。 視需要搜尋您的 Data Factory，在 [DATA FACTORY] 頁面中開啟 Data Factory，然後按一下 [編寫及部署] 來啟動 Data Factory 編輯器。   
2. 在樹狀檢視中按一下現有的 **連結服務** ，以查看其 JSON 定義或建立需要資料管理閘道 (例如︰SQL Server 或 Oracle) 的連結服務。
3. 在 JSON 編輯器中，為 **gatewayName** 屬性輸入閘道的名稱。
4. 在 **connectionString** 中輸入**資料來源**屬性的伺服器名稱。
5. 在 **connectionString** 中輸入**初始目錄**屬性的資料庫名稱。    
6. 在命令列上按一下 [加密] 按鈕，以啟動 Click Once **認證管理員**應用程式。 您應該會看見 [設定認證]  對話方塊。

    ![[設定認證] 對話方塊](./media/data-factory-data-management-gateway/setting-credentials-dialog.png)
7. 在 [設定認證]  對話方塊中，執行下列步驟：
   1. 選取您要 Data Factory 服務用來連接到資料庫的 **驗證** 。
   2. 在 [使用者名稱]  設定中輸入可存取資料庫的使用者名稱。
   3. 在 [密碼]  設定中輸入使用者的密碼。  
   4. 按一下 [確定]  以加密認證並關閉對話方塊。
8. 您現在應該會在 **connectionString** 中看到 **encryptedCredential** 屬性。

    ```JSON
    {
        "name": "SqlServerLinkedService",
        "properties": {
            "type": "OnPremisesSqlServer",
            "description": "",
            "typeProperties": {
                "connectionString": "data source=myserver;initial catalog=mydatabase;Integrated Security=False;EncryptedCredential=eyJDb25uZWN0aW9uU3R",
                "gatewayName": "adftutorialgateway"
            }
        }
    }
    ```
如果您從閘道器電腦以外的另一台電腦存取入口網站，您必須確定「認證管理員」應用程式可以連接到閘道器電腦。 如果應用程式無法連接閘道器電腦，它不會允許您設定資料來源的認證，以及測試資料來源的連接。  

當您使用**設定認證**應用程式時，入口網站會使用在閘道機器上**閘道組態管理員**的 [憑證] 索引標籤中指定的憑證來加密認證。

如果您要尋找以 API 為基礎的方法來加密認證，可以使用 [New-AzureRmDataFactoryEncryptValue](https://docs.microsoft.com/powershell/module/azurerm.datafactories/new-azurermdatafactoryencryptvalue) PowerShell Cmdlet 來加密認證。 此 cmdlet 會使用閘道器設定用來加密認證的憑證。 您需將加密認證新增到 JSON 中 **connectionString** 的 **EncryptedCredential** 元素中。 您需搭配 [New-AzureRmDataFactoryLinkedService](https://docs.microsoft.com/powershell/module/azurerm.datafactories/new-azurermdatafactorylinkedservice) Cmdlet 或在「Data Factory 編輯器」中使用 JSON。

```JSON
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```

使用「Data Factory 編輯器」來設定認證還有另一個方法。 如果您使用該編輯器來建立 SQL Server 連結服務，並以純文字輸入認證，系統就會使用 Data Factory 服務所擁有的憑證來加密該認證。 它「不會」使用已設定讓閘道使用的憑證。 雖然這種方法在某些情況下可能快一點，但也比較不安全。 因此，建議您只在開發/測試用途才採用此方法。

## <a name="powershell-cmdlets"></a>PowerShell Cmdlet
本節說明如何使用 Azure PowerShell Cmdlet 建立和註冊閘道。

1. 在系統管理員模式下啟動 **Azure PowerShell** 。
2. 請執行下列命令並輸入您的 Azure 認證，登入您的 Azure 帳戶。

    ```PowerShell
    Connect-AzureRmAccount
    ```
3. 使用 **New-AzureRmDataFactoryGateway** Cmdlet 來建立邏輯閘道器，如下所示：

    ```PowerShell
    $MyDMG = New-AzureRmDataFactoryGateway -Name <gatewayName> -DataFactoryName <dataFactoryName> -ResourceGroupName ADF –Description <desc>
    ```
    **範例命令和輸出**：

    ```
    PS C:\> $MyDMG = New-AzureRmDataFactoryGateway -Name MyGateway -DataFactoryName $df -ResourceGroupName ADF –Description “gateway for walkthrough”

    Name              : MyGateway
    Description       : gateway for walkthrough
    Version           :
    Status            : NeedRegistration
    VersionStatus     : None
    CreateTime        : 9/28/2014 10:58:22
    RegisterTime      :
    LastConnectTime   :
    ExpiryTime        :
    ProvisioningState : Succeeded
    Key               : ADF#00000000-0000-4fb8-a867-947877aef6cb@fda06d87-f446-43b1-9485-78af26b8bab0@4707262b-dc25-4fe5-881c-c8a7c3c569fe@wu#nfU4aBlq/heRyYFZ2Xt/CD+7i73PEO521Sj2AFOCmiI
    ```

1. 在 Azure PowerShell 中，切換至資料夾：**C:\Program Files\Microsoft Data Management Gateway\2.0\PowerShellScript\\**。 執行與區域變數 **$Key** 相關聯的 **RegisterGateway.ps1**，如下列命令所示。 此指令碼會向您稍早建立的邏輯閘道註冊您機器上安裝的用戶端代理程式。

    ```PowerShell
    PS C:\> .\RegisterGateway.ps1 $MyDMG.Key
    ```
    ```
    Agent registration is successful!
    ```
    您可以使用 IsRegisterOnRemoteMachine 參數在遠端電腦上註冊閘道器。 範例：

    ```PowerShell
    .\RegisterGateway.ps1 $MyDMG.Key -IsRegisterOnRemoteMachine true
    ```
2. 您可以使用 **Get-AzureRmDataFactoryGateway** Cmdlet 取得 Data Factory 中的閘道器器清單。 當 [狀態] 顯示為 [線上] 時，表示您的閘道器已就緒可供使用。

    ```PowerShell        
    Get-AzureRmDataFactoryGateway -DataFactoryName <dataFactoryName> -ResourceGroupName ADF
    ```
您可以使用 **Remove-AzureRmDataFactoryGateway** Cmdlet 移除閘道器，並使用 **Set-AzureRmDataFactoryGateway** Cmdlet 更新閘道器的說明。 如需這些 Cmdlet 的語法及其他詳細資訊，請參閱 Data Factory Cmdlet 參考文件。  

### <a name="list-gateways-using-powershell"></a>使用 PowerShell 列出閘道器

```PowerShell
Get-AzureRmDataFactoryGateway -DataFactoryName jasoncopyusingstoredprocedure -ResourceGroupName ADF_ResourceGroup
```

### <a name="remove-gateway-using-powershell"></a>使用 PowerShell 移除閘道器

```PowerShell
Remove-AzureRmDataFactoryGateway -Name JasonHDMG_byPSRemote -ResourceGroupName ADF_ResourceGroup -DataFactoryName jasoncopyusingstoredprocedure -Force
```


## <a name="next-steps"></a>後續步驟
* 請參閱 [在內部部署和雲端資料存放區之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文。 在該逐步解說中，您會建立一個使用閘道將資料從內部部署 SQL Server 資料庫移到 Azure Blob 的管線。  
