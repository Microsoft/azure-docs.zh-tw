---
title: 使用適用於開發人員的 Azure DevTest Labs | Microsoft Docs
description: 了解如何使用適用於開發人員案例的 Azure DevTest Labs。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 22e070e5-3d1a-49fe-9d4c-5e07cb0b7fe2
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 96432abe619ea23c1a06735567d00660e5430550
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38633276"
---
# <a name="use-azure-devtest-labs-for-developers"></a>使用適用於開發人員的 Azure DevTest Labs
Azure DevTest Labs 可以用來實作許多重要案例，但是其中一個主要案例是使用 DevTest Labs 來託管開發人員的開發電腦。 在此案例中，DevTest Labs 提供以下優點：

- 開發人員可以快速視需要佈建開發電腦。
- 開發人員可以隨時輕鬆自訂他們的開發電腦。
- 系統管理員可以確定下列事項以控制成本：
  - 開發人員無法取得超過開發所需的 VM。
  - VM 不使用時會關閉。 

![使用 DevTest Labs 進行訓練](./media/devtest-lab-developer-lab/devtest-lab-developer-lab.png)

在本文中，您會了解可用來符合開發人員需求的各種 Azure DevTest Labs 功能，以及可供遵循的實驗室詳細設定步驟。

## <a name="implementing-developer-environments-with-azure-devtest-labs"></a>實作使用 Azure DevTest Labs 的開發人員環境
1. **建立實驗室** 
   
    實驗室是 Azure DevTest Labs 的起點。 建立實驗室之後就可以執行各種工作，例如將使用者 (開發人員) 新增至實驗室、設定原則來控制成本，以及定義可以快速建立的 VM 映像等等。  
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [在 Azure 研測實驗室中建立實驗室](devtest-lab-create-lab.md) |了解如何在 Azure 入口網站的 Azure DevTest Labs 中建立實驗室。 |
2. **使用現成的 Marketplace 映像和自訂映像在幾分鐘內建立 VM** 
   
    您可以從 Azure Marketplace 的各種映像中挑選現成的映像，供實驗室使用。 如果現成映像不符合您的需求，您可以透過使用 Azure Marketplace 中的現成映像建立實驗室 VM、安裝所需的所有軟體，並將該 VM 儲存為實驗室中的自訂映像，以建立自訂映像。

    如果您要使用自訂映像，請考慮使用映像處理站建立和散發您的映像。 映像處理站是設定即程式碼的解決方案，定期自動建置及散發已設定的映像。 這可以在使用基礎作業系統建立 VM 之後，節省手動設定系統所需的時間。
  
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [設定 Azure Marketplace 映像](devtest-lab-configure-marketplace-images.md) |了解如何將 Azure Marketplace 映像加入白名單；只讓您想要提供給開發人員使用的映像可供選取。|
   | [建立自訂映像](devtest-lab-create-template.md) |預先安裝所需軟體以建立自訂映像，讓開發人員可以使用自訂映像快速建立 VM。|
   | [了解映像處理站](https://blogs.msdn.microsoft.com/devtestlab/2017/04/17/video-custom-image-factory-with-azure-devtest-labs/) |觀看如何設定及使用映像處理站的說明影片。|

3. **建立可供開發人員電腦重複使用的範本** 
   
    Azure DevTest Labs 中的公式是用來建立 VM 的預設屬性值清單。 您可以在實驗室中挑選映像、VM 大小 (CPU 和 RAM 的組合) 和虛擬網路以建立公式。 每位開發人員都能查看實驗室中的公式，並使用它來建立 VM。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [管理研發/測試實驗室公式來建立 VM](devtest-lab-manage-formulas.md) |了解如何挑選映像、VM 大小 (CPU 和 RAM 的組合) 和虛擬網路以建立公式。|

4. **建立構件以彈性自訂 VM**

   構件是在佈建 VM 之後用來部署和設定您的應用程式。 構件可以是：

   - 您想要在 VM 上安裝的工具 - 例如，代理程式、Fiddler 及 Visual Studio。
   - 您想要在 VM 上執行的動作 - 例如，複製儲存機制。
   - 您想要測試的應用程式。

   許多構件為現成可用。 如果想要符合特定需求的更多自訂，您可以建立自己的自訂構件。

   按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [為您的 DevTest Labs VM 建立自訂的構件](devtest-lab-artifact-author.md) |在實驗室中為虛擬機器建立自己的自訂構件。|
   | [在 Azure DevTest Labs 中新增可以存放自訂構件和 Azure Resource Manager 範本來使用的 Git 存放庫](devtest-lab-add-artifact-repo.md) |了解如何在您自己的私用 Git 儲存機制中儲存您的自訂構件。|

5. **控制成本**
   
    Azure DevTest Labs 可讓您在實驗室中設定原則，以指定開發人員可在實驗室中建立的 VM 數目上限。 
   
    如果您的開發人員小組有既定的工作時程，而您想要在一天中的某個特定時間停止所有的 VM，然後在隔天自動重新啟動，只要設定實驗室自動關閉和自動啟動原則即可輕鬆達到目的。 
   
    最後，當應用程式開發完成時，您可以執行單一 PowerShell 指令碼一次刪除所有 VM。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [定義實驗室原則](devtest-lab-set-lab-policy.md) |在實驗室中設定原則來控制成本。 |
   | [使用 PowerShell 指令碼刪除所有實驗室 VM](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) |當開發完成時，在一次作業中刪除所有實驗室。|

1. **將虛擬網路新增至 VM** 
   
    每當建立一個實驗室時，DevTest Labs 就會建立新的虛擬網路 (VNET)。 如果您已設定自己的 VNET – 例如，使用 ExpressRoute 或站對站 VPN – 您可以將此 VNET 新增至實驗室的虛擬網路設定，以便在建立 VM 時使用。

    此外，還有 Azure Active Directory 網域聯結構件可用，它會在建立 VM 時將 VM 加入網域。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [在 Azure DevTest Labs 中設定虛擬網路](devtest-lab-configure-vnet.md) |了解如何使用 Azure 入口網站在 Azure DevTest Labs 中設定虛擬網路。|

6. **與每位開發人員共用實驗室**
   
    使用您與開發人員共用的連結即可直接存取實驗室。 他們甚至不必有 Azure 帳戶，只要有 [Microsoft 帳戶](devtest-lab-faq.md#what-is-a-microsoft-account) 即可。 開發人員看不到其他開發人員建立的 VM。  
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [在 Azure DevTest Labs 中將開發人員新增至實驗室](devtest-lab-add-devtest-user.md) |使用 Azure 入口網站將開發人員新增至實驗室。|
   | [使用 PowerShell 指令碼將開發人員新增至實驗室](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell) |使用 PowerShell 自動將開發人員新增到實驗室。 |
   | [取得實驗室連結](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab) |了解開發人員如何透過超連結直接存取實驗室。|

7. **為更多小組自動建立實驗室** 
   
    您可以建立 Resource Manager 範本並一再使用它來建立相同的實驗室，以自動建立實驗室，包括自訂設定。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [使用 Resource Manager 範本建立實驗室](devtest-lab-faq.md#how-do-i-create-a-lab-from-a-resource-manager-template) |在 Azure DevTest Labs 中使用 Resource Manager 範本建立實驗室。 |

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

