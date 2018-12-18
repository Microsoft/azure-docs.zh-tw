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
ms.openlocfilehash: 9fa18b14b82376a25bb434acd770d340b1ef9262
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30197029"
---
如果您連線時遇到問題，請檢查下列項目︰

- 如果您已匯出用戶端憑證，確定您使用預設值 [如果可能的話，包含憑證路徑中的所有憑證] 將它匯出為 .pfx 檔案。 當您使用此值匯出它時，根憑證資訊也會一併匯出。 在用戶端電腦上安裝憑證時，.pfx 檔案內含的根憑證也會安裝於用戶端電腦上。 用戶端電腦必須已安裝根憑證資訊。 若要檢查，請移至 [管理使用者憑證] 並瀏覽至 [受信任的根憑證授權單位]\[憑證]。 確認已列出根憑證。 必須要有根憑證驗證才可運作。

- 如果您使用的是藉由企業 CA 解決方案簽發的憑證，而在驗證時發生問題，請檢查用戶端憑證上的驗證順序。 您可以按兩下用戶端憑證，然後移至 [詳細資料] > [增強金鑰使用方法]，來檢查驗證清單順序。 請確定清單所顯示的第一個項目是「用戶端驗證」。 如果不是，您就必須根據以「用戶端驗證」作為清單中第一個項目的「使用者」範本來簽發用戶端憑證。

- 如需其他 P2S 疑難排解詳細資訊，請參閱[針對 P2S 連線進行疑難排解](../articles/vpn-gateway/vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)。