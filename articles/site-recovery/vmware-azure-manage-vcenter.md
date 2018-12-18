---
title: " 在 Azure Site Recovery 中管理 VMware vCenter Server | Microsoft Doc"
description: 本文說明如何在 Azure Site Recovery 中新增並管理 VMware vCenter。
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.devlang: na
ms.topic: conceptual
ms.date: 06/20/2018
ms.author: ramamill
ms.openlocfilehash: 6f3edf8e5d7a6fda1795991ac0a21cc316c29414
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37950439"
---
# <a name="manage-vmware-vcenter-servers"></a>管理 VMware vCenter Server 

本文討論可以在 VMware vCenter 上執行的各種 Site Recovery 作業。 開始之前，請確認[必要條件](vmware-physical-azure-support-matrix.md#replicated-machines)。


## <a name="set-up-an-account-for-automatic-discovery"></a>設定帳戶以進行自動探索

Site Recovery 需要存取 VMware，才能讓處理序伺服器自動探索虛擬機器，以及針對虛擬機器進行容錯移轉和容錯回復。 建立帳戶以進行存取，如下所示：

1. 登入設定伺服器電腦。
2. 使用 [桌面] 捷徑開啟啟動 cspsconfigtool.exe。
3. 按一下 [管理帳戶] 索引標籤上的 [新增帳戶]。

  ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)
1. 提供帳戶詳細資料，並按一下 [確定] 加以新增。  帳戶應該具有下表摘要說明的權限。 

帳戶資訊需要花費約 15 分鐘的時間以和 Site Recovery 服務進行同步處理。

### <a name="account-permissions"></a>帳戶權限

|**Task** | **帳戶** | **權限** | **詳細資料**|
|--- | --- | --- | ---|
|**自動探索/遷移 (不含容錯回復)** | 您需要至少一個唯讀使用者 | 資料中心物件 –> 傳播至子物件、role=Read-only | 在資料中心層級指派的使用者，且能夠存取資料中心內的所有物件。<br/><br/> 如果要限制存取權，請將具備 [傳播至子物件] 物件的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、虛擬機器及網路)。|
|**複寫/容錯移轉** | 您需要至少一個唯讀使用者| 資料中心物件 –> 傳播至子物件、role=Read-only | 在資料中心層級指派的使用者，且能夠存取資料中心內的所有物件。<br/><br/> 如果要限制存取權，請將具備 [傳播至子物件] 物件的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、虛擬機器及網路)。<br/><br/> 適用於移轉用途，而不是完整複寫、容錯移轉、容錯回復。|
|**複寫/容錯移轉/容錯回復** | 建議您建立具有必要權限的角色 (AzureSiteRecoveryRole)，然後將角色指派給 VMware 使用者或群組 | 資料中心物件 –> 傳播至子物件、role=AzureSiteRecoveryRole<br/><br/> 資料存放區 -> 配置空間、瀏覽資料存放區、底層檔案作業、移除檔案、更新虛擬機器檔案<br/><br/> 網路 -> 網路指派<br/><br/> 資源 -> 指派 VM 至資源集區、移轉已關閉電源的 VM、移轉已開啟電源的 VM<br/><br/> 工作 -> 建立工作、更新工作<br/><br/> 虛擬機器 -> 組態<br/><br/> 虛擬機器 -> 互動 -> 回答問題、裝置連線、設定 CD 媒體、設定磁碟片媒體、電源關閉、電源開啟、VMware 工具安裝<br/><br/> 虛擬機器 -> 清查 -> 建立、註冊、取消註冊<br/><br/> 虛擬機器 -> 佈建 -> 允許虛擬機器下載、允許虛擬機器檔案上傳<br/><br/> 虛擬機器 -> 快照 -> 移除快照 | 在資料中心層級指派的使用者，且能夠存取資料中心內的所有物件。<br/><br/> 如果要限制存取權，請將具備 [傳播至子物件] 物件的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、虛擬機器及網路)。|


## <a name="add-vmware-server-to-the-vault"></a>將 VMware 伺服器新增至保存庫

1. 在 Azure 入口網站中，開啟您的保存庫 > [Site Recovery 基礎結構] > [設定伺服器]，然後開啟設定伺服器。
2. 在 [詳細資料] 頁面上，按一下 [+vCenter]。

[!INCLUDE [site-recovery-add-vcenter](../../includes/site-recovery-add-vcenter.md)]

## <a name="modify-credentials"></a>修改認證

修改用來連線至 vCenter Server 或 ESXi 主機的認證，如下所示：

1. 登入設定伺服器，並從桌面啟動 cspsconfigtool.exe。
2. 按一下 [管理帳戶] 索引標籤上的 [新增帳戶]。

  ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)
3. 提供新帳戶的詳細資料，並按一下 [確定] 加以新增。 該帳戶必須具備[上方](#account-permissions)所列的權限。
4. 在 Azure 入口網站中，開啟保存庫 > [Site Recovery 基礎結構] > [設定伺服器]，然後開啟設定伺服器。
5. 在 [詳細資料] 頁面上，按一下 [重新整理伺服器]。
6. [重新整理伺服器] 作業完成後，選取 [vCenter Server] 以開啟 vCenter [摘要] 頁面。
7. 在 [vCenter Server/vSphere 主機帳戶] 欄位中選取新增的帳戶，然後按一下 [儲存]。

    ![modify-account](./media/vmware-azure-manage-vcenter/modify-vcente-creds.png)

## <a name="delete-a-vcenter-server"></a>刪除 vCenter Server

1. 在 Azure 入口網站中，開啟您的保存庫 > [Site Recovery 基礎結構] > [設定伺服器]，然後開啟設定伺服器。
2. 在 [詳細資料] 頁面中，選取 vCenter Server。
3. 按一下 [刪除] 按鈕。

  ![delete-account](./media/vmware-azure-manage-vcenter/delete-vcenter.png)

> [!NOTE]
如果您需要修改 vCenter IP 位址、FQDN 或連接埠，則必須將 vCenter Server 刪除，再將它新增回入口網站。
