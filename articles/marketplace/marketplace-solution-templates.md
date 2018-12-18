---
title: Azure 應用程式解決方案範本供應項目發佈指南
description: 本文說明在 Marketplace 中發佈解決方案範本的需求
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
ms.openlocfilehash: 44d081a0666aa37ec0bf8eeac540b7a7f4f4f904
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39057880"
---
# <a name="azure-applications-solution-template-offer-publishing-guide"></a>Azure 應用程式：解決方案範本供應項目發佈指南

解決方案範本是在 Marketplace 中發佈解決方案的其中一個主要方法。 您可以使用本指南來了解這項供應項目的需求。 

這些交易供應項目是透過 Marketplace 來部署並計費的。 使用者會看到的呼籲行動是「立即取得」。

當您的解決方案除了簡單的 VM 之外，還需要以自動化方式進行額外的部署和設定時，請使用「Azure 應用程式：解決方案範本」供應項目類型。 您可以使用「Azure 應用程式：解決方案範本」來自動佈建一或多部 VM。 您也可以佈建網路和儲存體資源。 「Azure 應用程式：解決方案範本」供應項目類型可為單一 VM 和整個 IaaS 型解決方案提供自動化的優點。

## <a name="requirements-for-solution-templates"></a>解決方案範本的需求

|需求 |詳細資料  |
|---------|---------|
|計費和計量    |  資源會在客戶的 Azure 訂用帳戶中加以佈建。 隨用隨付 (PAYGO) 虛擬機器將透過 Microsoft 與客戶交易，並透過客戶的 Azure 訂用帳戶 (PAYGO) 計費 
在自備授權的案例中，雖然 Microsoft 會向客戶訂用帳戶中產生的基礎結構成本收費，但您將直接向客戶收取軟體授權費用        |
|Azure 相容的虛擬硬碟 (VHD)    |   VM 必須建置在 Windows 或 Linux 上。<ul> <li>如需有關建立 Linux VHD 的詳細資訊，請瀏覽＜建立與 Azure 相容的 VHD (以 Linux 為基礎)＞一節：[docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#2-create-an-azure-compatible-vhd-linux-based](https://docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#2-create-an-azure-compatible-vhd-linux-based)。</li> <li>如需有關建立 Windows VHD 的詳細資訊，請瀏覽＜建立與 Azure 相容的 VHD (以 Windows 為基礎)＞一節：[docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#3-create-an-azure-compatible-vhd-windows-based](https://docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#3-create-an-azure-compatible-vhd-windows-based)。</li> </ul>      |



## <a name="next-steps"></a>後續步驟
如果您還沒有這麼做，請 

- 在 Marketplace 中[註冊](https://azuremarketplace.microsoft.com/sell)

如果您已註冊，且要建立新的供應項目或使用現有的供應項目，請

- [登入 Cloud Partner 入口網站](https://cloudpartner.azure.com)以建立或完成您的供應項目
