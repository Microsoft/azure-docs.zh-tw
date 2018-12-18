---
title: 適用於 VM 的 Azure 監視器已知問題 | Microsoft Docs
description: 適用於 VM 的 Azure 監視器是結合了 Azure VM 作業系統健康情況與效能監控的 Azure 解決方案，其會自動探索應用程式元件和與其他資源的相依性，並且會對應它們之間的通訊。 本文涵蓋各項已知問題。
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
ms.date: 09/18/2018
ms.author: magoedte
ms.openlocfilehash: 819c3e74355cf80c7a998abb8b02b10c9e077059
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47062763"
---
# <a name="known-issues-with-azure-monitor-for-vms"></a>適用於 VM 的 Azure 監視器已知問題

下列為「適用於 VM 的 Azure 監視器」健康情況功能的已知問題：

- 此版本的健康情況準則時段和頻率無法修改。 
- 無法停用健康情況準則。 
- 上線之後，可能需要一點時間，資料才會顯示在 [Azure 監視器] -> [虛擬機器]-> [健康情況] 或是 [VM 資源] 刀鋒視窗-> [深入解析] 中
- 健康情況診斷體驗比任何其他檢視的更新速度更快，因此您在檢視之間切換時，資訊可能會延遲出現  
- 「適用於 VM 的 Azure 監視器」健康情況中提供的警示摘要，僅適用於受監視 Azure VM 偵測到的健康情況問題所引發的警示。
- 單一 VM 和 Azure 監視器中的 [警示清單] 檢視頁面會顯示在過去 30 天內監視條件設為「引發」的警示。  您無法對其進行設定。 不過，按一下摘要之後，一旦啟動 [警示清單] 檢視頁面，您就可以變更監視條件和時間範圍的篩選條件值。
- 在 [警示清單] 檢視頁面上，建議您不要修改 [資源類型]、[資源] 及 [監視服務] 篩選條件，因為這些篩選條件已設定為專用於解決方案 (相較於 [Azure 監視器] -> [警示] 清單檢視，此清單檢視會額外顯示某些欄位)。    
- 在 Linux VM 中，[健康情況診斷] 檢視具有 VM 的完整網域名稱，而不是使用者定義的 VM 名稱。
- 關閉 VM 會將其一些健康情況準則更新為重大狀態，並將其他健康情況準則更新為狀況良好狀態，且 VM 的淨狀態處於重大狀態。
- 無法修改健康情況警示的嚴重性，只能加以啟用或停用。  此外，某些嚴重性會根據健康情況準則的狀態來更新。
- 修改健康情況準則執行個體的任何設定，將會導致修改 VM 上相同類型的所有健康情況準則執行個體中的相同設定。 例如，如果對應至邏輯磁碟 C: 的磁碟可用空間健康情況準則執行個體閾值經過修改，那麼此閾值會套用到針對相同 VM 探索並監視的所有其他邏輯磁碟。   
- 某些 Windows 健康情況準則 (例如 DNS 用戶端服務健康情況) 的閾值無法修改，因為其狀況良好狀態已根據內容鎖定為服務或實體的 [執行中]、[可用] 狀態。  反之，值會以數字 4 代表，它在未來的版本中會轉換成實際的顯示字串。  
- 某些 Linux 健康情況準則 (例如邏輯磁碟健康情況) 的閾值無法修改，因為它們已設定為在狀況不良的狀態下觸發。  這些閾值會指示項目是上線/離線 (或開啟還是關閉)，且會顯示 1 或 0 的值來代表及表示相同內容。
- 更新任何資源群組中的 [資源群組] 篩選條件，同時使用大規模 [Azure 監視器] -> [虛擬機器] -> [健康情況] -> [任何] 清單檢視，並包含預先選取的訂用帳戶和資源群組，會導致清單檢視顯示 [沒有結果]。  請返回 [Azure 監視器] -> [虛擬機器] -> [健康情況] 索引標籤，並選取所需的訂用帳戶和資源群組，然後瀏覽至清單檢視。

## <a name="next-steps"></a>後續步驟
檢閱[將適用於 VM 的 Azure 監視器上線](monitoring-vminsights-onboard.md)，了解啟用虛擬機器監視的需求和方法。