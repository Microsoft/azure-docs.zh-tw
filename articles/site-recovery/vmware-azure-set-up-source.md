---
title: 設定以 Azure Site Recovery 從 VMware 複寫到 Azure 的來源環境 | Microsoft Docs
description: 本文說明如何設定您的內部部署環境，以利用 Azure Site Recovery 將 VMware 虛擬機器複寫到 Azure。
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: ramamill
ms.openlocfilehash: 1380c1bc820a815fae317a86fcd0ee4f46dd9aa5
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37952649"
---
# <a name="set-up-the-source-environment-for-vmware-to-azure-replication"></a>設定從 VMware 複寫到 Azure 的來源環境

本文說明如何設定您的來源內部部署環境，將 VMware 虛擬機器複寫到 Azure。 這包括選取您的複寫案例、將內部部署機器設定為 Site Recovery 組態伺服器以及自動探索內部部署 VM 等步驟。 

## <a name="prerequisites"></a>先決條件

本文假設您已經：
- 在 [Azure 入口網站](http://portal.azure.com)中[設定資源](tutorial-prepare-azure.md)。
- [設定內部部署 VMware](vmware-azure-tutorial-prepare-on-premises.md)，包括進行自動探索的專用帳戶。



## <a name="choose-your-protection-goals"></a>選擇您的保護目標

1. 在 Azure 入口網站中，移至 [復原服務] 保存庫刀鋒視窗，然後選取您的保存庫。
2. 在保存庫的 [資源] 功能表上，移至 [快速入門] > [Site Recovery] > [步驟 1: 準備基礎結構] > [保護目標]。

    ![選擇目標](./media/vmware-azure-set-up-source/choose-goals.png)
3. 在 [保護目標] 中選取 [至 Azure (To Azure)]，然後選擇 [是，使用 VMware vSphere Hypervisor (Yes, with VMware vSphere Hypervisor)]。 然後按一下 [確定] 。

    ![選擇目標](./media/vmware-azure-set-up-source/choose-goals2.png)

## <a name="set-up-the-configuration-server"></a>設定組態伺服器

您可以透過開放式虛擬化應用程式 (OVF) 範本，將組態伺服器設定為內部部署 VMware VM。 [深入了解](concepts-vmware-to-azure-architecture.md)要安裝在 VMware VM 上的元件。

1. 深入了解組態伺服器部署的[必要條件](vmware-azure-deploy-configuration-server.md#prerequisites)。
2. [檢查部署的容量數字](vmware-azure-deploy-configuration-server.md#capacity-planning)。
3. [下載](vmware-azure-deploy-configuration-server.md#download-the-template)和[匯入](vmware-azure-deploy-configuration-server.md#import-the-template-in-vmware) OVF 範本來設定執行組態伺服器的內部部署 VMware VM。 範本所提供的授權屬於評估授權，有效期為 180 天。 在此期間過後，客戶就必須使用所購買的授權來啟用時段。
4. 開啟 VMware VM，並在復原服務保存庫中[註冊它](vmware-azure-deploy-configuration-server.md#register-the-configuration-server-with-azure-site-recovery-services)。


## <a name="add-the-vmware-account-for-automatic-discovery"></a>新增用於自動探索的 VMware 帳戶

[!INCLUDE [site-recovery-add-vcenter-account](../../includes/site-recovery-add-vcenter-account.md)]

## <a name="connect-to-the-vmware-server"></a>連線至 VMware 伺服器

若要允許 Azure Site Recovery 探索在您內部部署環境中執行的虛擬機器，您必須將 VMware vCenter Server 或 vSphere ESXi 主機與 Site Recovery 連接。

選取 [+vCenter] 來開始連接 VMware vCenter Server 或 VMware vSphere ESXi 主機。

[!INCLUDE [site-recovery-add-vcenter](../../includes/site-recovery-add-vcenter.md)]


## <a name="common-issues"></a>常見問題
[!INCLUDE [site-recovery-vmware-to-azure-install-register-issues](../../includes/site-recovery-vmware-to-azure-install-register-issues.md)]


## <a name="next-steps"></a>後續步驟
在 Azure 中[設定您的目標環境](./vmware-azure-set-up-target.md)。
