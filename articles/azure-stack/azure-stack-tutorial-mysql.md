---
title: 在 Azure Stack 上提供高可用性 MySQL 資料庫 | Microsoft Docs
description: 了解如何使用 Azure Stack 來建立 MySQL 伺服器資源提供者主機電腦和高可用性 MySQL 資料庫。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/16/2018
ms.author: jeffgilb
ms.reviewer: quying
ms.openlocfilehash: ea3e6c2e616f2618200c1e3904786abd72bbd75d
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376800"
---
# <a name="tutorial-offer-highly-available-mysql-databases"></a>教學課程：提供高可用性 MySQL 資料庫

如果您是 Azure Stack 操作者，便可設定讓伺服器 VM 裝載 MySQL 伺服器資料庫。 在 MySQL 叢集已成功建立並受 Azure Stack 管理之後，已訂閱 MySQL 服務的使用者便可輕鬆建立高可用性 MySQL 資料庫。

此教學課程說明如何使用 Azure Stack Marketplace 項目來建立[具備複寫叢集的 MySQL](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.mysql-cluster)。 這個解決方案使用多個 VM，將資料庫從主要節點複寫至可設定數量的複本。 建立叢集之後，即可接著將其新增為「Azure Stack MySQL 主控伺服器」，然後使用者便可建立高可用性 MySQL 資料庫。

> [!IMPORTANT]
> **具備複寫功能的 MySQL** Azure Stack Marketplace 項目可能不是所有 Azure 雲端訂用帳戶環境都可使用。 請先確認您的訂用帳戶中有此 Marketplace 項目可供使用，再嘗試依照此教學課程中的其餘部分進行操作。

您將了解：

> [!div class="checklist"]
> * 從 Marketplace 項目建立 MySQL 伺服器叢集
> * 建立 Azure Stack MySQL 主控伺服器
> * 建立高可用性 MySQL 資料庫

在此教學課程中，將會使用可用的 Azure Stack Marketplace 項目來建立並設定一個有三部 VM 的 MySQL 伺服器叢集。 

開始進行此教學課程中的步驟之前，請確定已成功安裝 [MySQL Server 資源提供者](azure-stack-mysql-resource-provider-deploy.md)且 Azure Stack Marketplace 中提供下列項目：

> [!IMPORTANT]
> 必須備妥下列各項，才能建立 MySQL 叢集。

- [具備複寫功能的 MySQL](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.mysql-cluster)。 這是將用來部署 MySQL 叢集的 Bitnami 解決方案範本。
- [Debian 8 "Jessie"](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/credativ.Debian8backports?tab=Overview)。 由 credativ 所提供、含有適用於 Microsoft Azure 之反向移植核心的 Debian 8 "Jessie"。 Debian GNU/Linux 是最受歡迎其中一個 Linux 散發套件。
- [適用於 Linux 的自訂指令碼 2.0](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoft.custom-script-linux?tab=Overview)。 「自訂指令碼延伸模組」是一個工具，可用來執行佈建 VM 後的 VM 自訂工作。 將此延伸模組新增至虛擬機器後，它便可從 Azure 儲存體下載指令碼，並在 VM 上執行這些指令碼。 您也可以使用 Azure PowerShell Cmdlet 和「Azure 跨平台命令列介面」(xPlat CLI) 將「自訂指令碼延伸模組」工作自動化。
- 適用於 Linux 的 VM 存取延伸模組 1.4.7。 此 VM 存取延伸模組可讓您重設密碼、SSH 金鑰或 SSH 設定，以便重新取得 VM 存取權。 您也可以使用此延伸模組來新增具有密碼或 SSH 金鑰的新使用者，或刪除使用者。 此延伸模組的目標為 Linux VM。

若要深入了解如何將項目新增至 Azure Stack Marketplace，請參閱 [Azure Stack Marketplace 概觀](azure-stack-marketplace.md)。

您還需要一個 SSH 用戶端 (例如 [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html))，以在部署 Linux VM 之後登入這些 VM。

## <a name="create-a-mysql-server-cluster"></a>建立 MySQL 伺服器叢集 
請使用本節中的步驟，運用[具備複寫功能的 MySQL](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.mysql-cluster) Marketplace 項目來部署 MySQL伺服器叢集。 此範本會部署三個設定在高可用性 MySQL 叢集內的 MySQL 伺服器執行個體。 預設會建立下列資源：

- 虛擬網路
- 網路安全性群組
- 儲存體帳戶
- 可用性設定組
- 三個網路介面 (每個預設 VM 各一個)
- 公用 IP 位址 (用於主要的 MySQL 叢集 VM)
- 三個用以裝載 MySQL 叢集的 Linux VM

1. 登入系統管理入口網站：
    - 針對整合式系統部署，入口網站位址會依據您解決方案的區域和外部網域名稱而有所不同。 其格式將會是 https://adminportal.&lt;*區域*&gt;.&lt;*FQDN*&gt;。
    - 如果您使用 Azure Stack 開發套件 (ASDK)，則入口網站位址為 [https://adminportal.local.azurestack.external](https://adminportal.local.azurestack.external)。

2. 選取 [\+ 建立資源] > [計算]，然後選取 [具備複寫功能的 MySQL]。

   ![自訂範本部署](media/azure-stack-tutorial-mysqlrp/createcluster1.png)

3. 在 [基本] 頁面上提供基本部署資訊。 檢閱預設值並視需要變更，然後按一下 [確定]。<br><br>請至少提供下列資訊：
   - 部署名稱 (預設為 mysql)
   - 應用程式根密碼。 提供一個不含**任何特殊字元**的 12 字元英數字元密碼
   - 應用程式資料庫名稱 (預設為 bitnami)
   - 要建立的 MySQL 資料庫複本 VM 數目 (預設為 2)
   - 選取要使用的訂用帳戶
   - 選取要使用的資源群組，或建立一個新資源群組
   - 選取位置 (ASDK 的預設為本機)

   ![部署基本](media/azure-stack-tutorial-mysqlrp/createcluster2.png)

4. 在 [環境設定] 頁面上提供下列資訊，然後按一下 [確定]： 
   - 要用於安全殼層 (SSH) 驗證的密碼或 SSH 公開金鑰。 如果使用密碼，則密碼必須包含字母、數字，並且**可**包含特殊字元
   - VM 大小 (預設為標準 D1 v2 VM)
   - 資料磁碟大小 (GB)。按一下 [確定]

   ![環境設定](media/azure-stack-tutorial-mysqlrp/createcluster3.png)

5. 檢閱部署 [摘要]。 您也可以視需要下載自訂範本和參數，然後按一下 [確定]。

   ![總結](media/azure-stack-tutorial-mysqlrp/createcluster4.png)

6. 按一下 [購買] 頁面上的 [建立] 以開始部署。

   ![購買](media/azure-stack-tutorial-mysqlrp/createcluster4.png)

    > [!NOTE]
    > 部署將花費大約一小時的時間。 請先確定部署已完成且 MySQL 叢集已完全設定妥當，再繼續進行。 

7. 順利完成所有部署之後，請檢閱資源群組項目，然後選取 **mysqlip** 公用 IP 位址項目。 請記錄叢集之公用 IP 位址和公用 IP 的完整 FQDN。<br><br>您將需要將向「Azure Stack 操作員」提供此資訊，以便讓他們運用這個 MySQL 叢集來建立 MySQL 主控伺服器。


### <a name="create-a-network-security-group-rule"></a>建立網路安全性群組規則
預設並不會為 MySQL 設定進入主機 VM 的公用存取權。 若要讓 Azure Stack MySQL 資源提供者連線及管理 MySQL 叢集，就必須建立一個輸入網路安全性群組 (NSG) 規則。

1. 在系統管理員入口網站中，瀏覽至部署 MySQL 叢集時所建立的資源群組，然後選取網路安全性群組 (**default-subnet-sg**)：

   ![開啟](media/azure-stack-tutorial-mysqlrp/nsg1.png)

2. 選取 [輸入安全性規則]，然後按一下 [新增]。<br><br>在 [目的地連接埠範圍] 中 輸入**3306**，然後視需要在 [名稱] 和 [描述] 欄位中輸入描述。 按一下 [新增] 即可關閉輸入安全性規則對話方塊。

   ![開啟](media/azure-stack-tutorial-mysqlrp/nsg2.png)

### <a name="configure-external-access-to-the-mysql-cluster"></a>設定外部對 MySQL 叢集的存取權
必須先啟用外部存取權，才能將 MySQL 叢集新增為 Azure Stack MySQL 伺服器主機。

1. 在使用 SSH 用戶端的情況下，此範例會使用 [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)，請從能夠存取公用 IP 的電腦登入 MySQL 機器。 主要 MySQL VM 名稱通常結尾是 **0**，且已有指派的公用 IP。<br><br>請使用該公用 IP，並以使用者名稱 **bitnami** 及稍早所建立不含特殊字元的應用程式密碼登入 VM。

   ![LinuxLogin](media/azure-stack-tutorial-mysqlrp/bitnami1.png)


2. 在 SSH 用戶端視窗中，使用下列命令來確保 bitnami 服務已處於作用中及執行中狀態。 出現提示時，請再次提供 bitnami 密碼：

   `sudo service bitnami status`

   ![檢查服務](media/azure-stack-tutorial-mysqlrp/bitnami2.png)

3. 建立「Azure Stack MySQL 主控伺服器」要用來連線至 MySQL 的遠端存取使用者帳戶，然後結束 SSH 用戶端。<br><br>請執行下列命令，使用稍早所建立的根密碼以根身分登入 MySQL，然後建立一個新的系統管理員使用者，視您環境的需要取代 *\<username\>* 和 *\<password\>*。 在此範例中，要建立的使用者是命名為 **sqlsa**，並且使用強式密碼：

   ```mysql
   mysql -u root -p
   create user <username>@'%' identified by '<password>';
   grant all privileges on *.* to <username>@'%' with grant option;
   flush privileges;
   ```
   ![建立系統管理員使用者](media/azure-stack-tutorial-mysqlrp/bitnami3.png)


4. 記錄新的 MySQL 使用者資訊。<br><br>您將需要向「Azure Stack 操作員」提供這個使用者名稱和密碼，以及叢集的公用 IP 位址或公用 IP 的完整 FQDN，以便讓他們使用這個 MySQL 叢集來建立 MySQL 主控伺服器。


## <a name="create-an-azure-stack-mysql-hosting-server"></a>建立 Azure Stack MySQL 主控伺服器
在 MySQL 伺服器叢集已建立並設定妥當之後，「Azure Stack 操作員」必須建立「Azure Stack MySQL 主控伺服器」，才能將額外的容量提供給使用者來建立資料庫。 

請務必使用先前在建立 MySQL 叢集的資源群組時所記錄的 MySQL 叢集主要 VM 的公用 IP (**mysqlip**) 或公用 IP 的完整 FQDN。 此外，操作員還需要知道您所建立用來從遠端存取 MySQL 叢集資料庫的 MySQL 伺服器驗證認證。

> [!NOTE]
> 此步驟必須由「Azure Stack 操作員」從 Azure Stack 系統管理入口網站執行。

使用 MySQL 叢集的公用 IP 和 MySQL 驗證登入資訊，Azure Stack 操作員現在可以[使用新的 MySQL 叢集來建立 MySQL 主控伺服器](azure-stack-mysql-resource-provider-hosting-servers.md#connect-to-a-mysql-hosting-server)。 

此外，也請確定您已建立一些方案和供應項目，以供使用者建立 MySQL 資料庫。 操作員將需要把 **Microsoft.MySqlAdapter** 服務新增至方案中，並建立一個專門用於高可用性資料庫的新配額。 如需有關建立方案的詳細資訊，請參閱[方案、供應項目、配額和訂用帳戶概觀](azure-stack-plan-offer-quota-overview.md)。

> [!TIP]
> 將必須等到[已部署 MySQL 伺服器資源提供者](azure-stack-mysql-resource-provider-deploy.md)之後，才能新增 **Microsoft.MySqlAdapter** 服務。



## <a name="create-a-highly-available-mysql-database"></a>建立高可用性 MySQL 資料庫
在已建立、設定 MySQL 叢集並由「Azure Stack 操作員」新增為「Azure Stack MySQL 主控伺服器」之後，具有含 MySQL 伺服器資料庫功能之訂用帳戶的租用戶使用者便可依照本節中的步驟建立高可用性 MySQL 資料庫。 

> [!NOTE]
> 請從 Azure Stack 使用者入口網站，以具有可提供 MySQL 伺服器功能 (Microsoft.MySQLAdapter 服務) 之訂用帳戶的租用戶使用者身分執行這些步驟。

1. 登入使用者入口網站：
    - 針對整合式系統部署，入口網站位址會依據您解決方案的區域和外部網域名稱而有所不同。 其格式將會是 https://portal.&lt;*區域*&gt;.&lt;*FQDN*&gt;。
    - 如果您使用「Azure Stack 開發套件」(ASDK)，則使用者入口網站位址為 [https://portal.local.azurestack.external](https://portal.local.azurestack.external)。

2. 選取 [\+ 建立資源] > [資料 \+ 儲存體]，然後選取 [MySQL 資料庫]。<br><br>請提供必要的資料庫屬性資訊，包括名稱、定序、要使用的訂用帳戶，以及要用於部署的位置。 

   ![建立 MySQL 資料庫](./media/azure-stack-tutorial-mysqlrp/createdb1.png)

3. 選取 [SKU]，然後選擇要使用的適當「MySQL 主控伺服器 SKU」。 在此範例中，「Azure Stack 操作員」已建立 **MySQL-HA** SKU 來支援 MySQL 叢集資料庫的高可用性。

   ![選取 SKU](./media/azure-stack-tutorial-mysqlrp/createdb2.png)

4. 選取 [登入] > **[建立新的登入]**，然後提供要用於新資料庫的 MySQL 驗證認證。 完成時，請依序按一下 [確定] 和 [建立]，以開始資料庫部署程序。

   ![新增登入](./media/azure-stack-tutorial-mysqlrp/createdb3.png)

5. 當 MySQL 資料庫部署順利完成時，請檢閱資料庫屬性，以找出要用來連線至新高可用性資料庫的連接字串。 

   ![檢視連接字串](./media/azure-stack-tutorial-mysqlrp/createdb4.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 從 Marketplace 項目建立 MySQL 伺服器叢集
> * 建立 Azure Stack MySQL 主控伺服器
> * 建立高可用性 MySQL 資料庫

請前進到下一個教學課程，以了解如何：
> [!div class="nextstepaction"]
> [提供 Web 應用程式](azure-stack-tutorial-app-service.md)
