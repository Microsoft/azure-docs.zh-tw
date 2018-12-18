---
title: 使用複合資源在 Azure Automation State Configuration (DSC) 中撰寫 DSC 設定
description: 了解如何在 Azure Automation State Configuration (DSC) 中使用複合資源撰寫設定
keywords: powershell dsc, desired state configuration, powershell dsc azure, composite resources, 需要的狀態設定, 複合資源
services: automation
ms.service: automation
ms.component: dsc
author: DCtheGeek
ms.author: dacoulte
ms.date: 08/21/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: a1297b6cd84a012ff54f7d902c92687b2e284fe2
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45606898"
---
# <a name="composing-dsc-configurations-in-azure-automation-state-configuration-dsc-using-composite-resources"></a>使用複合資源在 Azure Automation State Configuration (DSC) 中撰寫 DSC 設定

要使用多個預期狀態設定 (DSC) 組態來管理資源時，最佳路徑是使用[複合資源](/powershell/dsc/authoringresourcecomposite)。 複合資源是巢狀且參數化的設定，用來在另一個設定中當作 DSC 資源。 這樣可建立複雜的設定，同時允許以個別方式管理及建置基礎複合資源 (參數化設定)。

Azure 自動化能夠[匯入和編譯複合資源](automation-dsc-compile.md#composite-resources)。 一旦複合資源匯入到自動化帳戶之後，您就可以使用 [State Configuration (DSC)] 頁面中的 [撰寫設定] 體驗。

## <a name="composing-a-configuration-from-composite-resources"></a>從複合資源撰寫設定

您必須先撰寫設定，才能在 Azure 入口網站中指派來自複合資源的設定。 您可以在 [設定] 或 [已編譯設定] 索引標籤上的 [State Configuration (DSC)] 頁面上使用 [撰寫設定]。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 依序按一下左側的 [所有資源] 和您的自動化帳戶名稱。
1. 在 [自動化帳戶] 頁面上，選取 [組態管理] 之下的 [State Configuration (DSC)]。
1. 在 [State Configuration (DSC)] 頁面上，按一下 [設定] 或 [已編譯設定] 索引標籤，然後按一下頁面頂端功能表中的 [撰寫設定]。
1. 在 [基本] 步驟中，提供新的設定名稱 (必要)，然後在您要納入新設定之每個複合資源列上的任意位置按一下，然後按一下 [下一步] 或按一下 [原始程式碼] 步驟。 針對後續步驟，我們選取了 **PSExecutionPolicy** 和 **RenameAndDomainJoin** 複合資源。
   ![撰寫設定頁面之基本步驟的螢幕擷取畫面](./media/compose-configurationwithcompositeresources/compose-configuration-basics.png)
1. **原始程式碼**步驟會顯示所選取之複合資源的已撰寫設定看起來會像怎樣。 您可以看到所有參數的合併，和它們如何傳遞給複合資源。 當您檢閱完新的原始程式碼之後，按一下 [下一步] 或按一下 [參數] 步驟。
   ![撰寫設定頁面之原始程式碼步驟的螢幕擷取畫面](./media/compose-configurationwithcompositeresources/compose-configuration-sourcecode.png)
1. 在 [參數] 步驟中，每個複合資源具有的參數都已公開，如此才能提供參數。 如果參數具有描述，則會顯示在參數欄位旁邊。 如果欄位是 **PSCredential** 型別參數，設定的下拉式清單會提供目前自動化帳戶中的 **Credential** 物件清單。 另外也可使用 [+ 新增認證] 選項。 一旦已提供所有必要參數，按一下 [儲存並編譯]。
   ![撰寫設定頁面之參數步驟的螢幕擷取畫面](./media/compose-configurationwithcompositeresources/compose-configuration-parameters.png)

新的設定儲存之後，便會提交以進行編譯。 就像任何已匯入的設定，您可以檢視編譯作業狀態。 如需詳細資訊，請參閱[檢視編譯作業](automation-dsc-getting-started.md#viewing-a-compilation-job)。

當編譯已順利完成時，新的設定會顯示在 [已編譯設定] 索引標籤中。一旦設定顯示在此索引標籤中，就可以使用[將節點重新指派至不同的節點組態](automation-dsc-getting-started.md#reassigning-a-node-to-a-different-node-configuration)中的步驟，將它指派給受控節點。

## <a name="next-steps"></a>後續步驟

- 若要開始使用，請參閱[開始使用 Azure 自動化狀態設定](automation-dsc-getting-started.md)
- 若要深入了解如何將節點上架，請參閱[將機器上架交由 Azure Automation State Configuration 管理](automation-dsc-onboarding.md)
- 若要了解如何編譯 DSC 組態，以將它們指派給目標節點，請參閱[編譯 Azure Automation State Configuration 中的組態](automation-dsc-compile.md)
- 如需 PowerShell Cmdlet 參考，請參閱 [Azure 自動化狀態設定 Cmdlet](/powershell/module/azurerm.automation/#automation)
- 如需定價資訊，請參閱 [Azure 自動化狀態設定的定價](https://azure.microsoft.com/pricing/details/automation/)
- 若要查看在持續部署管線中使用 Azure 自動化狀態設定的範例，請參閱[使用 Azure 自動化狀態設定和 Chocolatey 的持續部署](automation-dsc-cd-chocolatey.md)