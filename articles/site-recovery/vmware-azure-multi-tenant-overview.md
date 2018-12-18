---
title: 使用 Azure Site Recovery 將 VMware VM 複寫至 Azure (CSP) 的多租用戶支援概觀 | Microsoft Docs
description: 提供在多租用戶環境中，透過 CSP 方案對於租用戶訂用帳戶之 Azure Site Recovery 支援的概觀。
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.devlang: na
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: 4cd0c3baf385a864e2173de9bbce897bb5066dae
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352866"
---
# <a name="overview-of-multi-tenant-support-for-vmware-replication-to-azure-with-csp"></a>使用 CSP 將 VMware 複寫至 Azure 的多租用戶支援概觀

[Azure Site Recovery](site-recovery-overview.md) 支援租用戶訂用帳戶的多租用戶環境。 對於透過 Microsoft Cloud 解決方案提供者 (CSP) 方案建立及管理的租用戶訂用帳戶，它也支援多租用戶。

本文提供實作及管理多租用戶 VMware 到 Azure 複寫的概觀。

## <a name="multi-tenant-environments"></a>多租用戶環境

多租用戶模型主要有三種：

* **共用主機服務提供者 (HSP)**：合作夥伴擁有實體基礎結構，而使用者則共用資源 (如 vCenter、資料中心、實體儲存體等) 以將多租用戶的虛擬機器裝載在同一個基礎結構。 合作夥伴可提供災害復原管理做為受控服務，租用戶也可以擁有災害復原做為自助服務方案。

* **專用主機服務提供者**：合作夥伴擁有實體基礎結構，但使用專用資源 (如多個 vCenters、實體資料存放區等) 在個別的基礎結構上裝載每個租用戶的虛擬機器。 合作夥伴可提供災害復原管理做為受控服務，租用戶也可以擁有災害復原做為自助服務方案。

* **受控服務提供者 (MSP)** – 客戶擁有裝載虛擬機器的實體基礎結構，而合作夥伴則提供災害復原支援及管理。

## <a name="shared-hosting-services-provider-hsp"></a>共用主機服務提供者 (HSP)

另外兩個案例是共用主機案例的子集，且使用相同的原則。 共用主機指南最後面會詳述其中的差異。

多租用戶案例的基本需求是必須隔離租用戶。 一個租用戶無法觀察另一個租用戶裝載的內容。 這個需求在自助管理環境中，比在合作夥伴管理的環境中來得重要，甚至有重大影響。 本文假設租用戶隔離是必要條件。

下圖顯示這個架構。

![包含一個 vCenter 的共用 HSP](./media/vmware-azure-multi-tenant-overview/shared-hosting-scenario.png)  

**包含一個 vCenter 伺服器的共用主機**

在圖表中，每個客戶都有個別的管理伺服器。 這個設定可以將租用戶限制於只能存取租用戶特定 VM，以達到租用戶隔離的目的。 VMware VM 複寫會使用組態伺服器來探索 VM，並安裝代理程式。 相同的原則適用於多租用戶環境，但是另外使用 vCenter 存取控制來限制 VM 探索。

資料隔離需求表示所有基礎結構機密資訊 (如存取認證) 對租用戶保持公開。 基於這個理由，建議所有管理伺服器的元件都維持在合作夥伴的獨佔控制之下。 管理伺服器元件包括：

* 組態伺服器
* 處理序伺服器
* 主要目標伺服器

個別相應放大處理序伺服器也是在合作夥伴的控制之下。

## <a name="configuration-server-accounts"></a>組態伺服器帳戶

多租用戶案例中的每個組態伺服器都使用兩個帳戶：

- **vCenter 存取帳戶**：此帳戶可用來探索租用戶 VM。 它有指派給它的 vCenter 存取權限。 為了避免存取權限洩漏，建議合作夥伴自行在組態工具中輸入這些認證。

- **虛擬機器存取帳戶**：此帳戶是用來在租用戶 VM 上，透過自動推送安裝行動服務代理程式。 這通常是租用戶可能會提供給合作夥伴的網域帳戶，或可能由合作夥伴直接管理的帳戶。 如果租用戶不想直接與合作夥伴分享詳細資料，他們可以透過限時的組態伺服器存取權來輸入認證。 或者，也可以在合作夥伴的協助下手動安裝行動服務代理程式。

## <a name="vcenter-account-requirements"></a>vCenter 帳戶需求

使用獲得特殊角色指派的帳戶來設定組態伺服器。

- 針對每個 vCenter 物件，必須將角色指派套用於 vCenter 存取帳戶，而且不要傳播到子物件。 此設定可以確保租用戶隔離，因為存取傳播可能會導致意外存取其他物件。

    ![傳播到子物件選項](./media/vmware-azure-multi-tenant-overview/assign-permissions-without-propagation.png)

- 替代方法是在資料中心物件指派使用者帳戶和角色，並將這些傳播到子物件。 然後將每個物件 (例如屬於其他租用戶的 VM) 的**無存取**角色，授與應該無法存取特定租用戶的帳戶。 此組態相當繁瑣。 它會意外公開存取控制，因為會對每個新的子物件自動授與從父物件繼承的存取權。 因此建議採用第一種方法。

### <a name="create-a-vcenter-account"></a>建立 vCenter 帳戶

1. 藉由複製預先定義的「唯讀」角色來建立新的角色，並以易記名稱 (例如此範例中顯示的 Azure_Site_Recovery) 命名。
2. 將下列權限指派給這個角色：

    * **資料存放區**：配置空間、瀏覽資料存放區、底層檔案作業、移除檔案、更新虛擬機器檔案
    * **網路**：網路指派
    * **資源**：指派虛擬機器至資源集區、移轉已關閉電源的虛擬機器、移轉已開啟電源的虛擬機器
    * **工作**：建立工作、更新工作
    * **VM - 組態**：所有
    - **VM - 互動** > 回答問題、裝置連線、設定 CD 媒體、設定磁碟片媒體、電源關閉、電源開啟、VMware 工具安裝
    - **VM - 清查** > 從現有建立、建立新的、註冊、取消註冊
    - **VM - 佈建** > 允許虛擬機器下載、允許虛擬機器檔案上傳
    - **VM - 快照集管理** > 移除快照集

        ![[編輯角色] 對話方塊](./media/vmware-azure-multi-tenant-overview/edit-role-permissions.png)

3. 針對不同物件，指派存取層級給 vCenter 帳戶 (在租用戶組態伺服器中使用)：

>| Object | 角色 | 備註 |
>| --- | --- | --- |
>| vCenter | 唯讀 | 只用以允許 vCenter 存取以管理其他物件。 如果帳戶不會提供給租用戶，或用於任何 vCenter 上的管理作業，則可以移除此權限。 |
>| 資料中心 | Azure_Site_Recovery |  |
>| 主機和主機叢集 | Azure_Site_Recovery | 再次確認存取權在物件層級，以限制只有可存取的主機在容錯移轉前和容錯回復後有租用戶虛擬機器。 |
>| 資料存放區和資料存放區叢集 | Azure_Site_Recovery | 同上。 |
>| 網路 | Azure_Site_Recovery |  |
>| 管理伺服器 | Azure_Site_Recovery | 包含 CS 以外所有元件 (CS、PS 及 MT) 的存取權。 |
>| 租用戶 VM | Azure_Site_Recovery | 確保特定租用戶的任何新租用戶虛擬機器也會有此存取權，否則將無法透過 Azure 入口網站探索它們。 |

現在已經完成 vCenter 帳戶存取權。 這個步驟可以滿足完成容錯回復作業的最小權限需求。 也可以搭配現有的原則使用這些存取權限。 只要將現有的權限集修改成包含上面步驟 2 詳述的角色權限即可。

### <a name="failover-only"></a>僅限容錯移轉
若要限制災害復原作業僅限容錯移轉 (亦即，不含容錯回復功能)，請使用先前的程序，但有下列例外狀況：

- 不要將 Azure_Site_Recovery 角色指派給 vCenter 存取帳戶，而是僅將「唯讀」角色指派給帳戶。 這個權限集合會允許虛擬機器複寫和容錯移轉，而不允許容錯回復。
- 以上程序的其餘部分都保持原狀。 為了確保租用戶隔離及限制虛擬機器探索，所有權限仍僅指派到物件層級而未傳播到子物件。

### <a name="deploy-resources-to-the-tenant-subscription"></a>將資源部署到租用戶訂用帳戶

1. 在 Azure 入口網站上，建立「資源群組」，然後對於每個一般程序部署「復原服務」保存庫。
2. 下載保存庫註冊金鑰。
3. 使用保存庫註冊金鑰為租用戶註冊 CS。
4. 針對兩個存取帳戶輸入認證：用來存取 vCenter 伺服器的帳戶，以及用來存取 VM 的帳戶。

    ![管理員設定伺服器帳戶](./media/vmware-azure-multi-tenant-overview/config-server-account-display.png)

### <a name="register-servers-in-the-vault"></a>在保存庫中註冊伺服器

1. 在 Azure 入口網站您稍早建立的保存庫中，使用您建立的 vCenter 帳戶，向組態伺服器註冊 vCenter 伺服器。
2. 針對每個一般程序的 Site Recovery 完成「準備基礎結構」程序。
3. 現在可以開始複寫 VM。 確認 [複寫] > [選取虛擬機器] 僅顯示租用戶的 VM。

## <a name="dedicated-hosting-solution"></a>專用主機解決方案

如下圖所示，專用主機方案的架構差異是每個租用戶的基礎結構完全針對該租用戶設定。

![architecture-shared-hsp](./media/vmware-azure-multi-tenant-overview/dedicated-hosting-scenario.png)  
**含多個 vCenters 的專用主機案例**

## <a name="managed-service-solution"></a>受控服務解決方案

如下圖所示，受控服務方案本身的架構差異是每個租用戶的基礎結構在實體上與其他租用戶的基礎結構分開。 此案例通常是在租用戶擁有基礎結構時存在，而且需要解決方案提供者來管理災害復原。

![architecture-shared-hsp](./media/vmware-azure-multi-tenant-overview/managed-service-scenario.png)  
**含多個 vCenters 的受控服務案例**

## <a name="next-steps"></a>後續步驟
- [深入了解](site-recovery-role-based-linked-access-control.md) Site Recovery 中的角色型存取控制。
- 深入了解[為 VMware VM 設定以 Azure 作為目標的災害復原](vmware-azure-tutorial.md)。
- 深入了解[具有 VMWare VM 之 CSP 的多租用戶](vmware-azure-multi-tenant-csp-disaster-recovery.md)。
