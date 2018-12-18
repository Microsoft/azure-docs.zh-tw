---
title: Azure 合作夥伴與客戶使用狀況屬性
description: 有關如何追蹤客戶 Azure Marketplace 解決方案使用狀況的概觀
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
documentationcenter: ''
author: yijenj
manager: nunoc
editor: ''
ms.assetid: e8d228c8-f9e8-4a80-9319-7b94d41c43a6
ms.service: marketplace
ms.workload: ''
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 10/05/2018
ms.author: yijenj
ms.openlocfilehash: 99df133b9f626f970189df578c6d107086b9dab9
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854995"
---
# <a name="azure-partner-customer-usage-attribution"></a>Azure 合作夥伴客戶使用狀況屬性

身為 Azure 的軟體合作夥伴，您的解決方案需要 Azure 元件或必須直接部署在 Azure 基礎結構上。 部署合作夥伴解決方案並佈建其自有 Azure 資源的客戶會發現深入了解部署狀態以及對 Azure 成長的影響並不容易。 獲得概略的部署狀況資訊之後，您就可獲得與 Microsoft 銷售團隊相同的資訊，並獲得 Microsoft 合作夥伴計劃的信用額度。   

Microsoft 現在提供一個模型，協助合作夥伴以更好的方式追蹤客戶在 Azure 上部署之軟體的 Azure 使用狀況。 這個新方法使用 Azure Resource Manager 來協調 Azure 服務的部署。

身為 Microsoft 合作夥伴，您可以將 Azure 使用狀況與您代表客戶佈建的任何 Azure 資源相關聯。 您可以透過 Azure Marketplace、快速入門存放庫、私人 GitHub 存放庫與一對一客戶支援來構成此關聯關係。 若要啟用追蹤，有兩種方法可用：

- Azure Resource Manager 範本：Resource Manager 範本或解決方案範本可部署 Azure 服務以執行合作夥伴的軟體。 合作夥伴可建立 Resource Manager 範本以定義其 Azure 解決方案的基礎結構與設定。 Resource Manager 範本可讓您與您的客戶在其生命週期中部署您的解決方案。 您可以確信您的資源會以一致的狀態部署。 

- Azure Resource Manager API：合作夥伴可直接呼叫 Resource Manager API，來部署 Resource Manager 範本或產生 API 呼叫以直接佈建 Azure 服務。 

## <a name="use-resource-manager-templates"></a>使用 Resource Manager 範本

許多合作夥伴解決方案都使用 Resource Manager 範本來部署在客戶的訂用帳戶上。 如果您在 Azure Marketplace 中、GitHub 上或快速入門中，已有可用的 Resource Manager 範本，修改範本以啟用這個新追蹤方法的程序應該相當簡單。 如果您未使用 Azure Resource Manager 範本，現在這裡有一些連結，可協助您更了解 Resource Manager 範本與其建立方式： 

*   [建立及部署第一個 Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-create-first-template)
*   [建立 Azure Marketplace 的解決方案範本](https://docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-solution-template-creation)

## <a name="add-a-guid-to-your-template"></a>將 GUID 新增到您的範本

若要新增全域唯一識別碼 (GUID)，您只需要在主要範本檔案中進行一處修改：

1. 建立 GUID (例如 eb7927c8-dd66-43e1-b0cf-c346a422063)。

1. 開啟 Resource Manager 範本。

1. 在主要範本檔案中加入新的資源。 資源只需要置於 **mainTemplate.json** 或 **azuredeploy.json** 檔案中，而非任何巢狀或連結的範本中。

1. 在 **pid-** 前置詞後方輸入 GUID 值 (例如 pid-eb7927c8-dd66-43e1-b0cf-c346a422063)。

1. 檢查範本是否有任何錯誤。

1. 在適當的存放庫中重新發佈範本。

### <a name="sample-template-code"></a>範例範本程式碼

![範例範本程式碼](media/marketplace-publishers-guide/tracking-sample-code-for-lu-1.PNG)

## <a name="use-the-resource-manager-apis"></a>使用 Resource Manager API

在某些情況下，您可能會想要直接對 Resource Manager REST API 進行呼叫，以部署 Azure 服務。 [Azure 支援多個 SDK](https://docs.microsoft.com/azure/#pivot=sdkstools) 以啟用此作業。 您可以使用其中一個 SDK，或直接呼叫 REST API 來部署資源。

如果您使用 Resource Manager 範本，應該依照稍早的指示標記您的解決方案。 如果您不是使用 Resource Manager 範本，並進行直接 API 呼叫，仍可標記您的部署以與 Azure 資源的使用量建立關聯。 

### <a name="tag-a-deployment-with-the-resource-manager-apis"></a>使用 Resource Manager API 標記部署

針對此追蹤方法，當您設計您的 API 呼叫時，請在要求中的使用者代理程式標頭中包括 GUID。 針對每個供應項目或 SKU 新增 GUID。 使用 **pid-** 前置詞設定字串格式，並包括合作夥伴產生的 GUID。 以下是可插入到使用者代理程式中的 GUID 格式範例： 

![範例 GUID 格式](media/marketplace-publishers-guide/tracking-sample-guid-for-lu-2.PNG)

> [!Note]
> 此字串的格式至關重要。 若未包括 **pid-** 前置詞，就無法查詢資料。 不同的 SDK 會以不同的方式追蹤。 若要實作此方法，請檢閱支援和適用於慣用 Azure SDK 的方法。 

### <a name="example-the-python-sdk"></a>範例：Python SDK

針對 Python，請使用 **config** 屬性。 您只能將屬性新增到 UserAgent。 以下是範例：

![將屬性新增到使用者代理程式](media/marketplace-publishers-guide/python-for-lu.PNG)

> [!Note]
> 為每個用戶端新增屬性。 沒有全域靜態設定。 您可以標記用戶端處理站，以確定每個用戶端都在正在追蹤。 如需詳細資訊，請參閱 [GitHub 上的此用戶端處理站範例](https://github.com/Azure/azure-cli/blob/7402fb2c20be2cdbcaa7bdb2eeb72b7461fbcc30/src/azure-cli-core/azure/cli/core/commands/client_factory.py#L70-L79)。

#### <a name="tag-a-deployment-by-using-the-azure-powershell"></a>使用 Azure PowerShell 標記部署

若您透過 Azure PowerShell 部署資源，請使用下列方法附加您的 GUID：

```
[Microsoft.Azure.Common.Authentication.AzureSession]::ClientFactory.AddUserAgent("pid-eb7927c8-dd66-43e1-b0cf-c346a422063")
```

#### <a name="tag-a-deployment-by-using-the-azure-cli"></a>使用 Azure CLI 標記部署

當您使用 Azure CLI 來附加您的 GUID 時，請設定 **AZURE_HTTP_USER_AGENT** 環境變數。 您可以在指令碼的範圍內設定此變數。 您也可以針對殼層範圍設定全域變數：

```
export AZURE_HTTP_USER_AGENT='pid-eb7927c8-dd66-43e1-b0cf-c346a422063'
```

## <a name="register-guids-and-offers"></a>註冊 GUID 與供應項目

若要在我們的追蹤中包括 GUID，必須註冊 GUID。  

範本 GUID 的所有註冊都會透過 Azure Marketplace 雲端合作夥伴入口網站 (CPP) 來完成。 

在您將 GUID 加入至範本或使用者代理程式中，並在 CPP 中註冊該 GUID 之後，便會追蹤所有部署。 

1. 適用於 [Azure Marketplace](http://aka.ms/listonazuremarketplace)，並可取得 CPP 的存取權。

   * 合作夥伴必須[在 CPP 中擁有設定檔](https://docs.microsoft.com/azure/marketplace/become-publisher)。 我們鼓勵您在 Azure Marketplace 或 AppSource 中列出該供應項目。
   * 合作夥伴可以註冊多個 GUID。
   * 合作夥伴可以註冊為非 Marketplace 解決方案範本與供應項目註冊 GUID。
 
1. 登入 [Cloud Partner 入口網站](https://cloudpartner.azure.com/)。

1. 在右上角，選取您的帳戶圖示，然後選取 [發行者設定檔]。

   ![選取發行者設定檔](media/marketplace-publishers-guide/guid-image-for-lu.png)

1. 在 [設定檔] 頁面上，選取 [新增追蹤 GUID]。

   ![選取 [新增追蹤 GUID]](media/marketplace-publishers-guide/guid-how-to-add-tracking.png)

1. 在 [追蹤 GUID] 方塊中，輸入您的追蹤 GUID。 只輸入 GUID，不要輸入 **pid-** 前置詞。 在 [自訂描述] 方塊中，輸入您的供應項目名稱或描述。

   ![[設定檔] 頁面](media/marketplace-publishers-guide/guid-dev-center-login.png)
   
   ![輸入 GUID 與供應項目描述](media/marketplace-publishers-guide/guid-dev-center-example.png)

1. 若要註冊多個 GUID，請再次選取 [新增追蹤 GUID]。 頁面上會出現額外的方塊。

   ![再次選取 [新增追蹤 GUID]](media/marketplace-publishers-guide/guid-dev-center-example-add.png)
   
   ![輸入另一個 GUID 與供應項目描述](media/marketplace-publishers-guide/guid-dev-center-example-description.png)

1. 選取 [ **儲存**]。

   ![選取 [儲存]](media/marketplace-publishers-guide/guid-dev-center-save.png)

在您將 GUID 加入至範本或使用者代理程式中，並在 CPP 中註冊該 GUID 之後，便會追蹤所有部署。 

## <a name="verify-the-guid-deployment"></a>驗證 GUID 部署 

在您修改範本並執行測試部署之後，您可以使用下列 PowerShell 指令碼來擷取您已部署並標記的資源。 

您可以使用指令碼來確認 GUID 是否已成功加入至您的 Resource Manager 範本。 指令碼不適用於 Resource Manager API 部署。

登入 Azure。 選取具有您想要在執行指令碼之前驗證之部署的訂用帳戶。 在部署的訂用帳戶內容中執行指令碼。

部署的 **GUID** 與 **resourceGroup** 名稱是必要參數。

您可以在 GitHub 上取得[原始指令碼](https://gist.github.com/bmoore-msft/ae6b8226311014d6e7177c5127c7eba1#file-verify-deploymentguid-ps1)。

```
Param(
    [GUID][Parameter(Mandatory=$true)]$guid,
    [string][Parameter(Mandatory=$true)]$resourceGroupName
)

# Get the correlationId of the pid deployment

$correlationId = (Get-AzureRmResourceGroupDeployment -ResourceGroupName 
$resourceGroupName -Name "pid-$guid").correlationId

# Find all deployments with that correlationId

$deployments = Get-AzureRmResourceGroupDeployment -ResourceGroupName $resourceGroupName | Where-Object{$_.correlationId -eq $correlationId}

# Find all deploymentOperations in a deployment by name
# PowerShell doesn't surface outputResources on the deployment
# or correlationId on the deploymentOperation

foreach ($deployment in $deployments){

# Get deploymentOperations by deploymentName
# then the resourceId for any create operation

($deployment | Get-AzureRmResourceGroupDeploymentOperation | Where-Object{$_.properties.provisioningOperation -eq "Create" -and $_.properties.targetResource.resourceType -ne "Microsoft.Resources/deployments"}).properties.targetResource.id

}
```

## <a name="create-guids"></a>建立 GUID

GUID 是具有 32 個十六進位數字的參考號碼。 若要建立 GUID 來進行追蹤，您應該使用 GUID 產生器。 有多個[線上 GUID 產生器](https://www.bing.com/search?q=guid%20generator&qs=n&form=QBRE&sp=-1&ghc=2&pq=guid%20g&sc=8-6&sk=&cvid=0BAFAFCD70B34E4296BB97FBFA3E1B4E)可以使用。

為每個供應項目與散發通道都建立唯一的 GUID。 若您已使用範本部署兩個解決方案，且每個解決方案都可以在 Azure Marketplace 與 GitHub 上取得，您必須建立四個 GUID：

*   Azure Marketplace 中的供應項目 A 
*   GitHub 上的供應項目 A
*   Azure Marketplace 中的供應項目 B 
*   GitHub 上的供應項目 B

回報是由合作夥伴值 (Microsoft 合作夥伴 ID) 與 GUID 所完成。 

您也能以更細微的層級來追蹤 GUID (例如，供應項目的變體 SKU)。

## <a name="notify-your-customers"></a>通知您的客戶

合作夥伴應該通知其客戶有關使用 Resource Manager GUID 追蹤的部署。 Microsoft 會向合作夥伴回報與這些部署關聯的 Azure 使用狀況。 下列範例包括您可以用來通知客戶有關這些部署的內容。 在範例中，將 \<PARTNER> 取代為您的公司名稱。 合作夥伴應該確保通知符合他們的資料隱私權和收集原則，包括讓客戶從追蹤排除的選項。 

### <a name="notification-for-resource-manager-template-deployments"></a>Resource Manager 範本部署的通知

當您部署此範本時，Microsoft 可以透過已部署的 Azure 資源來識別 \<PARTNER> 軟體的安裝。 Microsoft 可讓用來支援軟體的 Azure 資源相互關聯。 Microsoft 會收集這項資訊，以透過其產品提供最佳體驗以及經營業務。 Microsoft 隱私權原則 (位於 https://www.microsoft.com/trustcenter) 規範此資料的收集與控管。 

### <a name="notification-for-sdk-or-api-deployments"></a>SDK 或 API 部署的通知

當您部署 \<PARTNER> 軟體時，Microsoft 可以透過已部署的 Azure 資源來識別 \<PARTNER> 軟體的安裝。 Microsoft 可讓用來支援軟體的 Azure 資源相互關聯。 Microsoft 會收集這項資訊，以透過其產品提供最佳體驗以及經營業務。 Microsoft 隱私權原則 (位於 https://www.microsoft.com/trustcenter) 規範此資料的收集與控管。

## <a name="get-support"></a>取得支援

若您需要協助，請依照這些步驟執行。

1. 移至[支援連結](https://go.microsoft.com/fwlink/?linkid=844975)。 

1. 在 [問題類型] 下，選取 [Marketplace 上線]。

1. 針對您的問題選擇 [類別]：

   - 針對使用狀況關聯問題，請選取 [其他]。
   - 針對 Azure Marketplace CPP 存取問題，請選取 [存取問題]。
   
    ![選擇問題類別](media/marketplace-publishers-guide/lu-article-incident.png)

1. 選取 [提出要求]。

1. 在下一頁，輸入必要值。 選取 [繼續]。

1. 在下一頁，輸入必要值。

   > [!Important] 
   > 在 [事件標題] 方塊中，輸入 **ISV 使用狀況追蹤**。 請詳細描述您的問題。
   
   ![輸入事件標題的 ISV 使用狀況追蹤](media/marketplace-publishers-guide/guid-dev-center-help-hd%201.png)

1. 完成表單，然後選取 [提交]。

## <a name="faq"></a>常見問題集

**將 GUID 加入至範本的優點為何？**

Microsoft 為合作夥伴提供一個客戶部署範本檢視和受影響的使用量見解。 Microsoft 與合作夥伴也可以使用此資訊來加強與銷售團隊之間的合作關係。 Microsoft 與合作夥伴可以使用該資料來取得個別合作夥伴對 Azure 成長的更一致檢視。 

**何人能將 GUID 加入至範本？**

追蹤資源的主要目的是將合作夥伴的解決方案連結到客戶的 Azure 使用量。 使用量資料會繫結到合作夥伴的 Microsoft 合作夥伴網路身分識別 (MPN ID)。 合作夥伴可以在 CPP 中取得回報資料。

**加入 GUID 之後能否予以變更？**
 
是，客戶或實作合作夥伴可自訂範本並能變更或移除該 GUID。 建議合作夥伴主動向其客戶與合作夥伴，描述資源角色與 GUID，以防止移除或編輯該追蹤 GUID。 變更 GUID 只會影響新的部署和資源，現有的部署和資源不受影響。

**何時才能使用報告功能？**

報告搶鮮版 (Beta) 很快就會推出。 回報功能將會與 CPP 整合。

**我可以從類似 GitHub 的非 Microsoft 存放庫追蹤部署的範本嗎？**

是，只要有 GUID，當範本部署時，都會追蹤使用量。 合作夥伴必須要在 CPP 中有設定檔，才能註冊在 Azure Marketplace 外部發佈的相關範本。 

**從 Azure Marketplace 和從類似 GitHub 的其他存放庫部署範本，是否有差異？**

是，在 Azure Marketplace 發佈供應項目的合作夥伴可從 Azure Marketplace 收到部署的更詳細資料。 合作夥伴向 Azure Marketplace 入口網站和 Azure 入口網站中的客戶公開其供應項目，將獲益良多。 Azure Marketplace 中的供應項目也可為合作夥伴產生潛在客戶。

**假設我要建立自訂範本用於進行個別的客戶業務開發呢？**

您仍可將 GUID 加入至範本。 若您使用現有的已註冊 GUID，它會包括在回報中。 如果您建立新 GUID，則必須註冊該新 GUID 才能將它納入追蹤。

**客戶也會收到報告嗎？**

客戶可以在 Azure 入口網站內，追蹤其個別資源或客戶所定義資源群組的使用情況。   

**此追蹤方法和數位記錄合作夥伴 (DPOR) 類似嗎？**

這種將部署與使用狀況連結至合作夥伴解決方案的新方法，提供將合作夥伴解決方案連結到 Azure 使用量的機制。 DPOR 可將諮詢 (系統整合者) 或管理 (受控服務提供者) 合作夥伴與客戶的 Azure 訂用帳戶建立關聯。   
