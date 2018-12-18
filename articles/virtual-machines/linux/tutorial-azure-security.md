---
title: 教學課程 - 針對 Azure 中的 Linux VM 使用 Azure 資訊安全中心 | Microsoft Docs
description: 在本教學課程中，您會了解 Azure 資訊安全中心的功能，以協助保護 Azure 中的 Linux 虛擬機器。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/11/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 562fc267a056d6908af5b89fd7a93e858f1c6165
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47092606"
---
# <a name="tutorial-use-azure-security-center-to-monitor-linux-virtual-machines"></a>教學課程：使用 Azure 資訊安全中心來監視 Linux 虛擬機器

Azure 資訊安全中心可協助您了解 Azure 資源的安全性作法。 資訊安全中心提供了整合式的安全性監視功能。 它可以偵測到可能不會被察覺的威脅。 在本教學課程中，您將會了解 Azure 資訊安全中心，以及要如何︰
 
> [!div class="checklist"]
> * 設定資料收集功能
> * 設定安全性原則
> * 檢視及修正組態的健康狀態問題
> * 檢閱所偵測到的威脅

## <a name="security-center-overview"></a>資訊安全中心概觀

資訊安全中心會找出潛在的虛擬機器 (VM) 組態問題和針對性的安全性威脅。 這些項目可能包括缺少網路安全性群組的 VM、磁碟未加密，以及遠端桌面通訊協定 (RDP) 暴力破解攻擊。 此資訊會以容易看懂的圖表形式顯示在資訊安全中心儀表板上。

若要存取資訊安全中心儀表板，請在 Azure 入口網站的功能表上選取 [資訊安全中心]。 在儀表板上，您可以看到 Azure 環境的安全性健康狀態、找到目前建議項目的計數，以及檢視目前的威脅警示狀態。 展開每個高階圖表就能查看更多詳細資料。

![資訊安全中心儀表板](./media/tutorial-azure-security/asc-dash.png)

資訊安全中心不只能探索資料，它還會提供建議以讓您解決所偵測到的問題。 例如，如果 VM 在部署時未連結網路安全性群組，資訊安全中心便會顯示建議，並指出可供採取的補救步驟。 您不需要離開資訊安全中心便能自動補救。  

![建議](./media/tutorial-azure-security/recommendations.png)

## <a name="set-up-data-collection"></a>設定資料收集功能

您必須先設定資訊安全中心的資料收集功能，才能了解 VM 的安全性組態。 這包含開啟資料收集，以在訂用帳戶的所有 VM 上自動安裝 Microsoft Monitoring Agent。

1. 在資訊安全中心儀表板上，按一下 [安全性原則] 並選取您的訂用帳戶。 
2. 針對 [資料收集]，在 [自動佈建] 中，選取 [開啟]。
3. 針對 [預設工作區設定]，將它保留為 [使用資訊安全中心建立的工作區 (預設)]。
4. 在 [安全性事件] 下方，保留預設選項 [通用]。
4. 按一下頁面頂端的 [儲存]。 

系統隨即會在所有 VM 上安裝資訊安全中心的資料收集代理程式，並開始收集資料。 

## <a name="set-up-a-security-policy"></a>設定安全性原則

安全性原則可用來定義原則項目，讓資訊安全中心收集其資料並提出建議。 您可以對不同的 Azure 資源集合套用不同的安全性原則。 雖然系統預設會根據所有原則項目來評估 Azure 資源，但您可以針對所有 Azure 資源或某個資源群組來關閉個別的原則項目。 若要深入了解資訊安全中心的安全性原則，請參閱[在 Azure 資訊安全中心設定安全性原則](../../security-center/security-center-policies.md)。 

設定整個訂用帳戶的安全性原則：

1. 在 [資訊安全中心] 儀表板上，選取 [安全性原則]，然後選取您的訂用帳戶。
2. 在 [安全性原則] 刀鋒視窗上，選取 [安全性原則]。 
3. 在 [安全性原則 - 安全性原則] 刀鋒視窗上，開啟或關閉您要套用至訂用帳戶的原則項目。
4. 當您選好設定時，請選取刀鋒視窗頂端的 [儲存]。 

![唯一原則](./media/tutorial-azure-security/unique-policy.png)

## <a name="view-vm-configuration-health"></a>檢視 VM 組態健康狀態

在開啟資料收集功能並設定好安全性原則後，資訊安全中心會開始提供警示和建議。 VM 在部署時就已安裝好資料收集代理程式。 之後，系統就會在資訊安全中心內填入新 VM 的資料。 若要深入了解 VM 組態的健康狀態，請參閱[在資訊安全中心內保護您的 VM](../../security-center/security-center-virtual-machine-recommendations.md)。 

隨著資料的收集，系統會彙總每個 VM 和相關 Azure 資源的資源健康狀態。 此資訊會以容易看懂的圖表形式來顯示。 

若要檢視資源健康狀態︰

1.  在 [資訊安全中心] 儀表板的 [預防] 下方，選取 [計算]。 
2.  在 [計算] 刀鋒視窗上，選取 [VM 和電腦]。 此檢視會提供所有 VM 組態的狀態摘要。

![計算健康狀態](./media/tutorial-azure-security/compute-health.png)

若要查看某個 VM 的所有建議，請選取該 VM。 

## <a name="remediate-configuration-issues"></a>補救組態問題：

在資訊安全中心開始填入組態資料後，系統會根據您設定的安全性原則來提出建議。 例如，如果 VM 在設置時沒有相關聯的網路安全性群組，系統會建議您建立一個安全性群組。 

若要查看所有建議項目的清單︰ 

1. 在資訊安全中心儀表板上，選取 [建議]。
2. 選取特定建議。 適用該項建議的所有資源清單隨即會出現。
3. 若要套用建議，請選取資源。 
4. 遵循補救步驟的指示來進行。 

在許多情況下，資訊安全中心會提供可行步驟，供您執行建議卻又無須離開資訊安全中心。 在下列範例中，資訊安全中心會偵測到輸入規則未受限制的網路安全性群組。 在建議頁面中，您可以選取 [編輯輸入規則] 按鈕。 用以修改規則的 UI 會隨即出現。 

![建議](./media/tutorial-azure-security/remediation.png)

建議在執行補救後會標示為已解決。 

## <a name="view-detected-threats"></a>檢視偵測到的威脅

除了資源組態建議外，資訊安全中心也會提供威脅偵測警示。 安全性警示功能會彙總從每個 VM、Azure 網路記錄和連線合作夥伴解決方案所收集到的資料，以偵測不利於 Azure 資源的安全性威脅。 若要深入了解資訊安全中心的威脅偵測功能，請參閱 [Azure 資訊安全中心的偵測功能](../../security-center/security-center-detection-capabilities.md)。

若要使用安全性警示功能，須將資訊安全中心的定價層從「免費」提升為「標準」。 當您改用這個較高的定價層時，會有 60 天的**免費試用**。 

若要變更定價層：  

1. 在資訊安全中心儀表板上，按一下 [安全性原則] 並選取您的訂用帳戶。
2. 選取 [定價層]。
3. 選取 [標準]，然後按一下刀鋒視窗頂端的 [儲存]。


在變更定價層後，一旦系統偵測到安全性威脅，安全性警示圖表就會開始填入資料。

![安全性警示](./media/tutorial-azure-security/security-alerts.png)

選取警示以檢視資訊。 例如，您可以看到威脅、偵測時間、所有威脅嘗試和建議補救步驟等項目的描述。 在下列範例中，系統會偵測到 RDP 暴力破解攻擊，且這項 RDP 攻擊嘗試已失敗 294 次。 資訊安全中心會提供建議的解決方案。

![RDP 攻擊](./media/tutorial-azure-security/rdp-attack.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已設定 Azure 資訊安全中心，然後在資訊安全中心檢閱了 VM。 您已了解如何︰

> [!div class="checklist"]
> * 設定資料收集功能
> * 設定安全性原則
> * 檢視及修正組態的健康狀態問題
> * 檢閱所偵測到的威脅

前進到下一個教學課程，深入了解如何利用 Jenkins、GitHub、Docker 建立 CI/CD 管線。

> [!div class="nextstepaction"]
> [以 Jenkins、GitHub 及 Docker 建立 CI/CD 基礎結構](tutorial-jenkins-github-docker-cicd.md)

