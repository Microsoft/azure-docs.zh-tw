---
title: 從 Azure 入口網站安裝 Chef 用戶端
description: 了解如何從 Azure 入口網站部署及設定您的 Chef 用戶端
keywords: azure, chef, devops, 用戶端, 安裝, 入口網站
ms.service: virtual-machines-linux
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 05/15/2018
ms.topic: article
ms.openlocfilehash: e121cd038b8becee1e9c4c12659dbbee0696a9f1
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44378647"
---
# <a name="install-the-chef-client-from-the-azure-portal"></a>從 Azure 入口網站安裝 Chef 用戶端
從 Azure 入口網站建立或修改 Linux 或 Windows 虛擬機器時，您可以將 Chef 擴充功能新增至虛擬機器。 這篇文章會逐步引導您進行使用新 Linux 虛擬機器的程序。

## <a name="prerequisites"></a>必要條件
- **Azure 訂用帳戶**：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- **Chef**：如果您沒有作用中 Chef 帳戶，請註冊[免費試用版託管 Chef](https://manage.chef.io/signup)。 若要依照本文章中的指示，您需要 Chef 帳戶的下列值： 
    - organization_validation 金鑰
    - rb
    - run_list

## <a name="install-the-chef-extension-on-a-new-linux-virtual-machine"></a>在新的 Linux 虛擬機器上安裝 Chef 擴充功能
在本節中，首先您會使用 Azure 入口網站建立 Linux 機器。 在過程中，您也會看到如何在新的虛擬機器上安裝 Chef 擴充功能。

1. 瀏覽至 [Azure 入口網站](http://portal.azure.com)。

1. 從左邊的功能表上，選取 [虛擬機器] 選項。 如果 [虛擬機器] 選項未顯示，請選取 [所有服務]，然後選取 [虛擬機器]。

1. 在 [虛擬機器] 索引標籤上，選取 [新增]。

    ![在 Azure 入口網站中新增新的虛擬機器](./media/chef-extension-portal/add-vm.png)

1. 在 [計算] 索引標籤上，選取所需的作業系統。 針對這個示範，請選取 **[Ubuntu Server]**。

1. 在 **[Ubuntu Server]** 索引標籤上，選取 **[Ubuntu Server 16.04 LTS]**。

    ![建立 Ubuntu 虛擬機器時，指定您需要的版本](./media/chef-extension-portal/ubuntu-server-version.png)

1. 在 **[Ubuntu Server 16.04 LTS]** 索引標籤上，選取 [建立]。

    ![Ubuntu 會提供其產品的額外資訊](./media/chef-extension-portal/create-vm.png)

1. 在 [建立虛擬機器] 索引標籤上，選取 [基本]。

1. 在 [基本] 索引標籤上，指定下列值，然後選取 [確定]。

    - **名稱** - 輸入新虛擬機器的名稱。
    - **虛擬機器磁碟類型** - 針對儲存體磁碟類型指定 **SSD** 或 **HDD**。 如需 Azure 上虛擬機器磁碟類型的詳細資訊，請參閱[虛擬機器的高效能進階儲存體與受控磁碟](/azure/virtual-machines/windows/premium-storage)一文。
    - **使用者名稱** - 輸入使用者名稱，系統已在虛擬機器上授與此使用者系統管理員權限。
    - **驗證類型** - 選取 [密碼]。 您也可以選取 **SSH 公開金鑰**，並提供 SSH 公開金鑰值。 針對本示範的目的 (以及在快照集中)，會選取 [密碼]。
    - **密碼**和**確認密碼** - 請輸入使用者的密碼。
    - **使用 Azure Active Directory 進行登入** - 請選取 [停用]。
    - **訂用帳戶** - 如果您有多個 Azure 訂用帳戶，請選取所需的訂用帳戶。
    - **資源群組** - 請輸入資源群組的名稱。
    - **位置** - 選取 [美國東部]。

    ![用來建立虛擬機器的 [基本] 索引標籤](./media/chef-extension-portal/add-vm-basics.png)

1. 請在 [選擇大小] 索引標籤上，選取虛擬機器的大小，然後選取 [選取]。

1. 在 [設定] 索引標籤上，系統會根據您在上一個索引標籤中選取的值，為您填入大部分的值。 選取 [擴充功能]。

    ![擴充功能會透過 [設定] 索引標籤新增至虛擬機器](./media/chef-extension-portal/add-vm-select-extensions.png)

1. 請在 [擴充功能] 索引標籤上，選取 [新增擴充功能]。

    ![請選取 [新增擴充功能] 以將擴充功能新增至虛擬機器](./media/chef-extension-portal/add-vm-add-extension.png)

1. 請在 [新增資源] 索引標籤上，選取 [Linux Chef 擴充功能 (1.2.3)]。

    ![Chef 具有 Linux 和 Windows 虛擬機器的擴充功能](./media/chef-extension-portal/select-linux-chef-extension.png)

1. 請在 [Linux Chef 擴充功能] 索引標籤上，選取 [建立]。

1. 請在 [安裝擴充功能] 索引標籤上，指定下列值，然後選取 [確定]。

    - **Chef 伺服器 URL** - 請輸入包含組織名稱的 Chef 伺服器 URL，例如 *https://api.chef.io/organization/hessco*。
    - **Chef 節點名稱** - 請輸入 Chef 節點名稱。 可以是任何值。
    - **執行清單** - 請輸入要新增至機器的 Chef 執行清單。 此項目可保留為空白。
    - **驗證用戶端名稱** - 請輸入 Chef 驗證用戶端名稱。 例如 *tarcher-validator*。
    - **驗證金鑰** - 選取檔案，其中包含啟動載入您的機器時使用的驗證金鑰。 
    - **用戶端組態檔** - 請選取 Chef 用戶端的組態檔。 此項目可保留為空白。
    - **Chef 用戶端版本** - 請輸入要安裝的 Chef 用戶端版本。 此項目可保留為空白。 若保留空白值，則會安裝最新的版本。 
    - **SSL 驗證模式** - 請選取 [無] 或 [對等]。 在示範中我們選取 [無]。
    - **Chef 環境** - 請輸入這個節點所屬的 Chef 環境。 此項目可保留為空白。
    - **加密 Databag 祕密** - 請選取包含加密 Databag 祕密的檔案，讓這部機器存取。 此項目可保留為空白。
    - **Chef 伺服器 SSL 憑證** - 請選取指派給 Chef 伺服器的 SSL 憑證。 此項目可保留為空白。

    ![在 Linux 虛擬機器上安裝 Chef 伺服器](./media/chef-extension-portal/install-extension.png)

1. 返回 [擴充功能] 索引標籤後，選取 [確定]。

1. 返回 [設定] 索引標籤後，選取 [確定]。

1. 返回 [建立] 索引標籤後 (顯示您選取及輸入的選項摘要)，請確認資訊以及**使用條款**，然後選取 [建立]。

建立及部署具有 Chef 擴充功能的虛擬機器程序完成時，通知會指出作業成功或失敗。 此外，新虛擬機器的資源頁面一旦建立，就會在 Azure 入口網站中自動開啟。

![在 Linux 虛擬機器上安裝 Chef 伺服器](./media/chef-extension-portal/resource-created.png)

## <a name="next-steps"></a>後續步驟
* [使用 Chef 在 Azure 上建立 Windows 虛擬機器](/azure/virtual-machines/windows/chef-automation)
