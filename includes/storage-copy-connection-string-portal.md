---
title: 包含檔案
description: 包含檔案
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 04/04/2018
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: bef314e60b962e39b3a55d0fb7acb40d4d1b4c32
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38732873"
---
## <a name="copy-your-credentials-from-the-azure-portal"></a>從 Azure 入口網站複製您的認證

範例應用程式需要驗證對您儲存體帳戶的存取。 若要進行驗證，您必須以連接字串的形式為應用程式提供您的儲存體帳戶認證。 若要檢視您的儲存體帳戶認證：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。
2. 找出您的儲存體帳戶。
3. 在儲存體帳戶概觀的 [設定] 區段中，選取 [存取金鑰]。 您的帳戶存取金鑰隨即出現，此外也會顯示每個金鑰的完整連接字串。   
4. 尋找 [金鑰1] 下方的 [連接字串] 值，然後按一下 [複製] 按鈕來複製連接字串。 在下一個步驟中，您會將連接字串值新增至環境變數。

    ![顯示如何從 Azure 入口網站複製連接字串的螢幕擷取畫面](media/storage-copy-connection-string-portal/portal-connection-string.png)
