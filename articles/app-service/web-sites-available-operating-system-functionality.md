---
title: Azure App Service 上的作業系統功能
description: 了解 Azure App Service 上 Web 應用程式、行動應用程式後端和 API Apps 可以使用的作業系統功能
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: mollybos
ms.assetid: 39d5514f-0139-453a-b52e-4a1c06d8d914
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/01/2016
ms.author: cephalin
ms.openlocfilehash: 00b5f9c78000fbb9bf86e8c1d8b06e3645795a12
ms.sourcegitcommit: 3c3488fb16a3c3287c3e1cd11435174711e92126
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34850149"
---
# <a name="operating-system-functionality-on-azure-app-service"></a>Azure App Service 上的作業系統功能
本文說明在 [Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714)上執行的所有應用程式可用的一般基礎作業系統功能。 此功能包含檔案、網路、登錄存取、診斷記錄和事件。 

<a id="tiers"></a>

## <a name="app-service-plan-tiers"></a>App Service 計劃層
App Service 會在多租用戶裝載環境中執行客戶應用程式。 部署在「免費」和「共用」層的應用程式，會在共用虛擬機器的背景工作角色處理序中執行，而部署在「標準」和「Premium」層的應用程式，則會在與單一客戶應用程式相關聯的專用虛擬機器上執行。

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

因為 App Service 支援不同層之間的完美縮放體驗，所以強制 App Service 應用程式採用的安全性設定維持不變。 這可確保當 App Service 在不同層之間切換時，應用程式不會突然出現不同的行為，發生非預期的失敗。

<a id="developmentframeworks"></a>

## <a name="development-frameworks"></a>開發架構
App Service 定價層可控制應用程式可用的運算資源 (CPU、磁碟儲存體、記憶體和網路出口流量) 數量。 但是，不論調整為何層，應用程式可用的架構功能廣度維持不變。

App Service 支援各種開發架構，包含 ASP.NET、傳統 ASP、node.js、PHP 和 Python，在 IIS 中這些架構全都以擴充功能形式執行。 為了簡化和標準化安全性設定，App Service 應用程式通常會以預設的設定執行各種部署架構。 設定應用程式的其中一種方式，就是針對個別的開發架構自訂 API 介面區和功能。 App Service 採取了更普通的方式，也就是不論應用程式開發架構為何，均啟用作業系統功能的一般基礎。

下列各節簡單說明 App Service 應用程式可用的一般作業系統功能種類。

<a id="FileAccess"></a>

## <a name="file-access"></a>檔案存取
App Service 中的各種磁碟機，包含本機磁碟機和網路磁碟機。

<a id="LocalDrives"></a>

### <a name="local-drives"></a>本機磁碟機
基本上，App Service 是一項在 Azure PaaS (平台即服務) 基礎結構之上執行的服務。 因此，「附加」至虛擬機器的本機磁碟機就是任何在 Azure 中執行之背景工作角色可用的相同磁碟機類型。 其中包括：

- 作業系統磁碟機 (D:\ 磁碟機)
- 應用程式磁碟機，包含 App Service 專用的 Azure 封裝 cspkg 檔案 (客戶無法存取)
- 「使用者」磁碟機 (C:\ 磁碟機)，它的大小會隨著虛擬機器大小而異。 

請務必在您應用程式增加之際監視磁碟使用狀況。 如果達到磁碟配額時，可能會對您的應用程式造成不良影響。

<a id="NetworkDrives"></a>

### <a name="network-drives-aka-unc-shares"></a>網路磁碟機 (亦稱為 UNC 共用)
App Service 將所有使用者內容儲存在一組 UNC 共用上，此種獨特做法讓應用程式的部署和維護變得直接。 此種模型非常符合一種內容儲存體模式，該模式通常是由具有多個負載平衡伺服器的內部部署虛擬主機所使用。 

在 App Service 中，每個資料中心內都已建立許多 UNC 共用。 每個 UNC 共用都會配置每個資料中心內各客戶某個百分比的使用者內容。 此外，單一客戶訂閱的所有檔案內容一律放在相同的 UNC 共用上。 

由於 Azure 服務的運作方式，負責裝載 UNC 共用的特定虛擬機器將隨著時間而改變。 UNC 共用在 Azure 作業的正常過程中啟動和關閉時，一定會由不同的虛擬機器掛接。 基於這個理由，應用程式不得硬性假設 UNC 檔案路徑中的機器資訊會隨時保持穩定。 反之，應該使用 App Service 所提供的方便 *faux* 絕對路徑 **D:\home\site**。 這個假的絕對路徑提供了一種無需知道應用程式和使用者的可攜式方法，即可參考某人擁有的應用程式。 使用 **D:\home\site**，即可在應用程式之間傳輸共用檔案，而不需針對每次傳輸設定新的絕對路徑。

<a id="TypesOfFileAccess"></a>

### <a name="types-of-file-access-granted-to-an-app"></a>授與應用程式的檔案存取類型
在資料中心內的特定 UNC 共用上，每個客戶的訂用帳戶都有一個保留的目錄結構。 一個客戶可以在特定資料中心內建立多個應用程式，所以屬於單一客戶訂用帳戶的所有目錄都會建立於相同的 UNC 共用上。 此共用可能包括目錄 (內容、錯誤和診斷記錄的目錄)，以及原始檔控制所建立的舊版應用程式。 如同預期，應用程式的應用程式程式碼可以在執行階段讀取和寫入客戶的應用程式目錄。

在執行應用程式之虛擬機器附加的本機磁碟機上，App Service 會在 C:\ 磁碟機上保留許多空間作為應用程式專用暫存本機儲存體。 雖然應用程式可以完整讀寫自己的暫存本機儲存體，但是該儲存體的真正目的並非由應用程式程式碼直接使用。 其目的是要提供 IIS 和 Web 應用程式架構的暫存檔案儲存體。 App Service 也會限制每個網站可用的暫存本機儲存體數量，以免個別應用程式取用過量的本機檔案儲存體。

App Service 對於暫存本機儲存體的兩種使用方式範例：暫存 ASP.NET 檔案的目錄和 IIS 壓縮檔案的目錄。 ASP.NET 編譯系統會使用 "Temporary ASP.NET Files" 目錄作為暫存編譯快取位置。 IIS 則使用 "IIS Temporary Compressed Files" 目錄來儲存壓縮回應輸出。 這兩種檔案使用方式 (和其他方式) 都會在 App Service 中重新對應至每個應用程式的暫存本機儲存體。 此重新對應可確保功能如預期般繼續運作。

App Service 中的每個應用程式都會以隨機獨特的低權限背景工作角色處理序身分識別 (稱為「應用程式集區身分識別」) 執行，進一步的說明如下：[http://www.iis.net/learn/manage/configuring-security/application-pool-identities](http://www.iis.net/learn/manage/configuring-security/application-pool-identities)。 應用程式程式碼會將此識別用於作業系統磁碟機 (D:\ 磁碟機) 的基本唯讀存取。 這表示應用程式程式碼可以列出通用目錄結構和讀取作業系統磁碟機上的通用檔案。 雖然這似乎是有點廣泛的存取層級，但是當您在 Azure 託管服務中佈建背景工作角色和讀取磁碟機內容時，可以存取的目錄和檔案相同。 

<a name="multipleinstances"></a>

### <a name="file-access-across-multiple-instances"></a>跨越多個執行個體的檔案存取
主目錄包含應用程式的內容，而應用程式程式碼可以寫入其中。 如果有個應用程式在多個執行個體上執行，則主目錄會在所有執行個體當中共用，讓所有執行個體都可看見相同的目錄。 例如，如果應用程式將上傳的檔案儲存到主目錄，所有執行個體即可立即使用這些檔案。 

<a id="NetworkAccess"></a>

## <a name="network-access"></a>網路存取
應用程式程式碼可以使用 TCP/IP 和 UDP 型通訊協定，對公開外部服務的網際網路可存取端點進行輸出網路連線。 應用程式可以使用這些相同通訊協定來連接至 Azure&#151; 內的服務，例如，建立與 SQL Database 的 HTTPS 連線。

此外，應用程式可以在有限的情況下建立一個本機回送連線，並且讓應用程式在該本機回送通訊端上接聽。 這項功能主要是讓在本機回送通訊端上接聽的應用程式成為其功能的一部分。 每個應用程式都會看到「私人」回送連線。 應用程式 "A" 無法接聽由應用程式 "B" 建立的本機回送通訊端。

此外，還支援具名管道作為共同執行應用程式的不同處理序之間的處理序間通訊 (IPC) 機制。 例如，IIS FastCGI 模組會依賴具名管道來協調執行 PHP 頁面的個別處理序。

<a id="Code"></a>

## <a name="code-execution-processes-and-memory"></a>程式碼執行、處理序和記憶體
如先前所述，應用程式會使用隨機應用程式集區身分識別在低權限的背景工作角色處理序內部執行。 應用程式程式碼可以存取與背景工作處理序相關聯的記憶體空間，以及 CGI 處理序或其他應用程式所繁衍的任何子處理序。 但是，即使是位於相同的虛擬機器上，某個應用程式無法存取其他應用程式的記憶體或資料。

應用程式可以執行利用支援的 Web 開發架構撰寫的指令碼或頁面。 App Service 不會將任何 Web 應用程式架構設定設定為比較受限的模式。 例如，在 App Service 上執行的 ASP.NET 應用程式會以「完全」信任模式 (相對於比較受限的信任模式) 執行。 Web 架構 (包括傳統 ASP 和 ASP.NET) 可以呼叫處理序內 COM 元件 (但不能呼叫處理序外 COM 元件)，例如：預設在 Windows 作業系統上註冊的 ADO (ActiveX Data Objects)。

應用程式可以繁衍和執行任意程式碼。 允許應用程式執行的動作如下：繁衍命令殼層或執行 PowerShell 指令碼。 但是，即使可以從應用程式衍生任意程式碼和處理序，但可執行的程式和指令碼仍受限於授與父應用程式集區的權限。 例如，應用程式可以繁衍用於進行輸出 HTTP 呼叫的可執行檔，但這一個可執行檔無法嘗試解除虛擬機器的 IP 位址與其 NIC 的繫結。 低權限的程式碼能夠進行輸出網路呼叫，但嘗試在虛擬機器上重新設定網路設定則需要有系統管理權限。

<a id="Diagnostics"></a>

## <a name="diagnostics-logs-and-events"></a>診斷記錄和事件
記錄資訊是某些應用程式嘗試存取的另一組資料。 在 App Service 中執行的程式碼的可用記錄資訊類型包括應用程式所產生的診斷和記錄資訊，應用程式也可輕易存取此資訊。 

例如，作用中應用程式所產生的 W3C HTTP 記錄可在針對該應用程式建立的網路共用位置中的記錄目錄中取得，而如果客戶已設定 W3C 記錄至儲存體，則可在 Blob 儲存體中取得。 後面這個選項能夠收集大量的記錄，而不需擔心超出與網路共用相關聯的檔案儲存體限制。

同樣地，也可以使用 .NET 追蹤和診斷基礎結構來記錄 .NET 應用程式的即時診斷資訊，方法是選擇將追蹤資訊寫入至應用程式的網路共用，或者是 Blob 儲存體位置。

應用程式無法使用的診斷記錄和追蹤區域包括 Windows ETW 事件和一般 Windows 事件記錄 (例如 System、Application 和 Security 事件記錄)。 因為有可能全機器都可檢視 ETW 追蹤資訊 (只要具有適當的 ACL)，所以封鎖了 ETW 事件的讀取和寫入存取權。 開發人員可能會注意到用以讀寫 ETW 事件和一般 Windows 事件記錄的 API 呼叫似乎有作用，但這是因為 App Service「假裝」呼叫，以致似乎呼叫成功。 事實上，應用程式程式碼無法存取此事件資料。

<a id="RegistryAccess"></a>

## <a name="registry-access"></a>登錄存取
應用程式對於執行所在的虛擬機器，具有大部分 (雖然並非全部) 登錄的唯讀存取權。 實際上，這表示應用程式可存取允許本機使用者群組唯讀存取的登錄機碼。 目前無法提供讀取或寫入的其中一個登錄區域為 HKEY\_CURRENT\_USER 登錄區。

登錄的寫入存取權已遭封鎖，其中包括任何每一使用者登錄機碼的存取。 從應用程式的觀點來看，因為應用程式可以在不同虛擬機器之間移轉，所以 Azure 環境中的登錄寫入存取權並不可靠。 應用程式唯一可依賴的持續可寫入儲存體，就是儲存在 App Service UNC 共用上的各應用程式內容目錄結構。 

## <a name="more-information"></a>詳細資訊

[Azure Web 應用程式沙箱](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox) - 有關 App Service 執行環境的最新資訊。 本頁面由 App Service 開發團隊直接維護。

> [!NOTE]
> 如果您想在註冊 Azure 帳戶前開始使用 Azure App Service，請移至 [試用 App Service](https://azure.microsoft.com/try/app-service/)，即可在 App Service 中立即建立短期入門 Web 應用程式。 不需要信用卡；無需承諾。
> 
> 


