---
title: 使用 Azure Site Recovery 在 Azure 中設定處理序伺服器用於 VMware VM 和實體伺服器的容錯回復 | Microsoft Docs
description: 本文說明如何在 Azure 中設定處理序伺服器，用於 Azure VMs 到 VMware 的容錯回復。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 10/10/2018
ms.author: raynew
ms.openlocfilehash: a54fe3099854e6e5200a50ec6ef5db1401a223bf
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077607"
---
# <a name="set-up-a-process-server-in-azure-for-failback"></a>在 Azure 中設定容錯回復的處理序伺服器

使用 [Site Recovery](site-recovery-overview.md) 將 VMware VM 或實體伺服器容錯移轉到 Azure 後，當它們又恢復運作時，可以再容錯回復到內部部署網站。 為了容錯回復，您必須在 Azure 中設定暫時處理序伺服器，以處理從 Azure 到內部部署的複寫。 容錯回復完成後，便可以刪除此 VM。

## <a name="before-you-start"></a>開始之前

深入了解[重新保護](vmware-azure-reprotect.md)和[容錯回復](vmware-azure-failback.md)程序。

[!INCLUDE [site-recovery-vmware-process-server-prerequ](../../includes/site-recovery-vmware-azure-process-server-prereq.md)]

## <a name="deploy-a-process-server-in-azure"></a>在 Azure 中部署處理序伺服器

1. 在保存庫中，依序選取 [Site Recovery 基礎結構]> [管理] > [組態伺服器]，然後按一下組態伺服器。
2. 在伺服器頁面中，按一下 [+ 處理序伺服器]。
3. 在 [新增處理序伺服器] 頁面中，選取在 Azure 中部署處理序伺服器。
4. 指定 Azure 的設定，包括用於容錯移轉的訂用帳戶、資源群組、用於容錯移轉的 Azure 區域、Azure VM 所在的虛擬網路。 如果您使用多個 Azure 網路，每個網路中都要有一個處理序伺服器。

  ![新增處理伺服器資源庫項目](./media/vmware-azure-set-up-process-server-azure/add-ps-page-1.png)

4. 在 [伺服器名稱]、[使用者名稱]、[密碼] 中，指定處理序伺服器的名稱，以及將指派為伺服器上系統管理員權限的認證。
5. 指定要用於伺服器 VM 磁碟的儲存體帳戶、處理序伺服器 VM 所在的子網路、以及在 VM 啟動時要指派的伺服器 IP 位址。
6. 按一下 [確定] 按鈕開始部署處理序伺服器 VM。

>

## <a name="registering-the-process-server-running-in-azure-to-a-configuration-server-running-on-premises"></a>將處理伺服器 (在 Azure 中執行) 註冊到組態伺服器 (在內部部署上執行)

處理序伺服器 VM 啟動並運作後，您必須向內部部署組態伺服器註冊它，做法如下：

[!INCLUDE [site-recovery-vmware-register-process-server](../../includes/site-recovery-vmware-register-process-server.md)]


