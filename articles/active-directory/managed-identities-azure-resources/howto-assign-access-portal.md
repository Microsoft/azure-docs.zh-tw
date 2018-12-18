---
title: 如何使用 Azure 入口網站將 MSI 存取權指派給 Azure 資源
description: 提供如何使用 Azure 入口網站將一個資源的 MSI 存取權指派給另一個資源的逐步指示。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/14/2017
ms.author: daveba
ms.openlocfilehash: c2048583cde397ac3325fd149982b3a3db475566
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44157274"
---
# <a name="assign-a-managed-service-identity-access-to-a-resource-by-using-the-azure-portal"></a>使用 Azure 入口網站將受控服務識別 (MSI) 存取權指派給資源

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

以受控服務識別 (MSI) 設定 Azure 資源後，您就可以將 MSI 存取權提供給另一個資源，就像任何安全性主體一樣。 本文示範如何使用 Azure 入口網站將 Azure 虛擬機器或虛擬機器擴展集的 MSI 存取權提供給 Azure 儲存體帳戶。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

## <a name="use-rbac-to-assign-the-msi-access-to-another-resource"></a>使用 RBAC 將 MSI 存取權指派給另一個資源

您在 Azure 資源 ([例如 Azure VM](qs-configure-portal-windows-vm.md) 或 [Azure VMSS](qs-configure-portal-windows-vmss.md)) 上啟用 MSI 之後：

1. 使用您已設定 MSI 且與 Azure 訂用帳戶相關聯的帳戶，登入 [Azure 入口網站](https://portal.azure.com)。

2. 瀏覽至您要修改其存取控制的資源。 在此範例中，因為我們會將 Azure 虛擬機器和 Azure 虛擬機器擴展集存取權給予儲存體帳戶，所以瀏覽至儲存體帳戶。

3. 針對 Azure 虛擬機器，選取資源的 [存取控制 (IAM)] 頁面，然後選取 [+ 新增]。 接著，指定 [角色]、[指派權存權給「虛擬機器」]，以及指定資源所在的對應 [訂用帳戶] 和 [資源群組]。 在搜尋條件區域中，您應該會看到資源。 選取資源，然後選取 [儲存]。 

   ![存取控制 (IAM) 螢幕擷取畫面](./media/msi-howto-assign-access-portal/assign-access-control-iam-blade-before.png)  
   針對 Azure 虛擬機器擴展集，選取資源的 [存取控制 (IAM)] 頁面，然後選取 [+ 新增]。 然後指定 [角色]、[指派存取權的對象]。 在搜尋條件區域下，搜尋您的虛擬機器擴展集。 選取資源，然後選取 [儲存]。
   
   ![存取控制 (IAM) 螢幕擷取畫面](./media/msi-howto-assign-access-vmss-portal/assign-access-control-vmss-iam-blade-before.png)  

4. 回到 [存取控制 (IAM)] 主頁面後，您會看到資源 MSI 的新項目。

    Azure 虛擬機器：

   ![存取控制 (IAM) 螢幕擷取畫面](./media/msi-howto-assign-access-portal/assign-access-control-iam-blade-after.png)

    Azure 虛擬機器擴展集：

    ![存取控制 (IAM) 螢幕擷取畫面](./media/msi-howto-assign-access-vmss-portal/assign-access-control-vmss-iam-blade-after.png)

## <a name="troubleshooting"></a>疑難排解

如果資源的 MSI 沒有顯示在可用的身分識別清單中，請確認已正確啟用 MSI。 在我們的案例中，我們可以返回 Azure 虛擬機器來檢查下列情況：

- 查看 [設定] 頁面，確定 [MSI 已啟用] 的值為 [是]。
- 查看 [擴充功能] 頁面，並確定已成功部署 MSI 擴充功能 (**擴充功能**頁面不適用於 Azure 虛擬機器擴展集)。

如果其中任一項不正確，您可能需要在資源上重新部署 MSI，或針對部署失敗進行疑難排解。

## <a name="related-content"></a>相關內容

- 如需 MSI 的概觀，請參閱[受控服務識別概觀](overview.md)。
- 若要在 Azure 虛擬機器上啟用 MSI，請參閱[使用 Azure 入口網站設定 Azure VM 受控服務身分識別 (MSI)](qs-configure-portal-windows-vm.md)。
- 若要在 Azure 虛擬機器擴展集上啟用 MSI，請參閱[使用 Azure 入口網站設定 Azure 虛擬機器擴展集的受控服務身分識別 (MSI)](qs-configure-portal-windows-vmss.md)


