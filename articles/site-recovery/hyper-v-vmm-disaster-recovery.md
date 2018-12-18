---
title: 使用 Azure Site Recovery 設定內部部署網站間的 Hyper-V VM 災害復原 | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 設定內部部署網站間的 Hyper-V VM 災害復原。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/10/2018
ms.author: raynew
ms.openlocfilehash: b8bc34d3786293aeae2c184ce5c44f4622ec57f1
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078591"
---
# <a name="set-up-disaster-recovery-for-hyper-v-vms-to-a-secondary-on-premises-site"></a>設定從 Hyper-V VM 到次要內部部署網站的災害復原

[Azure Site Recovery](site-recovery-overview.md) 服務可藉由管理及協調內部部署電腦與 Azure 虛擬機器 (VM) 的複寫、容錯移轉及容錯回復，為您的災害復原策略做出貢獻。

本文說明如何設定從 System Center Virtual Machine Manager (VMM) 雲端中管理的內部部署 Hyper-V VM 到次要網站的災害復原。 在本文中，您將了解：

> [!div class="checklist"]
> * 準備內部部署 VMM 伺服器和 Hyper-V 主機
> * 建立 Site Recovery 的復原服務保存庫 
> * 設定來源和目標複寫環境。 
> * 設定網路對應 
> * 建立複寫原則
> * 啟用 VM 複寫

## <a name="prerequisites"></a>必要條件

若要完成此案例：

- 檢閱[案例架構和元件](hyper-v-vmm-architecture.md)。
- 請確定 VMM 伺服器和 Hyper-V 主機符合[支援需求](hyper-v-vmm-secondary-support-matrix.md)。
- 檢查您要複寫的 VM 符合[複寫的機器支援](hyper-v-vmm-secondary-support-matrix.md#replicated-vm-support)。
- 準備 VMM 伺服器以進行網路對應。

### <a name="prepare-for-network-mapping"></a>準備網路對應

[網路對應](hyper-v-vmm-network-mapping.md)會在來源與目標雲端中的內部部署 VMM VM 網路之間對應。 對應具有下列功能：

- 在容錯移轉之後將 VM 連線至適當的目標 VM 網路。 
- 以最佳方式將複本 VM 放在目標 Hyper-V 主機伺服器上。 
- 如果您未設定網路對應，複本 VM 將不會在容錯移轉之後連線到 VM 網路。

準備 VMM，如下所示：

1. 確定您在來源和目標 VMM 伺服器上有 [VMM 邏輯網路](https://docs.microsoft.com/system-center/vmm/network-logical)。
    - 來源伺服器上的邏輯網路應該與 Hyper-V 主機所在的來源雲端相關聯。
    - 目標伺服器上的邏輯網路應該與目標雲端相關聯。
1. 確定您在來源和目標 VMM 伺服器上有 [VMM 網路](https://docs.microsoft.com/system-center/vmm/network-virtual)。 VM 網路應連結至每個位置的邏輯網路。
2. 將來源 Hyper-V 主機上的 VM 連線至來源 VM 網路。 


## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫

[!INCLUDE [site-recovery-create-vault](../../includes/site-recovery-create-vault.md)]


## <a name="choose-a-protection-goal"></a>選擇保護目標

選取您要複寫的項目以及您要複寫到的位置。

1. 按一下 [Site Recovery] > [步驟 1: 準備基礎結構] > [保護目標]。
2. 選取 [到復原網站]，然後選取 [是，利用 Hyper-V]。
3. 選取 [是]，表示您使用 VMM 來管理 Hyper-V 主機。
4. 如果您有次要 VMM 伺服器，請選取 [是]。 如果您要在單一 VMM 伺服器上的雲端之間部署複寫，請按一下 [否] 。 然後按一下 [確定] 。


## <a name="set-up-the-source-environment"></a>設定來源環境

在 VMM 伺服器上安裝 Azure Site Recovery Provider，並在保存庫中探索和註冊伺服器。

1. 按一下 [準備基礎結構]  >  [來源]。
2. 在 [準備來源] 中，按一下 [+ VMM] 以新增 VMM 伺服器。
3. 在 [新增伺服器] 中，檢查 [System Center VMM 伺服器] 是否出現在 [伺服器類型] 中。
4. 下載 Azure Site Recovery Provider 安裝檔案。
5. 下載註冊金鑰。 您在安裝 Provider 時需要此金鑰。 該金鑰在產生後會維持 5 天有效。

    ![設定來源](./media/hyper-v-vmm-disaster-recovery/source-settings.png)

6. 在每個 VMM 伺服器上安裝 Provider。 您不需要明確地在 Hyper-V 主機上安裝任何項目。

### <a name="install-the-azure-site-recovery-provider"></a>安裝 Azure Site Recovery 提供者

1. 在每部 VMM 伺服器上執行 Provider 安裝檔案。 如果 VMM 部署在叢集中，請在第一次時安裝，如下所示︰
    -  將 Provider 安裝在作用中的節點上，並完成安裝以在保存庫中註冊該 VMM 伺服器。
    - 然後在其他節點上安裝 Provider。 叢集節點全都應該執行相同版本的 Provider。
2. 安裝程式會執行幾項必要條件檢查，並要求停止 VMM 服務的權限。 當安裝完成之後，VMM 服務將會自動重新啟動。 如果您安裝在 VMM 叢集上，您會收到停止叢集角色的提示。
3. 在 [Microsoft Update] 中，可以選擇根據您的 Microsoft Update 原則來安裝提供者更新。
4. 在 [安裝] 中，接受或修改預設安裝位置，然後按一下 [安裝]。
5. 安裝完成之後，按一下 [註冊] 以在保存庫中註冊伺服器。

    ![安裝位置](./media/hyper-v-vmm-disaster-recovery/provider-register.png)
6. 在 [保存庫名稱] 中，確認要註冊伺服器的保存庫名稱。 按 [下一步] 。
7. 在 [Proxy 連線] 中，指定 VMM 伺服器上執行的 Provider 連線至 Azure 的方式。
   - 您可以指定提供者應該直接或透過 proxy 連接至網際網路。 視需要指定 Proxy 設定。
   - 如果您使用 proxy，則會使用指定的 proxy 認證自動建立 VMM RunAs 帳戶 (DRAProxyAccount)。 設定 proxy 伺服器，讓此帳戶可以成功進行驗證。 您可以在 VMM 控主台 > [設定] > [安全性] > [執行身分帳戶]中修改 RunAs 帳戶設定。
   - 重新啟動 VMM 服務以更新變更。
8. 在 [註冊金鑰] 中，選取您下載並複製到 VMM 伺服器的金鑰。
9. 加密設定不適用於此案例。 
10. 在 [伺服器名稱] 中，指定保存庫中 VMM 伺服器的易記識別名稱。 在叢集中，指定 VMM 叢集角色名稱。
11. 在 [同步處理雲端中繼資料] 中，選取您是否要同步處理 VMM 伺服器上所有雲端的中繼資料。 這個動作只需要在每個伺服器上進行一次。 如果您不要同步處理所有雲端，請勿勾選此設定。 您可以在 VMM 主控台的雲端屬性中，個別同步處理每個雲端。
12. 按 [下一步]  ，完成此程序。 註冊後，Site Recovery 即可從 VMM 伺服器擷取中繼資料。 伺服器將顯示於保存庫中的 [伺服器]  >  [VMM伺服器] 中。
13. 當伺服器出現在保存庫之後，於 [來源] > [準備來源] 中選取 VMM 伺服器，然後選取 Hyper-V 主機所在的雲端。 然後按一下 [確定] 。


## <a name="set-up-the-target-environment"></a>設定目標環境

選取目標 VMM 伺服器和雲端：

1. 按一下 [準備基礎結構] > [目標]，然後選取目標 VMM 伺服器。
2. 隨即顯示與 Site Recovery 同步處理的 VMM 雲端。 選取目標雲端。

   ![目標](./media/hyper-v-vmm-disaster-recovery/target-vmm.png)


## <a name="set-up-a-replication-policy"></a>設定複寫原則

開始之前，確定採用此原則的所有主機都具有相同的作業系統。 如果主機執行不同版本的 Windows Server，您會需要多個複寫原則。

1. 若要建立新的複寫原則，請按一下 [準備基礎結構] > [複寫設定] > [+建立及關聯]。
2. 在 [建立及關聯原則] 中指定原則名稱。 來源和目標類型應該是 **Hyper-V**。
3. 在 [Hyper-V 主機版本] 中，選取在主機上執行的作業系統。
4. 在 [驗證類型] 和 [驗證連接埠] 中，指定主要與復原 Hyper-V 主機伺服器之間流量的驗證方式。
    - 除非您有有效的 Kerberos 環境，否則請選取 [憑證]。 Azure Site Recovery 會自動設定用於 HTTPS 驗證的憑證。 您不需要手動執行任何動作。
    - 根據預設，連接埠 8083 和 8084 (用於憑證) 將在 Hyper-V 主機伺服器上的 Windows 防火牆中開啟。
    - 如果您確實選取 [Kerberos]，Kerberos 票證將會用於主機伺服器的相互驗證。 Kerberos 只適用於在 Windows Server 2012 R2 或更新版本上執行的 Hyper-V 主機伺服器。
1. 在 [複製頻率] 中，指定您要在初始複寫後複寫差異資料的頻率 (每隔 30 秒、5 或 15 分鐘)。
2. 在 [復原點保留] 中，針對每個復原點指定保留期的長度 (以小時為單位)。 複寫的機器可以復原到週期內的任意點。
3. 在 [應用程式一致快照頻率] 中，指定建立包含應用程式一致快照之復原點的頻率 (1-12 小時)。 Hyper-V 使用兩種快照集：
    - **標準快照集**：提供整個虛擬機器的累加快照集。
    - **應用程式一致快照集**：建立 VM 內應用程式資料的時間點快照集。 磁碟區陰影複製服務(VSS) 可確保在建立快照集時，應用程式處於一致狀態。 啟用應用程式一致快照集，會影響來源 VM 上的應用程式效能。 設定一個值，此值小於您設定的其他復原點數目。
4. 在 [資料傳輸壓縮] 中，指定是否應該壓縮已傳輸的複寫資料。
5. 選取 [刪除複本 VM]，以指定如果您停用對來源 VM 的保護，則應刪除複本虛擬機器。 如果啟用此設定，當您停用對來源 VM 的保護時，便會從 Site Recovery 主控台中加以移除、在 VMM 主控台中移除 VMM 的 Site Recovery 設定，並刪除複本。
6. 在 [初始複寫方法] 中，如果您要透過網路進行複寫，請指定是否要啟動初始複寫，或將它排程。 若要節省網路頻寬，您可能要將它排程在離峰時間執行。 然後按一下 [確定] 。

     ![複寫原則](./media/hyper-v-vmm-disaster-recovery/replication-policy.png)
     
7. 新原則會自動與 VMM 雲端產生關聯。 在 [複寫原則] 中，按一下 [確定]。 


## <a name="enable-replication"></a>啟用複寫

1. 按一下 [複寫應用程式] > [來源]。 
2. 在 [來源] 中，選取 VMM 伺服器和您要複寫的 Hyper-V 主機所在的雲端。 然後按一下 [確定] 。
3. 在 [目標] 中，確認次要 VMM 伺服器和雲端。
4. 在 [虛擬機器] 中，從清單中選取您要保護的 VM。


您可以在 [作業] > [Site Recovery 作業] 中，追蹤 [啟用保護] 動作的進度。 在**完成保護**作業完成後，即完成初始複寫，且 VM 已可進行容錯移轉。

## <a name="next-steps"></a>後續步驟

[執行災害復原演練](hyper-v-vmm-test-failover.md)
