---
title: 將 IaaS 資源從傳統移轉至 Azure Resource Manager 的規劃 | Microsoft Docs
description: 將 IaaS 資源從傳統移轉至 Azure Resource Manager 的規劃
services: virtual-machines-windows
documentationcenter: ''
author: singhkays
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 78492a2c-2694-4023-a7b8-c97d3708dcb7
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 04/01/2017
ms.author: kasing
ms.openlocfilehash: 6d84c83efa194543ed10aaed82362021b7053476
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576199"
---
# <a name="planning-for-migration-of-iaas-resources-from-classic-to-azure-resource-manager"></a>將 IaaS 資源從傳統移轉至 Azure Resource Manager 的規劃
雖然 Azure Resource Manager 提供了許多令人讚嘆的功能，但請務必詳加規劃您的移轉作業，以確保一切順利進行。 詳細規劃可確保您在執行移轉活動期間不會遇到問題。

> [!NOTE]
> 下列方針中，絕大多數是由與客戶合作移轉大型環境的 Azure 客戶諮詢團隊和雲端方案架構設計人員所提供。 因此，本文件將隨著出現成功的新模式而繼續更新，請不定時回來查看，以了解是否有任何新的建議。

移轉旅程有四個一般階段：<br>

![移轉階段](../media/virtual-machines-windows-migration-classic-resource-manager/plan-labtest-migrate-beyond.png)

## <a name="plan"></a>規劃

### <a name="technical-considerations-and-tradeoffs"></a>技術考量和取捨

根據您的技術需求多寡、地理位置和作業實務，您可能要考慮：

1. 為什麼您的組織需要 Azure Resource Manager？  移轉的商業理由有哪些？
2. 選擇 Azure Resource Manager 的技術理由有哪些？  您要運用 Azure 服務的哪些其他功能 (如果有的話)？
3. 移轉中包含哪些應用程式 (或虛擬機器的集合)？
4. 移轉 API 支援哪些案例？  檢閱[不支援的功能和組態](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#unsupported-features-and-configurations)。
5. 您的作業團隊目前支援傳統和 Azure Resource Manager 中的應用程式/VM 嗎？
6. Azure Resource Manager 如何變更您的 VM 部署、管理、監視和報告處理程序 (如果有的話)？  需要更新部署指令碼嗎？
7. 警示專案關係人 (使用者、應用程式擁有者，以及基礎結構擁有者) 的通訊規劃為何？
8. 根據環境的複雜度，是否應該有應用程式無法供使用者和應用程式擁有者使用的維護期間？  如果是這樣，該期間需要多久？
9. 確保專案關係人了解並精通 Azure Resource Manager 的訓練規劃為何？
10. 移轉的程式管理或專案管理規劃為何？
11. Azure Resource Manager 移轉和其他相關技術藍圖的時間表為何？  它們之間的配合是否理想？

### <a name="patterns-of-success"></a>成功的模式

成功的客戶會進行詳細的規劃，並針對上述問題進行討論、記載和控管。  請務必與贊助者和專案關係人廣泛地溝通移轉規劃。  讓自己具備有關移轉選項的知識；強烈建議您閱讀以下的移轉文件集。

* [平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager) 的概觀](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [平台支援的從傳統移轉至 Azure Resource Manager 的技術深入探討](migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [將 IaaS 資源從傳統移轉至 Azure Resource Manager 的規劃](migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [使用 PowerShell 將 IaaS 資源從傳統移轉至 Azure Resource Manager](migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [使用 CLI 將 IaaS 資源從傳統移轉至 Azure Resource Manager](../linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [用於協助將 IaaS 資源從傳統移轉至 Azure Resource Manager 的社群工具](migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [檢閱最常見的移轉錯誤](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [檢閱有關將 IaaS 資源從傳統移轉至 Azure Resource Manager 的常見問題集](migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

### <a name="pitfalls-to-avoid"></a>要避免的陷阱

- 規劃失敗。  此移轉的技術步驟已經過證實，且結果是可預測的。
- 假設在平台支援的移轉 API 會負責處理所有案例。 閱讀[不支援的功能和組態](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#unsupported-features-and-configurations)來了解支援的案例。
- 未針對使用者規劃潛在的應用程式中斷。  規劃足夠的緩衝時間，來警告使用者可能無法使用應用程式的時間。


## <a name="lab-test"></a>實驗室測試

**複寫您的環境並執行測試移轉**
  > [!NOTE]
  > 現有環境的確切複寫是使用 Microsoft 支援服務未正式支援的社群貢獻工具來執行。 因此，這是**選擇性**步驟，但它是找出問題，而不需接觸生產環境的最佳方式。 如果無法使用社群提供的工具，則請閱讀以下的驗證/準備/中止試執行相關建議。
  >

  確保順利移轉的最佳方式，是以實際會進行的方式 (計算、網路和儲存體) 來進行您的實驗室測試。 這將有助於確保：

  - 完全獨立的實驗室或要測試的現有非生產環境。 我們建議使用可以重複移轉且可透過破壞性方式修改的完全獨立實驗室。  從實際的訂用帳戶收集/水合中繼資料的指令碼如下所示。
  - 在個別的訂用帳戶中建立實驗室是良好的作法。 原因在於實驗室會一再地進行重複清理，而具有分開、隔離的訂用帳戶將可降低不小心刪除實際項目的機會。

  使用 AsmMetadataParser 工具可以達到此目的。 [在這裡深入了解此工具](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/AsmToArmMigrationApiToolset)

### <a name="patterns-of-success"></a>成功的模式

以下是在許多較大型移轉中發現的問題。 這不是詳盡完整的清單，您應該參閱[不支援的功能和組態](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#unsupported-features-and-configurations)以取得更多詳細資料。  您可能會或可能不會發生這些技術問題，但如果您確實在嘗試移轉之前解決這些問題，將可確保獲得更順暢的體驗。

- **執行驗證/準備/中止試執行** - 這可能是確保傳統至 Azure Resource Manager 移轉是否成功的最重要步驟。 移轉 API 有三個主要步驟：驗證、準備和認可。 「驗證」將讀取傳統環境的狀態，並傳回所有問題的結果。 不過，由於 Azure Resource Manager 堆疊中可能存在一些問題，「驗證」將不會擷取每個項目。 移轉程序中的下一個步驟，也就是「準備」，會協助公開這些問題。 「準備」會將中繼資料從傳統移動至 Azure Resource Manager，但不會認可移動，也不會移除或變更傳統端上的任何項目。 試執行則牽涉到準備移轉，然後中止 (**不認可**) 移轉的準備。 驗證/準備/中止試執行的目標是要查看 Azure Resource Manager 堆疊中的所有中繼資料，加以檢查 (*以程式設計方式或在入口網站中*)，並確認所有項目移轉正確，以及解決技術問題。  它也會讓您大致了解移轉時間，以便規劃停機時間。  驗證/準備/中止不會造成任何使用者停機時間；因此，它對應用程式的使用不具破壞性。
  - 您需要在試執行之前解決以下項目，不過試執行測試也會安全地排清這些遺漏的準備步驟。 在企業移轉期間，我們發現試執行是確保移轉整備安全且無價的方式。
  - 當準備執行中時，將會鎖定整個虛擬網路的控制平面 (Azure 管理作業)；因此，無法在驗證/準備/中止期間對 VM 中繼資料進行變更。  但另一方面，任何應用程式功能 (RD、VM 使用量等) 將不會受到影響。  VM 的使用者將不會知道正在執行試執行。

- **ExpressRoute 線路和 VPN**。 目前具有授權連結的 ExpressRoute 閘道，無法在沒有停機時間的情況下進行移轉。 如需因應措施，請參閱[將 ExpressRoute 線路和相關聯的虛擬網路從傳統部署模型移轉至 Resource Manager 部署模型](../../expressroute/expressroute-migration-classic-resource-manager.md)。

- **VM 擴充功能** - 虛擬機器擴充功能可能是移轉執行中 VM 其中一個最大的障礙。 VM 擴充功能的修復可能需要多達 1-2 天，因此請適當地進行規劃。  需要使用中的 Azure 代理程式，才能向 VM 擴充功能回報執行中 VM 的狀態。 如果執行中 VM 的狀態恢復為已損毀，這種情況將會中止移轉。 代理程式本身不需要採用工作順序即可啟用移轉，但如果 VM 上存在擴充功能，則將需要工作代理程式與輸出網際網路連線 (具有 DNS)，移轉才能繼續進行。
  - 如果在移轉期間遺失對 DNS 伺服器的連線，則在準備移轉之前，必須先從每個 VM 中移除 BGInfo 第 1 版\*以外的所有 VM 擴充功能，並在 Azure Resource Manager 移轉之後，陸續重新新增回 VM。  **這僅適用於執行中的 VM。**  如果 VM 已停止 (解除配置)，則不需要移除 VM 擴充功能。

  > [!NOTE]
  > 許多擴充功能 (例如 Azure 診斷和資訊安全中心監視) 在移轉之後將會自行重新安裝，因此移除這些功能不是問題。

  - 此外，請確定網路安全性群組不會限制輸出網際網路存取。 某些網路安全性群組組態可能會發生此情況。 需要輸出網際網路存取 (和 DNS)，才能將 VM 擴充功能移轉至 Azure Resource Manager。
  - BGInfo 擴充功能有兩個版本，稱為第 1 版和第 2 版。  

      - 如果 VM 使用 BGInfo 第 1 版擴充功能，您可以將此擴充功能保留原狀。 移轉 API 會略過此擴充功能。 移轉之後，可以加入 BGInfo 擴充功能。
      - 如果 VM 使用以 JSON 為基礎的 BGInfo 第 2 版擴充功能，則會使用 Azure 入口網站建立 VM。 若代理程式正在運作，並擁有對外的網際網路存取 (和 DNS)，移轉 API 在移轉至 Azure Resource Manager 時會包含此擴充功能。

  - **修復選項 1**。 如果您知道您的 VM 在不會有輸出網際網路存取、使用中 DNS 服務，和使用中 Azure 代理程式，則請在準備移轉之前解除安裝所有 VM 擴充功能，然後在移轉之後重新安裝 VM 擴充功能。
  - **修復選項 2**。 如果 VM 擴充功能會造成太大的問題，另一個選擇是在移轉前將所有 VM 關閉/解除配置。 移轉已取消配置的 VM，然後在 Azure Resource Manager 端重新啟動。 優點是 VM 擴充功能也會一併移轉。 缺點是會遺失所有對外公開的虛擬 IP (可能是非入門者)；很明顯地 VM 將會關閉，對工作應用程式造成更大的影響。

    > [!NOTE]
    > 如果已針對要移轉的執行中 VM 設定 Azure 資訊安全中心原則，必須先停止安全性原則再移除擴充功能，否則會在移除擴充功能之後，自動在 VM 上重新安裝安全性監視擴充功能。

- **可用性設定組** - 若要將虛擬網路 (vNet) 移轉到 Azure Resource Manager，傳統部署 (也就是雲端服務) 所包含的 VM 必須全部位在一個可用性設定組中，或是所有 VM 均不能在任何可用性設定組中。 雲端服務中有一個以上的可用性設定組與 Azure Resource Manager 不相容，將會中止移轉。  此外，不能有一些 VM 在可用性設定組中，而一些 VM 則不在可用性設定組中。 若要解決此問題，您必須修復或重新改組雲端服務。  因為這可能會耗費大量時間，請詳細規劃。

- **Web/背景工作角色部署** - 無法將包含 Web 和背景工作角色的雲端服務移轉至 Azure Resource Manager。 開始移轉之前，必須先從虛擬網路移除 Web/背景工作角色。  典型的解決方案是只將 Web/背景工作角色執行個體移至也與 ExpressRoute 線路連結的不同傳統虛擬網路，或是將程式碼移轉至較新的 PaaS 應用程式服務 (此討論已超出本文件的範圍)。 在先前的重新部署案例中，建立新的傳統虛擬網路、將 Web/背景工作角色移動/重新部署至該新虛擬網路，然後從要移動的虛擬網路中刪除部署。 不需要變更程式碼。 新的[虛擬網路對等互連](../../virtual-network/virtual-network-peering-overview.md)功能可用來將包含 Web/背景工作角色的傳統虛擬網路與在相同 Azure 區域中的其他虛擬網路 (例如要移轉的虛擬網路) 形成對等互連 (**在虛擬網路移轉完成之後，因為不能移轉已形成對等互連的虛擬網路**)，因此可提供相同功能，而不會損失效能且沒有延遲/頻寬罰則。 由於新增了[虛擬網路對等互連](../../virtual-network/virtual-network-peering-overview.md)，現在可輕鬆地移轉Web/背景工作角色部署，而不會封鎖對 Azure Resource Manager 的移轉。

- **Azure Resource Manager 配額** - Azure 區域對於傳統和 Azure Resource Manager 有個別的配額/限制。 即使在未取用新硬體的移轉案例中 *(我們正在將現有的 VM 從傳統交換至 Azure Resource Manager)*，Azure Resource Manager 配額仍必須具有足夠的容量，才可開始進行移轉。 下列是我們發現會造成問題的主要限制。  開啟配額支援票證來提高限制。

    > [!NOTE]
    > 必須在與您要移轉的目前環境相同的區域中提高這些限制。
    >

    - 網路介面
    - 負載平衡器
    - 公用 IP
    - 靜態公用 IP
    - 核心
    - 網路安全性群組
    - 路由表

    您可以透過最新版本的 Azure PowerShell，使用下列命令來檢查您目前的 Azure Resource Manager 配額。

    **計算** *(核心，可用性設定組)*

    ```powershell
    Get-AzureRmVMUsage -Location <azure-region>
    ```

    **網路** *(虛擬網路、靜態公用 IP、公用 IP、網路安全性群組、網路介面、負載平衡器、路由表)*

    ```powershell
    Get-AzureRmUsage /subscriptions/<subscription-id>/providers/Microsoft.Network/locations/<azure-region> -ApiVersion 2016-03-30 | Format-Table
    ```

    **儲存體** *(儲存體帳戶)*

    ```powershell
    Get-AzureRmStorageUsage
    ```

- **Azure Resource Manager API 節流限制** - 如果您有一個夠大的環境 (例如， VNET 中 > 400 個 VM)，您在 Azure Resource Manager 中可能會達到寫入時預設的 API 節流限制 (目前為 `1200 writes/hour`)。 開始之前移轉，您應該提出支援票證來為您的訂用帳戶提高此限制。


- **佈建逾時 VM 狀態** - 如果任何 VM 的狀態為 `provisioning timed out`，這個問題必須在移轉前解決。 您只能利用停機時間解除佈建/重新佈 VM (刪除、保留磁碟，並重新建立 VM)。

- **RoleStateUnknown VM 狀態** - 如果因為 `role state unknown` 錯誤訊息使得移轉中止，請使用入口網站檢查 VM，並確定它正在執行中。 此錯誤一般會在幾分鐘之後自行消失 (不需補救)，並且是虛擬機器 `start`、`stop`、`restart` 作業期間經常會看到的暫時性類型。 **建議做法：** 幾分鐘後再重試移轉。

- **Fabric 叢集不存在** - 在某些情況下，某些 VM 由於各種奇怪的原因而無法移轉。 其中一種已知的情況為，如果是最近才建立 VM (在過去一個星期內左右)，然後在尚無法因應 Azure Resource Manager 工作負載的 Azure 叢集登陸時會發生此情況。  您會收到錯誤，指出 `fabric cluster does not exist` 而無法移轉 VM。 通常等候幾天就可解決此特定問題，因為叢集很快就會啟用 Azure Resource Manager。 不過，有一個立即的解決方法是對 VM `stop-deallocate`，然後再繼續進行移轉，並且在移轉之後，於 Azure Resource Manager 中啟動 VM 備份。

### <a name="pitfalls-to-avoid"></a>要避免的陷阱

- 請勿採取捷徑而略過驗證/準備/中止試執行移轉。
- 在驗證/準備/中止步驟期間，您可能發生的問題，幾乎全都會浮現。

## <a name="migration"></a>移轉

### <a name="technical-considerations-and-tradeoffs"></a>技術考量和取捨

因為您已瀏覽了環境的已知問題，現在您已經準備就緒。

對於實際的移轉，您可能要考慮：

1. 規劃及排程虛擬網路 (最小的移轉單位)，並逐漸增加優先順序。  先進行簡單的虛擬網路，然後進行較複雜的虛擬網路。
2. 大部分客戶會有非生產環境和生產環境。  最後才對生產環境進行排程。
3. **(選擇性)** 排程維護停機時間，並加上許多緩衝時間，以因應非預期的問題發生。
4. 萬一發生問題，請連絡並配合您的支援團隊。

### <a name="patterns-of-success"></a>成功的模式

應該考慮來自_實驗室測試_一節的技術指引，並在實際移轉之前改善狀況。  經過足夠測試後，實際上無需太擔心移轉作業。  對於生產環境而言，有類似受信任的 Microsoft 合作夥伴或 Microsoft Premier 服務等額外支援會有所幫助。

### <a name="pitfalls-to-avoid"></a>要避免的陷阱

未經過完整測試可能會造成移轉的問題與延遲。  

## <a name="beyond-migration"></a>移轉之外

### <a name="technical-considerations-and-tradeoffs"></a>技術考量和取捨

既然您已採用 Azure Resource Manager，請善加利用平台。  閱讀 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)，找出相關的其他優點。

需考量的事項：

- 將移轉與其他活動統合。  大部分客戶會選擇應用程式維護期間。  如果是的話，您可能要使用這個停機時間來啟用其他 Azure Resource Manager 功能，例如加密和移轉至受控磁碟。
- 再次瀏覽 Azure Resource Manager 的技術和商業理由；僅在 Azure Resource Manager 上啟用適用於您的環境的其他可用服務。
- 利用 PaaS 服務現代化您的環境。

### <a name="patterns-of-success"></a>成功的模式

現在請明確設定要在 Azure Resource Manager 中啟用的服務。  許多客戶發現下列各個項目對其 Azure 環境深具吸引力：

- [角色型存取控制](../../azure-resource-manager/resource-group-overview.md#access-control)。
- [Azure Resource Manager 範本使得部署更容易且更受控制](../../azure-resource-manager/resource-group-overview.md#template-deployment)。
- [標籤](../../azure-resource-manager/resource-group-using-tags.md)。
- [活動控制](../../azure-resource-manager/resource-group-audit.md)
- [Azure 原則](../../azure-policy/azure-policy-introduction.md)

### <a name="pitfalls-to-avoid"></a>要避免的陷阱

請記住您開始從傳統移轉至 Azure Resource Manager 的原因。  原始的商業理由為何？ 您是否達成商業理由？


## <a name="next-steps"></a>後續步驟

* [平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager) 的概觀](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [平台支援的從傳統移轉至 Azure Resource Manager 的技術深入探討](migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [使用 PowerShell 將 IaaS 資源從傳統移轉至 Azure Resource Manager](migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [使用 CLI 將 IaaS 資源從傳統移轉至 Azure Resource Manager](../linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [從傳統 VPN 閘道到 Resource Manager 的移轉](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-classic-resource-manager-migration)
* [將 ExpressRoute 線路和相關聯的虛擬網路從傳統部署模型遷移至 Resource Manager 部署模型](https://docs.microsoft.com/azure/expressroute/expressroute-migration-classic-resource-manager)
* [用於協助將 IaaS 資源從傳統移轉至 Azure Resource Manager 的社群工具](migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [檢閱最常見的移轉錯誤](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [檢閱有關將 IaaS 資源從傳統移轉至 Azure Resource Manager 的常見問題集](migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
