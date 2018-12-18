---
title: Azure Marketplace 的虛擬機器供應項目發佈指南
description: 本文說明發佈虛擬機器的需求，以及從 Marketplace 部署免費試用軟體的需求。
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
documentationcenter: ''
author: ellacroi
manager: nunoc
editor: ''
ms.assetid: ''
ms.service: marketplace
ms.workload: ''
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 07/09/2018
ms.author: ellacroi
ms.openlocfilehash: b8caeab7f08ffeee81492b01750cbb255e172872
ms.sourcegitcommit: a1140e6b839ad79e454186ee95b01376233a1d1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43144517"
---
# <a name="virtual-machine-offer-publishing-guide"></a>虛擬機器供應項目發佈指南

虛擬機器映像是在 Azure Marketplace 中發佈解決方案的其中一個主要方法。 您可以使用本指南來了解這項供應項目的需求。 

這些交易供應項目是透過 Marketplace 來部署並計費的。 使用者會看到的呼籲行動是「立即取得」。

## <a name="free-trial"></a>免費試用 

您可以讓使用者在使用自備授權 (BYOL) 計費模型時，可存取有限期間的軟體授權，以便測試您的供應項目。 以下是部署這項供應項目的需求。 

|需求  |詳細資料  |
|---------|---------|
|免費試用期與試用版體驗     |   客戶可以在限時內免費試用您的應用程式。 客戶不需要為您的供應項目支付任何授權或訂閱費用。 客戶不需要為基礎 Microsoft 第一方產品或服務支付費用。 所有試用版選項都會部署到您的 Azure 訂用帳戶。 您可以全權控制成本最佳化和管理。 您可以選擇免費試用或互動式示範。 無論您的選擇為何，免費試用都必須提供客戶一段預先設定的時間來試用您的供應項目，且無須支付任何額外費用。|
|輕鬆設定、立即可用的解決方案    |  應用程式的設定和安全必須既簡單又快速。       |
|可用性 / 執行時間    |    SaaS 應用程式或平台的執行時間必須至少達到 99.9%。     |
|Azure Active Directory     |    您的供應項目必須允許 Azure Active Directory (Azure AD) 同盟單一登入 (SSO) (Azure AD 同盟 SSO) 並啟用同意功能。     |

## <a name="test-drive"></a>試用產品

您會透過基礎結構即服務 (IaaS) 或 SaaS 應用程式部署一或多部虛擬機器。 試用產品發佈選項的優點是，可在合作夥伴所主持的引導式導覽帶領下，以自動化方式佈建虛擬機器或整個解決方案。 試用產品可為您的客戶提供評估，而無須額外收費。 客戶不必是現有的 Azure 客戶，即可參與試用體驗。 

請經由 [amp-testdrive](mailto:amp-testdrive@microsoft.com) 與我們連絡，以開始使用。 

|需求  |詳細資料 |
|---------|---------|
| 您有 Marketplace 應用程式   |    透過 IaaS 或 SaaS 的一或多部虛擬機器。      |

## <a name="interactive-demo"></a>互動式示範

您利用互動式示範，為客戶提供引導式的解決方案體驗。 互動式示範發佈選項的優點是，無須進行複雜解決方案的複雜佈建，即可提供試用版體驗。 

## <a name="virtual-machine-offer"></a>虛擬機器供應項目

當您要將虛擬設備部署到與客戶相關的訂用帳戶時，請使用「虛擬機器」供應項目類型。 使用「預付型方案」或「自備授權」(BYOL) 授權模型，即可啟用完整的 VM 商務功能。 Microsoft 會主控商務交易，並代表您向客戶收費。 您獲得的好處是，可以使用客戶與 Microsoft 之間慣用的付款關係，包括任何 Enterprise 合約。

>[!NOTE]
>目前與 Enterprise 合約相關的承諾用量金額可用於您 VM 的 Azure 使用量，但不能用於您的軟體授權費用。  

>[!NOTE]
>若將映像和定價當作私人供應項目來發佈，就可以只讓一組特定的客戶探索和部署您的 VM。 私人供應項目可讓您為最親密的客戶建立專屬的供應項目，以及提供自訂軟體和條款。 這些自訂條款可讓您突顯各式各樣的案例，包括具有特殊定價和條款的實地主導交易，以及限制版軟體的優先存取權。 私人供應項目可讓您建立具有特定定價或產品詳細資料的新 SKU，為一組有限的客戶提供該特定的定價或產品。  
*   如需有關「私人供應項目」的詳細資訊，請瀏覽「Azure Marketplace 上的私人供應項目」頁面：[azure.microsoft.com/blog/private-offers-on-azure-marketplace](https://azure.microsoft.com/blog/private-offers-on-azure-marketplace)。  

| 需求 | 詳細資料 |  
|:--- |:--- | 
| 計費和計量 | 您的 VM 必須支援 BYOL 或每月計費「預付型方案」。 |  
| Azure 相容的虛擬硬碟 (VHD) | VM 必須建置在 Windows 或 Linux 上。<ul> <li>如需有關建立 Linux VHD 的詳細資訊，請瀏覽＜建立與 Azure 相容的 VHD (以 Linux 為基礎)＞一節：[docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#2-create-an-azure-compatible-vhd-linux-based](https://docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#2-create-an-azure-compatible-vhd-linux-based)。</li> <li>如需有關建立 Windows VHD 的詳細資訊，請瀏覽＜建立與 Azure 相容的 VHD (以 Windows 為基礎)＞一節：[docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#3-create-an-azure-compatible-vhd-windows-based](https://docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#3-create-an-azure-compatible-vhd-windows-based)。</li> </ul> |  

## <a name="next-steps"></a>後續步驟

如果您還沒有這麼做，請 

- 在市集中[註冊](https://azuremarketplace.microsoft.com/sell)

如果您已註冊，且要建立新供應項目或使用現有供應項目，請

- [登入 Cloud Partner 入口網站](https://cloudpartner.azure.com)以建立或完成您的供應項目
