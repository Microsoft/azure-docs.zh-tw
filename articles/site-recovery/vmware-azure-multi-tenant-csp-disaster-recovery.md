---
title: 使用 Site Recovery 和雲端解決方案提供者 (CSP) 方案在多租用戶環境中設定 VMware 到 Azure 的複寫 | Microsoft Docs
description: 說明如何透過 CSP 建立和管理租用戶訂用帳戶，並在多租用戶設定中部署 Azure Site Recovery
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: bd356a7e4285f8663b91da1f200cc2ae3c266bf3
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49354113"
---
# <a name="set-up-vmware-replication-in-a-multi-tenancy-environment-with-the-cloud-solution-provider-csp-program"></a>使用雲端解決方案提供者 (CSP) 方案在多租用戶環境中設定 VMware 複寫

[CSP 計畫](https://partner.microsoft.com/en-US/cloud-solution-provider)會針對 Microsoft 雲端服務 (包括 Office 365、Enterprise Mobility Suite 和 Microsoft Azure) 樹立雙贏的案例。 藉由 CSP，合作夥伴全面掌握與客戶的端對端關係，並且成為主要的關係連絡窗口。 合作夥伴可以為客戶部署 Azure 訂用帳戶，並將這些訂用帳戶與他們自己的加值自訂供應項目結合。

使用 [Azure Site Recovery](site-recovery-overview.md)，身為合作夥伴的您可以直接透過 CSP 管理客戶的災害復原。 或者，您可以使用 CSP 設定 Site Recovery 環境，並讓客戶藉由自助服務的方式管理自己的災害復原需求。 在這兩種情況下，合作夥伴是 Site Recovery 與客戶之間的連絡窗口。 合作夥伴負責維護客戶關係，以及向客戶收取 Site Recovery 使用量的費用。

本文說明身為合作夥伴的您如何針對多租用戶 VMware 複寫案例，透過 CSP 建立和管理租用戶訂用帳戶。

## <a name="prerequisites"></a>必要條件

若要設定 VMware 複寫，您需要執行下列作業：

- [準備](tutorial-prepare-azure.md) Azure 資源，包括 Azure 訂用帳戶、Azure 虛擬網路和儲存體帳戶。
- [準備](vmware-azure-tutorial-prepare-on-premises.md)內部部署 VMware 伺服器和 VM。
- 針對每個租用戶建立可與租用戶 VM 及 vCenter 伺服器通訊的個別管理伺服器。 只有身為合作夥伴的您具有此管理伺服器的存取權限。 深入了解[多租用戶環境](vmware-azure-multi-tenant-overview.md)。

## <a name="create-a-tenant-account"></a>建立租用戶帳戶

1. 透過 [Microsoft 合作夥伴中心](https://partnercenter.microsoft.com/) 登入 CSP 帳戶。
2. 在 [儀表板] 功能表上，選取 [客戶]。
3. 在開啟的頁面上，按一下 [新增客戶] 按鈕。
4. 在 [新客戶] 頁面中，填入租用戶的帳戶資訊詳細資料。

    ![[帳戶資訊] 頁面](./media/vmware-azure-multi-tenant-csp-disaster-recovery/customer-add-filled.png)

5. 然後按 [下一步：訂用帳戶]。
6. 在訂用帳戶選取頁面上，選取 [Microsoft Azure] 核取方塊。 您可以立即或在其他任何時候新增其他訂用帳戶。
7. 在 [檢閱] 頁面上，確認租用戶詳細資料，然後按一下 [提交]。
8. 您建立租用戶帳戶後，確認網頁隨即出現，顯示預設帳戶的詳細資料和該訂用帳戶的密碼。 儲存資訊並在稍後視需要透過 Azure 入口網站登入網頁來變更密碼。

您可以與租用戶共用這些既有的資訊，也可以視需要建立和共用不同的帳戶。

## <a name="access-the-tenant-account"></a>存取租用戶帳戶

您可以透過 Microsoft 夥伴中心儀表板存取租用戶的訂用帳戶。

1. 在 [客戶] 頁面上，按一下租用戶帳戶的名稱。
2. 在租用戶帳戶的 [訂用帳戶] 頁面中，您可以監視現有帳戶的訂用帳戶，並且視需要新增更多訂用帳戶。
3. 若要管理租用戶的災復原作業，請選取 [所有資源 (Azure 入口網站)]。 這會授與您租用戶的 Azure 訂用帳戶存取權限。

    ![[所有資源] 連結](./media/vmware-azure-multi-tenant-csp-disaster-recovery/all-resources-select.png)  

4. 您可以按一下 Azure 入口網站右上方的 [Azure Active Directory] 連結，以確認存取權限。

    ![[Azure Active Directory] 連結](./media/vmware-azure-multi-tenant-csp-disaster-recovery/aad-admin-display.png)

您現在可以在 Azure 入口網站中為租用戶執行及管理所有 Site Recovery 作業。 若要透過 CSP 存取租用戶訂用帳戶來進行受控災害復原，請遵循先前所述的程序。

## <a name="assign-tenant-access-to-the-subscription"></a>為租用戶指派訂用帳戶的存取權

1. 確定已設定災害復原基礎結構。 無論災害復原類型是受控或自助，合作夥伴都必須透過 CSP 入口網站存取租用戶訂用帳戶。 設定保存庫，並且向租用戶訂用帳戶註冊基礎結構。
2. 將[您建立的帳戶](#create-a-tenant-account)提供給租用戶。
3. 您可以透過 CSP 入口網站將新的使用者新增至租用戶訂用帳戶，方法如下：

    a) 移至租用戶的 CSP 訂用帳戶頁面，然後選取 [使用者與授權] 選項。

      ![租用戶的 CSP 訂用帳戶頁面](./media/vmware-azure-multi-tenant-csp-disaster-recovery/users-and-licences.png)

      b) 現在藉由輸入相關詳細資料並選取權限，或是使用 CSV 檔案上傳使用者清單，來建立新的使用者。
    c) 在您建立新的使用者之後，返回 Azure 入口網站。 在 [訂用帳戶] 頁面中，選取相關的訂用帳戶。
    d) 選取 [存取控制 (IAM)]，然後按一下 [新增] 以使用相關的存取層級新增使用者。 在按一下存取層級之後開啟的頁面上，會自動顯示透過 CSP 入口網站建立的使用者。

      ![新增使用者](./media/vmware-azure-multi-tenant-csp-disaster-recovery/add-user-subscription.png)

- *參與者*角色就足以執行大多數的管理作業。 除了變更存取層級 (必須具有「擁有者」存取層級) 之外，具有此存取層級的使用者可以在訂用帳戶上執行任何作業。
- Site Recovery 也有三個[預先定義的使用者角色](site-recovery-role-based-linked-access-control.md)，可視需要用來進一步限制存取層級。

## <a name="multi-tenant-environments"></a>多租用戶環境

多租用戶模型主要有三種：

* **共用主機服務提供者 (HSP)**：合作夥伴擁有實體基礎結構，而使用者則共用資源 (如 vCenter、資料中心、實體儲存體等) 以將多租用戶的虛擬機器裝載在同一個基礎結構。 合作夥伴可提供災害復原管理做為受控服務，租用戶也可以擁有災害復原做為自助服務方案。

* **專用主機服務提供者**：合作夥伴擁有實體基礎結構，但使用專用資源 (如多個 vCenters、實體資料存放區等) 在個別的基礎結構上裝載每個租用戶的虛擬機器。 合作夥伴可提供災害復原管理做為受控服務，租用戶也可以擁有災害復原做為自助服務方案。

* **受控服務提供者 (MSP)** – 客戶擁有裝載虛擬機器的實體基礎結構，而合作夥伴則提供災害復原支援及管理。

依照本文所述，透過設定租用戶訂用帳戶，您可以快速開始在任何相關的多租用戶模型中啟用客戶。 您可以在[這裡](vmware-azure-multi-tenant-overview.md)進一步了解不同的多租用戶模型，和啟用內部部署存取控制的資訊。

## <a name="next-steps"></a>後續步驟
- 深入了解如何使用[角色型存取控制](site-recovery-role-based-linked-access-control.md)來管理 Azure Site Recovery 部署。
- 深入了解 VMware 至 Azure [複寫架構](vmware-azure-architecture.md)。
- 針對將 VMware VM 複寫到 Azure [檢閱教學課程](vmware-azure-tutorial.md)。
深入了解[多租用戶環境](vmware-azure-multi-tenant-overview.md)，以將 VMware VM 複寫至 Azure。
