---
title: 如何設定雲端服務 (傳統) -入口網站 |Microsoft Docs
description: 了解如何在 Azure 中設定雲端服務。 了解更新雲端服務組態和設定角色執行個體的遠端存取。 這些範例使用 Azure 入口網站。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: f16fcfe227663958279281659b09929a4cd2d386
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98743418"
---
# <a name="how-to-configure-and-azure-cloud-service-classic"></a>如何設定 Azure 雲端服務 (傳統) 

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

您可以在 Azure 入口網站中設定雲端服務的最常用設定。 或者，如果您想要直接更新組態檔，可以下載要更新的服務組態檔、上傳更新過的檔案，然後將雲端服務更新為使用這些組態變更。 使用上述任一種方式，都會將組態更新推送到所有角色執行個體。

您也可以管理雲端服務角色的執行個體，或從遠端桌面存取它們。

每個角色至少必須有兩個角色執行個體，Azure 才能確保服務在組態更新期間有 99.95% 的可用性。 如此才能讓一個虛擬機器在受到更新時，還有另一個虛擬機器可以處理用戶端要求。 如需詳細資訊，請參閱 [服務等級協定](https://azure.microsoft.com/support/legal/sla/)。

## <a name="change-a-cloud-service"></a>變更雲端服務

開啟 [Azure 入口網站](https://portal.azure.com/)之後，瀏覽至您的雲端服務。 您可以從這裡管理許多層面。

![設定頁面](./media/cloud-services-how-to-configure-portal/cloud-service.png)

[設定] 或 [所有設定] 連結將開啟 [設定]，讓您可以變更 **屬性**、變更 **設定**、管理 **憑證**、設定 **警示規則**，以及管理可存取此雲端服務的 **使用者**。

![Azure 雲端服務設定](./media/cloud-services-how-to-configure-portal/cs-settings-blade.png)

### <a name="manage-guest-os-version"></a>管理客體作業系統版本

根據預設，Azure 會將客體作業系統定期更新為您在服務組態 (.cscfg) 中指定的作業系統系列內最新支援的映像，例如 Windows Server 2016。

如果您需要將目標設為特定的作業系統版本，可以在 [設定] 中進行設定。

![設定作業系統版本](./media/cloud-services-how-to-configure-portal/cs-settings-config-guestosversion.png)

>[!IMPORTANT]
> 選擇特定的作業系統版本會停用作業系統自動更新，並使修補變成您的責任。 您必須確定您的角色執行個體將接收更新，否則您可能會會應用程式暴露在安全性弱點之下。

## <a name="monitoring"></a>監視

您可以將警示新增至雲端服務。 按一下 [**設定**  >  **警示規則**  >  **新增警示**]。

![醒目提示 [警示規則] 選項的 [設定] 的螢幕擷取畫面，其中反白顯示並以紅色標示，並以紅色顯示 [新增警示] 選項。](./media/cloud-services-how-to-configure-portal/cs-alerts.png)

您可以從這裡設定警示。 您可以使用 [計量] 下拉式方塊，以設定下列資料類型的警示。

* 磁碟讀取
* 磁碟寫入
* 網路輸入
* 網路輸出
* CPU 百分比

![[新增警示規則] 窗格的螢幕擷取畫面，其中已設定所有設定選項。](./media/cloud-services-how-to-configure-portal/cs-alert-item.png)

### <a name="configure-monitoring-from-a-metric-tile"></a>從計量圖格設定監視

  >  您可以在雲端服務的 [**監視**] 區段中，按一下其中一個計量圖格，而不是使用設定 **警示規則**。

![雲端服務監視](./media/cloud-services-how-to-configure-portal/cs-monitoring.png)

您可以從這裡自訂與圖格一起使用的圖表，或新增警示規則。

## <a name="reboot-reimage-or-remote-desktop"></a>重新啟動、重新安裝映像或遠端桌面

您可以透過 [Azure 入口網站 (安裝遠端桌面)](cloud-services-role-enable-remote-desktop-new-portal.md)、[PowerShell](cloud-services-role-enable-remote-desktop-powershell.md) 或 [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md) 來設定遠端桌面。

若要重新開機、重新安裝映像或從遠端登入雲端服務，請選取雲端服務執行個體。

![雲端服務執行個體](./media/cloud-services-how-to-configure-portal/cs-instance.png)

您接著可以起始遠端桌面連線、從遠端重新啟動執行個體，或從遠端重新安裝執行個體的映像 (開始使用全新的映像)。

![雲端服務執行個體按鈕](./media/cloud-services-how-to-configure-portal/cs-instance-buttons.png)

## <a name="reconfigure-your-cscfg"></a>重新設定 .cscfg

您可能需要透過[服務組態 (cscfg)](cloud-services-model-and-package.md#cscfg) 檔案來重新設定雲端服務。 首先，您需要下載 .cscfg 檔案，進行修改，然後上傳。

1. 按一下 [設定] 圖示或 [所有設定] 連結，以開啟 [設定]。

    ![設定頁面](./media/cloud-services-how-to-configure-portal/cloud-service.png)
2. 按一下 [ **組態** ] 項目。

    ![設定刀鋒視窗](./media/cloud-services-how-to-configure-portal/cs-settings-config.png)
3. 按一下 [ **下載** ] 按鈕。

    ![下載](./media/cloud-services-how-to-configure-portal/cs-settings-config-panel-download.png)
4. 在您更新服務組態檔之後，請上傳和套用組態更新：

    ![上傳](./media/cloud-services-how-to-configure-portal/cs-settings-config-panel-upload.png)
5. 選取 .cscfg 檔案，然後按一下 [ **確定**]。

## <a name="next-steps"></a>後續步驟

* 了解如何 [部署雲端服務](cloud-services-how-to-create-deploy-portal.md)。
* 設定 [自訂功能變數名稱](cloud-services-custom-domain-name-portal.md)。
* [管理您的雲端服務](cloud-services-how-to-manage-portal.md)。
* 設定 [TLS/SSL 憑證](cloud-services-configure-ssl-certificate-portal.md)。



