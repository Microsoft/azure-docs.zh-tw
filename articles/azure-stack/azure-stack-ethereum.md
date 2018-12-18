---
title: Azure Stack 以太坊區塊鏈解決方案範本
description: 教學課程：使用自訂的解決方案範本，在 Azure Stack 上部署和設定聯盟以太坊區塊鏈網路
services: azure-stack
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 09/13/2018
ms.topic: tutorial
ms.service: azure-stack
ms.reviewer: seyadava
ms.custom: mvc
manager: femila
ms.openlocfilehash: acfa94799f36728f4e0041f1a51403edf6ffe37e
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48239467"
---
# <a name="tutorial-use-the-azure-stack-ethereum-blockchain-solution-template"></a>教學課程：使用 Azure Stack 以太坊區塊鏈解決方案範本

以太坊解決方案範本的設計訴求是，讓您以最低限度的 Azure 和以太坊知識，更輕鬆且更快速地部署和設定多成員的聯盟以太坊區塊鏈網路。

透過少數使用者輸入及 Azure Stack 租用戶入口網站中的一鍵部署，每個成員都可以佈建他們的網路應用。 每個成員的網路應用都會包含一組負載平衡的交易節點 (應用程式或使用者可透過與其互動來提交交易)、一組用來記錄交易的採礦節點和網路虛擬設備 (NVA)。 後續的連線步驟會與 NVA 連線，藉以建立已完整設定的多成員區塊鏈網路閘道。

若要設定，您將必須：

> [!div class="checklist"]
> * 選擇部署架構
> * 部署獨立、聯盟領導者或聯盟成員網路

## <a name="prerequisites"></a>必要條件

[從 Marketplace](azure-stack-download-azure-marketplace-item.md) 下載最新項目：

* Ubuntu Server 16.04 LTS
* Windows Server 2016
* Custom Script for Linux 2.0
* Windows 的自訂指令碼延伸模組

如需區塊鏈案例的詳細資訊，請參閱[以太坊工作量證明聯盟解決方案範本](../blockchain/templates/ethereum-deployment.md)。

## <a name="deployment-architecture"></a>部署架構

此解決方案範本可以部署單一成員或多成員的以太坊聯盟網路。 虛擬網路會使用網路虛擬設備和連線資源在鏈結拓撲中連線。 

## <a name="deployment-use-cases"></a>部署使用案例

範本可以透過各種方式來部署以太坊聯盟以供領導者和成員加入，以下是已完成測試的部署方式：

- 在具有 Azure AD 或 AD FS 的多節點 Azure Stack 上，使用相同訂用帳戶或不同訂用帳戶部署領導者和成員。
- 在單一節點 Azure Stack (具有 Azure AD) 上，使用相同訂用帳戶部署領導者和成員。

### <a name="standalone-and-consortium-leader-deployment"></a>獨立和聯盟領導者部署

聯盟領導者範本會在網路中設定第一個成員的配置。 

1. [從 GitHub 下載領導者範本](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/ethereum-consortium-blockchain/marketplace/ConsortiumLeader/mainTemplate.json)
2. 在 Azure Stack 系統管理入口網站中，選取 [+ 建立資源] > [範本部署] 以從自訂範本進行部署。
3. 選取 [編輯範本] 來編輯新的自訂範本。
4. 在右側的編輯窗格中，複製並貼上您先前下載的領導者範本 JSON。
    
    ![編輯領導者範本](media/azure-stack-ethereum/edit-leader-template.png)

5. 選取 [ **儲存**]。
6. 選取 [編輯參數] 並完成部署的範本參數。
    
    ![編輯領導者範本參數](media/azure-stack-ethereum/edit-leader-parameters.png)

    參數名稱 | 說明 | 允許的值 | 範例值
    ---------------|-------------|----------------|-------------
    NAMEPREFIX | 用來作為所部署資源命名基底的字串。 | 1 到 6 個英數字元 | eth
    AUTHTYPE | 用來向虛擬機器驗證的方法。 | 密碼或 SSH 公開金鑰 | 密碼
    ADMINUSERNAME | 每個所部署 VM 的管理員使用者名稱 | 1 - 64 個字元 | gethadmin
    ADMINPASSWORD (驗證類型 = 密碼)| 每個所部署虛擬機器的管理員帳戶密碼。 密碼必須包含下列其中 3 項要求：1 個大寫字元、1 個小寫字元、1 個數字與 1 個特殊字元。 <br />所有 VM 一開始都有相同的密碼，但您可以在佈建之後變更密碼。|12 - 72 個字元|
    ADMINSSHKEY (驗證類型 = sshPublicKey) | 用於遠端登入的安全殼層金鑰。 | |
    GENESISBLOCK | 代表自訂創世區塊的 JSON 字串。  可選擇是否指定此參數的值。 | |
    ETHEREUMACCOUNTPSSWD | 用來保護以太坊帳戶的系統管理員密碼。 | |
    ETHEREUMACCOUNTPASSPHRASE | 用來產生私密金鑰的複雜密碼，此金鑰會與以太坊帳戶相關聯。 | |
    ETHEREUMNETWORKID | 聯盟的網路識別碼。 | 使用介於 5 到 999,999,999 之間的任意值 | 72
    CONSORTIUMMEMBERID | 與聯盟網路每個成員相關聯的識別碼。   | 這個識別碼在網路中應該是唯一的。 | 0
    NUMMININGNODES | 採礦節點的數目。 | 介於 2 到 15。 | 2
    MNNODEVMSIZE | 採礦節點的 VM 大小。 | | Standard_A1
    MNSTORAGEACCOUNTTYPE | 採礦節點的儲存體效能。 | | Standard_LRS
    NUMTXNODES | 交易節點數目。 | 介於 1 到 5。 | 1
    TXNODEVMSIZE | 交易節點的 VM 大小。 | | Standard_A1
    TXSTORAGEACCOUNTTYPE | 交易節點的儲存體效能。 | | Standard_LRS
    BASEURL | 用來從中取得相依範本的基底 URL。 | 除非您想要自訂部署範本，否則請使用預設值。 | 

7. 選取 [確定] 。
8. 在 [自訂部署] 中，指定 [訂用帳戶]、[資源群組] 和 [資源群組位置]。
    
    ![領導者部署參數](media/azure-stack-ethereum/leader-deployment-parameters.png)

    參數名稱 | 說明 | 允許的值 | 範例值
    ---------------|-------------|----------------|-------------
    訂用帳戶 | 要對其部署聯盟網路的訂用帳戶 | | 取用訂用帳戶
    資源群組 | 要對其部署聯盟網路的資源群組。 | | EthereumResources
    位置 | 資源群組的 Azure 區域。 | | local

8. 選取 [建立] 。

部署可能需要 20 分鐘或更久的時間才能完成。

部署完成之後，您可以檢閱 **Microsoft.Template** 的部署摘要，位於資源群組的部署區段中。 摘要中包含可用來加入聯盟成員的輸出值。

若要確認領導者的部署，請瀏覽領導者的管理網站。 您可以在 **Microsoft.Template** 部署的 [輸出] 區段中找到管理網站位址。  

![領導者部署摘要](media/azure-stack-ethereum/ethereum-node-status.png)

### <a name="joining-consortium-member-deployment"></a>加入聯盟成員部署

1. [從 GitHub 下載聯盟成員範本](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/ethereum-consortium-blockchain/marketplace/JoiningMember/mainTemplate.json)
2. 在 Azure Stack 系統管理入口網站中，選取 [+ 建立資源] > [範本部署] 以從自訂範本進行部署。
3. 選取 [編輯範本] 來編輯新的自訂範本。
4. 在右側的編輯窗格中，複製並貼上您先前下載的領導者範本 JSON。
5. 選取 [ **儲存**]。
6. 選取 [編輯參數] 並完成部署的範本參數。

    參數名稱 | 說明 | 允許的值 | 範例值
    ---------------|-------------|----------------|-------------
    NAMEPREFIX | 用來作為所部署資源命名基底的字串。 | 1 到 6 個英數字元 | eth
    AUTHTYPE | 用來向虛擬機器驗證的方法。 | 密碼或 SSH 公開金鑰 | 密碼
    ADMINUSERNAME | 每個所部署 VM 的管理員使用者名稱 | 1 - 64 個字元 | gethadmin
    ADMINPASSWORD (驗證類型 = 密碼)| 每個所部署虛擬機器的管理員帳戶密碼。 密碼必須包含下列其中 3 項要求：1 個大寫字元、1 個小寫字元、1 個數字與 1 個特殊字元。 <br />所有 VM 一開始都有相同的密碼，但您可以在佈建之後變更密碼。|12 - 72 個字元|
    ADMINSSHKEY (驗證類型 = sshPublicKey) | 用於遠端登入的安全殼層金鑰。 | |
    CONSORTIUMMEMBERID | 與聯盟網路每個成員相關聯的識別碼。   | 這個識別碼在網路中應該是唯一的。 | 0
    NUMMININGNODES | 採礦節點的數目。 | 介於 2 到 15。 | 2
    MNNODEVMSIZE | 採礦節點的 VM 大小。 | | Standard_A1
    MNSTORAGEACCOUNTTYPE | 採礦節點的儲存體效能。 | | Standard_LRS
    NUMTXNODES | 交易節點數目。 | 介於 1 到 5。 | 1
    TXNODEVMSIZE | 交易節點的 VM 大小。 | | Standard_A1
    TXSTORAGEACCOUNTTYPE | 交易節點的儲存體效能。 | | Standard_LRS
    CONSORTIUMDATA | 指向相關聯盟組態資料的 URL，而此組態資料是由另一個成員的部署所提供。 您可以在領導者的部署輸出上找到此值。 | |
    REMOTEMEMBERVNETADDRESSSPACE | 領導者的 NVA IP 位址。 您可以在領導者的部署輸出上找到此值。 | | 
    REMOTEMEMBERNVAPUBLICIP | 領導者的 NVA IP 位址。 您可以在領導者的部署輸出上找到此值。 | | 
    CONNECTIONSHAREDKEY | 聯盟網路中建立連線的成員之間所預先建立的祕密。 | |
    BASEURL | 範本的基底 URL。 | 除非您想要自訂部署範本，否則請使用預設值。 | 

7. 選取 [確定] 。
8. 在 [自訂部署] 中，指定 [訂用帳戶]、[資源群組] 和 [資源群組位置]。

    參數名稱 | 說明 | 允許的值 | 範例值
    ---------------|-------------|----------------|-------------
    訂用帳戶 | 要對其部署聯盟網路的訂用帳戶 | | 取用訂用帳戶
    資源群組 | 要對其部署聯盟網路的資源群組。 | | MemberResources
    位置 | 資源群組的 Azure 區域。 | | local

8. 選取 [建立] 。

部署可能需要 20 分鐘或更久的時間才能完成。

部署完成之後，您可以在資源群組的 [部署] 區段中檢閱 **Microsoft.Template** 的部署摘要。 摘要中包含可用來將聯盟成員連線的輸出值。

若要確認成員的部署，請瀏覽成員的管理網站。 您可以在 Microsoft.Template 部署的 [輸出] 區段中找到管理網站位址。

![成員部署摘要](media/azure-stack-ethereum/ethereum-node-status-2.png)

如上圖所示，成員的節點狀態是**未執行**。 這是因為成員和領導者之間未建立連線。 成員和領導者之間的連線是雙向連線。 當您部署成員時，範本會自動建立從成員到領導者的連線。 若要建立從領導者到成員的連線，請移至下一個步驟。

### <a name="connect-member-and-leader"></a>讓成員和領導者連線

此範本會建立從領導者到遠端成員的連線。 

1. [從 GitHub 下載讓成員和領導者連線的範本](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/ethereum-consortium-blockchain/marketplace/Connection/mainTemplate.json)
2. 在 Azure Stack 系統管理入口網站中，選取 [+ 建立資源] > [範本部署] 以從自訂範本進行部署。
3. 選取 [編輯範本] 來編輯新的自訂範本。
4. 在右側的編輯窗格中，複製並貼上您先前下載的領導者範本 JSON。
    
    ![編輯連線範本](media/azure-stack-ethereum/edit-connect-template.png)

5. 選取 [ **儲存**]。
6. 選取 [編輯參數] 並完成部署的範本參數。
    
    ![編輯連線範本參數](media/azure-stack-ethereum/edit-connect-parameters.png)

    參數名稱 | 說明 | 允許的值 | 範例值
    ---------------|-------------|----------------|-------------
    MEMBERNAMEPREFIX | 領導者的名稱前置詞。 您可以在領導者的部署輸出上找到此值。  | 1 到 6 個英數字元 | |
    MEMBERROUTETABLENAME | 領導者的路由表名稱。 您可以在領導者的部署輸出上找到此值。 |  | 
    REMOTEMEMBERVNETADDRESSSPACE | 成員的位址空間。 您可以在成員的部署輸出上找到此值。 | |
    CONNECTIONSHAREDKEY | 聯盟網路中建立連線的成員之間所預先建立的祕密。  | |
    REMOTEMEMBERNVAPUBLICIP | 成員的 NVA IP 位址。 您可以在成員的部署輸出上找到此值。 | |
    MEMBERNVAPRIVATEIP | 領導者的私人 NVA IP 位址。 您可以在領導者的部署輸出上找到此值。 | |
    LOCATION | 您 Azure Stack 環境的位置。 | | local
    BASEURL | 範本的基底 URL。 | 除非您想要自訂部署範本，否則請使用預設值。 | 

7. 選取 [確定] 。
8. 在 [自訂部署] 中，指定 [訂用帳戶]、[資源群組] 和 [資源群組位置]。
    
    ![連線部署參數](media/azure-stack-ethereum/connect-deployment-parameters.png)

    參數名稱 | 說明 | 允許的值 | 範例值
    ---------------|-------------|----------------|-------------
    訂用帳戶 | 領導者的訂用帳戶。 | | 取用訂用帳戶
    資源群組 | 領導者的資源群組。 | | EthereumResources
    位置 | 資源群組的 Azure 區域。 | | local

8. 選取 [建立] 。

部署完成之後，領導者和成員需要幾分鐘的時間才會開始通訊。 若要確認部署，請重新整理成員的管理網站。 成員節點的狀態應該是執行中。 

![驗證部署](media/azure-stack-ethereum/ethererum-node-status-3.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 選擇部署架構
> * 部署獨立、聯盟領導者或聯盟成員網路

若要深入了解以太坊和 Azure，請參閱：

> [!div class="nextstepaction"]
> [區塊鏈技術與應用程式](https://azure.microsoft.com/solutions/blockchain/)
