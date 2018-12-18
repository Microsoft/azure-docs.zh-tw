---
title: Azure 受控應用程式概觀 | Microsoft Docs
description: 說明 Azure 受控應用程式的概念
services: managed-applications
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.date: 10/04/2018
ms.author: tomfitz
ms.openlocfilehash: 67797f3bc5eadc126bad1051e793a8d3bc4590e5
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48800939"
---
# <a name="azure-managed-applications-overview"></a>Azure 受控應用程式概觀

Azure 受控應用程式可讓您提供雲端解決方案，讓取用者方便部署及操作。 您會實作基礎結構，並且提供持續的支援。 若要讓受控應用程式可供所有客戶使用，請將它發佈到 Azure Marketplace 中。 若要讓它僅可供組織中的使用者使用，請將它發佈到內部目錄。 

受控應用程式與 Marketplace 中的解決方案範本類似，只有一個主要差異。 在受控應用程式中，資源會部署到應用程式發行者所管理的資源群組。 資源群組存在於客戶的訂用帳戶，但發行者租用戶中的身分識別可以存取資源群組。 身為發行者，您可以指定解決方案持續支援的成本。

## <a name="advantages-of-managed-applications"></a>受控應用程式的優點

受控應用程式會降低取用者使用您解決方案的障礙。 他們不需要具備雲端基礎結構的專業知識，就可以使用您的解決方案。 取用者只有重要資源的有限存取權，不必擔心管理時會犯下錯誤。 

受控應用程式可讓您與取用者建立持續的關係。 定義用來管理應用程式的條款，所有費用都是透過 Azure 帳單處理。

雖然客戶在其訂用帳戶中部署這些受控應用程式，但他們不需要維護、更新或服務它們。 您可以確保所有客戶都使用核准的版本。 客戶不需要開發特定應用程式特定領域的知識，就可以管理這些應用程式。 客戶會自動取得應用程式更新，而不必擔心應用程式的疑難排解和診斷問題。 

對於 IT 小組，受控應用程式可讓您為組織中的使用者提供預先核准的解決方案。 您知道這些解決方案都符合組織標準。

## <a name="types-of-managed-applications"></a>受控應用程式的類型

您可以在外部或內部發佈受控應用程式。

![在內部或外部發佈](./media/overview/manage_app_options.png)

### <a name="service-catalog"></a>服務類別目錄

服務類別目錄是組織中使用者適用的已核准解決方案的內部目錄。 您會使用目錄來符合組織標準，同時為組織提供一些解決方案。 員工會使用目錄輕鬆尋找其 IT 部門所建議和核准的應用程式。 他們會看到組織中的其他人所共用的受控應用程式。

如需發佈服務類別目錄受控應用程式的詳細資訊，請參閱[建立服務類別目錄應用程式](publish-service-catalog-app.md)。

### <a name="marketplace"></a>Marketplace

想要為其服務開立帳單的廠商，可以透過 Azure Marketplace 讓受控應用程式可供使用。 廠商發佈應用程式之後，應用程式就可供組織外部的使用者使用。 使用此方法，受控服務提供者 (MSP)、獨立軟體廠商 (ISV) 和系統整合者 (SI) 可以將其解決方案提供給所有 Azure 客戶。

如需將受控應用程式發佈到 Marketplace 的詳細資訊，請參閱[建立 Marketplace 應用程式](publish-marketplace-app.md)。

## <a name="resource-groups-for-managed-applications"></a>受控應用程式的資源群組

一般來說，受控應用程式的資源位於兩個資源群組中。 取用者會管理一個資源群組，發行者會管理另一個資源群組。 在定義受控應用程式時，發行者會指定存取的層級。 Azure 中的所有資料提供者目前都不支援限制存取[資料作業](../role-based-access-control/role-definitions.md)。

下圖顯示的案例是發行者要求受控資源群組的擁有者角色。 發行者會為取用者對這個資源群組進行唯讀鎖定。 獲取受控資源群組存取權的發行者身分識別免除鎖定。

![資源群組存取](./media/overview/access.png)

### <a name="application-resource-group"></a>應用程式資源群組

這個資源群組會保存受控應用程式執行個體。 這個資源群組只能包含一個資源。 受控應用程式的資源類型為 **Microsoft.Solutions/applications**。

取用者具有資源群組的完整存取權，並且使用它來管理受控應用程式的生命週期。

### <a name="managed-resource-group"></a>受控資源群組

這個資源群組會保存受控應用程式所需的所有資源。 例如，這個資源群組包含解決方案的虛擬機器、儲存體帳戶和虛擬網路。 因為取用者不會為受控應用程式管理個別資源，所以取用者具有這個資源群組的有限存取權。 這個資源群組的發行者存取權會對應至受控應用程式定義中指定的角色。 例如，發行者可能會要求這個資源群組的「擁有者」或「參與者」角色。

當取用者刪除受控應用程式時，也會刪除受控資源群組。

## <a name="azure-policy"></a>Azure 原則

您可以將 [Azure 原則](../azure-policy/azure-policy-introduction.md)套用至您的受控應用程式。 您要套用原則，以確保受控應用程式的已部署執行個體符合資料和安全性需求。 如果您的應用程式與敏感性資料互動，請確定您已評估其保護方式。 例如，如果您的應用程式是與 Office 365 中的資料互動，則套用原則以確定資料加密已啟用。

## <a name="next-steps"></a>後續步驟

在本文中，您會了解使用受控應用程式的優點。 移至下一篇文章以建立受控應用程式定義。

> [!div class="nextstepaction"]
> [快速入門：發佈 Azure 受控應用程式定義](publish-managed-app-definition-quickstart.md)
