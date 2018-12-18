---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 03/09/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: d4075f644ed1ce42479e35544f0e1442488f78eb
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/16/2018
ms.locfileid: "29958906"
---
1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 從左上角開始，按一下 [建立資源] > **[計算]** > [Windows Server 2016 Datacenter]。

    ![瀏覽至入口網站中的 Azure VM 映像](./media/virtual-machines-common-portal-create-fqdn/marketplace-new.png)

3. 在 Windows Server 2016 Datacenter 中，選取傳統部署模型。 按一下 [建立]。

    ![顯示在入口網站中可用 Azure VM 映像的螢幕擷取畫面](./media/virtual-machines-common-portal-create-fqdn/deployment-classic-model.png)

## <a name="1-basics-blade"></a>1.基本概念刀鋒視窗

[基本] 刀鋒視窗需要虛擬機器的系統管理資訊。

1. 輸入虛擬機器的**名稱**。 在此範例中，HeroVM 是虛擬機器的名稱。 名稱必須為 1-15 個字元，不能包含特殊字元。

2. 輸入用來在 VM 上建立本機帳戶的**使用者名稱**和強式**密碼**。 此本機帳戶用來登入及管理 VM。 在此範例中，azureuser 是使用者名稱。

 密碼長度必須是 8-123 個字元，且符合下列四個複雜性需求的其中三項：1 個小寫字元、1 個大寫字元、1 個數字和 1 個特殊字元。 進一步了解 [使用者名稱和密碼需求](../articles/virtual-machines/windows/faq.md)。

3. [訂用帳戶] 是選擇性的。 一個常見的設定是「隨用隨付」。

4. 選取現有的**資源群組**，或輸入新群組的名稱。 在此範例中，HeroVMRG 是資源群組的名稱。

5. 選取您要執行 VM 的 Azure Datacenter **位置**。 在此範例中，East US 就是位置。

6. 當您完成時，按 [下一步] 繼續下一個刀鋒視窗。

    ![顯示用於設定 Azure VM 之 [基本概念] 刀鋒視窗上的設定的螢幕擷取畫面](./media/virtual-machines-common-portal-create-fqdn/basics-blade-classic.png)

## <a name="2-size-blade"></a>2.大小刀鋒視窗

[大小] 刀鋒視窗會識別 VM 的設定詳細資料，並列出各種選擇，包括作業系統、處理器數目、磁碟儲存體類型，以及估計的每月使用成本。  

選擇 VM 大小，然後按一下 [選取] 以繼續。 在此範例中，DS1\_V2 Standard 就是 VM 大小。

  ![顯示您可以選取之 Azure VM 大小的 [大小] 刀鋒視窗的螢幕擷取畫面](./media/virtual-machines-common-portal-create-fqdn/vm-size-classic.png)


## <a name="3-settings-blade"></a>3.設定刀鋒視窗

[設定] 刀鋒視窗需要儲存體和網路選項。 您可以接受預設設定。 Azure 會視需要建立適當的項目。

如果您選取支援的虛擬機器大小，就能藉由選取 [磁碟類型] 中的 [進階 (SSD)]，嘗試使用 Azure 進階儲存體。

當您完成變更時，請按一下 [確定] 。

## <a name="4-summary-blade"></a>4.摘要刀鋒視窗

[摘要] 刀鋒視窗會列出先前刀鋒視窗中所指定的設定。 當您準備好製作映像時，按一下 [確定]。

 ![提供所指定虛擬機器設定的 [摘要] 刀鋒視窗報告](./media/virtual-machines-common-portal-create-fqdn/summary-blade-classic.png)

建立虛擬機器後，入口網站會在 [所有資源] 底下列出新的虛擬機器，並在儀表板上顯示虛擬機器圖格。 同時也會建立並列出對應的雲端服務和儲存體帳戶。 虛擬機器和雲端服務都會自動啟動，而且它們的狀態會顯示為 [執行中] 。

 ![設定 VM 代理程式和需擬機器端點](./media/virtual-machines-common-portal-create-fqdn/portal-with-new-vm.png)
