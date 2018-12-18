---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 517ebbf739c64c0364dc21386fee86ebc740e997
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/20/2018
ms.locfileid: "30198123"
---
當您產生用戶端憑證時，它會自動安裝於您用來產生它的電腦上。 如果您想要在另一部用戶端電腦上安裝用戶端憑證，您必須匯出您所產生的用戶端憑證。

1. 若要匯出用戶端憑證，請開啟 [管理使用者憑證]。 根據預設，您產生的用戶端憑證位於 'Certificates - Current User\Personal\Certificates'。 在您想要匯出的用戶端憑證上按一下滑鼠右鍵，然後依序按一下 [所有工作]、[匯出]，以開啟 [憑證匯出精靈]。

  ![匯出](./media/vpn-gateway-certificates-export-client-cert-include/export.png)
2. 在 [憑證匯出精靈] 中，按 [下一步] 繼續作業。

  ![下一頁](./media/vpn-gateway-certificates-export-client-cert-include/next.png)
3. 選取 [是，匯出私密金鑰]，然後按 [下一步]。

  ![匯出私密金鑰](./media/vpn-gateway-certificates-export-client-cert-include/privatekeyexport.png)
4. 在 [匯出檔案格式] 頁面上，保留選取預設值。 務必選取 [如果可能的話，包含憑證路徑中的所有憑證]。 此設定會額外匯出成功的用戶端驗證所需的根憑證資訊。 若缺少這項資訊，用戶端驗證即會因為用戶端沒有信任的根憑證而失敗。 然後按 [下一步] 。

  ![匯出檔案格式](./media/vpn-gateway-certificates-export-client-cert-include/includeallcerts.png)
5. 在 [安全性]  頁面上，您必須保護私密金鑰。 如果您選取要使用密碼，請務必記錄或牢記您為此憑證設定的密碼。 然後按 [下一步] 。

  ![security](./media/vpn-gateway-certificates-export-client-cert-include/security.png)
6. 在 [要匯出的檔案] 中，[瀏覽] 到您要匯出憑證的位置。 針對 [檔案名稱] ，請為憑證檔案命名。 然後按 [下一步] 。

  ![要匯出的檔案](./media/vpn-gateway-certificates-export-client-cert-include/filetoexport.png)
7. 按一下 [完成]  以匯出憑證。

  ![完成](./media/vpn-gateway-certificates-export-client-cert-include/finish.png)