---
title: Azure 檔案同步內部部署防火牆和 Proxy 設定 | Microsoft Docs
description: Azure 檔案同步內部部署網路組態
services: storage
author: fauhse
ms.service: storage
ms.topic: article
ms.date: 08/08/2018
ms.author: fauhse
ms.component: files
ms.openlocfilehash: 25bec4bdbae5b206d23fa300f139f55f86b8167e
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434376"
---
# <a name="azure-file-sync-proxy-and-firewall-settings"></a>Azure 檔案同步 Proxy 和防火牆設定
Azure 檔案同步會將您的內部部署伺服器連線到 Azure 檔案服務，以啟用多網站同步處理和雲端層功能。 因此，內部部署伺服器必須連線到網際網路。 IT 系統管理員必須決定進入 Azure 雲端服務之伺服器的最佳路徑。

這篇文章會提供特定需求的深入解析，以及可以將伺服器成功且安全地連線到 Azure 檔案同步的選項。

> [!Important]
> Azure 檔案同步尚未針對儲存體帳戶支援防火牆與虛擬網路。

## <a name="overview"></a>概觀
Azure 檔案同步會作為 Windows Server、Azure 檔案共用和其他數個 Azure 服務之間的協調流程服務，以便如您的同步群組中所述同步處理資料。 為了讓 Azure 檔案同步正常運作，您必須將伺服器設定為與下列 Azure 服務通訊：

- Azure 儲存體
- Azure 檔案同步
- Azure Resource Manager
- 驗證服務

> [!Note]  
> Windows Server 上的 Azure 檔案同步代理程式會啟動對雲端服務的所有要求，因而只需要從防火牆的觀點考慮輸出流量。 <br /> 沒有任何 Azure 服務會啟動與 Azure 檔案同步代理程式的連線。

## <a name="ports"></a>連接埠
Azure 檔案同步會透過 HTTPS 以獨佔方式移動檔案資料和中繼資料，而且要求連接埠 443 開放連出。
如此一來，所有流量都會加密。

## <a name="networks-and-special-connections-to-azure"></a>網路以及 Azure 的特殊連線
Azure 檔案同步代理程式沒有與 Azure 特殊通道 (例如 [ExpressRoute](../../expressroute/expressroute-introduction.md) 等) 相關的需求。

Azure 檔案同步會運用任何可用的方法來允許連線到 Azure，自動調整各種網路特性 (例如頻寬、延遲) ，以及提供系統管理控制權以進行微調。 目前不會提供所有功能。 如果您想要設定特定行為，請透過 [Azure 檔案服務 UserVoice](https://feedback.azure.com/forums/217298-storage?category_id=180670) 讓我們知道。

## <a name="proxy"></a>Proxy
Azure 檔案同步支援應用程式特定和整部電腦的 Proxy 設定。

**應用程式特定的 Proxy 設定**可特別針對 Azure 檔案同步流量允許 Proxy 的組態。 代理程式 3.0.12.0 版或更新版本可支援應用程式特定的 Proxy 設定，且可以在代理程式安裝期間進行設定，或藉由使用 Set-StorageSyncProxyConfiguration PowerShell Cmdlet。

以下 PowerShell 命令可用來設定應用程式特定的 Proxy 設定：
```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Set-StorageSyncProxyConfiguration -Address <url> -Port <port number> -ProxyCredential <credentials>
```
**整部電腦的 Proxy 設定**對於 Azure 檔案同步代理程式而言是透明的，因為伺服器的整個流量都會透過 Proxy 路由傳送。

若要設定整部電腦的 Proxy 設定，請遵循下列步驟： 

1. 設定 .NET 應用程式的 Proxy 設定 

  - 編輯這兩個檔案：  
    C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config  
    C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config

  - 在 machine.config 檔案中新增 <system.net> 區段 (在 <system.serviceModel> 區段底下)。  將 127.0.01:8888 變更為 Proxy 伺服器的 IP 位址和連接埠。 
  ```
      <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
          <proxy autoDetect="false" bypassonlocal="false" proxyaddress="http://127.0.0.1:8888" usesystemdefault="false" />
        </defaultProxy>
      </system.net>
  ```

2. 設定 WinHTTP Proxy 設定 

  - 從提高權限的命令提示字元或 PowerShell 中執行下列命令，以查看現有的 Proxy 設定：   

    netsh winhttp show proxy

  - 從提高權限的命令提示字元或 PowerShell 中執行下列命令 (將 127.0.01:8888 變更為 Proxy 伺服器的 IP 位址和連接埠)：  

    netsh winhttp set proxy 127.0.0.1:8888

3. 從提高權限的命令提示字元或 PowerShell 中執行下列命令，以重新啟動儲存體同步代理程式服務： 

      net stop filesyncsvc

      注意：儲存體同步代理程式 (filesyncsvc) 服務會在停止後自動啟動。

## <a name="firewall"></a>防火牆
如前一節所述，連接埠 443 必須開放連出。 根據您資料中心、分公司或區域的原則，可能會想要或需要進一步限制透過此連接埠送至特定網域的流量。

下表描述通訊所需的網域：

| 服務 | 網域 | 使用量 |
|---------|----------------|------------------------------|
| **Azure Resource Manager** | https://management.azure.com | 任何使用者呼叫 (例如 PowerShell) 都會通過這個 URL，包括初始伺服器註冊呼叫。 |
| **Azure Active Directory** | https://login.windows.net | Azure Resource Manager 呼叫必須由已驗證的使用者進行。 為了獲致成功，此 URL 用於進行使用者驗證。 |
| **Azure Active Directory** | https://graph.windows.net/ | 在部署 Azure 檔案同步時，將會在訂用帳戶的 Azure Active Directory 中建立服務主體。 針對該目的使用此 URL。 此主體用於將最小權限集合委派給 Azure 檔案同步服務。 執行 Azure 檔案同步之初始設定的使用者，必須是具有訂用帳戶擁有者權限的已驗證使用者。 |
| **Azure 儲存體** | &ast;.core.windows.net | 當伺服器下載檔案時，如果伺服器可以直接與儲存體帳戶中的 Azure 檔案共用通話，就可以更有效率地執行資料移動。 伺服器具有只允許存取目標檔案共用的 SAS 金鑰。 |
| **Azure 檔案同步** | &ast;.one.microsoft.com | 初始伺服器註冊之後，伺服器會收到該區域中 Azure 檔案同步服務執行個體的區域 URL。 伺服器可以使用 URL 直接且有效率地與處理其同步的執行個體進行通訊。 |
| **Microsoft PKI** | http://www.microsoft.com/pki/mscorp  http://ocsp.msocsp.com | 一旦安裝了 Azure 檔案同步代理程式，系統就會使用 PKI URL 來下載與 Azure 檔案同步服務和 Azure 檔案共用通訊時所需的中繼憑證。 OCSP URL 是用來檢查憑證的狀態。 |

> [!Important]
> 當允許前往 &ast;.one.microsoft.com 的流量時，伺服器可能會有前往同步服務以外的流量。 子網域底下有許多 Microsoft 服務可用。

如果 &ast;.one.microsoft.com 太廣泛，您可只允許對 Azure 檔案同步服務的明確區域執行個體進行通訊，藉此限制伺服器的通訊。 要選擇哪個執行個體，取決於您將伺服器部署及註冊到哪個儲存體同步服務區域。 在下表中，該區域稱為「主要端點 URL」。

基於商務持續性和災害復原 (BCDR) 的理由，您可能已在異地備援 (GRS) 儲存體帳戶中指定 Azure 檔案共用。 如果情況確實如此，則在發生持久的區域中斷時，Azure 檔案共用會容錯移轉至配對的區域。 Azure 檔案同步會使用相同的區域配對作為儲存體。 因此，如果您使用 GRS 儲存體帳戶，則需要啟用其他 URL，以供伺服器向 Azure 檔案同步的配對區域聯繫。下表將此稱為「配對的區域」。 此外，也必須啟用流量管理員設定檔 URL。 這可確保在發生容錯移轉時，網路流量會順暢地重新路由傳送至配對的區域，這個行為在下表中稱為「探索 URL」。

| 區域 | 主要端點 URL | 配對的區域 | 探索 URL |
|--------|---------------------------------------|--------|---------------------------------------|
| 澳洲東部 | https://kailani-aue.one.microsoft.com | 澳洲東南部 | https://kailani-aue.one.microsoft.com |
| 澳大利亞東南部 | https://kailani-aus.one.microsoft.com | 澳洲東部 | https://tm-kailani-aus.one.microsoft.com |
| 加拿大中部 | https://kailani-cac.one.microsoft.com | 加拿大東部 | https://tm-kailani-cac.one.microsoft.com |
| 加拿大東部 | https://kailani-cae.one.microsoft.com | 加拿大中部 | https://tm-kailani.cae.one.microsoft.com |
| 美國中部 | https://kailani-cus.one.microsoft.com | 美國東部 2 | https://tm-kailani-cus.one.microsoft.com |
| 東亞 | https://kailani11.one.microsoft.com | 東南亞 | https://tm-kailani11.one.microsoft.com |
| 美國東部 | https://kailani1.one.microsoft.com | 美國西部 | https://tm-kailani1.one.microsoft.com |
| 美國東部 2 | https://kailani-ess.one.microsoft.com | 美國中部 | https://tm-kailani-ess.one.microsoft.com |
| 北歐 | https://kailani7.one.microsoft.com | 西歐 | https://tm-kailani7.one.microsoft.com |
| 東南亞 | https://kailani10.one.microsoft.com | 東亞 | https://tm-kailani10.one.microsoft.com |
| 英國南部 | https://kailani-uks.one.microsoft.com | 英國西部 | https://tm-kailani-uks.one.microsoft.com |
| 英國西部 | https://kailani-ukw.one.microsoft.com | 英國南部 | https://tm-kailani-ukw.one.microsoft.com |
| 西歐 | https://kailani6.one.microsoft.com | 北歐 | https://tm-kailani6.one.microsoft.com |
| 美國西部 | https://kailani.one.microsoft.com | 美國東部 | https://tm-kailani.one.microsoft.com |

- 如果您使用本地備援 (LRS) 或區域備援 (ZRS) 儲存體帳戶，您只需要啟用 [主要端點 URL] 底下所列的 URL。

- 如果您使用異地備援 (GRS) 儲存體帳戶，請啟用三個 URL。

**範例：** 您在 `"West US"` 部署儲存體同步服務，並向其註冊伺服器。 在此案例中，要允許伺服器與之通訊的 URL 是：

> - https://kailani.one.microsoft.com (主要端點：美國西部)
> - https://kailani1.one.microsoft.com (配對的容錯移轉區域：美國東部)
> - https://tm-kailani.one.microsoft.com (主要區域的探索 URL)

## <a name="summary-and-risk-limitation"></a>摘要和風險限制
本文件中稍早的清單包含 Azure 檔案同步目前與其通訊的 URL。 防火牆必須能夠允許輸出至這些網域的流量。 Microsoft 致力於讓這份清單保持最新狀態。

設定網域限制防火牆規則可以提高安全性。 如果使用這些防火牆組態，則必須記住 URL 會新增並甚至可能隨著時間而改變。 請定期查看此文章。

## <a name="next-steps"></a>後續步驟
- [規劃 Azure 檔案同步部署](storage-sync-files-planning.md)
- [部署 Azure 檔案同步](storage-sync-files-deployment-guide.md)
