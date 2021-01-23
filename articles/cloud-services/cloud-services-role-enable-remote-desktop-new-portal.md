---
title: 使用入口網站來啟用角色的遠端桌面
description: 如何設定的 Azure 雲端服務應用程式以允許遠端桌面連線
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 8fa0d3c0e29c53e6fe9cb32ddf02168686be1efe
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98743248"
---
# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services-classic"></a>針對 Azure 雲端服務 (傳統) 中的角色啟用遠端桌面連線

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

> [!div class="op_single_selector"]
> * [Azure 入口網站](cloud-services-role-enable-remote-desktop-new-portal.md)
> * [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
> * [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md)

遠端桌面可讓您存取 Azure 內執行中角色的桌面。 您可以使用遠端桌面連線來疑難排解和診斷執行中應用程式的問題。

您可以在開發期間藉由在服務定義中包含遠端桌面模組啟用遠端桌面連線，也可以選擇透過遠端桌面延伸模組啟用遠端桌面。 慣用的方法是使用遠端桌面延伸模組，因為即使在部署應用程式之後，您仍然可以啟用遠端桌面，無需重新部署您的應用程式。

## <a name="configure-remote-desktop-from-the-azure-portal"></a>從 Azure 入口網站設定遠端桌面

Azure 入口網站會使用遠端桌面延伸模組方法，因此即使在應用程式部署之後，您也可以啟用遠端桌面。 雲端服務的 **遠端桌面** 設定可讓您啟用遠端桌面、變更用來與虛擬機器連線的本機系統管理員帳戶、驗證中使用的憑證，並設定到期日。

1. 按一下 [雲端服務]，然後依序選取雲端服務的名稱及 [遠端桌面]。

    ![顯示雲端服務遠端桌面的影像](./media/cloud-services-role-enable-remote-desktop-new-portal/CloudServices_Remote_Desktop.png)

2. 選擇要針對個別的角色或所有角色啟用遠端桌面，然後將切換器的值變更為 **啟用**。

3. 填入必要的使用者名稱、密碼、到期日及憑證欄位。

    ![雲端服務的遠端桌面](./media/cloud-services-role-enable-remote-desktop-new-portal/CloudServices_Remote_Desktop_Details.png)

   > [!WARNING]
   > 當您首次啟用遠端桌面並選取 [確定] (打勾記號) 時，所有角色執行個體都將重新啟動。 若要防止重新啟動，角色上必須安裝用來將密碼加密的憑證。 若要防止重新啟動，請[上傳雲端服務的憑證](cloud-services-configure-ssl-certificate-portal.md#step-3-upload-a-certificate)，然後再回到這個對話方塊。

4. 在 [角色] 中，選取想要更新的角色，或選取 [全部] 以更新所有角色。

5. 完成設定更新時，選取 [儲存]。 在您的角色執行個體準備好接受連線之前，會需要幾分鐘的時間。

## <a name="remote-into-role-instances"></a>角色執行個體的遠端存取

針對角色啟用遠端桌面之後，就能直接從 Azure 入口網站將連線初始化：

1. 按一下 [執行個體] 以開啟 [執行個體] 設定。
2. 選取已設定「遠端桌面」的角色執行個體。
3. 按一下 [連接] 下載 RDP 檔案的角色執行個體。

    ![雲端服務遠端桌面映射](./media/cloud-services-role-enable-remote-desktop-new-portal/CloudServices_Remote_Desktop_Connect.png)

4. 按一下 [開啟]，然後按一下 [連接] 以啟動遠端桌面連線。

>[!NOTE]
> 如果您的雲端服務設置在 NSG 後方，您可能需要建立規則以允許連接埠 **3389** 和 **20000** 上的流量。  遠端桌面使用者連接埠 **3389**。  系統已為雲端服務執行個體進行負載平衡，因此您無法直接控制要連線的執行個體。  *RemoteForwarder* 與 *RemoteAccess* 代理程式會管理 RDP 流量，並允許用戶端傳送 RDP Cookie 並指定要連線的個別執行個體。  *RemoteForwarder* 與 *RemoteAccess* 代理程式要求您必須開啟連接埠 **20000***，此連接埠在您使用 NSG 的情況下可能是封鎖的。

## <a name="additional-resources"></a>其他資源

[如何設定雲端服務](cloud-services-how-to-configure-portal.md)
