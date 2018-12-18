---
title: Azure Active Directory Domain Services：開始使用 | Microsoft Docs
description: 使用 Azure 入口網站啟用 Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: ace1ed4a-bf7f-43c1-a64a-6b51a2202473
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/23/2018
ms.author: maheshu
ms.openlocfilehash: c167f40e09ac877d00d348285383106fc5ba41bc
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503905"
---
# <a name="enable-azure-active-directory-domain-services-using-the-azure-portal"></a>使用 Azure 入口網站啟用 Azure Active Directory Domain Services


## <a name="before-you-begin"></a>開始之前
請參閱 [Azure Active Directory Domain Services 的網路考量](active-directory-ds-networking.md)。


## <a name="task-2-configure-network-settings"></a>工作 2：設定網路設定
下一個設定工作是建立 Azure 虛擬網路與其中的專用子網路。 您可在虛擬網路內的這個子網路中啟用 Azure Active Directory Domain Services。 您也可以挑選現有的虛擬網路，並且建立其中的專用子網路。

1. 按一下 [虛擬網路] 以選取虛擬網路。
    > [!NOTE]
    > **新部署不支援傳統虛擬網路。** 新部署不支援傳統虛擬網路。 繼續支援在傳統虛擬網路中部署的現有受控網域。 Microsoft 將在不久的將來，讓您將現有的受控網域從傳統虛擬網路移轉到 Resource Manager 虛擬網路。
    >

2. 在 [選擇虛擬網路] 頁面上，您會看到所有現有的虛擬網路。 您只會看到隸屬於您在 [基本] 精靈分頁上選取之資源群組和 Azure 位置的虛擬網路。
3. 選擇應該在其中啟用 Azure AD Domain Services 的虛擬網路。 您可以選取現有的虛擬網路或建立新的虛擬網路。

  > [!TIP]
  > **啟用 Azure AD Domain Services 後，無法將受控網域移至不同的虛擬網路。** 挑選要啟用受控網域的適當虛擬網路。 建立受控網域之後，您無法在未刪除受控網域的情況下，將其移動至不同的虛擬網路。 在繼續之前，建議您檢閱 [Azure Active Directory Domain Services 的網路考量](active-directory-ds-networking.md)。  
  >

4. **建立虛擬網路：** 若要建立新的虛擬網路，請按一下 [新建]。 針對 Azure AD Domain Services 使用專用子網路。 例如，建立具有名稱 'DomainServices' 的子網路，讓其他系統管理員容易了解子網路內部署的內容。 完成後，按一下 [確定]。

    ![挑選虛擬網路](./media/getting-started/domain-services-blade-network-pick-vnet.png)

  > [!WARNING]
  > 請務必挑選私人 IP 位址空間內的位址空間。 不是您所擁有的 IP 位址位於公用位址空間中會導致 Azure AD 網域服務內發生錯誤。

5. **現有的虛擬網路：** 如果您計畫挑選現有的虛擬網路，[使用虛擬網路延伸模組建立專用子網路](../virtual-network/virtual-network-manage-subnet.md#add-a-subnet)，然後挑選該子網路。 按一下 [虛擬網路] 以選取現有的虛擬網路。 按一下 [子網路] 以挑選現有虛擬網路中的專用子網路，在其中啟用新的受控網域。 完成後，按一下 [確定]。

    ![挑選虛擬網路內的子網路](./media/getting-started/domain-services-blade-network-pick-subnet.png)

  > [!NOTE]
  > **選取子網路的指導方針**
  > 1. 針對 Azure AD Domain Services 使用專用子網路。 不要將任何其他虛擬機器部署到此子網路。 此設定可讓您為工作負載/虛擬機器設定網路安全性群組 (NSG)，而不會中斷受控網域。 如需詳細資訊，請參閱 [Azure Active Directory Domain Services 的網路考量](active-directory-ds-networking.md)。
  2. 請勿選取閘道子網路來部署 Azure AD Domain Services，因為它不是支援的設定。
  3. 您選取的子網路在其位址空間必須具有至少 3 到 5 個可用 IP 位址。
  >

6. 完成時，按一下 [確定] 以繼續前往精靈的 [Administrator 群組] 分頁。


## <a name="next-step"></a>後續步驟
[工作 3：設定系統管理群組並啟用 Azure AD Domain Services](active-directory-ds-getting-started-admingroup.md)
