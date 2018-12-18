---
title: 包含檔案
description: 包含檔案
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 09/19/2018
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 903074c78180ab2cd755abcf4207232f2851804e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47019672"
---
[Azure 檔案](../articles/storage/files/storage-files-introduction.md)透過 [Azure Active Directory (Azure AD) Domain Services](../articles/active-directory-domain-services/active-directory-ds-overview.md)，支援「透過 SMB (伺服器訊息區) (預覽) 進行以身分識別為基礎的驗證」。 您加入網域的 Windows 虛擬機器 (VM) 可以使用 [Azure AD](../articles/active-directory/fundamentals/active-directory-whatis.md) 認證存取 Azure 檔案共用。 

Azure AD 會使用 [ 角色型存取控制 (RBAC)](../articles/role-based-access-control/overview.md) 驗證身分識別，例如使用者、群組或服務主體。 您可以定義自訂 RBAC 角色，內有幾組用來存取 Azure 檔案的常用權限集。 在指派自訂 RBAC 角色給 Azure AD 身分識別時，會根據這些權限將 Azure 檔案共用的存取權授予身分識別。

在預覽中，Azure 檔案對所有檔案和檔案共用中的目錄，均支援 [NTFS DACL](https://technet.microsoft.com/library/2006.01.howitworksntfs.aspx) 的保留、繼承和強制執行。 如果您是從檔案共用將資料複製到 Azure 檔案服務，或從 Azure 檔案服務將資料複製到檔案共用，都可以指定維持 NTFS DACL。 如此一來，您可以使用 Azure 檔案實作備份作業，同時在內部部署檔案共用與雲端檔案共用之間保留您的 NTFS DACL。 

> [!NOTE]
> 預覽版本的 Linux VM 不支援透過 SMB 的 Azure AD 驗證。 只支援 Windows Server VM。
>
> Azure AD 驗證僅適用於在 2018 年 9 月 24 日之後建立的儲存體帳戶。
