---
title: 準備目標環境 (VMware/實體至 Azure) | Microsoft Docs
description: 此文章說明如何準備您的目標 Azure 環境，用於將 VMware VM 和實體伺服器複寫至 Azure。
services: site-recovery
author: bsiva
manager: abhemraj
ms.service: site-recovery
ms.topic: article
ms.date: 09/28/2018
ms.author: bsiva
ms.openlocfilehash: 948812f05697362978ad041566d22977efec92a1
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434631"
---
# <a name="prepare-the-target-environment-vmwarephysical-to-azure"></a>準備目標環境 (VMware/實體至 Azure)

此文章將說明如何準備您的目標 Azure 環境，以開始將 VMware 虛擬機器或實體伺服器複寫至 Azure。

## <a name="prerequisites"></a>必要條件

此文章假設：
- 您已經在 [Azure 入口網站](http://portal.azure.com "Azure 入口網站")上建立復原服務保存庫以保護您的來源機器
- 您已經將內部部署環境設定為將來源 [VMware 虛擬機器](vmware-azure-set-up-source.md)或[實體伺服器](physical-azure-set-up-source.md)複寫至 Azure。

## <a name="prepare-target"></a>準備目標

完成**步驟 1：選取保護目標**和**步驟 2︰準備來源**之後，即會進入**步驟 3︰目標**

![準備目標](./media/vmware-azure-set-up-target/prepare-target-vmware-to-azure.png)

1. **訂用帳戶︰** 從下拉式功能表中選取您想要的訂用帳戶，以將虛擬機器或實體伺服器複寫至該訂用帳戶。
2. **部署模型︰** 選取部署模型 (傳統或資源管理員)

根據所選的部署模型，會執行驗證以確保您的目標訂用帳戶中有至少一個相容的儲存體帳戶和虛擬網路，以便將您的虛擬機器或實體伺服器複寫和容錯移轉至其中。

驗證成功完成後，按一下 [確定] 以移至下一個步驟。

如果您沒有相容的資源管理員儲存體帳戶或虛擬網路，您可以按一下頁面頂端的 [+ 儲存體帳戶] 或 [+ 網路] 按鈕來建立一個。

## <a name="next-steps"></a>後續步驟
[設定複寫設定](vmware-azure-set-up-replication.md)。
