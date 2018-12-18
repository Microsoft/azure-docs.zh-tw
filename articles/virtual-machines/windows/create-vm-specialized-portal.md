---
title: 在 Azure 入口網站中從特製化 VHD 建立 Windows VM | Microsoft Docs
description: 在 Azure 入口網站中從 VHD 建立新的 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 01/09/2018
ms.author: cynthn
ms.openlocfilehash: 428003747b7c746a2849a042e54647e86361c562
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34716555"
---
# <a name="create-a-vm-from-a-vhd-using-the-azure-portal"></a>使用 Azure 入口網站從 VHD 建立 VM


有數種方法能在 Azure 中建立 VM。 如果您已有可用的 VHD，或想從現有 VM 複製 VHD 以使用，您可以將該 VHD 連接為 OS 磁碟來建立新的 VM。 此程序會將 VHD 作為 OS 磁碟*附加*至新的 VM。

您也可以從已刪除之 VM 的 VHD 建立新的 VM。 例如，如果您有運作不正常的 Azure VM，您可以刪除該 VM，然後使用 VHD 來建立新的 VM。 您可以重複使用相同的 VHD，或是建立快照集來建立該 VHD 的複本，然後從快照集建立新的受控磁碟。 這需要進行另外幾個步驟，但可確保您能保留原始的 VHD，並提供可視需要作為備援的快照集。

您有想用來在 Azure 中建立 VM 的內部部署 VM。 您可以上傳 VHD，然後將它附加至新的 VM。 若要上傳 VHD，您必須使用 PowerShell 或其他工具來將它上傳至儲存體帳戶，然後從 VHD 建立受控磁碟。 如需詳細資訊，請參閱[上傳特製化 VHD](create-vm-specialized.md#option-2-upload-a-specialized-vhd)

如果您想要使用 VM 或 VHD 來建立多個 VM，則不應使用此方法。 針對較大型的部署，您應該[建立映像](capture-image-resource.md)，然後[使用該映像來建立多個 VM](create-vm-generalized-managed.md)。


## <a name="copy-a-disk"></a>複製磁碟

建立快照集，然後從該快照集建立磁碟。 這可讓您保留原始 VHD 作為備援。

1. 在左側功能表中，按一下 [所有資源]。
2. 在 [所有類型] 下拉式清單中，取消選取 [全選]，然後向下捲動並選取 [磁碟]，以找到可用的磁碟。
3. 按一下您要使用的磁碟。 該磁碟的 [概觀] 頁面隨即開啟。
4. 在 [概觀] 頁面中，於上方功能表按一下 [+ 建立快照集]。 
5. 輸入快照集的名稱。
6. 選擇快照集的 [資源群組]。 您可以使用現有的資源群組，或建立一個新的群組。
7. 選擇要使用標準 (HDD) 或進階 (SDD) 儲存體。
8. 完成時，請按一下 [建立] 來建立快照集。
9. 建立快照集之後，請按一下左側功能表中的 [+ 建立資源]。
10. 在搜尋列中，輸入**受控磁碟**，然後從清單中選取 [受控磁碟]。
11. 在 [受控磁碟] 頁面上，按一下 [建立]。
12. 輸入磁碟的名稱。
13. 選擇磁碟的 [資源群組]。 您可以使用現有的資源群組，或建立一個新的群組。 這也會是您從磁碟所建立 VM 的資源群組。
14. 選擇要使用標準 (HDD) 或進階 (SDD) 儲存體。
15. 在 [來源類型] 中，請務必選取 [快照集]。
16. 在 [來源快照集] 下拉式清單中，選取您想要使用的快照集。
17. 視需要做出其他調整，然後按一下 [建立] 來建立磁碟。

## <a name="create-a-vm-from-a-disk"></a>從磁碟建立 VM

有了要使用的受控磁碟 VHD 之後，便可以在入口網站中建立 VM。

1. 在左側功能表中，按一下 [所有資源]。
2. 在 [所有類型] 下拉式清單中，取消選取 [全選]，然後向下捲動並選取 [磁碟]，以找到可用的磁碟。
3. 按一下您要使用的磁碟。 該磁碟的 [概觀] 頁面隨即開啟。
在 [概觀] 頁面中，請確定列出的 [磁碟狀態] 為 [未連結]。 如果不是，您可能必須將磁碟和 VM 中斷連接，或刪除 VM 以釋出該磁碟。
4. 在窗格頂端的功能表中，按一下 [+ 建立 VM]。
5. 在新 VM 的 [基本] 頁面上輸入一個名稱，然後選取現有的資源群組或建立一個新的群組。
6. 在 [大小] 頁面上，選取 VM 大小頁面，然後按一下 [選取]。
7. 在 [設定] 頁面上，您可以讓入口網站建立所有的新資源，或選取現有的**虛擬網路**和**網路安全性群組**。 入口網站一律會為新 VM 建立新的 NIC 和公用 IP 位址。 
8. 視需要對監視選項做出變更並新增擴充功能。
9. 完成時按一下 [確定]。 
10. 如果 VM 設定通過驗證，請按一下 [確定] 以開始部署。

## <a name="next-steps"></a>後續步驟

您也可以使用 PowerShell 來[將 VHD 上傳至 Azure 並建立特製化 VM](create-vm-specialized.md)。


