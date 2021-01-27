---
title: 使用入口網站建立 Azure 共用映射庫
description: 瞭解如何使用 Azure 入口網站來建立和共用虛擬機器映射。
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 11/06/2019
ms.author: cynthn
ms.openlocfilehash: 25cd75035a814fd718cc1101e6575f78c50f105e
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98879692"
---
# <a name="create-an-azure-shared-image-gallery-using-the-portal"></a>使用入口網站建立 Azure 共用映射庫

[共用映像資源庫](../shared-image-galleries.md)可簡化跨組織共用自訂映像。 自訂映像類似 Marketplace 映像，但您要自行建立它們。 自訂映像可用於啟動部署工作，例如，預先載入應用程式、應用程式設定和其他 OS 設定。 

共用映像資源庫可讓您與 AAD 租用戶中區域內或跨區域組織中的其他人共用自訂 VM 映像。 選擇您要共用的映像、您要開放使用的區域，以及您要共用的對象。 您可以建立多個資源庫，讓您可以根據邏輯群組共用映像。 

資源庫是最上層資源，可 (Azure RBAC) 提供完整的 Azure 角色型存取控制。 可建立映像版本，並且您可以選擇將每個映像版本複寫到不同的 Azure 區域集合。 資源庫僅適用於受控映像。

共用映像庫具有多個資源類型。 我們將在這篇文章中使用或建置這些資源類型：


[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]

<br>


逐步完成本文之後，請視需要取代資源群組和 VM 名稱。


[!INCLUDE [virtual-machines-common-shared-images-portal](../../../includes/virtual-machines-common-shared-images-portal.md)]
 
## <a name="create-vms"></a>建立 VM

現在您可以建立一或多個新的 Vm。 此範例會在 *美國東部* 資料中心的 *myResourceGroup* 中建立名為 *myVM* 的 VM。

1. 移至您的映射定義。 您可以使用資源篩選來顯示所有可用的映射定義。
1. 在您映射定義的頁面上，從頁面頂端的功能表中選取 [ **建立 VM** ]。
1. 針對 [ **資源群組**]，選取 [ **建立新** 的]，並輸入 *myResourceGroup* 作為 [名稱]。
1. 在 [ **虛擬機器名稱**] 中，輸入 *myVM*。
1. 在 [區域] 中，選取 [美國東部]。
1. 針對 [ **可用性選項**]，保留 [ *不需要基礎結構冗余*] 的預設值。
1.  `latest` 如果您是從映射定義的頁面開始，則影像的值會自動填入映射版本。
1. 針對 [ **大小**]，從可用大小清單中選擇 VM 大小，然後選擇 [ **選取**]。
1. 在 [ **系統管理員帳戶**] 下，如果映射是一般化的，您需要提供使用者名稱，例如 *>azureuser* 和密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](faq.md#what-are-the-password-requirements-when-creating-a-vm)。 如果您的映射是特製化的，使用者名稱和密碼欄位將呈現灰色，因為會使用來源 VM 的使用者名稱和密碼。
1. 如果您想要允許遠端存取 VM，請在 [ **公用輸入埠**] 下，選擇 [ **允許選取的埠** ]，然後從下拉式清單中選取 **RDP (3389)** 。 如果您不想要允許遠端存取 VM，請將 [ **未** 選取] 用於 **公用輸入埠**。
1. 當您完成時，請選取頁面底部的 [ **檢查 + 建立** ] 按鈕。
1. 在 VM 通過驗證之後，請選取頁面底部的 [ **建立** ] 以開始部署。


## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以刪除資源群組、虛擬機器和所有相關資源。 若要這樣做，請選取虛擬機器的資源群組，選取 [刪除]，然後確認要刪除的資源群組名稱。

如果您想要刪除個別資源，您需要以相反順序刪除它們。 例如，若要刪除映射定義，您必須刪除從該映射建立的所有映射版本。

## <a name="next-steps"></a>後續步驟

您也可以使用範本建立共用映像庫資源。 有數個 Azure 快速入門範本可以使用： 

- [建立共用映像資源庫](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [在共用映像資源庫中建立映像定義](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [在共用映像資源庫中建立映像版本](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [從映像版本建立 VM](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

如需共用映像資源庫的詳細資訊，請參閱[概觀](../shared-image-galleries.md)。 若遇到任何問題，請參閱[針對共用映像資源庫問題進行疑難排解](../troubleshooting-shared-images.md)。