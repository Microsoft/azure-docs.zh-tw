---
title: 如何管理 Azure 地圖服務帳戶和金鑰 | Microsoft Docs
description: 您可以使用 Azure 入口網站來管理 Azure 地圖服務帳戶以及管理存取金鑰。
author: dsk-2015
ms.author: dkshir
ms.date: 09/11/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 29fd63139d31d146cfc67153f35bd57650b08cfa
ms.sourcegitcommit: f983187566d165bc8540fdec5650edcc51a6350a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2018
ms.locfileid: "45541614"
---
# <a name="how-to-manage-your-azure-maps-account-and-keys"></a>如何管理 Azure 地圖服務帳戶和金鑰

您可以透過 Azure 入口網站管理 Azure 地圖服務帳戶和金鑰。 一旦擁有帳戶和金鑰之後，您就可以在網站或行動裝置應用程式中實作 API。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-a-new-account"></a>建立新帳戶

1. 登入 [Azure 入口網站](http://portal.azure.com)。

2. 按一下 Azure 入口網站左上角的 [建立資源]。

3. 搜尋並選取 [地圖服務]，然後按一下 [建立]。

4. 輸入新帳戶的資訊。

![在入口網站中輸入帳戶資訊](./media/how-to-manage-account-keys/new-account-portal.png)

## <a name="manage-keys-on-the-account-page"></a>在帳戶頁面上管理金鑰

建立帳戶後，您會獲得兩個隨機產生的金鑰。 當您想要擷取地圖資料或建立新的 JavaScript 地圖執行個體時，可以使用這兩個金鑰來對地圖服務 API 進行驗證。

您可以在 Azure 入口網站中找到金鑰。 瀏覽至帳戶，然後從功能表中選取 [金鑰]。

![在入口網站中管理帳戶金鑰](./media/how-to-manage-account-keys/account-keys-portal.png)

您可以從這個頁面複製金鑰，也可以產生新的金鑰。

## <a name="delete-an-account"></a>刪除帳戶

您可以從 Azure 入口網站刪除帳戶。 瀏覽至帳戶概觀頁面，然後選取 [刪除]。

![在入口網站中刪除您的帳戶](./media/how-to-manage-account-keys/account-delete-portal.png)

您將會看見確認頁面。 您可以藉由鍵入帳戶名稱來確認刪除該帳戶。

## <a name="next-steps"></a>後續步驟

* 了解如何使用 Azure 地圖服務搜尋API 來[搜尋地址](./how-to-search-for-address.md)。
* 了解如何使用 Azure 地圖服務的 [JavaScript 地圖控制項](./how-to-use-map-control.md)。
