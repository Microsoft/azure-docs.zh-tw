---
title: 了解使用服務和建置應用程式時 Azure 與 Azure Stack 之間的主要差異 | Microsoft Docs
description: 使用服務或為 Azure Stack 建置應用程式時，您需要了解的事項。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: c81f551d-c13e-47d9-a5c2-eb1ea4806228
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 09/12/2018
ms.author: sethm
ms.openlocfilehash: 9f229caa76059db403ced5b74c7a35ec2f0b8e9d
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44721875"
---
# <a name="key-considerations-using-services-or-building-apps-for-azure-stack"></a>關鍵考量：使用服務或為 Azure Stack 建置應用程式

在使用服務或為 Azure Stack 建置應用程式之前，您必須先了解 Azure Stack 與 Azure 之間的差異。 此文章指出當您以 Azure Stack 作為混合式雲端開發環境時的主要考量事項。

## <a name="overview"></a>概觀

Azure Stack 是混合式雲端平台，可讓您從您的公司或服務提供者的資料中心使用 Azure 服務。 您可以在 Azure Stack 上建置應用程式，然後將它部署至 Azure Stack、Azure 或 Azure 混合式雲端。

您的 Azure Stack 操作員會告訴您有哪些服務可供您使用，以及如何取得支援。 他們會透過其自訂方案和供應項目來提供這些服務。

Azure 技術內容會假設應用程式是為了 Azure 服務 (而不是 Azure Stack) 開發。 建置和部署應用程式到 Azure Stack 時，必須了解一些主要差異，例如：

* Azure Stack 提供 Azure 中所提供服務和功能的**子集**。
* 您的公司或服務提供者可以選擇他們想要提供的服務。 可用的選項可能包括自訂的服務或應用程式。 他們可能會提供自己的自訂文件。
* 您必須使用正確的 Azure Stack 特定端點 (例如，入口網站位址的 URL 和 Azure Resource Manager 端點)。
* 您必須使用 Azure Stack 所支援的 PowerShell 和 API 版本。 使用支援的版本可確保您的應用程式能在 Azure Stack 和 Azure 中運作。

## <a name="cheat-sheet-high-level-differences"></a>功能提要：高層差異

下表描述 Azure Stack 與 Azure 之間的高層差異。 當您為 Azure Stack 進行開發，或使用 Azure Stack 服務時，請記住這些差異。
*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

| 領域 | Azure (全域) | Azure Stack |
| -------- | ------------- | ----------|
| 由誰操作？ | Microsoft | 您的組織或服務提供者。|
| 尋求支援的連絡對象？ | Microsoft | 如果是整合環境，請連絡您的 Azure Stack 操作員 (位於您的組織或服務提供者) 以取得支援。<br><br>如需 Azure Stack 開發套件的支援，請瀏覽 [Microsoft 論壇](https://social.msdn.microsoft.com/Forums/home?forum=azurestack)。 因為開發套件屬於評估環境，因此 Microsoft 客戶支援服務 (CSS) 不會提供官方支援。
| 可用服務 | 查看 [Azure 產品](https://azure.microsoft.com/services/?b=17.04b)的清單。 可用的服務因 Azure 區域而異。 | Azure Stack 支援 Azure 服務的子集。 實際服務會因您組織或服務提供者選擇提供的項目而異。
| Azure Resource Manager 端點* | https://management.azure.com | 如果是 Azure Stack 整合系統，請使用您 Azure Stack 操作員所提供的端點。<br><br>如果是開發套件，請使用： https://management.local.azurestack.external
| 入口網站 URL* | [https://portal.azure.com](https://portal.azure.com) | 針對 Azure Stack 整合系統，請移至您 Azure Stack 操作員所提供的 URL。<br><br>如果是開發套件，請使用： https://portal.local.azurestack.external
| 區域 | 您可以選取想要部署的目標區域。 | 如果是 Azure Stack 整合系統，請使用您系統上可用的區域。<br><br>若為開發套件，區域一律為**本機**。
| 資源群組 | 資源群組可以跨區域。 | 就整合系統和開發套件兩者而言，只有一個區域。
|支援的命名空間、資源類型和 API 版本 | 最新版本 (或未過時的較早版本)。 | Azure Stack 支援特定版本。 請參閱此文章的＜版本需求＞一節。
| | |

*如果您是 Azure Stack 操作員，請參閱[使用系統管理員入口網站](../azure-stack-manage-portals.md)和[管理基本知識](../azure-stack-manage-basics.md)，以了解詳細資訊。

## <a name="helpful-tools-and-best-practices"></a>實用工具和最佳作法
 
 Microsoft 提供數個工具和指引來協助您為 Azure Stack 進行開發。

| 建議 | 參考 | 
| -------- | ------------- | 
| 在開發人員工作站上安裝正確的工具。 | - [安裝 PowerShell](azure-stack-powershell-install.md)<br>- [下載工具](azure-stack-powershell-download.md)<br>- [設定 PowerShell](azure-stack-powershell-configure-user.md)<br>- [安裝 Visual Studio](azure-stack-install-visual-studio.md) 
| 檢閱有關下列各項的相關資訊：<br>- Azure Resource Manager 範本考量<br>- 如何尋找快速入門範本<br>- 使用原則模組可協助您使用 Azure 來為 Azure Stack 進行開發 | [開發 Azure Stack](azure-stack-developer.md) | 
| 檢閱並遵循範本的最佳作法。 | [Resource Manager 快速入門範本](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md#best-practices)
| | |

## <a name="version-requirements"></a>版本需求

Azure Stack 支援特定版本的 Azure PowerShell 和 Azure 服務 API。 使用支援的版本以確保您的應用程式可以同時部署至 Azure Stack 和 Azure。

若要確定您使用的是正確版本的 Azure PowerShell，請使用 [API 版本設定檔](azure-stack-version-profiles.md)。 若要判斷您可以使用的最新 API 版本設定檔，請找出您所使用的 Azure Stack 組建。 您可以向您的 Azure Stack 系統管理員取得此資訊。

>[!NOTE]
 如果您使用 Azure Stack 開發套件，而且您具有管理存取權，請參閱[管理更新](../azure-stack-updates.md#determine-the-current-version)的＜判斷目前的版本＞一節來判斷 Azure Stack 組建。

對於其他 API，執行下列 PowerShell 命令以輸出您的 Azure Stack 訂用帳戶中支援的命名空間、資源類型和 API 版本。 請注意，可能仍有屬性層級的差異。 (若要讓此命令運作，您必須已經為 Azure Stack 環境[安裝](azure-stack-powershell-install.md)和[設定](azure-stack-powershell-configure-user.md) PowerShell。 您也必須有 Azure Stack 供應項目的訂用帳戶。)

```powershell
Get-AzureRmResourceProvider | Select ProviderNamespace -Expand ResourceTypes | Select * -Expand ApiVersions | `
Select ProviderNamespace, ResourceTypeName, @{Name="ApiVersion"; Expression={$_}} 
```

範例輸出 (已截斷)：![Get-AzureRmResourceProvider 命令的範例輸出](media/azure-stack-considerations/image1.png)
 
## <a name="next-steps"></a>後續步驟

如需在服務層級差異的詳細資訊，請參閱：

* [Azure Stack 中虛擬機器的考量](azure-stack-vm-considerations.md)
* [Azure Stack 中儲存體的考量](azure-stack-acs-differences.md)
* [Azure Stack 網路服務的注意事項](azure-stack-network-differences.md)
