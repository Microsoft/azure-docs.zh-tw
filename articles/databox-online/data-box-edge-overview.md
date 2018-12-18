---
title: Microsoft Azure Data Box Edge 概觀 | Microsoft Docs
description: 說明 Azure Data Box Edge，這是針對 Azure 進行網路型傳輸使用實體裝置的儲存體解決方案。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 09/24/2018
ms.author: alkohli
ms.openlocfilehash: 6422482a31ad8022b795cb357c37ad8560781767
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166588"
---
# <a name="what-is-azure-data-box-edge-preview"></a>什麼是 Azure Data Box Edge (預覽)？ 

Azure Data Box Edge 是允許您處理資料並將它透過網路傳送到 Azure 的儲存體解決方案。 此文章為您提供 Data Box Edge 解決方案、其優點與主要功能，以及適合部署此裝置之案例的概觀。 

Data Box Edge 會使用由 Microsoft 所提供的實體裝置來加快安全資料傳輸的速度。 實體裝置位在您的內部部署環境中，而您將會使用 NFS 與 SMB 通訊協定將資料寫入其中。 

Data Box Edge 具有資料箱閘道的所有閘道功能。 資料箱更有具備 AI 能力的邊緣運算功能，有助於在將資料移至 Azure 區塊 Blob、分頁 Blob 或 Azure 檔案的期間，對資料進行分析、處理或篩選。  

> [!IMPORTANT]
> - Data Box Edge 目前處於預覽狀態。 [註冊](#sign-up)以使用此服務！
> - 部署此解決方案之前，請先檢閱[預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


## <a name="use-cases"></a>使用案例

Azure Data Box Edge 為具備 AI 能力的邊緣運算裝置，並具有網路資料傳輸功能。 以下是適合將 Data Box Edge 用於資料傳輸的各種案例。

- **預先處理資料** - 從內部部署或 IoT 裝置分析資料，以在接近資料產生的位置快速取得結果。 Data Box Edge 會將完整的資料集傳輸到雲端，以執行更進階的處理或更深入的分析。  預先處理功能可用來： 

    - 彙總資料。
    - 修改資料 (例如移除個人識別資訊 (PII))。
    - 對所需的資料進行子集處理，並傳輸到雲端以進行更深入的分析。
    - 分析並對 IoT 事件做出反應。 

- **推斷 Azure Machine Learning** - 透過使用 Data Box Edge，您可以執行 Machine Learning (ML) 模型，以在資料被傳送到雲端之前快速取得能用來據以採取動作的結果。 完整資料集會完成傳輸，以繼續重新定型並改善您的 ML 模型。

- **透過網路將資料傳輸到 Azure** - 使用 Data Box Edge 來輕鬆且快速地將資料傳輸到 Azure，以進行進一步的運算和分析，或用於封存目的。 

## <a name="benefits"></a>優點

Data Box Edge 具有下列優點：

- **資料傳輸容易** - 使將資料移入及移出 Azure 儲存體和使用區域網路共用一樣容易。  
- **高效能** - 能為針對 Azure 的來回傳輸提供高效能的表現。 
- **快速存取** - 能對最新的檔案進行快取，以供您快速存取內部部署檔案。  
- **限制頻寬使用量** - 當系統在尖峰營運時段對網路進行節流以限制使用量時，資料也可以寫入至 Azure。  
- **轉換資料** - 在資料移至 Azure 期間，提供對該資料進行分析、處理或篩選的功能。

## <a name="key-capabilities"></a>主要功能

Data Box Edge 具有下列功能：

|功能 |說明  |
|---------|---------|
|高效能     | 完全自動化且高度最佳化的資料傳輸與頻寬。|
|支援的通訊協定     | 針對資料擷取支援標準 SMB 與 NFS 通訊協定。 <br> 如需支援版本的詳細資訊，請移至 [Data Box Edge 系統需求](http://aka.ms/dbe-docs) \(英文\)。|
|運算       |允許對資料進行分析、處理、篩選。|
|資料存取     | 能使用雲端 API 直接從 Azure 儲存體 Blob 與 Azure 檔案存取資料，以在雲端中進行額外的資料處理。|
|快速存取     | 裝置上的本機快取可讓您快速存取最近使用的檔案。|
|離線上傳     | 中斷連線模式支援離線上傳案例。|
|資料重新整理     | 能夠以來自雲端的最新檔案對本機檔案進行重新整理。|
|加密    | 支援使用 BitLocker 在本機加密資料，並透過 *https* 保護對雲端的資料傳輸。       |
|災害復原     | 內建的網路復原功能。        |


## <a name="features-and-specifications"></a>功能和規格

Data Box Edge 實體裝置具有下列功能：

| 功能/規格                                          | 說明              |
|---------------------------------------------------------|--------------------------|
| 維度   | 寬度：17.25 英吋，深度：27.25 英吋，高度：1.75 英吋<br>(不含兩側固定片及 PSU 把手)  |            
| 機架空間|置於機架中時為 1U|
| 纜線| 2 條電源纜線<br>2 條 1 Gbps RJ45 纜線<br>2 條 10 Gbps SFP+ 銅纜線|
| 元件|2 個內建電源供應器 (PSU)|
| CPU|2 顆分別具有 10 個核心的 Intel Xeon 處理器  |
| 記憶體| 64 GB RAM|
| 磁碟| 8 個 NVMe SSD，每個硬碟的大小為 1.6 TB <br> 如果有單一 NVMe SSD 失敗，系統便會失敗。 |
| 本機儲存體容量| 12.8 TB 的總容量|
| 網路介面| 2 個 1 GbE 介面 - 1 個為管理用途 (使用者無法設定)，用於初始設定。 另一個資料介面可由使用者設定，且預設為 DHCP。 <br>2 個 25 GbE 介面 - 這些也能以 10 GbE 介面的形式運作。 這些資料介面可由使用者設定為 DHCP (預設) 或靜態。 <br> 2 個 25 GbE 介面 - 這些資料介面可由使用者設定為 DHCP (預設) 或靜態。|

## <a name="components"></a>元件

Data Box Edge 解決方案由 Data Box Edge 資源、Data Box Edge 實體裝置與本機 Web UI 所組成。

* **Data Box Edge 實體裝置** - 由 Microsoft 所提供的 1U 機架裝載伺服器，可設定來將資料傳送到 Azure。 
    
* **Data Box Edge 資源** - Azure 入口網站中的資源，可讓您從不同的地理位置存取 Web 介面來管理 Data Box Edge 裝置。 使用 Data Box Edge 資源來建立及 管理資源、檢視和管理裝置與警示，以及管理共用。  

    <!--![The Data Box Edge service in Azure portal](media/data-box-overview/data-box-Edge-service1.png)-->

    如需詳細資訊，請移至[使用 Data Box Edge 服務來管理您的 Data Box Edge 裝置](http://aka.ms/dbe-docs)。

* **資料箱本機 Web UI** - 使用本機 Web UI 來對 Data Box Edge 裝置執行診斷、關機和重新啟動、檢視複製記錄，以及連絡 Microsoft 支援服務以提出服務要求。

    <!--![The Data Box Edge local web UI](media/data-box-Edge-overview/data-box-Edge-local-web-ui.png)-->

    如需有關使用 Web 型 UI 的資訊，請移至[使用 Web 型 UI 來管理資料箱](http://aka.ms/dbe-docs)。


## <a name="region-availability"></a>區域可用性

Data Box Edge 實體裝置、Azure 資源，以及您要將資料傳輸至其中的目標儲存體帳戶，並不需要全都位於相同的區域。

- **資源可用性** - 在針對此版本，下列區域提供 Data Box Edge 資源：
    - **美國** - 美國西部 2 與美國東部
    - **歐盟** - 西歐
    - **亞太地區** - 東南亞

- **目的地儲存體帳戶** - 儲存資料的儲存體帳戶在所有 Azure 區域都可用。 

    若要獲得最佳效能，儲存資料箱資料的儲存體帳戶應該要位在接近裝置所在區域的區域。 儲存體帳戶的位置若離裝置遙遠，將會導致較長的延遲時間與較緩慢的效能。 


## <a name="sign-up"></a>註冊

Data Box Edge 處於預覽狀態，而且您必須註冊。 請執行下列步驟以註冊 Data Box Gateway︰

1. 登入 Azure 入口網站：[https://aka.ms/databox-edge](https://aka.ms/databox-edge)。

2. 挑選您要用於 Data Box Edge 預覽的訂用帳戶。 選取您要部署 Data Box Edge 資源的區域。 在 [Data Box Edge] 選項中，按一下 [註冊]。

    ![Data Box Edge 註冊 3](media/data-box-edge-overview/data-box-edge-sign-up3.png)

3.  回答關於資料所在國家/地區、時間範圍、資料傳輸的目標 Azure 服務、網路頻寬，以及資料傳輸頻率的問題。 檢閱 [隱私權與條款]，然後選取 [Microsoft 可以使用您的電子郵件地址連絡您] 核取方塊。

    ![Data Box Edge 註冊 4](media/data-box-edge-overview/data-box-edge-sign-up4.png)

4. 在註冊並啟用預覽後，您便可以訂購 Data Box Edge。

## <a name="next-steps"></a>後續步驟

- 檢閱 [Data Box Edge 系統需求](http://aka.ms/dbe-docs)。
- 了解 [Data Box Edge 限制](http://aka.ms/dbe-docs)。
- 在 Azure 入口網站中部署 [Azure Data Box Edge](http://aka.ms/dbe-docs)。




