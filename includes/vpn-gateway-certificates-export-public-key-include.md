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
ms.openlocfilehash: 4ae4cfb91fb3a746c73d6b098a1adc9e4dee8698
ms.sourcegitcommit: caebf2bb2fc6574aeee1b46d694a61f8b9243198
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/12/2018
ms.locfileid: "35414700"
---
建立自我簽署根憑證之後，請匯出根憑證公開金鑰.cer 檔案 (而非私密金鑰)。 您稍後會將此檔案上傳至 Azure。 下列步驟可協助您匯出自我簽署根憑證的 .cer 檔案：

1. 若要取得憑證的 .cer 檔案，請開啟 [管理使用者憑證]。 找出自我簽署的根憑證，通常位於 '[憑證 - 目前的使用者]\[個人]\[憑證]' 中，然後按一下滑鼠右鍵。 按一下 [所有工作]，然後按一下 [匯出]。 這會開啟 [憑證匯出精靈] 。 如果您在「目前使用者\個人\憑證」底下找不到憑證，可能是因為您已針對本機電腦憑證開啟 [憑證管理員] (標題是相對於「憑證 - 目前使用者」的「憑證 - 本機電腦」)。 若要在目前使用者範圍開啟 [憑證管理員]，請從相同的 PowerShell (在其中是藉由輸入 ```certmgr``` 來建立憑證) 啟動它。

  ![匯出](./media/vpn-gateway-certificates-export-public-key-include/export.png)
2. 在精靈中，按 [下一步]。

  ![匯出憑證](./media/vpn-gateway-certificates-export-public-key-include/exportwizard.png)
3. 選取 [否，不要匯出私密金鑰]，然後按 [下一步]。

  ![不要匯出私密金鑰](./media/vpn-gateway-certificates-export-public-key-include/notprivatekey.png)
4. 在 [匯出檔案格式] 頁面上，選取 [Base-64 編碼 X.509 (.CER)]，然後按 [下一步]。

  ![Base-64 編碼](./media/vpn-gateway-certificates-export-public-key-include/base64.png)
5. 針對 [要匯出的檔案]，[瀏覽] 至您要匯出憑證的位置。 針對 [檔案名稱] ，請為憑證檔案命名。 然後按 [下一步] 。

  ![瀏覽](./media/vpn-gateway-certificates-export-public-key-include/browse.png)
6. 按一下 [完成]  以匯出憑證。

  ![完成](./media/vpn-gateway-certificates-export-public-key-include/finish.png)
7. 已成功匯出您的憑證。

  ![成功](./media/vpn-gateway-certificates-export-public-key-include/success.png)
8. 匯出的憑證如下所示：

  ![已匯出](./media/vpn-gateway-certificates-export-public-key-include/exported.png)
9. 如果您使用「記事本」開啟匯出的憑證，您會看到類似於此範例的內容。 以藍色標示的部分包含上傳至 Azure 的資訊。 如果您使用「記事本」開啟您的憑證，但並未顯示這樣的內容，這通常表示您未使用 Base-64 編碼 X.509 (.CER) 格式加以匯出。 此外，如果您想要使用不同的文字編輯器，請了解某些編輯器可能會在背景中導入非預期的格式。 這可能會在此憑證中的文字上傳至 Azure 時產生問題。

  ![使用記事本開啟](./media/vpn-gateway-certificates-export-public-key-include/notepad.png)
