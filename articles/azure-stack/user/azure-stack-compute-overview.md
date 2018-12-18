---
title: Azure Stack 虛擬機器簡介
description: 了解 Azure Stack 虛擬機器
services: azure-stack
author: sethmanheim
manager: femila
ms.service: azure-stack
ms.topic: get-started-article
ms.date: 09/05/2018
ms.author: sethm
ms.reviewer: kivenkat
ms.openlocfilehash: a0d75b13369cff4e99bef6f57a3b01f3d8eee6e4
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227158"
---
# <a name="introduction-to-azure-stack-virtual-machines"></a>Azure Stack 虛擬機器簡介

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure Stack 會以一種依需求、可調整的計算資源形式提供虛擬機器 (VM)。 當您對於計算環境所需的控制權比其他選擇還要多時，您可以選擇 VM。 在您建立 VM 之前，本文會提供詳細資料。

Azure Stack VM 供虛擬化的彈性，您不需要管理叢集或個別機器。 不過，您仍然需要執行工作來維護 VM，例如設定、修補和安裝在 VM 上執行的軟體。

Azure Stack 虛擬機器有多種用途。 例如︰

- **開發和測試**  
    Azure Stack VM 提供快速又簡單的方法來建立電腦，讓電腦具備撰寫和測試應用程式所需的特定設定。

- **雲端中的應用程式**  
    因為對於應用程式的需求會變動，在 Azure Stack 中的 VM 上執行應用程式會較具經濟效益。 當您需要 VM 時便支付額外的 VM，而當您不需要時便關閉這些 VM。

- **擴充的資料中心**  
    Azure Stack 虛擬網路中的虛擬機器很容易連線至組織的網路或 Azure。

您的應用程式所使用的 VM 數目可以相應增加或相應放大為符合個人需求的任何內容。

## <a name="what-do-i-need-to-think-about-before-creating-a-vm"></a>我在建立 VM 之前需要先考慮什麼？

當您在 Azure Stack 中建置應用程式基礎結構時，一定會面臨許多設計考量。 在您開始建立基礎結構之前，仔細考量 VM 的下列層面是很重要的︰

- 應用程式資源的名稱。
- VM 的大小。
- 可建立的 VM 數目上限。
- VM 執行的作業系統。
- VM 啟動後的設定。
- VM 需要的相關資源。

### <a name="naming"></a>命名

虛擬機器會被指派名稱，也具有作業系統中所設定的電腦名稱。 VM 的名稱最多可為 15 個字元。

如果您使用 Azure Stack 來建立作業系統磁碟，則電腦名稱和虛擬機器名稱相同。 如果您上傳並使用您自己的映像 (該映像包含先前所設定的作業系統)，並用它來建立虛擬機器，則名稱可能會不同。 當您上傳自己的映像檔時，作業系統中的電腦名稱和虛擬機器名稱最好相同。

### <a name="vm-size"></a>VM 大小

您使用的 VM 大小取決於您要執行的工作負載。 您所選的大小會決定例如處理電源、記憶體和儲存體容量等因素。 Azure Stack 提供各種不同的大小來支援許多用途。

### <a name="vm-limits"></a>VM 限制

訂用帳戶設有預設配額限制，可能會影響如何部署專案的許多 VM。 每一訂用帳戶目前的限制是每一區域 20 個 VM。

### <a name="operating-system-disks-and-images"></a>作業系統磁碟和映像

虛擬機器是使用虛擬硬碟 (VHD) 來儲存其作業系統 (OS) 和資料。 VHD 也能夠使用於您可以選擇用來安裝 OS 的映像。
Azure Stack 提供一個市集，適用於各種版本和類型的作業系統。 Marketplace 映像是依映像發行者、供應項目、SKU 和版本 (版本通常會指定為最新版本) 來識別。

下表顯示一些方法讓您找到映像的資訊：

|方法|說明|
|---------|---------|
|Azure Stack 入口網站|當您選取要使用的影像時，會自動為您指定值。|
|Azure Stack PowerShell|`Get-AzureRMVMImagePublisher -Location "location"`<br>`Get-AzureRMVMImageOffer -Location "location" -Publisher "publisherName"`<br>`Get-AzureRMVMImageSku -Location "location" -Publisher "publisherName" -Offer "offerName"`|
|REST API     |[列出映像發行者](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publishers)<br>[列出映像供應項目](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publisher-offers)<br>[列出映像 SKU](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus)|

您可以選擇上傳並使用自己的映像。 如果這樣做，則不會使用發行者名稱、供應項目和 SKU。

### <a name="extensions"></a>擴充功能

VM 擴充可透過部署後設定及自動化工作，讓您的 VM 有更多功能。
可以使用擴充功能來完成這些常見工作︰

- **執行自訂指令碼**  
    「自訂指令碼擴充」可在佈建 VM 時執行您的指令碼，以協助您在 VM 上設定工作負載。

- **部署和管理設定**  
    「PowerShell 期望狀態設定 (DSC) 擴充」可協助您在 VM 上設定 DSC 來管理設定和環境。

- **收集診斷資料**  
    「Azure 診斷擴充」可協助您設定 VM 來收集診斷資料，用來監視應用程式的健康情況。

### <a name="related-resources"></a>相關資源

下表中的資源由 VM 使用，在建立 VM 時必須存在或已建立。


|資源|必要|說明|
|---------|---------|---------|
|資源群組|是|VM 必須包含在資源群組中。|
|儲存體帳戶|否|如果使用受控磁碟，VM 就不需要有儲存體帳戶來儲存其虛擬硬碟。 <br>如果使用非受控磁碟，VM 就需要有儲存體帳戶來儲存其虛擬硬碟。|
|虛擬網路|是|VM 必須是虛擬網路的成員。|
|公用 IP 位址|否|可以有公用 IP 位址指派給 VM，以從遠端存取它。|
|Linux|是|VM 需要網路介面以在網路中進行通訊。|
|資料磁碟|否|VM 可以包含資料磁碟來擴充儲存體功能。|

## <a name="create-your-first-vm"></a>建立您的第一個 VM

建立 VM 有多種選擇。 您的選擇取決於您的環境。
下表提供資訊來協助您開始建立 VM。


|方法|文章|
|---------|---------|
|Azure Stack 入口網站|使用 Azure Stack 入口網站建立 Windows 虛擬機器<br>[使用 Azure Stack 入口網站建立 Linux 虛擬機器](azure-stack-quick-linux-portal.md)|
|範本|Azure Stack 快速入門範本位於：<br> [https://github.com/Azure/AzureStack-QuickStart-Templates](https://github.com/Azure/AzureStack-QuickStart-Templates)|
|PowerShell|[在 Azure Stack 中使用 PowerShell 建立 Windows 虛擬機器](azure-stack-quick-create-vm-windows-powershell.md)<br>[在 Azure Stack 中使用 PowerShell 建立 Linux 虛擬機器](azure-stack-quick-create-vm-linux-powershell.md)|
|CLI|[在 Azure Stack 中使用 CLI 建立 Windows 虛擬機器](azure-stack-quick-create-vm-windows-cli.md)<br>[在 Azure Stack 中使用 CLI 建立 Linux 虛擬機器](azure-stack-quick-create-vm-linux-cli.md)|

## <a name="manage-your-vm"></a>管理您的 VM

您可以使用以瀏覽器為基礎的入口網站、支援指令碼處理的命令列工具，或直接透過 API 管理 VM。 您可能會執行的一般管理工作包括：

- 取得 VM 的相關資訊
- 連線至 VM
- 管理可用性
- 建立備份

### <a name="get-information-about-your-vm"></a>取得 VM 的相關資訊

下表顯示一些方法讓您取得 VM 的相關資訊。


|方法|說明|
|---------|---------|
|Azure Stack 入口網站|在 [中樞] 功能表中，按一下 [虛擬機器]，然後從清單中選取 VM。 在 VM 的頁面上，您可以存取概觀資訊、設定值及監視計量。|
|Azure PowerShell|在 Azure 和 Azure Stack 中，管理 VM 的方法很相似。 如需有關使用 PowerShell 的詳細資訊，請參閱下列 Azure 主題：<br>[使用 Azure PowerShell 模組建立和管理 Windows VM](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-manage-vm#understand-vm-sizes)|
|用戶端 SDK|在 Azure 和 Azure Stack 中，使用 C# 來管理 VM 的方法很相似。 如需詳細資訊，請參閱下列 Azure 主題：<br>[在 Azure 中使用 C# 建立和管理 Windows VM](https://docs.microsoft.com/azure/virtual-machines/windows/csharp)|

### <a name="connect-to-your-vm"></a>連接到您的 VM

在 Azure Stack 入口網站中，您可以使用 [連線] 按鈕來連線至 VM。

## <a name="next-steps"></a>後續步驟

- [Azure Stack 中虛擬機器的考量](azure-stack-vm-considerations.md)
