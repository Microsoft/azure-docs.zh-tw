---
title: 包含檔案
description: 包含檔案
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 05/17/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: cfe675ca269a69c7c2bfa67638acd0afbcd1c8ea
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34371281"
---
每個端點都有一個公用連接埠和一個私人連接埠：

* Azure 負載平衡器使用公用連接埠接聽從網際網路到虛擬機器的連入流量。
* 虛擬機器使用私人連接埠接聽目的地通常為虛擬機器上執行的應用程式或服務的連入流量。

當您使用 Azure 入口網站建立端點時，會提供 IP 通訊協定的預設值以及已知網路通訊協定的 TCP 或 UDP 通訊埠。 針對自訂端點，您必須指定正確的 IP 通訊協定 (TCP 或 UDP) 以及公用和私人連接埠。 若要將連入流量隨機分散到多部虛擬機器，您必須建立負載平衡的集合，其中包含多個端點。

建立端點之後，您可以使用存取控制清單 (ACL) 定義規則，根據來源 IP 位址允許或拒絕端點公用連接埠的連入流量。 不過，如果虛擬機器位於 Azure 虛擬網路，請改用網路安全性群組。 如需詳細資訊，請參閱 [關於網路安全性群組](../articles/virtual-network/security-overview.md)。

> [!NOTE]
> Azure 虛擬機器的防火牆組態會針對連接埠自動完成，該連接埠與 Azure 自動設定的遠端連線端點相關聯。 至於其他所有端點的指定連接埠，不會自動設定虛擬機器的防火牆。 您建立虛擬機器的端點時，需要確定虛擬機器的防火牆也允許端點組態相對應通訊協定和私人連接埠的流量。 若要設定防火牆，請參閱文件或虛擬機器上執行之作業系統的線上說明。
>
>

## <a name="create-an-endpoint"></a>建立端點
1. 如果您尚未登入 [Azure 入口網站](https://portal.azure.com)，請先登入。
2. 按一下 [虛擬機器] ，然後按一下要設定的虛擬機器名稱。
3. 按一下 [設定] 群組中的 [端點]。 [端點] 頁面會列出虛擬機器目前的所有端點。 Linux VM 依預設會顯示 SSH 的端點。 )

   <!-- ![Endpoints](./media/virtual-machines-common-classic-setup-endpoints/endpointswindows.png) -->
   ![端點](./media/virtual-machines-common-classic-setup-endpoints/endpointsblade.png)

4. 在端點項目上方的命令列中，按一下 [新增]。
5. 在 [新增端點] 頁面的 [名稱] 中，輸入端點的名稱。
6. 在 [通訊協定] 中選擇 [TCP] 或 [UDP]。
7. 在 [公用連接埠] 中，輸入網際網路連入流量的連接埠號碼。 在 [私人連接埠] 中，輸入虛擬機器所接聽的連接埠號碼。 這些連接埠號碼可以不同。 請確定已經設定虛擬機器的防火牆允許通訊協定 (在步驟 6 中) 和私人連接埠對應的流量。
10. 按一下 [確定] 。

新端點將會列在 [ **端點** ] 頁面上。

![端點建立成功](./media/virtual-machines-common-classic-setup-endpoints/endpointcreated.png)

## <a name="manage-the-acl-on-an-endpoint"></a>在端點上管理 ACL
為了定義可以傳送流量的電腦集合，端點上的 ACL 能夠根據來源 IP 位址限制流量。 請依照這些步驟，在端點上新增、修改或移除 ACL。

> [!NOTE]
> 如果端點屬於負載平衡集合，則對端點上的 ACL 所做的任何變更，都將套用至此集合的所有端點。
>
>

如果虛擬機器位於 Azure 虛擬網路，建議使用網路安全性群組而非 ACL。 如需詳細資訊，請參閱 [關於網路安全性群組](../articles/virtual-network/security-overview.md)。

1. 如果您尚未登入 Azure 入口網站，請先登入。
2. 按一下 [虛擬機器] ，然後按一下要設定的虛擬機器名稱。
3. 按一下 [端點] 。 從清單中選取適當的端點。 ACL 清單位於頁面底部。

   ![指定 ACL 詳細資料](./media/virtual-machines-common-classic-setup-endpoints/aclpreentry.png)

4. 使用清單中的資料列來新增、刪除或編輯 ACL 的規則並變更奇順序。 [ **遠端子網路** ] 值是網際網路連入流量的 IP 位址範圍，Azure 負載平衡器會根據流量的來源 IP 位址允許或拒絕流量。 請務必指定 CIDR 格式 (也就是位址前置詞格式) 的 IP 位址範圍。 例如 `10.1.0.0/8`。

 ![新增 ACL 項目](./media/virtual-machines-common-classic-setup-endpoints/newaclentry.png)


您可以使用規則僅允許網際網路上的電腦對應的特定電腦流量，或拒絕特定已知位址範圍的流量。

規則的評估順序是從第一個規則開始，一直到最後一個規則為止。 這表示規則應會以最寬鬆到最嚴格的順序來排列。 如需範例和詳細資訊，請參閱[什麼是網路存取控制清單](../articles/virtual-network/virtual-networks-acl.md)。
