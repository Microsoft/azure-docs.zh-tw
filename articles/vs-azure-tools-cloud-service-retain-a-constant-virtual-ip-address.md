---
title: 如何保持 Azure 雲端服務的固定虛擬 IP 位址 | Microsoft Docs
description: 了解如何確保 Azure 雲端服務的虛擬 IP 位址 (VIP) 不會變更。
services: visual-studio-online
author: ghogen
manager: douge
assetId: 4a58e2c6-7a79-4051-8a2c-99182ff8b881
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/21/2017
ms.author: ghogen
ms.openlocfilehash: 2f82663f6b53c6d4e32b8d655dcbd67a321d91ed
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141299"
---
# <a name="retain-a-constant-virtual-ip-address-for-an-azure-cloud-service"></a>保持 Azure 雲端服務的固定虛擬 IP 位址
當您更新裝載於 Azure 中的雲端服務時，您可能需要確保服務的虛擬 IP 位址 (VIP) 不會變更。 許多網域管理服務都使用網域名稱系統 (DNS) 註冊網域名稱。 DNS 只有在 VIP 保持不變時才能運作。 您可以使用 Azure 工具中的 [發佈精靈]  確保雲端服務的 VIP 不會在您更新它時變更。 如需有關如何使用雲端服務的 DNS 網域管理的詳細資訊，請參閱 [設定 Azure 雲端服務的自訂網域名稱](cloud-services/cloud-services-custom-domain-name-portal.md)。

## <a name="publish-a-cloud-service-without-changing-its-vip"></a>發佈雲端服務而不變更其 VIP
當您第一次將雲端服務的 VIP 部署至特定環境 (例如生產環境) 的 Azure 中，會配置雲端服務的 VIP。 只有當您明確刪除部署或由部署更新程序隱含地刪除部署時，VIP 才會變更。 若要保留 VIP，您不能刪除您的部署，且必須要確定 Visual Studio 不會自動刪除您的部署。 

您可以在 [發佈精靈] 中指定部署設定，該精靈可支援多種部署選項。 您可以指定全新部署或更新部署，後者可以是累加或同時的。 這兩種更新部署都會保留 VIP。 如需這些不同類型部署的定義，請參閱 [發佈 Azure 應用程式精靈](vs-azure-tools-publish-azure-application-wizard.md)。 此外，您可以在發生錯誤時，控制是否要刪除先前的雲端服務部署。 如果您沒有正確設定該選項，可能會意外變更 VIP。

## <a name="update-a-cloud-service-without-changing-its-vip"></a>更新雲端服務而不變更其 VIP
1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。 

2. 在 [方案總管] 中，於專案上按一下滑鼠右鍵。 選取捷徑功能表上的 [發佈]。

    ![Publish menu](./media/vs-azure-tools-cloud-service-retain-a-constant-virtual-ip-address/solution-explorer-publish-menu.png)

3. 在 [發佈 Azure 應用程式] 對話方塊中，選取您要部署到其中的 Azure 訂用帳戶。 視需要登入，然後選取 [下一步]。

    ![發佈 Azure 應用程式登入頁面](./media/vs-azure-tools-cloud-service-retain-a-constant-virtual-ip-address/azure-publish-signin.png)

4. 在 [一般設定] 索引標籤上，確認您要部署的目標雲端服務名稱、[環境]、[組建組態] 和 [服務組態] 全都正確無誤。

    ![發佈 Azure 應用程式一般設定索引標籤](./media/vs-azure-tools-cloud-service-retain-a-constant-virtual-ip-address/azure-publish-common-settings.png)

5. 在 [進階設定] 索引標籤上，確認 [部署標籤] 和 [儲存體帳戶] 都正確無誤。 確認已清除 [失敗時刪除部署] 核取方塊，並確認已選取 [部署更新] 核取方塊。 清除 [失敗時刪除部署] 核取方塊，可以確保在部署期間發生錯誤時不會遺失 VIP。 藉由選取 [部署更新] 核取方塊，您可以確定不會刪除您的部署，而且在重新發佈應用程式時不會遺失 VIP。 

    ![發佈 Azure 應用程式進階設定索引標籤](./media/vs-azure-tools-cloud-service-retain-a-constant-virtual-ip-address/azure-publish-advanced-settings.png)

6. 若要進一步指定您希望更新角色的方式，請選取 [部署更新] 旁的 [設定]。 選取 [累加式更新] 或 [同時更新]，然後選取 [確定]。 選擇 [累加式更新]，就會逐一更新應用程式的每個執行個體，讓應用程式隨時可供使用。 選擇 [同時更新]，即會同時更新應用程式的所有執行個體。 同時更新較為快速，但是您的服務可能無法在更新過程中使用。 完成後，選取 [下一步] 。

    ![發佈 Azure 應用程式部署設定頁面](./media/vs-azure-tools-cloud-service-retain-a-constant-virtual-ip-address/azure-publish-deployment-update-settings.png)

7. 在 [發佈 Azure 應用程式] 對話方塊中，選取 [下一步] 直到 [摘要] 頁面顯示為止。 確認您的設定，然後選取 [發佈]。
   
    ![發佈 Azure 應用程式摘要頁面](./media/vs-azure-tools-cloud-service-retain-a-constant-virtual-ip-address/azure-publish-summary.png)

## <a name="next-steps"></a>後續步驟
- [使用 Visual Studio 發佈 Azure 應用程式精靈](vs-azure-tools-publish-azure-application-wizard.md)

