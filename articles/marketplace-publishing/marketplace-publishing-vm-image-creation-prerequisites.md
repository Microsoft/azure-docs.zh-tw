---
title: 建立 Azure Marketplace 虛擬機器映像的技術性必要條件 | Microsoft Docs
description: 了解建立和部署虛擬機器映像到 Azure Marketplace 供他人購買的要求。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 63c16966-0304-4b17-a715-368a0a5ccb2c
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Azure
ms.workload: na
ms.date: 04/29/2016
ms.author: hascipio; v-divte
ms.openlocfilehash: af3e2ad623d8d7bfafe676411f9ae3fbee78aab8
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39714327"
---
# <a name="technical-prerequisites-for-creating-a-virtual-machine-image-for-the-azure-marketplace"></a>建立 Azure Marketplace 虛擬機器映像的技術性必要條件
開始之前，請先徹底閱讀程序，並且了解每個步驟執行的位置及原因。 在供應項目建立程序之前，您應該盡可能準備您的公司資訊和其他資料、下載必要的工具，和/或建立技術元件。 檢閱本文之後，您會更清楚這些項目。  

## <a name="download-needed-tools--applications"></a>下載需要的工具和應用程式
開始程序之前，您應該先準備好下列項目：

* 視您的目標作業系統而定，從 [Azure 下載](https://azure.microsoft.com/downloads/)頁面安裝 [Azure PowerShell Cmdlet](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/WindowsAzurePowershellGet.3f.3f.3fnew.appids) 或 [Linux 命令列介面工具](https://go.microsoft.com/fwlink/?LinkId=253472&clcid=0x409)。
* 從 CodePlex 安裝 Azure 儲存體總管。
* 下載並安裝適用於 Azure 認證的認證測試工具：
  * [http://go.microsoft.com/fwlink/?LinkID=526913](http://go.microsoft.com/fwlink/?LinkID=526913)。 您需要 Windows 架構的電腦才能執行認證工具。 如果您沒有 Windows 電腦，可以在 Azure 中使用 Windows 架構的 VM 執行工具。

## <a name="platforms-supported"></a>支援的平台
您可以在 Windows 或 Linux 上開發 Azure VM。 發佈程序的一些項目—例如建立與 Azure 相容的虛擬硬碟 (VHD)—會根據使用的作業系統而使用不同的工具和步驟。  

* 如果您使用 Linux，請參閱 [虛擬機器映像發佈指南](marketplace-publishing-vm-image-creation.md)的＜建立與 Azure 相容的 VHD (以 Linux 為基礎)＞一節。
* 如果您使用 Windows，請參閱 [虛擬機器映像發佈指南](marketplace-publishing-vm-image-creation.md)的＜建立與 Azure 相容的 VHD (以 Windows 為基礎)＞一節。

> [!NOTE]
> 您需要有 Windows 電腦的存取權才能︰
> 
> * 執行憑證驗證工具。
> * 建立送出 VHD 憑證的 VHD 共用存取簽章 URL。
> 
> 

## <a name="develop-your-vhd"></a>開發您的 VHD
您可以在雲端或內部部署中開發 Azure VHD：

* 雲端開發表示所有開發步驟都會在 Azure 的 VHD 上遠端執行。
* 內部部署開發則需要使用內部部署基礎結構下載 VHD 和開發。 雖然可行，但我們不建議您這麼做。 請注意，為內部部署的 Windows 或 SQL 開發時，需要有相關的內部部署授權金鑰。 建立 VM 之後即無法加入或安裝 SQL Server。 此外，您必須讓供應項目以 Azure 入口網站中核准的 SQL 映像為基礎。 如果您決定開發內部部署，必須執行一些不同於在雲端中開發的步驟。 您可以在 [建立內部部署 VM 映像](marketplace-publishing-vm-image-creation-on-premise.md)找到相關資訊。

[link-acct-creation]:marketplace-publishing-accounts-creation-registration.md
