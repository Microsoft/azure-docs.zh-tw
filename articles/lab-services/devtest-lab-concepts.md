---
title: 研發/測試實驗室概念 | Microsoft Docs
description: 了解研發/測試實驗室的基本概念，以及它如何讓您輕鬆地建立、管理和監視 Azure 虛擬機器
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 105919e8-3617-4ce3-a29f-a289fa608fb2
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2018
ms.author: spelluru
ms.openlocfilehash: ce0c51a3b97c57095a336128919c5e27f860905b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38676465"
---
# <a name="devtest-labs-concepts"></a>研發/測試實驗室概念
## <a name="overview"></a>概觀
下列清單包含重要的研發/測試實驗室概念和定義：

## <a name="labs"></a>實驗室
實驗室是包含一組資源 (例如虛擬機器 (VM)) 的基礎結構，可讓您藉由指定限制和配額更好地管理這些資源。

## <a name="virtual-machine"></a>虛擬機器
Azure VM 是由 Azure 所提供的[隨選且可調整的數種運算資源](https://docs.microsoft.com/azure/app-service/choose-web-site-cloud-service-vm)類型之一。 Azure VM 提供您虛擬化的彈性，而無需購買並維護執行它的實體硬體，不過您仍然需要藉由執行特定工作，例如設定、修補和安裝在其中執行的軟體來維護 VM。

[Azure 中的 Windows 虛擬機器概觀](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-overview)提供您在建立 VM 之前應該考慮的事項、建立方式及管理方式的相關資訊。

## <a name="claimable-vm"></a>可宣告 VM
Azure 可宣告 VM 是可供任何具有權限的實驗室使用者使用的虛擬機器。 實驗室系統管理員能以特定的基本映像和構件準備 VM，並將它們儲存至共用集區。 實驗室使用者接著可以在需要具有特定組態的 VM 時，從集區宣告具有該組態的運作中 VM。

可宣告的 VM 最初並不會指派給任何特定的使用者，但它會出現在所有使用者的 [可宣告的虛擬機器] 清單上。 當使用者宣告 VM 之後，該 VM 將會移至該使用者的 [我的虛擬機器] 區域，且其他使用者將無法宣告該 VM。

## <a name="environment"></a>環境
在研發/測試實驗室中，環境是指實驗室中 Azure 資源的集合。 [此部落格文章](https://blogs.msdn.microsoft.com/devtestlab/2016/11/16/connect-2016-news-for-azure-devtest-labs-azure-resource-manager-template-based-environments-vm-auto-shutdown-and-more/)討論如何從 Azure Resource Manager 範本建立多個 VM 環境。

## <a name="base-images"></a>基礎映像
基礎映像是 VM 映像，包含預先安裝並加以設定的所有工具與設定，可用來快速建立 VM。 您可以挑選現有的基本映像並加入構件來安裝測試代理程式，藉以佈建 VM。 您接著可以儲存佈建的 VM 做為基本映像，如此一來，不必針對 VM 的每個佈建重新安裝測試代理程式，就能使用該基本映像。

## <a name="artifacts"></a>構件
構件是在佈建 VM 之後用來部署和設定您的應用程式。 構件可以是：

* 您想要在 VM 上安裝的工具 - 例如，代理程式、Fiddler 及 Visual Studio。
* 您想要在 VM 上執行的動作 - 例如，複製儲存機制。
* 您想要測試的應用程式。

構件是 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) JSON 檔案，其中包含執行部署及套用組態的指示。

## <a name="artifact-repositories"></a>構件儲存機制
構件儲存機制是已簽入構件的 Git 儲存機制。 構件儲存機制可以加入組織中的多個實驗室，以便重複使用及共用。

## <a name="formulas"></a>公式
除了基底映像，公式提供快速 VM 佈建的機制。 研發/測試實驗室中的公式是用來建立實驗室 VM 的預設屬性值清單。
透過公式，可以建立具備相同屬性集 - 例如基底映像、VM 大小、虛擬網路以及成品 - 的 VM，而不需要每次都指定這些屬性。 透過公式建立 VM 時，可以依現況使用預設值，或修改預設值。

## <a name="policies"></a>原則
原則可協助控制實驗室中的成本。 例如，您可以建立一個原則，根據定義的排程自動關閉 VM。

## <a name="caps"></a>最高限度
最高限度是可將您實驗室中的成本浪費降至最低的機制。 例如，您可以設定一個最高限度，來限制每位使用者或每個實驗室中可建立的 VM 數目。

## <a name="security-levels"></a>安全性層級
安全性存取權是由 Azure 角色型存取控制 (RBAC) 所決定。 若要了解存取權的運作方式，了解 RBAC 所定義的權限、角色和範圍之間的差異將有所幫助。

* 權限 - 權限是針對特定動作所定義的存取權 (例如，針對所有虛擬機器的讀取權限)。
* 角色 - 角色是一組可以分組並指派給使用者的權限。 例如，「訂用帳戶擁有者」  角色擁有訂用帳戶內所有資源的存取權。
* 範圍 - 範圍是 Azure 資源階層的其中一個層級，例如，資源群組、單一實驗室或整個訂用帳戶。

在 DevTest Labs 的範圍內有兩種可用來定義使用者權限的角色類型︰實驗室擁有者和實驗室使用者。

* 實驗室擁有者 - 實驗室擁有者擁有實驗室內任何資源的存取權。 因此，實驗室擁有者可以修改原則、讀取和寫入任何 VM、變更虛擬網路等等。
* 實驗室使用者 - 實驗室使用者可以檢視所有實驗室資源，例如 VM、原則和虛擬網路，但不能修改原則或任何由其他使用者所建立的 VM。

若要查看如何在 DevTest Labs 建立自訂角色，請參閱 [將特定實驗室原則的權限授與使用者](devtest-lab-grant-user-permissions-to-specific-lab-policies.md)一文。

由於範圍是階層形式，當使用者擁有特定範圍的權限時，就會自動獲得該範圍所包含的每個較低層級範圍的權限。 例如，如果有使用者指派給訂用帳戶擁有者角色，他們就可以存取訂用帳戶中的所有資源，其中包括所有虛擬機器、所有虛擬網路和所有實驗室。 因此，訂用帳戶擁有者會自動繼承實驗室擁有者角色。 不過，若情形顛倒過來就不成立。 實驗室擁有者可存取實驗室，而實驗室是比訂用帳戶層級還低的範圍。 因此，實驗室擁有者將無法看到實驗室以外的虛擬機器、虛擬網路或任何資源。

## <a name="azure-resource-manager-templates"></a>Azure 資源管理員範本
此文章中討論的所有概念都可以使用 Azure Resource Manager 範本來設定，範本可以讓您定義 Azure 解決方案的基礎結構/設定，並重複以一致的狀態來部署。

[了解 Azure Resource Manager 範本的結構和語法](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates#template-format)描述 Azure Resource Manager 範本的結構，以及範本的各區段中可用的屬性。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
[在研測實驗室中建立實驗室](devtest-lab-create-lab.md)
