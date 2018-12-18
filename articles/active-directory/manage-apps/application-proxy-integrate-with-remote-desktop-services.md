---
title: 使用 Azure AD App Proxy 發佈遠端桌面 | Microsoft Docs
description: 涵蓋 Azure AD 應用程式 Proxy 連接器的基本概念。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: barbkess
ms.custom: it-pro
ms.reviewer: harshja
ms.openlocfilehash: 5d8af50e3007342a5cd46e4862623f2cf7145172
ms.sourcegitcommit: fc5555a0250e3ef4914b077e017d30185b4a27e6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39480416"
---
# <a name="publish-remote-desktop-with-azure-ad-application-proxy"></a>使用 Azure AD 應用程式 Proxy 發佈遠端桌面

遠端桌面服務和 Azure AD Application Proxy 可依同合作，提升不在公司網路內的工作者的產能。 

本文的目標對象是︰
- 目前的應用程式 Proxy 客戶，想要透過遠端桌面服務發佈內部部署應用程式，以提供更多使用者應用程式。
- 目前的遠端桌面服務客戶，想要使用 Azure AD 應用程式 Proxy 來減少其部署的 Attack Surface。 這種情況下，會向 RDS 提供一組有限的雙步驟驗證和條件式存取控制。

## <a name="how-application-proxy-fits-in-the-standard-rds-deployment"></a>應用程式 Proxy 如何放入標準 RDS 部署中

標準 RDS 部署包含 Windows Server 上執行的各種遠端桌面角色服務。 查看[遠端桌面服務架構](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/desktop-hosting-logical-architecture)，有多個部署選項。 不同於其他 RDS 部署選項，[使用 Azure AD 應用程式 Proxy 的 RDS 部署](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/desktop-hosting-logical-architecture) (如下圖所示) 具有執行連接器服務之伺服器的永久輸出連線。 其他部署會透過負載平衡器保留開啟的輸入連線。

![應用程式 Proxy 位於 RDS VM 與公用網際網路之間](./media/application-proxy-integrate-with-remote-desktop-services/rds-with-app-proxy.png)

在 RDS 部署中，RD Web 角色和 RD 閘道角色是在網際網路對應的電腦上執行。 這些端點是公開的，原因如下︰
- RD Web 會向使用者提供公用端點，可用來登入及檢視他們可以存取的各種內部部署應用程式和桌上型電腦。 在選取資源時，會使用作業系統上的原生應用程式來建立 RDP 連線。
- 一旦使用者啟動 RDP 連線後，RD 閘道就會派上用場。 RD 閘道會處理來自網際網路的加密 RDP 流量，並將它轉譯為使用者所連線的內部部署伺服器。 在此案例中，RD 閘道正在接收的流量是來自 Azure AD 應用程式 Proxy。

>[!TIP]
>如果您之前從未部署過 RDS，或在您開始前需要更多資訊，請了解如何[使用 Azure Resource Manager 和 Azure Marketplace 順暢地部署 RDS](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/rds-in-azure)。

## <a name="requirements"></a>需求

- 因為 Web 用戶端不支援應用程式 Proxy，所以使用遠端桌面 Web 用戶端以外的用戶端。

- RD Web 和 RD 閘道端點必須位於相同的電腦上，並具有一般的根。 將 RD Web 和 RD 閘道發佈為單一應用程式並搭配應用程式 Proxy，如此便能在這兩個應用程式之間擁有單一登入的體驗。

- 您應該已經[部署 RDS](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/rds-in-azure) 並[啟用應用程式 Proxy](application-proxy-enable.md)。

- 此案例假設您的終端使用者在透過 RD 網頁連線的 Windows 7 或 Windows 10 桌上型電腦上使用 Internet Explorer。 如果您需要支援其他作業系統，請參閱[其他用戶端設定的支援](#support-for-other-client-configurations)。

- 發行 RD Web 時，建議您使用相同的內部和外部 FQDN。 如果內部和外部 FQDN 不同，則應停用要求標頭轉譯，以避免用戶端接收無效的連結。 

- 在 Internet Explorer 上，啟用 RDS ActiveX 附加元件。

## <a name="deploy-the-joint-rds-and-application-proxy-scenario"></a>部署聯合 RDS 和應用程式 Proxy 案例

為您的環境設定 RDS 與 Azure AD 應用程式 Proxy 之後，請遵循步驟將兩種解決方案結合。 這些步驟會逐步解說發佈兩個 Web 對向 RDS 端點 (RD Web 和 RD 閘道) 作為應用程式，然後將您 RDS 上的流量導向以通過應用程式 Proxy。

### <a name="publish-the-rd-host-endpoint"></a>發佈 RD 主機端點

1. 使用下列值[發佈新的應用程式 Proxy 應用程式](application-proxy-publish-azure-portal.md)︰
   - 內部 URL：`https://\<rdhost\>.com/`，其中 `\<rdhost\>` 是 RD Web 和 RD 閘道共用的一般根。
   - 外部 URL︰會根據應用程式名稱自動填入這個欄位，但您可以修改它。 您的使用者在存取 RDS 時，將會移到此 URL。
   - 預先驗證方法︰Azure Active Directory
   - 轉譯 URL 標頭：否
2. 將使用者指派給已發佈 RD 應用程式。 並請確定它們都可存取 RDS。
3. 保留應用程式的單一登入方法，因為 **Azure AD 單一登入已停用**。 系統會要求您的使用者分別驗證一次 Azure AD 及 RD Web，但可單一登入 RD 閘道。
4. 移至 [Azure Active Directory] > [應用程式註冊] > [您的應用程式] > [設定]。
5. 選取 [內容] 並更新 [首頁 URL] 欄位，以指向 RD Web 端點 (例如 `https://\<rdhost\>.com/RDWeb`)。

### <a name="direct-rds-traffic-to-application-proxy"></a>將 RDS 資料流導向應用程式 Proxy

以系統管理員身分連線至 RDS 部署，並變更部署的 RD 閘道伺服器名稱。 此組態可確保連線經過 Azure AD 應用程式 Proxy 服務。

1. 連線到執行 RD 連線代理人角色的 RDS 伺服器。
2. 啟動**伺服器管理員**。
3. 選取左窗格中的 [遠端桌面服務]。
4. 選取 [概觀]。
5. 在 [部署概觀] 區段中，選取下拉式選單，然後選擇 [編輯部署內容]。
6. 在 [RD 閘道] 索引標籤中，將 [伺服器名稱] 變更為您在應用程式 Proxy 中所設定之 RD 主機端點的外部 URL。
7. 將 [登入方法] 欄位變更為 [密碼驗證]。

  ![在 RDS 上部署內容畫面](./media/application-proxy-integrate-with-remote-desktop-services/rds-deployment-properties.png)

8. 對於每個集合執行此命令。 使用您自己的資訊來取代 *\<yourcollectionname\>* 和 *\<proxyfrontendurl\>*。 此命令會啟用 RD Web 和 RD 閘道之間的單一登入，並將效能最佳化︰

   ```
   Set-RDSessionCollectionConfiguration -CollectionName "<yourcollectionname>" -CustomRdpProperty "pre-authentication server address:s:<proxyfrontendurl>`nrequire pre-authentication:i:1"
   ```

   **例如：**
   ```
   Set-RDSessionCollectionConfiguration -CollectionName "QuickSessionCollection" -CustomRdpProperty "pre-authentication server address:s:https://remotedesktoptest-aadapdemo.msappproxy.net/`nrequire pre-authentication:i:1"
   ```

9. 若要確認自訂 RDP 屬性的修改，以及檢視將會針對此集合從 RDWeb 下載的 RDP 檔案內容，請執行下列命令：
    ```
    (get-wmiobject -Namespace root\cimv2\terminalservices -Class Win32_RDCentralPublishedRemoteDesktop).RDPFileContents
    ```

現在您已設定遠端桌面，Azure AD 應用程式 Proxy 已接管作為 RDS 的網際網路對應元件。 您可以將 RD Web 和 RD 閘道機器上的其他公用網際網路對應端點移除。

## <a name="test-the-scenario"></a>測試案例

在 Windows 7 或 10 電腦上使用 Internet Explorer 來測試案例。

1. 移至您所設定的外部 URL，或在 [MyApps 面板](https://myapps.microsoft.com)中尋找您的應用程式。
2. 系統會要求您向 Azure Active Directory 進行驗證。 使用您指派給應用程式的帳戶。
3. 系統會要求您向 RD Web 進行驗證。
4. 一旦 RDS 驗證成功後，您可以選取所需的桌上型電腦或應用程式，然後開始使用。

## <a name="support-for-other-client-configurations"></a>其他用戶端設定的支援

本文中所述的設定是針對具有 Internet Explorer 和 RDS ActiveX 附加元件之 Windows 7 或 10 上的使用者。 不過，如果需要，您可以支援其他作業系統或瀏覽器。 差異在於您所使用的驗證方法。

| 驗證方法 | 支援的用戶端設定 |
| --------------------- | ------------------------------ |
| 預先驗證    | 使用 Internet Explorer + RDS ActiveX 附加元件的 Windows 7/10 |
| 通道 | 支援 Microsoft 遠端桌面應用程式的任何其他作業系統 |

預先驗證流程的安全性優點多於通道流程。 使用預先驗證，您可以使用內部部署資源的 Azure AD 驗證功能，例如單一登入、條件式存取和雙步驟驗證。 您也可以確定只有驗證過的流量到達您的網路。

若要使用通道驗證，只需要對本文中所列的步驟進行兩項修改：
1. 在 [[Publish the RD host endpoint] \(發佈 RD 主機端點)](#publish-the-rd-host-endpoint) 步驟 1 中，請將預先驗證方法設為 [通道]。
2. 在 [[Direct RDS traffic to Application Proxy] \(將 RDS 流量導向應用程式 Proxy)](#direct-rds-traffic-to-application-proxy) 中，完全略過步驟 8。

## <a name="next-steps"></a>後續步驟

[使用 Azure AD 應用程式 Proxy 啟用 SharePoint 的遠端存取](application-proxy-integrate-with-sharepoint-server.md)  
[使用 Azure AD 應用程式 Proxy 遠端存取應用程式的安全性考量](application-proxy-security.md)
