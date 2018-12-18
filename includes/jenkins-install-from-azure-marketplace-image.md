---
description: 包含檔案
author: tomarcher
manager: rloutlaw
ms.service: multiple
ms.workload: web
ms.devlang: na
ms.topic: include
ms.date: 03/12/2018
ms.author: tarcher
ms.custom: Jenkins
ms.openlocfilehash: d5a832909f060ad8c8b3f0e7c7ea4504e5e5aadb
ms.sourcegitcommit: 828d8ef0ec47767d251355c2002ade13d1c162af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36943465"
---
1. 在瀏覽器中開啟[適用於 Jenkins 的 Azure Marketplace 映像](https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview)。

1. 選取 [立即取得]。

    ![選取 [立即取得] 以開始 Jenkins Marketplace 映像的安裝程序。](./media/jenkins-install-from-azure-marketplace-image/jenkins-install-get-it-now.png)

1. 檢閱定價詳細資料和條款資訊之後，選取 [繼續]。

    ![Jenkins Marketplace 映像定價和條款資訊。](./media/jenkins-install-from-azure-marketplace-image/jenkins-install-pricing-and-terms.png)

1. 選取 [建立] 以在 Azure 入口網站中設定 Jenkins 伺服器。 

    ![安裝 Jenkins Marketplace 映像。](./media/jenkins-install-from-azure-marketplace-image/jenkins-install-create.png)

1. 在 [基本] 索引標籤中，指定下列值：

    - **Name** (名稱)：輸入 `Jenkins`。
    - **使用者名稱** - 輸入當登入執行 Jenkins 所在的虛擬機器時，要使用的使用者名稱。 使用者名稱必須符合[特定需求](/azure/virtual-machines/linux/faq#what-are-the-username-requirements-when-creating-a-vm)。
    - **驗證類型** - 選取 [SSH 公開金鑰]。
    - **SSH 公開金鑰** - 複製並貼上 RSA 公開金鑰，可以是單行格式 (開頭為 `ssh-rsa`) 或多行 PEM 格式。 您可以使用 Linux 和 macOS 上的 ssh-keygen 或 Windows 上的 PuTTYGen 來產生 SSH 金鑰。 如需有關 SSH 金鑰和 Azure 的詳細資訊，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](/azure/virtual-machines/linux/ssh-from-windows)一文。
    - **訂用帳戶** - 選取您要在其中安裝 Jenkins 的 Azure 訂用帳戶。
    - **資源群組** - 選取 [新建]，然後為資源群組 (作為組成 Jenkins 安裝之資源集合的邏輯容器) 輸入名稱。
    - **位置** - 選取 [美國東部]。

    ![在 [基本] 索引標籤中輸入 Jenkins 的驗證和資源群組資訊。](./media/jenkins-install-from-azure-marketplace-image/jenkins-configure-basic.png)

1. 選取 [確定] 以繼續前往 [其他設定] 索引標籤。 

1. 在 [其他設定] 索引標籤中，指定下列值：

    - **大小** - 為您的 Jenkins 虛擬機器選取適當大小選項。
    - **VM 磁碟類型** - 指定 HDD (硬碟) 或 SSD (固態硬碟)，以表示 Jenkins 虛擬機器可使用哪種儲存磁碟類型。
    - **虛擬網路** - (選擇性) 選取 [虛擬網路] 可修改預設設定。
    - **子網路** - 選取 [子網路]、確認資訊，並選取 [確定]。
    - **公用 IP 位址** - IP 位址名稱預設為您在上一頁中以尾碼為 -IP 指定的 Jenkins 名稱。 您可以選取選項來變更該預設值。
    - **網域名稱標籤** - 為 Jenkins 虛擬機器指定完整 URL 值。
    - **Jenkins 版本類型** - 從選項中選取所需的版本類型：`LTS`、`Weekly build` 或 `Azure Verified`。 `LTS` 和 `Weekly build` 選項在 [Jenkins LTS Release Line](https://jenkins.io/download/lts/) (英文) 一文中有說明。 `Azure Verified` 選項是指已經過驗證可在 Azure 上執行的 [Jenkins LTS 版本](https://jenkins.io/download/lts/) (英文)。 

    ![在 [設定] 索引標籤中輸入 Jenkins 的虛擬機器設定。](./media/jenkins-install-from-azure-marketplace-image/jenkins-configure-settings.png)

1. 選取 [確定] 以繼續前往 [整合設定] 索引標籤。

1. 在 [整合設定] 索引標籤中，指定下列值：

    - **服務主體** - 新增至 Jenkins 作為向 Azure 驗證的認證服務主體。 `Auto` 表示將由 MSI (受控服務識別) 建立主體。 `Manual` 表示應該由您建立主體。 
        - **應用程式識別碼**和**祕密** - 如果在 [服務主體] 選項選取 `Manual` 選項，則必須為服務主體指定 `Application ID` 和 `Secret`。 [建立服務主體](/cli/azure/create-an-azure-service-principal-azure-cli)時，請注意預設角色是**參與者**，其資格足以使用 Azure 資源。
    - **啟用雲端代理程式** - 指定代理程式的預設雲端範本，其中 `ACI` 是指 Azure 容器執行個體，`VM` 是指虛擬機器。 如果您不想要啟用雲端代理程式，也可以指定 `No`。

1. 選取 [確定] 以繼續前往 [摘要] 索引標籤。

1. 當 [摘要] 索引標籤顯示時，就會驗證輸入的資訊。 一旦您在索引標籤頂端看到 [通過驗證] 訊息，請選取 [確定]。 

    ![[摘要] 索引標籤會顯示並驗證您選取的選項。](./media/jenkins-install-from-azure-marketplace-image/jenkins-configure-summary.png)

1. 當 [建立] 索引標籤顯示時，請選取 [建立] 以建立 Jenkins 虛擬機器。 當您的伺服器準備就緒時，會在 Azure 入口網站中顯示通知。

    ![Jenkins 已就緒通知。](./media/jenkins-install-from-azure-marketplace-image/jenkins-install-notification.png)