---
title: Azure DevTest Labs 常見問題集 | Microsoft Docs
description: 尋找 Azure DevTest Labs 常見問題的解答。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: afe83109-b89f-4f18-bddd-b8b4a30f11b4
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: a295cad2bf1cafce4dc64909174e9417daa7918e
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38235443"
---
# <a name="azure-devtest-labs-faq"></a>Azure DevTest Labs 常見問題集
獲得一些關於 Azure DevTest Labs 最常見問題的解答。

**一般**
## <a name="what-if-my-question-isnt-answered-here"></a>如果這裡沒有解答我的問題該怎麼辦？
如果這裡未列出您的問題，請告訴我們，好讓我們能協助您找到答案。

* 將問題張貼在此常見問題集尾端。 與 Azure 快取小組和其他社群成員一起討論本文。
* 若要觸及更多讀者，可將問題張貼在 [Azure DevTest Labs MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureDevTestLabs)。 與 Azure DevTest Labs 小組和社群的其他成員交流。
* 若要提出功能要求，請將您的要求和想法提交到 [Azure DevTest Labs 使用者心聲](https://feedback.azure.com/forums/320373-azure-devtest-labs)。

## <a name="why-should-i-use-azure-devtest-labs"></a>為何應使用 Azure DevTest Labs？
Azure DevTest Labs 可讓您的小組節省時間和金錢。 開發人員可以使用數種不同的基底物件建立自己的環境。 他們也可以使用構件快速部署和設定應用程式。 透過使用自訂映像和公式，您可以將虛擬機器 (VM) 儲存為範本，並輕易地在小組中予以重現。 DevTest Labs 還提供數個可設定的原則，以供實驗室系統管理員用來減少浪費並管理小組的環境。 這些原則包括自動關機、成本臨界值、每一使用者的 VM 數上限和 VM 大小上限。 如需更深入的 DevTest Labs 說明，請參閱[概觀](devtest-lab-overview.md)或觀看[簡介影片](https://channel9.msdn.com/Blogs/Azure/what-is-azure-devtest-labs)。

## <a name="what-does-worry-free-self-service-mean"></a>「零煩惱的自助方式」是什麼意思？
零煩惱的自助方式表示開發人員和測試人員可以視需要建立自己的環境。 系統管理員可獲得保障，知道 DevTest Labs 可以協助他們儘可能減少浪費和管控成本。 系統管理員可以指定允許的 VM 大小、VM 數目上限，以及何時啟動和關閉 VM。 DevTest Labs 也可讓成本監視和警示設定動作變得簡單，以便協助您隨時注意實驗室資源的使用情形。

## <a name="how-can-i-use-devtest-labs"></a>如何使用 DevTest Labs？
每當您需要開發或測試環境，而且想要加以快速重現或以節省成本的原則進行管理時，DevTest Labs 便能派上用場。

以下是我們的客戶使用 DevTest Labs 的一些案例︰

* 集中在一個地方管理開發和測試環境。 使用原則來降低成本，並建立自訂映像讓整個小組共用建置。
* 使用自訂映像開發應用程式，以儲存整個開發階段的磁碟狀態。
* 依進度追蹤成本。
* 建立大量測試環境以進行品質保證測試。
* 使用構件和公式在各種環境輕鬆地設定和重現應用程式。
* 散發 VM 以進行黑客松 (Hackathon，共同作業的開發或測試工作)，然後在活動結束時輕鬆地解除佈建。

## <a name="how-am-i-billed-for-devtest-labs"></a>DevTest Labs 如何收費？
DevTest Labs 是免費的服務。 在 DevTest Labs 建立實驗室和設定原則、範本與構件都是免費的。 您只需要針對在實驗室內使用到的 Azure 資源 (例如 VM、儲存體帳戶和虛擬網路) 支付費用。 如需實驗室資源成本的詳細資訊，請參閱 [Azure DevTest Labs 定價](https://azure.microsoft.com/pricing/details/devtest-lab/)。


**Security**
## <a name="what-are-the-different-security-levels-in-devtest-labs"></a>DevTest Labs 中有哪些不同的安全性層級？
安全性存取權是由[角色型存取控制 (RBAC)](../role-based-access-control/built-in-roles.md) 所決定。 若要了解存取權的運作方式，了解 RBAC 所定義的權限、角色和範圍之間的差異將有所幫助。

* **權限**：權限是針對特定動作所定義的存取權。 例如，權限可以是對所有 VM 的讀取權限。
* **角色**：角色是一組可以分組並指派給使用者的權限。 例如，具有訂用帳戶擁有者角色的使用者可存取訂用帳戶內的所有資源。
* **範圍**：範圍是 Azure 資源階層中的層級。 例如，範圍可以是一個資源群組、單一實驗室或整個訂用帳戶。

在 DevTest Labs 的範圍內有兩種角色類型可以定義使用者的權限︰

* **實驗室擁有者**：實驗室擁有者擁有實驗室內所有資源的存取權。 實驗室擁有者可以修改原則、讀取和寫入任何 VM、變更虛擬網路等等。
* **實驗室使用者**：實驗室使用者可以檢視所有實驗室資源，例如 VM、原則和虛擬網路。 但是，實驗室使用者不能修改原則或任何由其他使用者所建立的 VM。 

您也可以在 DevTest Labs 中建立自訂角色。 若要了解如何在 DevTest Labs 建立自訂角色，請參閱[將特定實驗室原則的權限授與使用者](devtest-lab-grant-user-permissions-to-specific-lab-policies.md)。

因為範圍是階層形式，當使用者擁有特定範圍的權限時，就會自動獲得該範圍的每個較低層級範圍的權限。 例如，如果有使用者指派給訂用帳戶擁有者角色，該使用者就可以存取訂用帳戶中的所有資源。 這些資源包括所有 VM、所有虛擬網路和所有實驗室。 訂用帳戶擁有者會自動繼承實驗室擁有者角色。 不過，若情形顛倒過來就不成立。 實驗室擁有者可存取實驗室，而實驗室是比訂用帳戶層級還低的範圍。 因此，實驗室擁有者看不到該實驗室之外的 VM、虛擬網路或任何其他資源。

## <a name="how-do-i-create-a-role-to-allow-users-to-perform-a-specific-task"></a>如何建立可讓使用者執行特定工作的角色？
如需如何建立自訂角色並指派權限給該角色的完整文章，請參閱[將特定實驗室原則的權限授與使用者](devtest-lab-grant-user-permissions-to-specific-lab-policies.md)。 以下指令碼範例會建立「DevTest Labs 進階使用者」角色，其具有啟動和停止實驗室中所有 VM 的權限︰

    $policyRoleDef = Get-AzureRmRoleDefinition "DevTest Labs User"
    $policyRoleDef.Actions.Remove('Microsoft.DevTestLab/Environments/*')
    $policyRoleDef.Id = $null
    $policyRoleDef.Name = "DevTest Labs Advanced User"
    $policyRoleDef.IsCustom = $true
    $policyRoleDef.AssignableScopes.Clear()
    $policyRoleDef.AssignableScopes.Add("subscriptions/<subscription Id>")
    $policyRoleDef.Actions.Add("Microsoft.DevTestLab/labs/virtualMachines/Start/action")
    $policyRoleDef.Actions.Add("Microsoft.DevTestLab/labs/virtualMachines/Stop/action")
    $policyRoleDef = New-AzureRmRoleDefinition -Role $policyRoleDef  


**CI/CD 整合與自動化**
## <a name="does-devtest-labs-integrate-with-my-cicd-toolchain"></a>DevTest Labs 是否會與我的 CI/CD 工具鏈整合？
如果您使用 Visual Studio Team Services，則可以使用 [DevTest Labs 工作擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks)在 DevTest Labs 中自動執行發行管線。 您可以使用此擴充功能執行的一些工作包括：

* 自動建立並部署 VM。 您也可以使用 Azure 檔案複製或 PowerShell Team Services 工作來以最新的組建設定 VM。
* 自動在測試之後擷取 VM 的狀態，以利在相同 VM 重現錯誤以便進行進一步的調查。
* 當您不再需要 VM 時，於發行管線結束時加以刪除。

下列部落格文章提供關於使用 Team Services 擴充功能的指引和資訊︰

* [DevTest Labs 和 Visual Studio Team Services 擴充功能](https://blogs.msdn.microsoft.com/devtestlab/2016/06/15/azure-devtest-labs-vsts-extension/)
* [透過 Team Services 在現有 DevTest Labs 實驗室中部署新的 VM](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS)
* [使用 Team Services 發行管理來持續部署至 DevTest Labs](http://www.visualstudiogeeks.com/blog/DevOps/Use-VSTS-ReleaseManagement-to-Deploy-and-Test-in-AzureDevTestLabs)

如需其他的持續整合 (CI)/持續傳遞 (CD) 工具鏈，可藉由使用 [Azure PowerShell Cmdlet](../azure-resource-manager/resource-group-template-deploy.md) 和 [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Management.DevTestLabs/) 部署 [Azure Resource Manager 範本](https://aka.ms/dtlquickstarttemplate) 來實現相同的案例。 您也可以使用[適用於 DevTest Labs 的 REST API](http://aka.ms/dtlrestapis) 來與您的工具鏈整合。  


**虛擬機器**
## <a name="why-cant-i-see-vms-on-the-virtual-machines-blade-that-i-see-in-devtest-labs"></a>為何無法在 [虛擬機器] 刀鋒視窗上看到我在 DevTest Labs 中看到的 VM？
當您在 DevTest Labs 中建立 VM 時，您會獲得該 VM 的存取權限。 在 [實驗室] 刀鋒視窗上以及 [虛擬機器] 刀鋒視窗上，您都可以檢視 VM。 指派到 DevTest Labs 實驗室使用者角色的使用者，可以在實驗室的 [所有虛擬機器] 刀鋒視窗上看到實驗室內所建立的所有 VM。 不過，具有 DevTest Labs 實驗室使用者角色的使用者不會自動獲得其他使用者所建立之 VM 資源的讀取權限。 因此，這些 VM 不會顯示在 [虛擬機器] 刀鋒視窗上。

## <a name="what-is-the-difference-between-a-custom-image-and-a-formula"></a>自訂映像和公式有何差異？
自訂映像是虛擬硬碟 (VHD)。 公式則是可以透過其他設定來加以設定，然後儲存並重現的映像。 如果您想要使用相同的不可變基底映像快速建立幾個環境，自訂映像可能較合適。 如果您想要使用最新版本來重現 VM 的組態、將 VM 的組態重現為虛擬網路或子網路的一部分，或是將 VM 的組態重現為特定大小的 VM，則公式可能較合適。 如需更深入的說明，請參閱[比較 DevTest Labs 中的自訂映像和公式](devtest-lab-comparing-vm-base-image-types.md)。

## <a name="how-do-i-create-multiple-vms-from-the-same-template-at-once"></a>如何從相同範本一次建立多個 VM？
有兩個選項可供您透過相同範本同時建立多個 VM：
* 您可以使用 [Visual Studio Team Services 工作擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks)。 
* 您可以在建立 VM 時[產生 Resource Manager 範本](devtest-lab-add-vm.md#save-azure-resource-manager-template)，然後[從 Windows PowerShell 部署 Resource Manager 範本](../azure-resource-manager/resource-group-template-deploy.md)。

## <a name="how-do-i-move-my-existing-azure-vms-into-my-devtest-labs-lab"></a>如何將現有 Azure VM 移到 DevTest Labs 實驗室？
若要將現有 VM 複製到 DevTest Labs：

1. 使用 [Windows PowerShell 指令碼](https://github.com/Azure/azure-devtestlab/blob/master/Scripts/CopyVHDFromVMToLab.ps1)來複製現有 VM 的 VHD 檔案。
2. 在 DevTest Labs 實驗室內[建立自訂映像](devtest-lab-create-template.md)。
3. 在實驗室中從自訂映像建立 VM。

## <a name="can-i-attach-multiple-disks-to-my-vms"></a>是否可以在 VM 連接多個磁碟？
是，您可以將多個磁碟連結至 VM。  

## <a name="if-i-want-to-use-a-windows-os-image-for-my-testing-do-i-have-to-purchase-an-msdn-subscription"></a>如果我想要使用 Windows 作業系統映像進行測試，是否應購買 MSDN 訂用帳戶？
若要對 Azure 中的開發或測試使用 Windows 用戶端 OS 映像 (Windows 7 或更新版本)，您必須執行下列其中一項動作：

- [購買 MSDN 訂閱](https://www.visualstudio.com/products/how-to-buy-vs)。
- 如果您有 Enterprise 合約，請以 [Enterprise 開發/測試供應項目](https://azure.microsoft.com/offers/ms-azr-0148p)建立 Azure 訂用帳戶。

如需每個 MSDN 供應項目之 Azure 點數的詳細資訊，請參閱 [Visual Studio 訂閱者的每月 Azure 點數](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)。

## <a name="how-do-i-automate-the-process-of-uploading-vhd-files-to-create-custom-images"></a>如何自動執行上傳 VHD 檔案的程序以建立自訂映像？
若要自動上傳 VHD 檔案以建立自訂映像，您有兩個選擇：

* 使用 [AzCopy](../storage/common/storage-use-azcopy.md#upload-blobs-to-blob-storage) 將 VHD 檔案複製或上傳到與實驗室相關聯的儲存體帳戶。
* 使用 [Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)。 儲存體總管是在 Windows、OSX 和 Linux 上執行的獨立應用程式。   

若要尋找與實驗室相關聯的目的地儲存體帳戶︰

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
2. 在左功能表上選取 [資源群組]。
3. 尋找並選取與您的實驗室相關聯的資源群組。
4. 在 [概觀] 底下，選取其中一個儲存體帳戶。
5. 選取 [Blob] 。
6. 在清單中尋找要上傳的項目。 如果不存在任何項目，請返回步驟 4，然後嘗試另一個儲存體帳戶。
7. 在 AzCopy 命令中使用 [URL] 作為目的地。

## <a name="how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab"></a>如何自動進行刪除實驗室中所有 VM 的程序？
您可以在 Azure 入口網站中從實驗室刪除 VM。 您也可以使用 PowerShell 指令碼刪除實驗室中的所有 VM。 在下列範例中，修改 **Values to change** 註解底下的參數值。 您可以從 Azure 入口網站中的 [實驗室] 窗格擷取 `subscriptionId`、`labResourceGroup` 及 `labName` 值。

    # Delete all the VMs in a lab.

    # Values to change:
    $subscriptionId = "<Enter Azure subscription ID here>"
    $labResourceGroup = "<Enter lab's resource group here>"
    $labName = "<Enter lab name here>"

    # Sign in to your Azure account.
    Connect-AzureRmAccount

    # Select the Azure subscription that has the lab. This step is optional
    # if you have only one subscription.
    Select-AzureRmSubscription -SubscriptionId $subscriptionId

    # Get the lab that has the VMs that you want to delete.
    $lab = Get-AzureRmResource -ResourceId ('subscriptions/' + $subscriptionId + '/resourceGroups/' + $labResourceGroup + '/providers/Microsoft.DevTestLab/labs/' + $labName)

    # Get the VMs from that lab.
    $labVMs = Get-AzureRmResource | Where-Object {
              $_.ResourceType -eq 'microsoft.devtestlab/labs/virtualmachines' -and
              $_.ResourceName -like "$($lab.ResourceName)/*"}

    # Delete the VMs.
    foreach($labVM in $labVMs)
    {
        Remove-AzureRmResource -ResourceId $labVM.ResourceId -Force
    }

**構件**
## <a name="what-are-artifacts"></a>何謂構件？
構件是可用來在 VM 中部署最新版本或開發工具的可自訂項目。 在建立 VM 時，請將構件連結至 VM。 VM 佈建好之後，構件就會部署並設定 VM。 您可以在[公用 GitHub 存放庫](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts)取得各種既存構件。 您也可以[撰寫您自己的構件](devtest-lab-artifact-author.md)。


**實驗室組態**
## <a name="how-do-i-create-a-lab-from-a-resource-manager-template"></a>如何從 Resource Manager 範本建立實驗室？
我們提供一個[實驗室 Azure Resource Manager 範本的 GitHub 存放庫](https://aka.ms/dtlquickstarttemplate)，您可以依原狀部署，或是加以修改來為您的實驗室建立自訂範本。 每個範本都包含一個連結，以供您在自己的 Azure 訂用帳戶中依原狀部署該實驗室。 或者，您可以自訂範本並[使用 PowerShell 或 Azure CLI 來進行部署](../azure-resource-manager/resource-group-template-deploy.md)。

## <a name="why-are-my-vms-created-in-different-resource-groups-with-arbitrary-names-can-i-rename-or-modify-these-resource-groups"></a>為何我的 VM 會建立在具有任意名稱的不同資源群組中？ 我是否可以重新命名或修改這些資源群組？
資源群組是以這種方式的建立，因此 DevTest Labs 可以管理 VM 的使用者權限和存取權。 雖然您可以將 VM 移至另一個資源群組，並使用您想要的名稱，但建議您不要變更資源群組。 我們正致力於改善這個體驗，以增加更多彈性。   

## <a name="how-many-labs-can-i-create-under-the-same-subscription"></a>在相同的訂用帳戶下可建立幾個實驗室？
每個訂用帳戶可以建立的實驗室數目並無特定限制， 但每個訂用帳戶可使用的資源數量則有限制。 您可以閱讀 [Azure 訂用帳戶的限制和配額](../azure-subscription-service-limits.md)和[如何增加這些限制值](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests)。

## <a name="how-many-vms-can-i-create-per-lab"></a>每個實驗室可以建立幾個 VM？
每個實驗室可以建立的 VM 數目並無特定限制， 不過，每個訂用帳戶可使用的資源 (VM 核心、公用 IP 位址等) 則有限制。 您可以閱讀 [Azure 訂用帳戶的限制和配額](../azure-subscription-service-limits.md)和[如何增加這些限制值](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests)。

## <a name="how-do-i-share-a-direct-link-to-my-lab"></a>如何共用我實驗室的直接連結？

1. 在 Azure 入口網站中，移到實驗室。
2. 從瀏覽器複製實驗室 URL，然後與您的實驗室使用者共用。

> [!NOTE]
> 實驗室使用者是具有 [Microsoft 帳戶](#what-is-a-microsoft-account)的外部使用者，但不屬於組織的 Active Directory 執行個體成員，該使用者在嘗試存取共用的連結時可能會看到錯誤訊息。 如果外部使用者看到錯誤訊息，請要求使用者先在 Azure 入口網站右上角選取其名稱。 然後，使用者就可以在功能表的 [目錄] 區段中，選取實驗室所在的目錄。
>
>

## <a name="what-is-a-microsoft-account"></a>什麼是 Microsoft 帳戶？
Microsoft 帳戶是您使用 Microsoft 裝置和服務來執行幾乎所有作業時所使用的帳戶。 此帳戶是可供用來登入 Skype、Outlook.com、OneDrive、Windows phone 和 Xbox Live 的電子郵件地址和密碼。 單一帳戶意味著不論您移到哪個裝置，檔案、相片、連絡人及設定都會跟著您移動。

> [!NOTE]
> Microsoft 帳戶以前稱為「Windows Live ID」。
>
>


**疑難排解**
## <a name="my-artifact-failed-during-vm-creation-how-do-i-troubleshoot-it"></a>在建立 VM 時，我的構件失敗了。 我該如何進行疑難排解？
若要了解如何取得失敗構件的記錄，請參閱[如何診斷 DevTest Labs 中的構件失敗](devtest-lab-troubleshoot-artifact-failure.md)。

## <a name="why-isnt-my-existing-virtual-network-saving-properly"></a>為何我現有的虛擬網路未能正確儲存？
其中一個可能原因是您的虛擬網路名稱包含句點。 若是如此，請嘗試移除句點，或將句點取代為連字號。 然後，再試著儲存虛擬網路一次。

## <a name="why-do-i-get-a-parent-resource-not-found-error-when-i-provision-a-vm-from-powershell"></a>為何我從 PowerShell 佈建 VM 時遇到「找不到父資源」錯誤？
當某資源為另一個資源的父資源時，父資源必須在建立子資源之前就存在。 如果父資源不存在，您會看到 **ParentResourceNotFound** 訊息。 如果您未在父資源上指定相依性，子資源可能會在父資源之前進行部署。

VM 是資源群組中實驗室下的子資源。 當您使用 Resource Manager 範本透過 PowerShell 部署 VM 時，在 PowerShell 指令碼中提供的資源群組名稱應該是實驗室的資源群組名稱。 如需詳細資訊，請參閱[對常見的 Azure 部署錯誤進行疑難排解](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-common-deployment-errors#parentresourcenotfound)。

## <a name="where-can-i-find-more-error-information-if-a-vm-deployment-fails"></a>當 VM 部署失敗時，我可以在哪裡找到更多錯誤資訊？
VM 部署錯誤會擷取至活動記錄中。 您可以在實驗室 VM 刀鋒視窗 (在您從 [我的虛擬機器] 清單中選取 VM 之後，該刀鋒視窗便會出現) 中資源功能表上的 [稽核記錄] 或 [虛擬機器診斷] 底下找到實驗室 VM 活動記錄。

系統有時候會在 VM 部署開始之前就發生部署錯誤。 舉例來說，如果您超過隨 VM 一起建立之資源的訂用帳戶限制時。 在此情況下，系統會將錯誤詳細資料擷取到實驗室層級的活動記錄中。 活動記錄位於 [組態和原則] 設定底部。 如需在 Azure 中使用活動記錄的詳細資訊，請參閱[檢視活動記錄以稽核對資源的動作](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit)。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]
