---
title: StorSimple 雲端設備 Update 3 | Microsoft Docs
description: 了解如何建立、部署和管理 Microsoft Azure 虛擬網路中的 StorSimple 雲端設備。 (適用於 StorSimple Update 3 和更新版本)。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/08/2017
ms.author: alkohli
ms.openlocfilehash: 33be58ae3ac5fcc8d0b35b240f9f378ccce134cc
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387679"
---
# <a name="deploy-and-manage-a-storsimple-cloud-appliance-in-azure-update-3-and-later"></a>部署和管理 Azure 中的 StorSimple 雲端設備 (Update 3 和更新版本)

## <a name="overview"></a>概觀

StorSimple 8000 系列雲端設備是 Microsoft Azure StorSimple 解決方案提供的另一項功能。 StorSimple 雲端設備會在 Microsoft Azure 虛擬網路中的虛擬機器上執行，您可以使用它將主機上的資料進行備份和複製。

本文說明在 Azure 中部署和管理 StorSimple 雲端設備的逐步程序。 閱讀本文之後，您將能夠：

* 了解雲端設備與實體裝置的差異。
* 能夠建立和設定雲端設備。
* 連線至雲端設備。
* 了解如何使用雲端設備。

本教學課程適用於所有執行 Update 3 和更新版本的 StorSimple 雲端設備。

#### <a name="cloud-appliance-model-comparison"></a>雲端設備模型比較

StorSimple 雲端設備可以在兩種模型中使用，標準 8010 (前身為 1100) 和進階 8020 (於 Update 2 引進)。 下表顯示兩個模型的比較。

| 裝置型號 | 8010<sup>1</sup> | 8020 |
| --- | --- | --- |
| **最大容量** |30 TB |64 TB |
| **Azure VM** |Standard_A3 (4 核心、7 GB 記憶體)| Standard_DS3 (4 核心、14 GB 記憶體)|
| **區域可用性** |所有 Azure 區域 |支援進階儲存體和 DS3 Azure VM 的 Azure 區域<br></br>使用[這份清單](https://azure.microsoft.com/regions/services/)，查看您的區域是否提供「虛擬機器 > DS 系列」和「儲存體 > 磁碟儲存體」。 |
| **儲存體類型** |將 Azure 標準儲存體使用於本機磁碟<br></br> 了解如何 [建立標準儲存體帳戶](../storage/common/storage-create-storage-account.md) |將 Azure 進階儲存體使用於本機磁碟<sup>2</sup> <br></br>了解如何[建立進階儲存體帳戶](../virtual-machines/windows/premium-storage.md) |
| **工作負載指引** |從備份的檔案的項目層級擷取 |雲端開發和測試案例 <br></br>低延遲和更高的效能工作負載<br></br>災害復原的次要裝置 |

<sup>1</sup> *前身為 1100*。

<sup>2</sup> *8010 和 8020 會將 Azure 標準儲存體使用於雲端層。差異僅存在於裝置內的本機層*。

## <a name="how-the-cloud-appliance-differs-from-the-physical-device"></a>雲端設備與實體裝置的差異

StorSimple 雲端設備是純軟體的 StorSimple 版本，在 Microsoft Azure 虛擬機器中的單一節點上執行。 雲端設備支援無法使用實體裝置時的災害復原案例。 雲端設備適合用於從備份擷取的項目等級、內部部署災害復原和雲端開發和測試案例。

#### <a name="differences-from-the-physical-device"></a>與實體裝置的差異

下表顯示 StorSimple 雲端設備和 StorSimple 實體裝置之間的一些主要差異。

|  | 實體裝置 | 雲端設備 |
| --- | --- | --- |
| **位置** |位於資料中心。 |在 Azure 中執行。 |
| **網路介面** |有六個網路介面：DATA 0 到 DATA 5。 |只有一個網路介面：DATA 0。 |
| **註冊** |在初始設定步驟期間註冊。 |註冊是個別的工作。 |
| **服務資料加密金鑰** |在實體裝置上重新產生金鑰，然後以新的金鑰更新雲端設備。 |無法從雲端設備重新產生。 |
| **支援的磁碟區類型** |支援固定在本機和分層的磁碟區。 |只支援分層的磁碟區。 |

## <a name="prerequisites-for-the-cloud-appliance"></a>雲端設備的必要條件

下列各節說明 StorSimple 雲端設備的設定必要條件。 部署雲端設備之前，請檢閱使用雲端設備的安全性考量。

[!INCLUDE [StorSimple Cloud Appliance security](../../includes/storsimple-8000-cloud-appliance-security.md)]

#### <a name="azure-requirements"></a>Azure 需求

佈建雲端設備之前，您需要在 Azure 環境中做好下列準備：

* 請確定您的資料中心已部署和執行 StorSimple 8000 系列實體裝置 (模型 8100 或 8600)。 使用與您意圖建立 StorSimple 雲端設備相同的 StorSimple 裝置管理員服務註冊此裝置。
* 針對雲端設備，[在 Azure 中設定虛擬網路](../virtual-network/manage-virtual-network.md#create-a-virtual-network)。 如果使用進階儲存體，您必須在支援進階儲存體的 Azure 區域中建立虛擬網路。 進階儲存體區域是對應到[依區域的 Azure 服務清單](https://azure.microsoft.com/regions/services/)中，磁碟儲存體資料列的區域。
* 建議您使用 Azure 提供的預設 DNS 伺服器，而非指定您自己的 DNS 伺服器名稱。 如果您的 DNS 伺服器名稱無效，或者如果 DNS 伺服器無法正確解析 IP 位址，則建立雲端設備將會失敗。
* 點對站及站對站都是選用的，但並非必要。 如有需要，您可以針對更進階的案例設定這些選項。
* 您可以將 [Azure 虛擬機器](../virtual-machines/virtual-machines-windows-quick-create-portal.md) (主機伺服器) 建立在可使用由雲端設備公開之磁碟區的虛擬網路中。 這些伺服器必須符合下列需求：

  * 是已安裝 iSCSI 啟動器軟體的 Windows 或 Linux VM。
  * 正在與雲端設備相同的虛擬網路中執行。
  * 能夠透過雲端設備的內部 IP 位址連線至雲端設備的 iSCSI 目標。
  * 確定您已在相同虛擬網路上設定 iSCSI 和雲端流量的支援。

#### <a name="storsimple-requirements"></a>StorSimple 需求

建立雲端設備之前，對 StorSimple 裝置管理員服務進行下列更新：

* 針對要作為雲端設備之主機伺服器的 VM，新增[存取控制記錄](storsimple-8000-manage-acrs.md)。
* 使用與雲端設備位於相同區域的[儲存體帳戶](storsimple-8000-manage-storage-accounts.md#add-a-storage-account)。 若儲存體帳戶位於不同區域，可能導致效能不佳。 您可以搭配雲端設備使用標準或進階儲存體帳戶。 如何建立[標準儲存體帳戶](../storage/common/storage-create-storage-account.md)或[進階儲存體帳戶](../virtual-machines/windows/premium-storage.md)的詳細資訊
* 針對建立雲端設備使用與用於資料之儲存體帳戶不同的儲存體帳戶。 使用相同的儲存體帳戶可能導致效能不佳。

開始之前，請確定您擁有下列資訊：

* 具有存取認證的 Azure 入口網站帳戶。
* 已註冊 StorSimple 裝置管理員服務之實體裝置的服務資料加密金鑰副本。

## <a name="create-and-configure-the-cloud-appliance"></a>建立和設定雲端設備

執行這些程序之前，請確定您符合[雲端設備的必要條件](#prerequisites-for-the-cloud-appliance)。

執行下列步驟來建立 StorSimple 雲端設備。

### <a name="step-1-create-a-cloud-appliance"></a>步驟 1：建立雲端設備

執行下列步驟來建立 StorSimple 雲端設備。

[!INCLUDE [Create a cloud appliance](../../includes/storsimple-8000-create-cloud-appliance-u2.md)]

如果此步驟中的雲端設備建立失敗，您可能沒有網際網路的連線能力。 如需詳細資訊，請在建立雲端設備時，移至[針對網際網路連線失敗進行疑難排解](#troubleshoot-internet-connectivity-errors)。

### <a name="step-2-configure-and-register-the-cloud-appliance"></a>步驟 2：設定和註冊雲端設備

開始此程序之前，請確定您擁有服務資料加密金鑰的副本。 當您向 StorSimple 裝置管理員服務註冊您的第一個 StorSimple 實體裝置時，會建立服務資料加密金鑰。 系統會指示您將它儲存在安全的位置。 如果您沒有服務資料加密金鑰的複本，就必須連絡 Microsoft 支援服務以尋求協助。

執行下列步驟來設定和註冊 StorSimple 雲端設備。

[!INCLUDE [Configure and register a cloud appliance](../../includes/storsimple-8000-configure-register-cloud-appliance.md)]

### <a name="step-3-optional-modify-the-device-configuration-settings"></a>步驟 3：(選擇性) 修改裝置組態設定

如果您想要使用 CHAP、StorSimple Snapshot Manager 或變更裝置系統管理員密碼，下一節會說明 StorSimple 雲端設備所需的裝置組態設定。

#### <a name="configure-the-chap-initiator"></a>設定 CHAP 起始端

此參數包含您的雲端設備 (目標) 預期從嘗試存取磁碟區的啟動器 (伺服器) 收到的認證。 在這個驗證過程中，啟動器將提供 CHAP 使用者名稱和 CHAP 密碼，以便向您的裝置識別其身分。 如需詳細步驟，請移至 [設定裝置的 CHAP](storsimple-8000-configure-chap.md#unidirectional-or-one-way-authentication)。

#### <a name="configure-the-chap-target"></a>設定 CHAP 目標

此參數包含的認證，是您的雲端設備在啟用 CHAP 功能的啟動器要求相互或雙向驗證時所使用的。 在這個驗證程序中，您的雲端設備將使用反向 CHAP 使用者名稱和反向 CHAP 密碼，向啟動器識別其身分。

> [!NOTE]
> CHAP 目標設定為全域設定。 套用這些設定時，與雲端設備連線的所有磁碟區都會使用 CHAP 驗證。

如需詳細步驟，請移至 [設定裝置的 CHAP](storsimple-8000-configure-chap.md#bidirectional-or-mutual-authentication)。

#### <a name="configure-the-storsimple-snapshot-manager-password"></a>設定 StorSimple Snapshot Manager 密碼

StorSimple Snapshot Manager 軟體位於您的 Windows 主機上，而且可讓系統管理員以本機和雲端快照的形式管理 StorSimple 裝置的備份。

> [!NOTE]
> 對雲端設備而言，您的 Windows 主機就是 Azure 虛擬機器。

當您在 StorSimple Snapshot Manager 中設定裝置時，系統將提示您提供 StorSimple 裝置 IP 位址和密碼，來驗證您的儲存裝置。 如需詳細步驟，請移至 [設定 StorSimple Snapshot Manager 密碼](storsimple-8000-change-passwords.md#set-the-storsimple-snapshot-manager-password)。

#### <a name="change-the-device-administrator-password"></a>變更裝置系統管理員密碼

當您使用 Windows PowerShell 介面來存取雲端設備時，需要輸入裝置系統管理員密碼。 為了確保資料的安全性，您必須變更此密碼後，才能使用雲端設備。 如需詳細步驟，請移至 [設定裝置系統管理員密碼](../storsimple/storsimple-8000-change-passwords.md#change-the-device-administrator-password)。

## <a name="connect-remotely-to-the-cloud-appliance"></a>遠端連線至雲端設備

預設不會啟用透過 Windows PowerShell 介面遠端存取您的雲端設備。 您必須先將雲端設備上的遠端管理啟用，才能在用來存取雲端設備的用戶端上加以啟用。

下列雙步驟程序描述如何從遠端連線到您的雲端設備。

### <a name="step-1-configure-remote-management"></a>步驟 1：設定遠端管理

執行下列步驟來設定 StorSimple 雲端設備的遠端管理。

[!INCLUDE [Configure remote management via HTTP for cloud appliance](../../includes/storsimple-8000-configure-remote-management-http-device.md)]

### <a name="step-2-remotely-access-the-cloud-appliance"></a>步驟 2：遠端存取雲端設備

將雲端設備上的遠端管理啟用後，使用 Windows PowerShell 遠端，從相同虛擬網路內部的另一個虛擬機器連線到該設備。 例如，您可以從已設定且用來連線至 iSCSI 的主機 VM 進行連線。 在大部分的部署中，您要開啟公用端點，才能存取可用來存取雲端設備的主機 VM。

> [!WARNING]
> **為了加強安全性，強烈建議您在連接到端點時應使用 HTTPS，並且在完成 PowerShell 遠端工作階段之後刪除端點。**

您必須遵循[遠端連線至 StorSimple 裝置](storsimple-8000-remote-connect.md)中的程序，設定雲端設備的遠端處理功能。

## <a name="connect-directly-to-the-cloud-appliance"></a>直接連線至雲端設備

您也可以直接連線至雲端設備。 若要從虛擬網路或 Microsoft Azure 環境以外的另一部電腦直接連線至雲端設備，就必須建立其他端點。

執行下列步驟，在雲端設備上建立公用端點。

[!INCLUDE [Create public endpoints on a cloud appliance](../../includes/storsimple-8000-create-public-endpoints-cloud-appliance.md)]

建議您從相同虛擬網路內部的另一部虛擬機器進行連接，因為此做法可將虛擬網路上的公用端點數降到最低。 在此情況下，透過遠端桌面工作階段連線到虛擬機器，然後依照您在區域網路上設定任何其他 Windows 用戶端的方式，將該虛擬機器進行設定即可。 您不需要附加公用連接埠號碼，因為該連接埠是已知的。

## <a name="get-private-ip-for-the-cloud-appliance"></a>取得雲端設備的私人 IP

若要將雲端應用裝置連線至相同虛擬網路中的主機伺服器，您需要雲端應用裝置的內部或私人 IP 位址。 執行下列步驟來取得雲端應用裝置的私人 IP 位址

1. 移至您雲端應用裝置的基礎虛擬機器。 虛擬機器與您的雲端應用裝置具有相同的名稱。 移至 [所有資源]，提供雲端應用裝置的名稱和訂用帳戶，然後選取作為虛擬機器的類型。 在所呈現的虛擬機器清單中，選取及按一下對應至雲端應用裝置的虛擬機器。

     ![選取您雲端應用裝置的虛擬機器](./media/storsimple-8000-cloud-appliance-u2/sca-vm.png)

2. 移至 [設定 > 網路]。 在右窗格中，您會看到雲端應用裝置的私人 IP 位址。 請將此記下。

    ![取得您雲端應用裝置的私人 IP 位址](./media/storsimple-8000-cloud-appliance-u2/sca-private-ip-vm-networking.png)

## <a name="work-with-the-storsimple-cloud-appliance"></a>使用 StorSimple 雲端設備

現在您已建立並設定 StorSimple 雲端設備，準備好開始使用它。 您可以使用雲端設備上的磁碟區容器、磁碟區及備份原則，如同實體 StorSimple 裝置上使用的一樣。 唯一的差異，在於您必須確認是從您的裝置清單中選取雲端設備。 請參閱[使用 StorSimple 裝置管理員服務來管理雲端設備](storsimple-8000-manager-service-administration.md)，以取得雲端設備各種管理工作的逐步程序。

下列各節將討論您在使用雲端設備時會遇到的一些差異。

### <a name="maintain-a-storsimple-cloud-appliance"></a>維護 StorSimple 雲端設備

由於它是純軟體裝置，相較於實體裝置的維護，雲端設備的維護可說是最基本的。

您不能更新雲端設備。 若要建立新的雲端設備，請使用最新版本的軟體。


### <a name="storage-accounts-for-a-cloud-appliance"></a>雲端設備的儲存體帳戶

建立儲存體帳戶的目的，是供 StorSimple 裝置管理員服務、雲端設備和實體裝置使用。 當您建立儲存體帳戶時，建議您在易記名稱中使用區域識別碼。 這有助於確保區域在所有系統元件中都是一致的。 在雲端設備中，所有元件都務必位於相同區域中，以防止發生效能問題。

如需逐步程序，請移至 [新增儲存體帳戶](storsimple-8000-manage-storage-accounts.md#add-a-storage-account)。

### <a name="deactivate-a-storsimple-cloud-appliance"></a>停用 StorSimple 雲端設備

停用雲端設備時，動作會將佈建它時所建立的 VM 和資源刪除。 停用雲端設備之後，就無法將它還原為先前的狀態。 將雲端設備停用之前，請確實將其依存的用戶端和主機停止或刪除。

停用雲端設備會導致下列動作：

* 會將雲端設備移除。
* 會將針對雲端設備建立的作業系統磁碟和資料磁碟移除。
* 保留在佈建期間建立的託管服務和虛擬網路。 如果您不使用它們，就應該手動加以刪除。
* 會將針對雲端設備建立的雲端快照集保留。

如需逐步程序，請移至 [停用及刪除 StorSimple 裝置](storsimple-8000-deactivate-and-delete-device.md)。

只要雲端設備在 StorSimple 裝置管理員服務刀鋒視窗上顯示為已停用，您就能從 [裝置] 刀鋒視窗的裝置清單中，將該雲端設備刪除。

### <a name="start-stop-and-restart-a-cloud-appliance"></a>將雲端設備啟動、停止和重新啟動
不同於 StorSimple 實體裝置，在 StorSimple 雲端設備上並沒有開啟電源或關閉電源按鈕可供使用。 但是，有時您可能需要將雲端設備停止並重新啟動。

若要將雲端設備啟動、停止和重新啟動，最簡單的方式是透過虛擬機器服務刀鋒視窗。 執行虛擬機器服務。 從 VM 清單中，找出對應至您雲端設備 (相同名稱) 的 VM，然後按一下 VM 名稱。 當您檢視虛擬機器刀鋒視窗時，雲端設備狀態為 [執行中]，因為它在建立之後，依預設會加以啟動。 您隨時都能啟動、停止及重新啟動虛擬機器。

[!INCLUDE [Stop and restart cloud appliance](../../includes/storsimple-8000-stop-restart-cloud-appliance.md)]

### <a name="reset-to-factory-defaults"></a>重設為原廠預設值
如果您決定想要重新設定全新的雲端設備，請將它停用並刪除，然後建立新的雲端設備。

## <a name="fail-over-to-the-cloud-appliance"></a>容錯移轉至雲端設備
災害復原 (DR) 是針對其設計 StorSimple 雲端設備的重要案例之一。 在此案例中，實體 StorSimple 裝置或整個資料中心可能無法使用。 幸運的是，您可以使用雲端設備，在替代位置中還原作業。 在 DR 期間，來源裝置的磁碟區容器會將擁有權變更，並移轉到雲端設備。

DR 的必要條件如下︰

* 會建立及設定雲端設備。
* 磁碟區容器內的所有磁碟區皆為離線。
* 您要容錯移轉的磁碟區容器具有相關聯的雲端快照集。

> [!NOTE]
> * 當使用雲端設備作為 DR 的次要裝置時，請記住，8010 有 30 TB 的標準儲存體，8020 有 64 TB 的進階儲存體。 較高容量的 8020 雲端設備可能比較適合 DR 案例。

如需逐步程序，請移至[容錯移轉至雲端設備](storsimple-8000-device-failover-cloud-appliance.md)。

## <a name="delete-the-cloud-appliance"></a>將雲端設備刪除
如果您先前已設定並使用 StorSimple 雲端設備，但現在不想再因為使用而產生計算費用，您就必須將雲端設備停止。 停止雲端設備會將 VM 取消配置。 這個動作將會停止您訂用帳戶上所產生的費用。 儲存體會收取作業系統費用，但資料磁碟將會繼續。

若要將所有費用停止，您就必須將雲端設備刪除。 若要將雲端設備所建立的備份刪除，您可以將裝置停用或刪除。 如需詳細資訊，請參閱 [停用及刪除 StorSimple 裝置](storsimple-8000-deactivate-and-delete-device.md)。

[!INCLUDE [Delete a cloud appliance](../../includes/storsimple-8000-delete-cloud-appliance.md)]

## <a name="troubleshoot-internet-connectivity-errors"></a>針對網際網路連線錯誤進行疑難排解
在雲端設備建立期間，如果無法連線到網際網路，建立步驟就會失敗。 若要針對網際網路連線失敗進行疑難排解，請在 Azure 入口網站中執行下列步驟︰

1. [在 Azure 入口網站中建立 Windows 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal)。 此虛擬機器應該與雲端設備使用相同的儲存體帳戶、VNet 和子網路。 如果 Azure 中已經有使用相同儲存體帳戶、VNet 和子網路的現有 Windows Server 主機，您也可以使用它來針對網際網路連線進行疑難排解。
2. 遠端登入在先前步驟中建立的虛擬機器。
3. 在虛擬機器內開啟命令視窗 (Win + R，然後輸入 `cmd`)。
4. 在命令提示字元執行下列命令。

    `nslookup windows.net`
5. 如果 `nslookup` 失敗，網際網路連線失敗就會導致雲端設備無法註冊 StorSimple 裝置管理員服務。
6. 對虛擬網路進行必要的變更，確保雲端設備能夠存取 Azure 網站，例如 _windows.net_。

## <a name="next-steps"></a>後續步驟
* 了解如何[使用 StorSimple 裝置管理員服務來管理雲端設備](storsimple-8000-manager-service-administration.md)。
* 了解如何 [從備份組還原 StorSimple 磁碟區](storsimple-8000-restore-from-backup-set-u2.md)。
