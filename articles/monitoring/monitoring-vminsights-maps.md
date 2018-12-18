---
title: 如何使用適用於 VM 的 Azure 監視器來檢視應用程式相依性 | Microsoft Docs
description: 對應是適用於 VM 的 Azure 監視器的一項功能，可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 本文提供如何將它用於各種案例的詳細資料。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: magoedte
ms.openlocfilehash: e75409e791c00dc0a5bec591aecfbaa019df8f81
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225157"
---
# <a name="using-azure-monitor-for-vms-map-to-understand-application-components"></a>使用適用於 VM 的 Azure 監視器對應來了解應用程式元件
檢視在 Azure 環境中執行的 Windows 和 Linux 虛擬機器上探索到的應用程式元件，可搭配適用於 VM 的 Azure 監視器使用下列兩種方式來觀測：直接從虛擬機器，或者從 Azure 監視器跨 VM 群組進行。 

本文將協助您了解這兩個檢視方塊之間的體驗，以及對應功能的使用方式。 如需設定適用於 VM 的 Azure 監視器相關資訊，請參閱[啟用適用於 VM 的 Azure 監視器](monitoring-vminsights-onboard.md)。

## <a name="sign-in-to-azure"></a>登入 Azure
在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。

## <a name="introduction-to-map-experience"></a>對應體驗簡介
深入了解如何檢視適用於單一虛擬機器或 VM 群組的對應之前，重要的是我們將提供一段功能簡介，讓您能夠了解資訊的呈現方式，以及視覺效果所呈現的內容。  

不論您是直接從 VM 或從 Azure 監視器選取對應功能，它都會呈現一致的體驗。  唯一差異在於從 Azure 監視器，您可以在一個對應中看見某個多層式應用程式或叢集的所有成員。

對應會透過下列方式來將 VM 相依性視覺化：探索在伺服器之間具有作用中網路連線的執行中處理序、輸入和輸出連線延遲，以及在指定的時間範圍內跨任何 TCP 連線架構的連接埠。  展開 VM 即可顯示處理序詳細資料，而且只會顯示與 VM 通訊的處理序。 連線到虛擬機器的前端用戶端計數會使用用戶端群組來表示。 VM 連線的後端伺服器計數則會顯示於伺服器連接埠群組中。 展開伺服器連接埠群組，以查看透過該連接埠連線的伺服器詳細清單。  

當您按一下虛擬機器時，[屬性] 窗格會在右側展開以顯示所選取項目的屬性，例如，作業系統所報告的系統資訊、Azure VM 的屬性，以及摘要說明探索到之連線的環圈圖。 

![電腦的系統屬性](./media/monitoring-vminsights-maps/properties-pane-01.png)

在窗格右側，按一下 [記錄事件] 圖示以切換窗格的焦點來顯示資料表清單，其已將從 VM 收集到的資料傳送到 Log Analytics，可供查詢。  按一下任一個列出的記錄類型，將會開啟 [記錄] 頁面，透過已針對特定虛擬機器篩選過的預先設定查詢來檢視該類型的結果。  

![[屬性] 窗格中的記錄搜尋清單](./media/monitoring-vminsights-maps/properties-pane-logs-01.png)

關閉 [記錄]* 並返回 [屬性] 窗格，然後選取 [警示] 來檢視警示，警示是針對 VM 從健康情況準則所引發的。 對應會與 Azure 警示整合，以顯示所選時間範圍內針對所選伺服器而引發的警示。 如果有最新警示，伺服器就會顯示一個圖示，而 [機器警示] 窗格會列出警示。 

![[屬性] 窗格中的 [機器警示]](./media/monitoring-vminsights-maps/properties-pane-alerts-01.png)

若要讓對應功能顯示相關警示，請建立針對特定電腦引發的警示規則。 若要建立適當的警示︰
- 包含依電腦群組的子句，例如，**by Computer interval 1 minute** (依 1 分鐘的電腦間隔)。
- 選擇依據計量量值來發出警示。

如需 Azure 警示和建立警示規則的詳細資訊，請參閱 [Azure 監視器中的整合警示](../monitoring-and-diagnostics/monitoring-overview-unified-alerts.md)

右上角的 [圖例] 選項會說明對應的符號和角色。  若要放大以進一步查看您的對應並四處移動它，可使用頁面右下角的縮放控制項來設定縮放層級，並使頁面符合目前頁面的大小。  

## <a name="connection-metrics"></a>連接計量
[連線] 窗格會針對透過 TCP 連接埠且來自 VM 的所選連線，顯示標準連線計量。 計量包括回應時間、每分鐘的要求、流量輸送量及連結。  

![網路連線能力圖表窗格範例](./media/monitoring-vminsights-maps/map-group-network-conn-pane-01.png)  

### <a name="failed-connections"></a>失敗的連線
對應中會顯示處理序和電腦的失敗連線，並以紅色虛線指出用戶端系統無法連線到處理序或連接埠。 任何具有相依性代理程式的系統如果就是嘗試進行失敗連線的系統，則會報告失敗的連線。 對應會觀測無法建立連線的 TCP 通訊端，藉以衡量此處理序。 連線失敗的原因可能是防火牆、用戶端或伺服器設定不正確，或遠端服務無法使用。

![對應中的失敗連線範例](./media/monitoring-vminsights-maps/map-group-failed-connection-01.png)

了解失敗的連線有助於進行疑難排解、移轉驗證、安全性分析，以及了解服務的整體架構。 失敗的連線有時無害，但它們通常直指問題所在，例如，容錯移轉環境突然變成無法連線，或兩個應用程式層在雲端移轉之後彼此無法通訊。

### <a name="client-groups"></a>用戶端群組
對應中的用戶端群組代表連線到對應機器的用戶端電腦。 單一用戶端群組代表個別處理序或機器的用戶端。

![對應中的用戶端群組範例](./media/monitoring-vminsights-maps/map-group-client-groups-01.png)

若要查看受監視的用戶端及用戶端群組中系統的 IP 位址，請選取群組。 群組的內容會列出在群組下方。  

![對應中的用戶端群組 IP 位址清單範例](./media/monitoring-vminsights-maps/map-group-client-group-iplist-01.png)

如果群組包含受監視和未受監視的用戶端，您可以在群組中選取環圈圖的適當區段來篩選用戶端。

### <a name="server-port-groups"></a>伺服器連接埠群組
伺服器連接埠群組代表伺服器上的伺服器連接埠，其具有來自對應機器的輸入連線。 此群組包含伺服器連接埠，以及與該連接埠連線的伺服器數目。 選取群組，以查看列出的個別伺服器和連線。 

![對應中的伺服器連接埠群組範例](./media/monitoring-vminsights-maps/map-group-server-port-groups-01.png)  

如果群組包含受監視和未受監視的伺服器，您可以在群組中選取環圈圖的適當區段來篩選伺服器。

## <a name="view-map-directly-from-a-virtual-machine"></a>直接從虛擬機器檢視對應 

若要直接從虛擬機器存取適用於 VM 的 Azure 監視器，請執行下列動作。

1. 在 Azure 入口網站中，選取 [虛擬機器]。 
2. 從清單中選擇 VM，然後在 [監視] 區段中選擇 [Insights (預覽)]。  
3. 選取 [對應] 索引標籤。

對應會以視覺化方式呈現指定時間範圍內的 VM 相依性，也就是執行中的處理序群組，以及具有作用中網路連線的處理序。  根據預設，對應會顯示過去 30 分鐘。  使用左上角的 [時間範圍] 選取器，可查詢過往時間範圍 (最長可達一小時)，以顯示相依性的過往情形 (例如在事件發生期間或變更發生之前)。  

![直接 VM 對應概觀](./media/monitoring-vminsights-maps/map-direct-vm-01.png)

## <a name="view-map-from-azure-monitor"></a>從 Azure 監視器檢視對應
從 Azure 監視器，對應功能可提供您虛擬機器及其相依性的全域檢視。  若要從 Azure 監視器存取對應功能，請執行下列動作。 

1. 在 Azure 入口網站中，選取 [監視]。 
2. 選擇 [Insights] 區段中的 [虛擬機器 (預覽)]。
3. 選取 [對應] 索引標籤。

![Azure 監視器多部 VM 對應概觀](./media/monitoring-vminsights-maps/map-multivm-azure-monitor-01.png)

從頁面頂端的 [工作區] 選取器，如果您具有多個 Log Analytics 工作區，請選擇已與解決方案整合且具有要向其報告之虛擬機器的 Log Analytics 工作區。  然後從 [群組] 選取器、訂用帳戶或資源群組進行選取，以檢視一段指定時間內一組 VM 及其與群組相符的相依性。  根據預設，對應會顯示過去 30 分鐘。  使用 [時間範圍] 選取器，可查詢過往時間範圍 (最長可達一小時)，以顯示相依性的過往情形 (例如在事件發生期間或變更發生之前)。   

## <a name="next-steps"></a>後續步驟
若要了解如何使用健康情況功能，請參閱[檢視 Azure VM 健康情況](monitoring-vminsights-health.md)，或者，若要找出 VM 效能的瓶頸和整體使用率，請參閱[檢視適用於 VM 的 Azure 監視器效能](monitoring-vminsights-performance.md)。 
