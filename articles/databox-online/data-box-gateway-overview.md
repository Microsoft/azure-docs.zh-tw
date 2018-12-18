---
title: Microsoft Azure 資料箱閘道概觀 | Microsoft Docs
description: 描述 Azure 資料箱閘道，這是可讓您將資料傳輸到 Azure 的虛擬設備儲存體解決方案
services: databox
author: alkohli
ms.service: databox
ms.topic: overview
ms.date: 09/24/2018
ms.author: alkohli
ms.openlocfilehash: 4f1ab6d955c81ce6f7b141eef42341f43bb379f6
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49165312"
---
# <a name="what-is-azure-data-box-gateway-preview"></a>什麼是 Azure 資料箱閘道 (預覽)？ 

Azure 資料箱閘道是可讓您順利將資料傳送到 Azure 的儲存體解決方案。 此文章為您提供的概觀涵蓋 Azure 資料箱閘道解決方案、優點、主要功能，以及您可以部署此裝置的案例。 

資料箱閘道是虛擬裝置，其基礎是佈建在虛化環境中或 Hypervisor 中的虛擬機器。 虛擬裝置位在您的內部部署環境中，且您使用 NFS 與 SMB 通訊協定將資料寫入其中。 裝置接著將您的資料傳輸到 Azure 區塊 Blob、分頁 Blob 或 Azure 檔案。 

> [!IMPORTANT]
> 資料箱閘道處於預覽狀態。 部署此解決方案之前，請先檢閱[預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="use-cases"></a>使用案例

需要將資料傳輸到雲端 (以用於雲端封存與災害復原) 或有需要以雲端規模來處理您的資料時，可以使用資料箱閘道。 以下是資料箱閘道可用於資料傳輸的各種案例。

- **雲端封存** - 使用資料箱閘道以安全且有效率的方式將數百 TB 的資料複製到 Azure 儲存體。 針對封存的案例，資料可以一次內嵌或以持續的方式內嵌。

- **資料彙總** - 將多個來源的資料彙總到 Azure 儲存體中的單一位置內，以進行資料處理及分析。  

- **與內部部署工作負載整合** - 與內部部署工作負載 (例如使用雲端儲存體且需要在本機存取常用檔案的備份和還原) 整合。 

## <a name="benefits"></a>優點

資料箱閘道具有下列優點：

- **資料傳輸容易** - 使將資料移入及移出 Azure 儲存體和使用區域網路共用一樣容易。  
- **高效能** - 免於網路資料傳輸的麻煩，傳入和傳出 Azure 都具有高效能。 
- **快速存取** - 為內部部署檔案的快速存取快取最新的檔案。  
- **限制頻寬使用量** - 當系統在尖峰營運時段對網路進行節流以限制使用量時，資料也可以寫入至 Azure。  

## <a name="key-capabilities"></a>主要功能

資料箱閘道具有下列功能：

|功能 |說明  |
|---------|---------|
|速度     | 完全自動化且高度最佳化的資料傳輸與頻寬。|
|支援的通訊協定     | 支援用於資料擷取的標準 SMB 與 NFS 通訊協定。 <br> 如需支援版本的詳細資訊，請移至[資料箱閘道系統需求](data-box-gateway-system-requirements.md)。|
|資料存取     | 能使用雲端 API 直接從 Azure 儲存體 Blob 與 Azure 檔案存取資料，以在雲端中進行額外的資料處理。|
|快速存取     | 裝置上的本機快取可讓您快速存取最近使用的檔案。|
|離線上傳     | 中斷連線模式支援離線上傳案例。|
|資料重新整理     | 能夠以來自雲端的最新檔案對本機檔案進行重新整理。|
|加密    | BitLocker 支援本機加密資料，以及透過 *https* 對雲端的安全資料傳輸       |
|災害復原     | 內建的網路災害復原        |


## <a name="specifications"></a>規格

資料箱閘道虛擬裝置具有下列規格：

| 規格                                          | 說明              |
|---------------------------------------------------------|--------------------------|
| 虛擬處理器 (核心)   | 最少 4 個 |            
| 記憶體  | 最少 8 GB|
| 可用性|單一節點|
| 磁碟| 作業系統磁碟：250 GB <br> 資料磁碟：最少 2 TB，精簡佈建且必須以 SSD 支援|
| 網路介面|1 或多個虛擬網路介面|
| 原生的檔案共用通訊協定|SMB 和 NFS  |
| 安全性| 驗證以解除存取裝置和資料的鎖定 <br> 資料傳輸使用 AES-256 位元加密|
| 管理性| 本機 Web UI - 初始設定、診斷及裝置電源管理 <br> Azure 入口網站 - 資料箱閘道裝置的日常管理工作       |


## <a name="components"></a>元件

資料箱閘道解決方案的組成包括資料箱閘道資源、資料箱閘道虛擬裝置和本機 Web UI。

* **資料箱閘道虛擬裝置** - 以您的虛擬化環境或 Hypervisor 中所佈建之虛擬機器為基礎的裝置，可讓您將資料傳送至 Azure。 
    
* **資料箱閘道資源** - Azure 入口網站中的資源，可讓您從 Web 介面管理資料箱閘道裝置，且您可以從不同地理位置存取該介面。 使用資料箱閘道資源來建立和管理資源、檢視和管理裝置，以及警示和管理共用。  

    <!--![The Data Box Gateway service in Azure portal](media/data-box-overview/data-box-Gateway-service1.png)-->

    <!--For more information, go to [Use the Data Box Gateway service to administer your Data Box Gateway device](data-box-gateway-portal-ui-admin.md).-->

* **資料箱本機 Web UI** - 使用本機 Web UI 對資料箱閘道執行診斷、關機和重新啟動、檢視複本記錄檔，以及連絡 Microsoft 支援服務以提出服務要求。

    <!--![The Data Box Gateway local web UI](media/data-box-gateway-overview/data-box-gateway-local-web-ui.png)-->

    <!-- For information about using the web-based UI, go to [Use the web-based UI to administer your Data Box](data-box-gateway-portal-ui-admin.md).-->


## <a name="region-availability"></a>區域可用性

Data Box Edge 實體裝置、Azure 資源，以及您要將資料傳輸至其中的目標儲存體帳戶，並不需要全都位於相同的區域。

- **資源可用性** - 在針對此版本，下列區域提供 Data Box Edge 資源：
    - **美國** - 美國西部 2 與美國東部
    - **歐盟** - 西歐
    - **亞太地區** - 東南亞

- **目的地儲存體帳戶** - 儲存資料的儲存體帳戶在所有 Azure 區域都可用。 

    若要獲得最佳效能，儲存資料箱資料的儲存體帳戶應該要位在接近裝置所在區域的區域。 儲存體帳戶的位置若離裝置遙遠，將會導致較長的延遲時間與較緩慢的效能。 


## <a name="next-steps"></a>後續步驟

- 檢閱[資料箱閘道的系統需求](data-box-gateway-system-requirements.md)。
- 了解[資料箱閘道的限制](data-box-gateway-limits.md)。
- 在 Azure 入口網站中部署 [Azure 資料箱閘道](data-box-gateway-deploy-prep.md)。




