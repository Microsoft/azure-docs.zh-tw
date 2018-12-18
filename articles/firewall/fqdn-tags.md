---
title: Azure 防火牆的 FQDN 標籤概觀
description: 了解 Azure 防火牆中的 FQDN 標籤
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 9/24/2018
ms.author: victorh
ms.openlocfilehash: 6dc7d20d31d9399355b2b3de90ea90f2f3e07af5
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224642"
---
# <a name="fqdn-tags-overview"></a>FQDN 標籤概觀

FQDN 標籤代表一群與知名 Microsoft 服務相關聯的完整網域名稱 (FQDN)。 您可以在應用程式規則中使用 FQDN 標籤，以允許必要的輸出網路流量通過防火牆。

例如，若要手動允許 Windows Update 網路流量通過防火牆，您需要根據 Microsoft 文件建立多個應用程式規則。 使用 FQDN 標籤，您可以建立應用程式規則、納入 **Windows Update** 標籤，而且現在流向 Microsoft Windows Update 端點的網路流量可以通過防火牆。

您無法建立您自己的 FQDN 標籤，也不能指定標籤內所要包含的 FQDN。 Microsoft 會管理 FQDN 標籤所包含的 FQDN，並將標籤更新為 FQDN 變更。 

<!--- screenshot of application rule with a FQDN tag.-->

下表顯示您目前可以使用的 FQDN 標籤。 Microsoft 會維護這些標籤，而且預計會定期新增其他標籤。

|FQDN 標籤  |說明  |
|---------|---------|
|Windows Update     |允許對 Microsoft Update 進行輸出存取，如[如何為軟體更新設定防火牆](https://technet.microsoft.com/library/bb693717.aspx)中所述。|
|Windows 診斷|允許對所有 [Windows 診斷端點](https://docs.microsoft.com/windows/privacy/configure-windows-diagnostic-data-in-your-organization#endpoints)進行輸出存取。|
|Microsoft Active Protection Service (MAPS)|允許對 [MAPS](https://cloudblogs.microsoft.com/enterprisemobility/2016/05/31/important-changes-to-microsoft-active-protection-service-maps-endpoint/) 進行輸出存取。|
|App Service Environment (ASE)|允許對 ASE 平台流量進行輸出存取。 此標籤未涵蓋客戶專屬的儲存體和 ASE 所建立的 SQL 端點。 這些項目應該透過[服務端點](../virtual-network/tutorial-restrict-network-access-to-resources.md)來啟用，或以手動方式新增。|
|Azure 備份|允許對 Azure 備份服務進行輸出存取。

> [!NOTE]
> 選取應用程式規則中的 FQDN 標籤時，[通訊協定:連接埠] 欄位必須設定為 [https]。

## <a name="next-steps"></a>後續步驟

若要了解如何部署 Azure 防火牆，請參閱[教學課程：使用 Azure 入口網站部署和設定 Azure 防火牆](tutorial-firewall-deploy-portal.md)。