---
title: Microsoft Azure Stack Edge Pro with GPU 概觀 | Microsoft Docs
description: 說明 Microsoft Azure Stack Edge Pro with GPU，這是使用實體裝置針對 Azure 進行網路型傳輸的儲存體解決方案。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 09/23/2020
ms.author: alkohli
ms.openlocfilehash: 059aad8b45aa449f5cd28c97f4ba55e2204c04ce
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/09/2020
ms.locfileid: "96937061"
---
# <a name="what-is-azure-stack-edge-pro-with-gpu"></a>Azure Stack Edge Pro with GPU 是什麼？

Azure Stack Edge Pro with GPU 為具備 AI 能力的邊緣運算裝置，並具有網路資料傳輸功能。 此文章為您提供 Azure Stack Edge Pro 解決方案的概觀、優點、主要功能，以及適合部署此裝置的案例。

Azure Stack Edge Pro with GPU 是一種硬體即服務解決方案。 Microsoft 可為您提供作為網路儲存體閘道的雲端管理裝置，並具有可加速 AI 推斷的內建圖形處理器 (GPU)。 

## <a name="use-cases"></a>使用案例

以下是 Azure Stack Edge Pro 可用於該邊緣的快速 Machine Learning (ML) 推斷，並在將資料傳送到 Azure 之前先進行預先處理的各種案例。

- **使用 Azure Machine Learning 進行推斷**：透過 Azure Stack Edge Pro，您可以執行 ML 模型，以便在將資料傳送到雲端之前快速取得可據以採取動作的結果。 完整資料集可以選擇性地進行傳輸，以繼續重新定型並改善您的 ML 模型。 如需如何在 Azure Stack Edge Pro 裝置上使用 Azure ML 硬體加速模型的詳細資訊，請參閱[在 Azure Stack Edge Pro 上部署 Azure ML 硬體加速模型](../machine-learning/how-to-deploy-fpga-web-service.md#deploy-to-a-local-edge-server) \(部分機器翻譯\)。

- **前置處理資料** - 先轉換資料，再透過計算選項 (例如容器化工作負載和虛擬機器) 將資料傳送至 Azure，以建立更容易操作的資料集。 預先處理功能可用來： 

    - 彙總資料。
    - 修改資料，例如移除個人資料。
    - 子集資料可將儲存體和頻寬最佳化，或用於進一步分析。
    - 分析並對 IoT 事件做出反應。 

- **透過網路將資料傳輸到 Azure**：使用 Azure Stack Edge Pro，輕鬆且快速地將資料傳輸到 Azure，以進行進一步的運算和分析，或基於封存目的使用。 

## <a name="key-capabilities"></a>主要功能

Azure Stack Edge Pro 具有下列功能：

|功能 |描述  |
|---------|---------|
|加速 AI 推斷| 由內建 GPU 啟用 (根據模型來決定啟用一個或兩個)。|
|Edge 計算      |支援 VM 和容器化工作負載，以允許分析、處理和篩選資料。 |
|資料存取     | 能使用雲端 API 直接從 Azure 儲存體 Blob 與 Azure 檔案存取資料，以在雲端中進行額外的資料處理。 裝置上的本機快取可用來快速存取最近使用的檔案。|
|雲端管理     |裝置與服務是透過 Azure 入口網站來管理的。  |
|離線上傳     | 中斷連線模式支援離線上傳案例。|
|支援的檔案傳輸通訊協定      | 支援用於資料擷取的標準 SMB、NFS 與 REST 通訊協定。 <br> 如需支援版本的詳細資訊，請參閱 [ 系統需求](azure-stack-edge-system-requirements.md)。|
|資料重新整理     | 能夠以來自雲端的最新檔案對本機檔案進行重新整理。|
|加密    | 支援使用 BitLocker 在本機加密資料，並透過 *https* 保護對雲端的資料傳輸。|
|頻寬節流設定| 節流以限制尖峰時間的頻寬使用量。|
<!--|ExpressRoute | 透過 ExpressRoute 增加安全性。 使用對等互連組態，從本機裝置到雲端儲存體端點的流量會透過 ExpressRoute 傳輸。 如需詳細資訊，請參閱 [ExpressRoute 概觀](../expressroute/expressroute-introduction.md)。|-->

## <a name="components"></a>元件

Azure Stack Edge Pro 解決方案由 Azure Stack Edge 資源、Azure Stack Edge Pro 實體裝置及本機 Web UI 所組成。

* **Azure Stack Edge Pro 實體裝置**：由 Microsoft 所提供的 1U 機架裝載伺服器，可設定來將資料傳送到 Azure。
    
* **Azure Stack Edge 資源**：Azure 入口網站中的資源，可讓您從不同的地理位置存取 Web 介面來管理 Azure Stack Edge Pro 裝置。 使用 Azure Stack Edge 資源來建立和管理資源、檢視和管理裝置與警示，以及管理共用。  

    如需詳細資訊，請移至[為您的 Azure Stack Edge Pro 裝置建立訂單](azure-stack-edge-gpu-deploy-prep.md#create-a-new-resource)。

* **Azure Stack Edge Pro 本機 Web UI** - Azure Stack Edge Pro 裝置上的瀏覽器型本機使用者介面，主要是用於裝置的初始組態。 也可使用本機 Web UI 對 Azure Stack Edge Pro 裝置執行診斷、關機和重新啟動、檢視複製記錄，以及連絡 Microsoft 支援服務以提出服務要求。

    如需使用 Web 型 UI 的相關資訊，請移至[使用 Web 型 UI 來管理 Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md)。

## <a name="region-availability"></a>區域可用性

Azure Stack Edge Pro 實體裝置、Azure 資源，以及您要傳輸資料的目標儲存體帳戶，不需全都位於相同區域。

- **資源可用性** - 針對此預覽版本，該資源可用於美國東部、西歐和東南亞區域。

- **裝置可用性** - 如需適用 Azure Stack Edge Pro 裝置的所有國家/地區的清單，請移至 [Azure Stack Edge Pro 定價](https://azure.microsoft.com/pricing/details/azure-stack/edge/#azureStackEdgePro)，參閱 [Azure Stack Edge Pro] 索引標籤中的 [可用性] 區段。
    
- **目的地儲存體帳戶** - 儲存資料的儲存體帳戶在所有 Azure 區域都可用。 若要獲得最佳效能，儲存 Azure Stack Edge Pro 資料的儲存體帳戶應位於接近裝置所在的位置。 儲存體帳戶的位置若離裝置遙遠，將會導致較長的延遲時間與較緩慢的效能。

## <a name="next-steps"></a>後續步驟

- 檢閱 [Azure Stack Edge Pro 系統需求](azure-stack-edge-gpu-system-requirements.md)。

- 了解 [Azure Stack Edge Pro 限制](azure-stack-edge-limits.md)。
- 在 Azure 入口網站中部署 [Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md)。
