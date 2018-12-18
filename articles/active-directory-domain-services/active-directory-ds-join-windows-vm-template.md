---
title: 將 Windows Server VM 加入 Azure Active Directory Domain Services | Microsoft Docs
description: 使用 Azure Resource Manager 範本將 Windows Server 虛擬機器加入受控網域。
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 4eabfd8e-5509-4acd-86b5-1318147fddb5
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: 40eb0e8133626e77a1f61ddfc5df69ecf1b958f2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950789"
---
# <a name="join-a-windows-server-virtual-machine-to-a-managed-domain-using-a-resource-manager-template"></a>使用 Resource Manager 範本將 Windows Server 虛擬機器加入受控網域
本文示範如何使用 Resource Manager 範本，將 Windows Server 虛擬機器加入至 Azure Active Directory Domain Services 之受控網域。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要執行本文中所列的工作，您需要︰
1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. 確保您尚未將受控網域的 IP 位址設定為虛擬網路的 DNS 伺服器。 如需詳細資訊，請參閱[如何更新 Azure 虛擬網路的 DNS 設定](active-directory-ds-getting-started-dns.md)
5. 完成[將密碼同步處理至您的 Azure AD Domain Services 受控網域](active-directory-ds-getting-started-password-sync.md)所需的步驟。


## <a name="install-and-configure-required-tools"></a>安裝並設定必要的工具
您可以使用下列選項之一來執行這份文件所述的步驟：
* **Azure PowerShell**：[安裝和設定](https://azure.microsoft.com/documentation/articles/powershell-install-configure/)
* **Azure CLI**：[安裝及設定](https://azure.microsoft.com/documentation/articles/xplat-cli-install/)


## <a name="option-1-provision-a-new-windows-server-vm-and-join-it-to-a-managed-domain"></a>選項 1：佈建新的 Windows Server VM，並將它加入至受控網域
**快速入門範本名稱**：[201-vm-domain-join](https://azure.microsoft.com/resources/templates/201-vm-domain-join/)

若要部署 Windows Server 虛擬機器，並將它加入至受控網域，請執行下列步驟：
1. 瀏覽至[快速入門範本](https://azure.microsoft.com/resources/templates/201-vm-domain-join/)。
2. 按一下 [ **部署至 Azure**]。
3. 在 [自訂部署] 頁面上，提供必要的資訊來佈建虛擬機器。
4. 選取 [Azure 訂用帳戶] 來佈建虛擬機器。 挑選已啟用 Azure AD Domain Services 的相同 Azure 訂用帳戶。
5. 選擇現有的**資源群組**或建立新群組。
6. 挑選要部署新虛擬機器的**位置**。
7. 在 [現有 VNET 名稱]，指定已部署 Azure AD Domain Services 受控網域之虛擬網路。
8. 在 [現有的子網路名稱] 中，指定想要部署此虛擬機器之虛擬網路內的子網路。 請勿在虛擬網路中選取閘道子網路。 此外，請勿選取部署受控網域中的專用子網路。
9. 在 [DNS 標籤首碼] 中，指定正在佈建虛擬機器的主機名稱。 例如，'contoso-win'。
10. 選取虛擬機器的適當 **VM 大小**。
11. 在 [要加入的網域] 中，指定受控網域之 DNS 網域名稱。
12. 在 [網域使用者名稱]，指定受控網域上的使用者帳戶名稱，您應使用它將 VM 加入受控網域。
13. 在 [網域密碼] 中，使用 'domainUsername' 參數指定網域使用者帳戶參照到的密碼。
14. 選擇性：您可以指定自訂 OU 的 **OU 路徑**，在其中新增虛擬機器。 如果您未指定此參數的值，要將虛擬機器新增到受控網域上的預設 **AAD DC 電腦** OU。
15. 在 [VM 系統管理員使用者名稱] 欄位中，指定虛擬機器的本機系統管理員帳戶名稱。
16. 在 [VM 系統管理員密碼] 欄位中，指定虛擬機器的本機系統管理員密碼。 提供虛擬機器的強式本機系統管理員密碼，以防止暴力密碼破解攻擊。
17. 按一下 [我同意上方所述的條款及條件]。
18. 按一下 [購買] 來佈建虛擬機器。

> [!WARNING]
> **處理密碼時多加注意。**
> 範本參數檔案包含網域帳戶的密碼，以及虛擬機器的本機系統管理員密碼。 請確定您未保留檔案共用上的此檔案，或其他共用位置。 完成虛擬機器部署後，我們建議您處置此檔案。
>

順利完成部署之後，您最近佈建的 Windows 虛擬機器已加入受控網域。


## <a name="option-2-join-an-existing-windows-server-vm-to-a-managed-domain"></a>選項 2：將現有 Windows Server VM 加入受控網域
**快速入門範本**：[201-vm-domain-join-existing](https://azure.microsoft.com/resources/templates/201-vm-domain-join-existing/)

若要將現有的 Windows Server 虛擬機器加入至受控網域，請執行下列步驟：
1. 瀏覽至[快速入門範本](https://azure.microsoft.com/resources/templates/201-vm-domain-join-existing/)。
2. 按一下 [ **部署至 Azure**]。
3. 在 [自訂部署] 頁面上，提供必要的資訊來佈建虛擬機器。
4. 選取 [Azure 訂用帳戶] 來佈建虛擬機器。 挑選已啟用 Azure AD Domain Services 的相同 Azure 訂用帳戶。
5. 選擇現有的**資源群組**或建立新群組。
6. 挑選要部署新虛擬機器的**位置**。
7. 在 [VM 清單] 欄位中，指定要加入至受控網域之現有虛擬機器的名稱。 使用逗號來分隔個別的 VM 名稱。 例如，**contoso-web, contoso-api**。
8. 在 [網域加入使用者名稱]，指定受控網域上的使用者帳戶名稱，您應使用它將 VM 加入受控網域。
9. 在 [網域加入使用者密碼] 中，使用 'domainUsername' 參數指定網域使用者帳戶參照到的密碼。
10. 在 [網域 FQDN] 中，指定受控網域之 DNS 網域名稱。
11. 選擇性：您可以指定自訂 OU 的 **OU 路徑**，在其中新增虛擬機器。 如果您未指定此參數的值，要將虛擬機器新增到受控網域上的預設 **AAD DC 電腦** OU。
12. 按一下 [我同意上方所述的條款及條件]。
13. 按一下 [購買] 來佈建虛擬機器。

> [!WARNING]
> **處理密碼時多加注意。**
> 範本參數檔案包含網域帳戶的密碼，以及虛擬機器的本機系統管理員密碼。 請確定您未保留檔案共用上的此檔案，或其他共用位置。 完成虛擬機器部署後，我們建議您處置此檔案。
>

順利完成部署之後，指定的 Windows 虛擬機器會加入至受控網域。


## <a name="related-content"></a>相關內容
* [Azure PowerShell 的概觀](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-4.4.0)
* [Azure 快速入門範本 - 網域加入新的 VM](https://azure.microsoft.com/resources/templates/201-vm-domain-join/)
* [Azure 快速入門範本 - 網域加入現有的 VM](https://azure.microsoft.com/resources/templates/201-vm-domain-join-existing/)
* [使用 Resource Manager 範本與 Azure PowerShell 來部署資源](../azure-resource-manager/resource-group-template-deploy.md)
