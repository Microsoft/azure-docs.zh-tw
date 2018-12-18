---
title: 概述如何建立及部署供應項目至 Marketplace | Microsoft Docs
description: 了解在 Azure Marketplace 中成為核准的 Microsoft 開發人員，以及建立及部署虛擬機器映像、範本、資料服務或開發人員服務所需的步驟
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 5343bd26-c6e4-4589-85b7-4a2c00bba8ab
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/05/2017
ms.author: hascipio
ms.openlocfilehash: 82580fbab68eab28a2027cd277213f1fb2a76e07
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39715996"
---
> [!NOTE]
> 本文件並非最新版本，且資訊並不準確。 如需在 Azure Marketplace 發佈供應項目的說明資訊，請改至 Azure Marketplace [賣方指南](https://docs.microsoft.com/azure/marketplace/seller-guide/cloud-partner-portal-seller-guide)。

# <a name="publish-and-manage-an-offer-in-the-azure-marketplace"></a>在 Azure Marketplace 中發佈和管理供應項目
此文章可協助開發人員建立解決方案、部署並管理他們在 Azure Marketplace 中列出的解決方案，讓其他 Azure 客戶與合作夥伴可進行購買及使用。

## <a name="marketplace-publishing"></a>Marketplace 發佈
身為 Azure 發行者，您可以將創新的解決方案或服務散發和販售給 Marketplace 中的其他開發人員、ISV 和 IT 專業人員。 透過 Marketplace，您可以接觸到想要快速開發其雲端型應用程式和行動解決方案的客戶。 如果您的解決方案是以商務使用者為目標，則您可能要考慮 [AppSource](http://appsource.microsoft.com) Marketplace。


## <a name="supported-types-of-solutions"></a>支援的解決方案類型
首先，您需要以發行者的身分，定義您的公司提供的解決方案類型。 Marketplace 支援下列幾種供應項目：

|解決方案類型|虛擬機器|解決方案範本|
|---|---|---|
|**定義**|預先配置的映像，包含完整安裝的作業系統以及一或多個應用程式。 虛擬機器映像提供在 Azure 虛擬機器服務中建立及部署虛擬機器所需的資訊。|可參考一或多個獨特 Azure 服務 (包含由其他賣家發佈的服務) 的資料結構。 Azure 訂戶能使用此結構，以單一、協調的方式部署一或多個供應項目。|
|**範例**|身為 Azure 發行者，您已建立並驗證帶有創新資料庫服務的 VM。 其他 Azure 訂戶願意購買此 VM 並部署至其雲端服務環境。|身為 Azure 發行者，您組合了 Azure 的幾項服務，以便快速部署負載平衡、強化安全性且高可用性的雲端服務。 其他 Azure 訂戶可藉由取得符合其目標的解決方案範本來節省時間。 他們不必手動尋找、取得、部署及設定相同或類似的 Azure 服務。|

> [!NOTE]
> 有些步驟會在不同類型的解決方案間共用，但有些步驟則因解決方案類型而有所不同。 本文提供簡短的概觀，讓您了解每種類型的解決方案必須完成哪些步驟。

## <a name="publish-a-solution"></a>發佈解決方案
![提名、註冊、發佈](media/marketplace-publishing-getting-started/img01.png)

### <a name="nominate-your-solution-for-pre-approval"></a>提名您的解決方案以供預先核准
若要在 Marketplace 中發佈虛擬機器[解決方案](https://createopportunity.azurewebsites.net)，請填妥 Microsoft Azure 認證**解決方案提名表單**。

>[!NOTE]
> 如果您與合作夥伴帳戶管理員或 DX 合作夥伴管理員合作，請要求他們提名讓您的解決方案能夠加入 Azure 認證計劃。 您也可以移至 [Microsoft Azure 認證](http://createopportunity.azurewebsites.net)網頁，然後填妥申請表單。 在 [Microsoft 贊助者連絡人] 方塊中輸入您的夥伴客戶經理或 DX 夥伴經理的電子郵件。

如果您符合 [Azure Marketplace 參與原則](http://go.microsoft.com/fwlink/?LinkID=526833)的資格條件，且您的應用程式獲得核准，我們就會開始與您合作將您的解決方案上架到 Marketplace。

### <a name="register-your-account-as-a-microsoft-seller"></a>將您的帳戶註冊為 Microsoft 賣方
將您的 Microsoft 帳戶註冊為 [Microsoft 開發人員帳戶](marketplace-publishing-accounts-creation-registration.md)。

### <a name="publish-your-solution"></a>發佈您的解決方案
若要在 Marketplace 中發佈解決方案，請遵循下列步驟︰
1. 滿足非技術性需求。

    a. 滿足[非技術性必要條件](marketplace-publishing-pre-requisites.md)。

    b. 滿足 [VM 技術性必要條件](marketplace-publishing-vm-image-creation-prerequisites.md)。

    c. 滿足[解決方案範本技術性必要條件](marketplace-publishing-solution-template-creation-prerequisites.md)。

2. 建立您的供應項目。

    a. 建立[虛擬機器](marketplace-publishing-vm-image-creation.md)供應項目。

    b. 建立[解決方案範本](marketplace-publishing-solution-template-creation.md)供應項目。

3. 建立您的供應項目[行銷內容](marketplace-publishing-push-to-staging.md)。

4. 在預備環境中測試您的供應項目。

    a. 在[預備環境](marketplace-publishing-vm-image-test-in-staging.md)中測試您的 VM 供應項目。

    b. 在[預備環境](marketplace-publishing-solution-template-test-in-staging.md)中測試您的解決方案範本供應項目。

5. 將您的供應項目部署至 [Marketplace](marketplace-publishing-push-to-production.md)。


### <a name="create-and-manage-a-virtual-machine-image"></a>建立和管理虛擬機器映像
使用這些資源來建立和管理 VM 映像︰
* 建立[內部部署](marketplace-publishing-vm-image-creation-on-premise.md) VM 映像。
* 在 [Azure 入口網站](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中建立執行 Windows 的虛擬機器。
* 在 [Azure 入口網站](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中建立執行 Linux 的虛擬機器。
* 針對 [VHD 建立](marketplace-publishing-vm-image-creation-troubleshooting.md)常見問題進行疑難排解。

## <a name="manage-your-solution"></a>管理您的解決方案
透過下列資源的協助來管理您的解決方案︰
* [閱讀虛擬機器供應項目的後期製作指南](marketplace-publishing-vm-image-post-publishing.md)
* [更新供應項目或 SKU 的非技術性詳細資料](marketplace-publishing-vm-image-post-publishing.md#update-the-nontechnical-details-of-an-offer-or-a-sku)
* [更新供應項目或 SKU 的技術性詳細資料](marketplace-publishing-vm-image-post-publishing.md#update-the-technical-details-of-a-sku)
* [在已列出的供應項目下新增 SKU](marketplace-publishing-vm-image-post-publishing.md#add-a-new-sku-under-a-listed-offer)
* [變更已列出 SKU 的資料磁碟計數](marketplace-publishing-vm-image-post-publishing.md#change-the-data-disk-count-for-a-listed-sku)
* [從 Marketplace 刪除已列出的供應項目](marketplace-publishing-vm-image-post-publishing.md)
* [從 Marketplace 刪除已列出的 SKU](marketplace-publishing-vm-image-post-publishing.md#delete-a-listed-sku-from-the-marketplace)
* [從 Marketplace 刪除已列出 SKU 的目前版本](marketplace-publishing-vm-image-post-publishing.md#delete-the-current-version-of-a-listed-sku-from-the-marketplace)
* [將列出價格還原成生產環境值](marketplace-publishing-vm-image-post-publishing.md#revert-the-listing-price-to-production-values)
* [將計費模式還原成生產環境值](marketplace-publishing-vm-image-post-publishing.md#revert-the-billing-model-to-production-values)
* [將已列出的 SKU 的可見性設定還原成生產環境值](marketplace-publishing-vm-image-post-publishing.md#revert-the-visibility-setting-of-a-listed-sku-to-the-production-value)

## <a name="additional-resources"></a>其他資源
[設定 Azure PowerShell](marketplace-publishing-powershell-setup.md)
