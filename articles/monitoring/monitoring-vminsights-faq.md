---
title: 適用於 VM 的 Azure 監視器常見問題集 | Microsoft Docs
description: 適用於 VM 的 Azure 監視器是 Azure 中的解決方案，結合了 Azure VM 作業系統的健康情況和效能監視，以及自動探索應用程式元件和其他資源的相依性，並對應它們之間的通訊。 本文將回答常見問題。
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
ms.date: 09/19/2018
ms.author: magoedte
ms.openlocfilehash: 308a447ff99cd11ad6a28df0bdb515764b0f546b
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063450"
---
# <a name="azure-monitor-for-vms-frequently-asked-questions"></a>適用於 VM 的 Azure 監視器常見問題集
此 Microsoft 常見問題集是 Microsoft Azure 中適用於 VM 的 Azure 監視器常見問題清單。 若您有任何關於解決方案的其他問題，請前往[討論論壇](https://feedback.azure.com/forums/34192--general-feedback)並張貼您的問題。 當問到常見問題時，我們會將其新增至此文章，以便其他人可以快速輕鬆地找到此問題。

## <a name="can-i-onboard-to-an-existing-workspace"></a>我可以將現有的工作區上線嗎？
如果您的虛擬機器已經連線到 Log Analytics 工作區，則您可以在上線到適用於 VM 的 Azure 監視器時繼續使用該工作區，但前提是它位於[此處](monitoring-vminsights-onboard.md#prerequisites)所列的其中一個支援區域。

上線時，我們會設定工作區的效能計數器，這將導致所有 VM 都會向工作區回報資料，開始收集此資訊，以便在適用於 VM 的 Azure 監視器中加以顯示和分析。  因此，您將會看到來自所有連線到所選工作區之 VM 的效能資料。  健康情況和對應功能只能針對您已指定來上線的 VM 加以啟用。

如需要啟用哪些效能計數器的詳細資訊，請參閱我們的[上線](monitoring-vminsights-onboard.md)文章。

## <a name="can-i-onboard-to-a-new-workspace"></a>我可以上線到新的工作區嗎？ 
如果您的 VM 目前並未連線到現有的 Log Analytics 工作區，您需要建立新的工作區來儲存資料。  如果您透過 Azure 入口網站為適用於 VM 的 Azure 監視器設定單一 Azure VM，建立新的預設工作區即會自動完成。

如果您選擇使用指令碼型方法，則可在[上線](monitoring-vminsights-onboard.md)文章中找到這些步驟。 

## <a name="what-do-i-do-if-my-vm-is-already-reporting-to-an-existing-workspace"></a>如果我的 VM 已經向現有的工作區回報，我該怎麼做？
如果您已經從虛擬機器收集資料，則您可能已經將它設為向現有的 Log Analytics 工作區回報資料。  只要該工作區位於我們支援的其中一個區域，您就可以針對該預先存在的工作區啟用適用於 VM 的 Azure 監視器。  如果您已經使用的工作區不在我們支援的其中一個區域，目前將無法上線到適用於 VM 的 Azure 監視器。  我們正積極努力地支援更多區域。

>[!NOTE]
>我們會針對工作區設定效能計數器，其會影響要向該工作區回報的所有 VM，而不論您是否已選擇要將它們上線到適用於 VM 的 Azure 監視器。 如需如何針對工作區設定效能計數器的詳細資訊，請參閱我們的[文件](../log-analytics/log-analytics-data-sources-performance-counters.md)。 如需針對適用於 VM 的 Azure 監視器所設定的計數器相關資訊，請參閱我們的[上線文件](monitoring-vminsights-onboard.md#performance-counters-enabled)。  

## <a name="why-did-my-vm-fail-to-onboard"></a>為什麼無法將我的 VM 上線？
從 Azure 入口網站將 Azure VM 上線時，請執行下列步驟：

* 如果已選取該選項，即會建立預設的 Log Analytics 工作區。
* 已針對所選工作區設定效能計數器。 如果這個步驟失敗，您會注意到某些效能圖表和表格不會顯示您所上線之 VM 的資料。 您可以執行[此處](monitoring-vminsights-onboard.md#enable-with-powershell)所述的 PowerShell 指令碼來修正此問題。
* 如果決定需要使用 Log Analytics 代理程式，可使用 VM 延伸模組在 Azure VM 上進行安裝。  
* 如果決定需要使用適用於 VM 的 Azure 監視器對應 Dependency Agent，可使用延伸模組在 Azure VM 上進行安裝。  
* 視需要設定支援健康情況功能的 Azure 監視器元件，並設定 VM 來回報健康情況資料。

在上線過程中，我們會檢查上述每一項的狀態，以便在入口網站中將通知狀態傳回給您。  設定工作區與代理程式安裝通常需要 5 至 10 分鐘。  在入口網站中檢視監視與健康情況資料需要額外的 5 至 10 分鐘。  

如果您已初始上線，並看到一則訊息指出必須將 VM 上線，則允許 VM 最多 30 分鐘的時間來完成此程序。 

## <a name="i-dont-see-some-or-any-data-in-the-performance-charts-for-my-vm"></a>我在適用於 VM 的效能圖表中看不到部分或任何資料
如果您在磁碟表格中或在某些效能圖表中看不到效能資料，則您可能未在工作區中設定效能計數器。 若要解決此問題，請執行下列 [PowerShell 指令碼](monitoring-vminsights-onboard.md#enable-with-powershell)。

## <a name="how-is-azure-monitor-for-vms-map-feature-different-from-service-map"></a>適用於 VM 的 Azure 監視器對應功能與服務對應有何不同？
適用於 VM 的 Azure 監視器對應功能會以服務對應為根據，但有下列差異：

* [對應] 檢視可以從 VM 刀鋒視窗，以及從 Azure 監視器下方適用於 VM 的 Azure 監視器進行存取。
* [對應] 中的連線現在可供點選，並在所選連線的側邊面板中顯示連線計量資料的檢視。
* 有一個新的 API 可用來建立對應，以便為更複雜的對應提供更好的支援。
* 受監視的 VM 現在包含於用戶端群組節點中，而環圈圖會顯示群組中受監視與未受監視的虛擬機器比例。  它也可以在展開群組時用來篩選機器清單。
* 受監視的虛擬機器現在包含於伺服器連接埠群組節點中，而環圈圖會顯示群組中受監視與未受監視的機器比例。  它也可以在展開群組時用來篩選機器清單。
* 對應樣式已更新，以便與 Application Insights 的應用程式對應更為一致。
* 側邊面板已更新，但尚未與服務對應中支援的項目完全整合：更新管理、變更追蹤、安全性及服務台。 
* 用來選擇要對應之群組與機器的選項已更新，現在支援訂用帳戶、資源群組、Azure 虛擬機器擴展集和雲端服務。
* 您無法在適用於 VM 的 Azure 監視器對應功能中建立新的服務對應機器群組。  

## <a name="why-do-my-performance-charts-show-dotted-lines"></a>為什麼我的效能圖表會顯示虛線？

發生的原因有數種。  萬一資料集合中有間隔，我們就會將線條描繪為虛線。  如果您針對啟用的效能計數器修改了資料取樣頻率 (預設設定是每隔 60 秒收集一次資料)，若您針對圖表選擇較窄的時間範圍，而且您的取樣頻率小於圖表中所使用的貯體大小 (例如，取樣頻率為每隔 10 分鐘，而圖表上的每個貯體是 5 分鐘)，則您會在圖表中看到虛線。  在此案例中，選擇更寬的時間範圍來檢視，應該會讓圖表線條顯示為實線，而非虛線。

## <a name="are-groups-supported-with-azure-monitor-for-vms"></a>適用於 VM 的 Azure 監視器支援群組嗎？
效能功能會根據特定工作區內強調顯示的資源群組支援群組，以及根據特定的 Azure 虛擬機器擴展集和雲端服務支援群組設定。

## <a name="how-do-i-see-the-details-for-what-is-driving-the-95th-percentile-line-in-the-aggregate-performance-charts"></a>如何查看彙總效能圖表中衍生第 95 條百分位數線之項目的詳細資料？
根據預設，此清單會排序以顯示針對所選計量的第 95 個百分位數具有最高值的 VM，但可用記憶體圖表除外，因為它會顯示具有第 5 個百分位數之最小值的機器。  按一下圖表，將利用已選取的適當計量來開啟 [N 大排行榜] 檢視。

## <a name="how-does-the-map-feature-handle-duplicate-ips-across-different-vnets-and-subnets"></a>對應功能如何跨不同的 VNet 和子網路處理重複的 IP？
如果您跨子網路和 VNet 使用 VM 或 Azure 虛擬機器擴展集來複製 IP 範圍，它可能會造成適用於 VM 的 Azure 監視器對應功能顯示不正確的資訊。 這是個已知的問題，而我們正在研究改善此體驗的選項。

## <a name="does-map-feature-support-ipv6"></a>對應功能支援 IPv6 嗎？
對應功能目前僅支援 IPv4，而我們正在研究對於 IPv6 的支援。 我們也支援在 IPv6 內部設定通道的 IPv4。

## <a name="when-i-load-a-map-for-a-resource-group-or-other-large-group-the-map-is-difficult-to-view"></a>當我針對資源群組或其他大型群組載入對應時，很難檢視該對應
儘管我們已改進對應來處理大型且複雜的設定，但我們理解到對應可擁有許多節點、連線，以及作為叢集使用的節點。  我們承諾會繼續加強支援以提高延展性。   

## <a name="why-does-the-network-chart-on-the-performance-tab-look-different-than-the-network-chart-on-the-azure-vm-overview-page"></a>[效能] 索引標籤上的網路圖表為什麼看起來與 [Azure VM 概觀] 頁面上的網路圖表不同呢？

適用於 Azure 虛擬機器的概觀頁面會根據客體 VM 中主機的活動量值來顯示圖表。  至於 [Azure VM 概觀] 上的網路圖表，它只會顯示將收費的網路流量。  這不包括虛擬網路間的流量。  針對適用於 VM 的 Azure 監視器顯示的資料和圖表，會以來自客體 VM 的資料為基礎，而網路圖表會顯示輸入和輸出到該 VM (包括虛擬網路間) 的所有 TCP/IP 流量。

## <a name="next-steps"></a>後續步驟
檢閱[將適用於 VM 的 Azure 監視器上線](monitoring-vminsights-onboard.md)，了解啟用虛擬機器監視的需求和方法。
