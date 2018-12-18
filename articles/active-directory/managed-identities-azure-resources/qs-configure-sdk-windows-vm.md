---
title: Azure SDK 可用來以 Azure 資源管理的受控識別設定 VM
description: 使用 Azure SDK 在 Azure VM 上設定和使用 Azure 資源的受控識別適用的逐步指示。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/28/2017
ms.author: daveba
ms.openlocfilehash: 3b485fd0f655588ec5ae7941bcb6a43ca7728134
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47106469"
---
# <a name="configure-a-vm-with-managed-identities-for-azure-resources-using-an-azure-sdk"></a>使用 Azure SDK 以 Azure 資源管理的受控識別設定 VM

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

適用於 Azure 資源的受控識別會在 Azure Active Directory (AD) 中為 Azure 服務提供自動的受控識別。 您可以使用此身分識別來向任何支援 Azure AD 驗證的服務進行驗證，不需要任何您程式碼中的認證。 

在本文中，您將了解如何使用 Azure SDK 來啟用和移除 Azure VM 的 Azure 資源受控識別。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

## <a name="azure-sdks-with-managed-identities-for-azure-resources-support"></a>Azure 資源受控識別支援的 Azure SDK 

Azure 透過一系列的 [Azure SDK](https://azure.microsoft.com/downloads) 來支援多個程式設計平台。 其中許多都已更新來支援 Azure 資源的受控識別，還提供對應的範例來示範用法。 這份清單會隨著新增額外的支援而更新：

| SDK | 範例 |
| --- | ------ | 
| .NET   | [從 Azure 資源的受控識別啟用的虛擬機器管理資源](https://azure.microsoft.com/resources/samples/aad-dotnet-manage-resources-from-vm-with-msi/) |
| Java   | [從 Azure 資源的受控識別啟用的虛擬機器管理儲存體](https://azure.microsoft.com/resources/samples/compute-java-manage-resources-from-vm-with-msi-in-aad-group/)|
| Node.js| [建立已啟用系統指派受控識別的虛擬機器](https://azure.microsoft.com/resources/samples/compute-node-msi-vm/) |
| Python | [建立已啟用系統指派受控識別的虛擬機器](https://azure.microsoft.com/resources/samples/compute-python-msi-vm/) |
| Ruby   | [建立已啟用系統指派身分識別的 Azure VM](https://azure.microsoft.com/resources/samples/compute-ruby-msi-vm/) |

## <a name="next-steps"></a>後續步驟

- 若要了解您還可以如何使用 Azure 入口網站、PowerShell、CLI 和資源範本，請參閱**設定 Azure VM 身分識別**底下的相關文章。
