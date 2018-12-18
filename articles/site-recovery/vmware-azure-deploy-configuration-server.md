---
title: 使用 Azure Site Recovery 部署 VMware 災害復原的組態伺服器 | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery 部署 VMware 災害復原的組態伺服器
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.openlocfilehash: 4222214705c42fe09d90d77faa7be63cc2a13206
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44025271"
---
# <a name="deploy-a-configuration-server"></a>部署設定伺服器

當您使用 [Azure Site Recovery](site-recovery-overview.md) 將 VMware VM 和實體伺服器災害復原到 Azure 時，便會部署內部部署設定伺服器。 設定伺服器會協調內部部署 VMware 與 Azure 之間的通訊。 它也會管理資料複寫。 本文會逐步引導您完成當您將 VMware VM 複寫至 Azure 時部署設定伺服器所需的步驟。 如果您必須設定實體伺服器複寫的設定伺服器，請[遵循這篇文章](physical-azure-set-up-source.md)。

>[!TIP]
您可以從[這裡](vmware-azure-architecture.md)了解組態伺服器在 Azure Site Recovery 架構中扮演的角色。

## <a name="deployment-of-configuration-server-through-ova-template"></a>透過 OVA 範本部署設定伺服器

設定伺服器必須搭配特定的最基本硬體和大小需求來設定為高可用性 VMware VM。 為了能夠便利且輕鬆地進行部署，Site Recovery 提供一個可下載的 OVA (開放式虛擬化應用程式) 範本，以設定能符合下列所有規定需求的設定伺服器。

## <a name="prerequisites"></a>必要條件

下表彙總了設定伺服器的最基本硬體需求。

[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

## <a name="capacity-planning"></a>容量規劃

組態伺服器的大小需求取決於可能的資料變更率。 請使用下表作為指南。

| **CPU** | **記憶體** | **快取磁碟大小** | **資料變更率** | **受保護的機器** |
| --- | --- | --- | --- | --- |
| 8 個 vCPU (2 個插槽 * 4 核心 \@ 2.5GHz) |16 GB |300 GB |500 GB 或更少 |複寫少於 100 部電腦。 |
| 12 個 vCPU (2 個插槽 * 6 核心 \@ 2.5GHz) |18 GB |600 GB |500 GB 至 1 TB |複寫 100-150 部機器。 |
| 16 個 vCPU (2 個插槽 * 8 核心 \@ 2.5GHz) |32 GB |1 TB |1 TB 至 2 TB |複寫 150-200 部機器。 |

如果您要複寫多個 VMware VM，請參閱[容量規劃考量](site-recovery-plan-capacity-vmware.md)。 執行用於 VMware 複寫的[部署規劃工具](site-recovery-deployment-planner.md)。

## <a name="download-the-template"></a>下載範本

1. 在保存庫中，移至 [準備基礎結構] > [來源]。
2. 在 [準備來源] 中，選取 [+設定伺服器]。
3. 在 [新增伺服器] 中，檢查 [VMware 的組態伺服器] 是否出現在 [伺服器類型] 中。
4. 下載適用於設定伺服器的「開放式虛擬化應用程式」(OVA) 範本。

  > [!TIP]
>您也可以直接從 [Microsoft 下載中心](https://aka.ms/asrconfigurationserver)下載最新版的設定伺服器範本。

>[!NOTE]
OVA 範本隨附的授權是有效期為 180 天的評估授權。 在此期間過後，客戶就必須使用所購買的授權來啟用時段。

## <a name="import-the-template-in-vmware"></a>在 VMware 中匯入範本

1. 使用 VMWare vSphere 用戶端，登入 VMware vCenter 伺服器或 vSphere ESXi 主機。
2. 在 [檔案] 功能表上，選取 [部署 OVF 範本] 以啟動 [部署 OVF 範本] 精靈。

     ![OVF 範本](./media/vmware-azure-deploy-configuration-server/vcenter-wizard.png)

3. 在 [選取來源] 中，輸入所下載 OVF 的位置。
4. 在 [檢閱詳細資料] 中，選取 [下一步]。
5. 在 [選取名稱和資料夾] 和 [選取設定] 中，接受預設設定。
6. 在 [選取儲存體] 中，於 [選取虛擬磁碟格式] 中選取 [Thick Provision Eager Zeroed]，以達到最佳效能。
7. 在精靈的其餘頁面中，接受所有的預設設定。
8. 在 [準備要完成] 中：

    * 若要使用預設設定來設定 VM，請選取 [在部署後開啟電源] > [完成]。

    * 若要新增額外的網路介面，請清除 [在部署後開啟電源]，然後選取 [完成]。 預設會使用單一 NIC 部署設定伺服器範本。 您可以在部署後新增其他 NIC。

## <a name="add-an-additional-adapter"></a>新增其他介面卡

如果您想要將其他 NIC 新增至設定伺服器，請在保存庫中註冊伺服器前新增。 註冊之後，便不支援新增其他介面卡。

1. 在 vSphere 用戶端詳細目錄中，以滑鼠右鍵按一下 VM 並選取 [編輯設定]。
2. 在 [硬體] 中，選取 [新增] > [乙太網路介面卡]。 然後，選取 [下一步]。
3. 選取介面卡類型和網路。
4. 若要在 VM 開啟時連線虛擬 NIC，請選取 [在電源開啟時連線]。 然後，選取 [下一步] > [完成] > [確定]。

## <a name="register-the-configuration-server-with-azure-site-recovery-services"></a>向 Azure Site Recovery 服務註冊設定伺服器

1. 從 VMWare vSphere 用戶端主控台開啟 VM。
2. VM 會開機進入 Windows Server 2016 安裝體驗。 接受授權合約，並輸入系統管理員密碼。
3. 在安裝完成之後，以系統管理員身分登入 VM。
4. 您第一次登入時，「Azure Site Recovery 設定工具」在幾秒內就會啟動。
5. 輸入用來向 Site Recovery 註冊設定伺服器的名稱。 然後，選取 [下一步]。
6. 此工具會檢查 VM 是否可連線到 Azure。 建立連線之後，選取 [登入] 以登入您的 Azure 訂用帳戶。 認證必須能夠存取您要在其中註冊組態伺服器的保存庫。
7. 此工具會執行一些設定工作，而後重新開機。
8. 再次登入機器。 設定伺服器管理精靈會在幾秒內**自動**啟動。

### <a name="configure-settings"></a>配置設定

1. 在設定伺服器管理精靈中，選取 [設定連線]，然後選取 NIC，處理序伺服器會用其來接收來自虛擬機器的複寫流量。 然後選取 [儲存]。 在設定後，您便無法變更此設定。
2. 在 [選取復原服務保存庫] 中，登入 Microsoft Azure，選取您的 Azure 訂用帳戶，以及相關的資源群組和保存庫。

    > [!NOTE]
    > 復原服務保存庫在註冊之後，即沒有變更的彈性。

3. 在 [安裝協力廠商軟體] 中，

    |案例   |要依循的步驟  |
    |---------|---------|
    |是否可以下載並手動安裝 MySQL？     |  是。 請下載 MySQL 並將它放在資料夾 **C:\Temp\ASRSetup** 資料夾中，然後手動安裝。 現在，當您接受條款 > 按一下 [下載並安裝] 時，入口網站會顯示「已經安裝」。 您可以繼續進行下一個步驟。       |
    |是否可以避免線上下載 MySQL？     |   是。 請將您的 MySQL 安裝程式應用程式放在 **C:\Temp\ASRSetup** 資料夾中。 接受條款 > 按一下 [下載並安裝]，入口網站將會使用您新增的安裝程式並安裝應用程式。 您可以繼續進行安裝後的下一個步驟。    |
    |我想要透過 Azure Site Recovery 下載並安裝 MySQL     |  接受授權合約，然後按一下 [下載並安裝]。 接著，您可以繼續進行安裝後的下一個步驟。       |
4. 在 [驗證設備設定] 中，必要條件會在您繼續之前進行驗證。
5. 在 [設定 vCenter 伺服器/vSphere ESXi 伺服器] 中，輸入 vCenter 伺服器或 vSphere 主機 (您要複寫的 VM 位於其上) 的 FQDN 或 IP 位址。 輸入伺服器所接聽的連接埠。 輸入要用於保存庫中 VMware 伺服器的易記名稱。
6. 輸入供設定伺服器用來連線至 VMware 伺服器的認證。 Site Recovery 會使用這些認證來自動探索可用於複寫的 VMware VM。 選取 [新增]，然後選取 [繼續]。 在這裡輸入的認證會儲存在本機。
7. 在 [設定虛擬機器認證] 中，輸入虛擬機器的使用者名稱和密碼，以在複寫期間自動安裝「行動服務」。 針對 **Windows** 電腦，此帳戶必須具備您要複寫之電腦的本機系統管理員權限。 針對 **Linux**，請提供根帳戶的詳細資料。
8. 選取 [完成設定] 以完成註冊。
9. 註冊完成之後，開啟 Azure 入口網站，確認 [復原服務保存庫] > [管理] > [Site Recovery 基礎結構] > [設定伺服器] 上已列出設定伺服器和 VMware 伺服器。

## <a name="faq"></a>常見問題集

1. 是否可以將已安裝組態伺服器的 VM 用於不同用途？

    **否**，建議您使用 VM 作為設定伺服器的唯一用途。 請務必遵循[必要條件](#prerequisites)所述的所有規格，以便有效管理災害復原。
2. 是否可以將已在設定伺服器中註冊的保存庫與新建立的保存庫交換？

    **否**，在保存庫向設定伺服器註冊之後，即無法變更。
3. 是否可以使用同一部設定伺服器來保護實體和虛擬機器？

    **是**，可以使用同一部組態伺服器來複寫實體和虛擬機器。 不過，實體機器只能容錯回復到 VMware VM。
4. 組態伺服器的用途為何而其使用位置為何？

    請參考 [VMware 到 Azure 複寫架構](vmware-azure-architecture.md)，以深入了解組態伺服器及其功能。
5. 哪裡可以找到最新版的設定伺服器？

    如需透過入口網站升級組態伺服器的步驟，請參閱[升級組態伺服器](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server)。 您也可以從 [Microsoft 下載中心](https://aka.ms/asrconfigurationserver)直接下載。
6. 哪裡可以下載設定伺服器的複雜密碼？

    請參閱[本文](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase)以下載複雜密碼。
7. 哪裡可以下載保存庫註冊金鑰？

    在 [復原服務保存庫] 的 [管理] > [Site Recovery Infrastructure] \(Site Recovery 基礎結構\) > [設定伺服器] 中。 在 [伺服器] 中，選取 [下載註冊金鑰] 以下載保存庫認證檔案。

## <a name="upgrade-the-configuration-server"></a>升級設定伺服器

若要將組態伺服器升級為最新版本，請遵循[這裡](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server)的步驟。

## <a name="manage-the-configuration-server"></a>管理組態伺服器

若要避免進行中的複寫中斷，請確定已向保存庫註冊組態伺服器後，組態伺服器的 IP 位址不會變更。 您可在[此處](vmware-azure-manage-configuration-server.md)深入了解常見組態伺服器管理工作。

## <a name="troubleshoot-deployment-issues"></a>為部署問題進行疑難排解

[!INCLUDE [site-recovery-vmware-to-azure-install-register-issues](../../includes/site-recovery-vmware-to-azure-install-register-issues.md)]



## <a name="next-steps"></a>後續步驟

為 [VMware VM](vmware-azure-tutorial.md) 設定以 Azure 作為目標的災害復原。
