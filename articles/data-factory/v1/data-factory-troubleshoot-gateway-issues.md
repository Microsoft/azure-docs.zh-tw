---
title: 針對資料管理閘道問題進行疑難排解 | Microsoft Docs
description: 提供秘訣以針對資料管理閘道相關問題進行疑難排解。
services: data-factory
author: nabhishek
manager: craigg
ms.assetid: c6756c37-4e5a-4d1e-ab52-365f149b4128
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/01/2017
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 9453099af8d9be21a97808733ffa8f8bfe5a364a
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37047559"
---
# <a name="troubleshoot-issues-with-using-data-management-gateway"></a>使用資料管理閘道針對問題進行疑難排解
本文提供使用資料管理閘道進行疑難問題排解的相關資訊。

> [!NOTE]
> 本文適用於 Azure Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [Data Factory 中的自我裝載整合執行階段](../create-self-hosted-integration-runtime.md)。

如需閘道的詳細資訊，請參閱[資料管理閘道](data-factory-data-management-gateway.md)文章。 如需使用閘道將資料從內部部署 SQL Server 資料庫移至 Microsoft Azure Blob 儲存體的逐步解說，請參閱[在內部部署與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)文章。

## <a name="failed-to-install-or-register-gateway"></a>無法安裝或註冊閘道
### <a name="1-problem"></a>1.問題
您在安裝和註冊閘道時看到此錯誤訊息，尤其是在下載閘道安裝檔案時。

`Unable to connect to the remote server". Please check your local settings (Error Code: 10003).`

#### <a name="cause"></a>原因
您嘗試安裝閘道的電腦因為網路問題，而無法從下載中心下載最新的閘道安裝檔案。

#### <a name="resolution"></a>解決方案
請檢查防火牆 proxy 伺服器設定，以查看這些設定是否會封鎖從電腦到[下載中心](https://download.microsoft.com/)的網路連線，並據以更新設定。

或者，您可以在能夠存取下載中心的其他電腦上，從[下載中心](https://www.microsoft.com/download/details.aspx?id=39717)下載最新閘道的安裝檔案。 然後您可以將安裝程式檔案複製到閘道主機電腦，並手動執行以安裝及更新閘道。

### <a name="2-problem"></a>2.問題
當您在 Azure 入口網站中按一下 [直接在這部電腦上安裝] 以嘗試安裝閘道時，您會看到此錯誤。

`Error:  Abort installing a new gateway on this computer because this computer has an existing installed gateway and a computer without any installed gateway is required for installing a new gateway.`  

#### <a name="cause"></a>原因
已在電腦上安裝閘道。

#### <a name="resolution"></a>解決方案
解除安裝電腦上的現有閘道，然後再次按一下 [直接在這部電腦上安裝] 連結。

### <a name="3-problem"></a>3.問題
註冊新的閘道時，您可能會看到此錯誤。

`Error: The gateway has encountered an error during registration.`

#### <a name="cause"></a>原因
因為下列原因之一，您可能會看到此訊息：

* 閘道金鑰的格式無效。
* 閘道金鑰已經失效。
* 已從入口網站重新產生閘道金鑰。  

#### <a name="resolution"></a>解決方案
從入口網站確認您是否使用正確的閘道金鑰。 如有需要，請重新產生金鑰並使用此金鑰來註冊閘道。

### <a name="4-problem"></a>4.問題
註冊閘道時，您可能會看到下列錯誤訊息。

`Error: The content or format of the gateway key "{gatewayKey}" is invalid, please go to azure portal to create one new gateway or regenerate the gateway key.`



![金鑰的內容或格式無效](media/data-factory-troubleshoot-gateway-issues/invalid-format-gateway-key.png)

#### <a name="cause"></a>原因
輸入閘道金鑰的內容或格式不正確。 其中一個原因可能是您只從入口網站複製部分金鑰，或使用無效的金鑰。

#### <a name="resolution"></a>解決方案
在入口網站中產生閘道金鑰，並使用 [複製] 按鈕來複製整個金鑰。 然後將它貼在這個視窗來註冊閘道。

### <a name="5-problem"></a>5.問題
註冊閘道時，您可能會看到下列錯誤訊息。

`Error: The gateway key is invalid or empty. Specify a valid gateway key from the portal.`

![閘道金鑰無效或是空的](media/data-factory-troubleshoot-gateway-issues/gateway-key-is-invalid-or-empty.png)

#### <a name="cause"></a>原因
已重新產生閘道金鑰，或已在 Azure 入口網站中刪除閘道。 如果資料管理閘道安裝程式不是最新的，也可能發生此問題。

#### <a name="resolution"></a>解決方案
檢查資料管理閘道安裝程式是否為最新版本，您可以在 Microsoft [下載中心](https://go.microsoft.com/fwlink/p/?LinkId=271260)找到最新版本。

如果安裝程式是最新的，但閘道仍然存在於入口網站上，請在 Azure 入口網站中重新產生閘道金鑰，使用 [複製] 按鈕來複製整個金鑰，然後將它貼在此視窗以註冊閘道。 否則，請重新建立閘道並從頭開始。

### <a name="6-problem"></a>6.問題
註冊閘道時，您可能會看到下列錯誤訊息。

`Error: Gateway has been online for a while, then shows “Gateway is not registered” with the status “Gateway key is invalid”`

![閘道金鑰無效或是空的](media/data-factory-troubleshoot-gateway-issues/gateway-not-registered-key-invalid.png)

#### <a name="cause"></a>原因
因為已刪除閘道，或已重新產生相關聯的閘道金鑰，所以會發生此錯誤。

#### <a name="resolution"></a>解決方案
如果已刪除閘道，請從入口網站重新建立閘道、按一下 [註冊]、從入口網站複製金鑰、將它貼上，然後嘗試重新註冊閘道。

如果閘道仍然存在，但已重新產生其金鑰，請使用新的金鑰來註冊閘道。 如果您沒有金鑰，則從入口網站再次重新產生金鑰。

### <a name="7-problem"></a>7.問題
註冊閘道時，您可能需要輸入憑證的路徑和密碼。

![指定憑證](media/data-factory-troubleshoot-gateway-issues/specify-certificate.png)

#### <a name="cause"></a>原因
此閘道之前已在其他電腦上註冊。 在閘道的初始註冊期間，加密憑證已經與閘道相關聯。 此憑證可由閘道自行產生或由使用者提供。  此憑證用來加密資料存放區 (連結的服務) 的認證。  

![匯出憑證](media/data-factory-troubleshoot-gateway-issues/export-certificate.png)

在不同的主機電腦上還原閘道時，註冊精靈會要求此憑證，以將先前使用此憑證加密的認證解密。  如果沒有這個憑證，新的閘道便無法將認證解密，而且與這個新閘道相關聯的後續複製活動執行會失敗。  

#### <a name="resolution"></a>解決方案
如果您已使用「資料管理閘道組態管理員」之 [設定] 索引標籤上的 [匯出] 按鈕，從原始閘道電腦匯出認證憑證，請使用這裡的憑證。

在復原閘道時，您無法略過此階段。 如果憑證遺失，您必須從入口網站刪除閘道，再重新建立新的閘道。  此外，藉由重新輸入其認證來更新與閘道相關的所有連結的服務。

### <a name="8-problem"></a>8.問題
您可能會看到下列錯誤訊息。

`Error: The remote server returned an error: (407) Proxy Authentication Required.`

#### <a name="cause"></a>原因
您的閘道處於需要 HTTP Proxy 才能存取網際網路資源的環境中，或 Proxy 的驗證密碼已變更但未在您的閘道中相應更新時，就會發生此錯誤。

#### <a name="resolution"></a>解決方案
遵循本文章的 [Proxy 伺服器考量](#proxy-server-considerations)一節中的指示，並透過「資料管理閘道組態管理員」設定 Proxy 設定。

## <a name="gateway-is-online-with-limited-functionality"></a>閘道在線上但功能受限
### <a name="1-problem"></a>1.問題
您看到閘道的狀態為 [在線上但功能受限]。

#### <a name="cause"></a>原因
基於下列其中一個原因，您看到閘道的狀態為 [在線上但功能受限]：

* 閘道無法透過 Azure 服務匯流排連線到雲端服務。
* 雲端服務無法透過服務匯流排連線到閘道。

當閘道在線上但功能受限時，您可能無法使用「Data Factory 複製精靈」來建立資料管線，以將資料複製到內部部署資料存放區，或從該資料存放區複製資料。 您可以使用入口網站中的「Data Factory 編輯器」、Visual Studio 或 Azure PowerShell 作為因應措施。

#### <a name="resolution"></a>解決方案
此問題 (在線上但功能受限) 的解決方式取決於是閘道無法連線到雲端服務，還是雲端服務無法連線到閘道。 下列各節將提供這些解決方式。

### <a name="2-problem"></a>2.問題
您看到下列錯誤。

`Error: Gateway cannot connect to cloud service through service bus`

![閘道無法連接到雲端服務](media/data-factory-troubleshoot-gateway-issues/gateway-cannot-connect-to-cloud-service.png)

#### <a name="cause"></a>原因
閘道無法透過服務匯流排連線到雲端服務。

#### <a name="resolution"></a>解決方案
請依照下列步驟操作，讓閘道回到線上：

1. 允許在閘道電腦和公司防火牆上執行 IP 位址輸出規則。 您可以從 Windows 事件記錄檔中找到 IP 位址 (識別碼 == 401)：嘗試以存取權限禁止的方式存取通訊端 XX.XX.XX.XX:9350。
* 設定閘道上的 Proxy 設定。 如需詳細資料，請參閱 [Proxy 伺服器考量](#proxy-server-considerations)一節。
* 在閘道電腦的 Windows 防火牆上與公司防火牆上啟用輸出連接埠 5671 和 9350-9354。 如需詳細資料，請參閱[連接埠和防火牆](#ports-and-firewall)一節。 此步驟為選用步驟，但基於效能考量，還是建議執行。

### <a name="3-problem"></a>3.問題
您看到下列錯誤。

`Error: Cloud service cannot connect to gateway through service bus.`

#### <a name="cause"></a>原因
暫時性網路連線錯誤。

#### <a name="resolution"></a>解決方案
請依照下列步驟操作，讓閘道回到線上：

1. 等候數分鐘，錯誤消失時，就會自動恢復連線。
* 如果錯誤持續發生，請重新啟動閘道服務。

## <a name="failed-to-author-linked-service"></a>無法編寫連結的服務
### <a name="problem"></a>問題
當您嘗試在入口網站上使用「認證管理員」來輸入新連結服務的認證，或更新現有連結服務的認證時，您會看到此錯誤。

`Error: The data store '<Server>/<Database>' cannot be reached. Check connection settings for the data source.`

當您看到此錯誤時，「資料管理閘道組態管理員」中的 [設定] 頁面看起來類似下列螢幕擷取畫面。

![無法觸達資料庫](media/data-factory-troubleshoot-gateway-issues/database-cannot-be-reached.png)

#### <a name="cause"></a>原因
閘道電腦上的 SSL 憑證可能已遺失。 閘道電腦無法載入目前用於 SSL 加密的憑證。 您也可能會看到類似下列訊息的事件記錄檔錯誤訊息。

 `Unable to get the gateway settings from cloud service. Check the gateway key and the network connection. (Certificate with thumbprint cannot be loaded.)`

#### <a name="resolution"></a>解決方案
請遵循下列步驟來解決問題︰

1. 啟動「資料管理閘道組態管理員」。
2. 切換到 [設定]  索引標籤。  
3. 按一下 [變更] 按鈕來變更 SSL 憑證。

   ![變更憑證按鈕](media/data-factory-troubleshoot-gateway-issues/change-button-ssl-certificate.png)
4. 選取新的憑證做為 SSL 憑證。 您可以使用您自己或任何組織所產生的任何 SSL 憑證。

   ![指定憑證](media/data-factory-troubleshoot-gateway-issues/specify-http-end-point.png)

## <a name="copy-activity-fails"></a>複製活動失敗
### <a name="problem"></a>問題
在入口網站中設定管線之後，您可能會發現下列 "UserErrorFailedToConnectToSqlserver" 失敗。

`Error: Copy activity encountered a user error: ErrorCode=UserErrorFailedToConnectToSqlServer,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Cannot connect to SQL Server`

#### <a name="cause"></a>原因
發生的原因各有不同，而緩和方式也隨之而異。

#### <a name="resolution"></a>解決方案
在連接到 SQL Database 之前，允許透過資料管理閘道用戶端上的連接埠 TCP/1433 進行輸出 TCP 連線。

如果目標資料庫是 Azure SQL Database，請同時檢查 Azure 的 SQL Server 防火牆設定。

請參閱下一節，以測試連往內部部署資料存放區的連線。

## <a name="data-store-connection-or-driver-related-errors"></a>資料存放區連線或驅動程式相關錯誤
如果您看到資料存放區連線或驅動程式相關的錯誤，請完成下列步驟︰

1. 在閘道機器上啟動「資料管理閘道組態管理員」。
2. 切換至 [診斷]  索引標籤。
3. 在 [測試連接] 中，新增閘道群組值。
4. 按一下 [測試] 來確認您是否可以透過使用連線資訊和認證，從閘道機器連線到內部部署資料來源。 如果在安裝驅動程式後測試連接仍然失敗，請重新啟動閘道器以讓它取得最新的變更。

![在 [診斷] 索引標籤中的測試連接](media/data-factory-troubleshoot-gateway-issues/test-connection-in-diagnostics-tab.png)

## <a name="gateway-logs"></a>閘道記錄檔
### <a name="send-gateway-logs-to-microsoft"></a>將閘道記錄檔傳送給 Microsoft
向「Microsoft 支援服務」尋求閘道問題疑難排解協助時，支援人員可能會要求您分享閘道記錄檔。 隨著閘道的發行，您可以透過在「資料管理閘道組態管理員」上點選按鈕兩次，共用所需的閘道記錄檔。    

1. 切換至「資料管理閘道組態管理員」中的 [診斷] 索引標籤。

    ![資料管理閘道診斷索引標籤](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-diagnostics-tab.png)
2. 按一下 [傳送記錄檔] 可看到以下對話方塊。

    ![資料管理閘道傳送記錄檔](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-dialog.png)
3. (選擇性) 按一下 [檢視記錄檔] 以在事件檢視器檢閱記錄檔。
4. (選擇性) 按一下 [隱私權] 以檢閱 Microsoft Web 服務隱私權聲明。
5. 當您滿意即將上傳的內容時，請按一下 [傳送記錄檔]，將過去 7 天的記錄檔傳送給 Microsoft 進行疑難排解。 您應該會看到「傳送記錄檔」作業的狀態，如下列螢幕擷取畫面所示。

    ![資料管理閘道傳送記錄檔狀態](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-status.png)
6. 作業完成之後，您會看到一個對話方塊，如下列螢幕擷取畫面所示。

    ![資料管理閘道傳送記錄檔狀態](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-result.png)
7. 儲存**報告識別碼**並與 Microsoft 支援服務共用。 報告識別碼是用來尋找為了進行疑難排解所上傳的閘道記錄檔。  報告識別碼也會儲存在事件檢視器。  查看事件識別碼 "25" 即可找到它，並檢查日期和時間。

    ![資料管理閘道傳送記錄檔報告識別碼](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-report-id.png)    

### <a name="archive-gateway-logs-on-gateway-host-machine"></a>在閘道主機電腦上封存閘道記錄檔
在某些情況下，您會有閘道問題但無法直接共用閘道記錄檔︰

* 您以手動方式安裝閘道及註冊閘道。
* 您嘗試在「資料管理閘道組態管理員」中使用重新產生的金鑰來註冊閘道。
* 您嘗試傳送記錄檔，但無法與閘道主機服務連線。

對於這些案例，您可以將閘道記錄檔儲存為 zip 檔案，並在連絡 Microsoft 支援服務時共用。 例如，如果您在註冊閘道時收到錯誤，如下列螢幕擷取畫面所示。   

![資料管理閘道註冊錯誤](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-registration-error.png)

按一下 [封存閘道記錄檔] 連結以封存和儲存記錄檔，然後與 Microsoft 支援服務共用 zip 檔案。

![資料管理閘道封存記錄檔](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-archive-logs.png)

### <a name="locate-gateway-logs"></a>找出閘道記錄檔
您可以在 Windows 事件記錄檔中找到詳細的閘道記錄檔資訊。

1. 啟動 Windows **事件檢視器**。
2. 找出 [應用程式及服務記錄檔]  >  [資料管理閘道] 資料夾中的記錄檔。

 針對閘道相關問題進行疑難排解時，請在事件檢視器中尋找錯誤層級的事件。

![資料管理閘道事件檢視器中的記錄檔](media/data-factory-troubleshoot-gateway-issues/gateway-logs-event-viewer.png)
