---
title: 監視 Azure CDN 資源的健康狀態 | Microsoft Docs
description: 了解如何使用 Azure 資源健康狀態監視 Azure CDN 資源的健康狀態。
services: cdn
documentationcenter: .net
author: zhangmanling
manager: zhangmanling
editor: ''
ms.assetid: bf23bd89-35b2-4aca-ac7f-68ee02953f31
ms.service: cdn
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 37fe208f5087f318e665e76825127854b4a11c98
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22997923"
---
# <a name="monitor-the-health-of-azure-cdn-resources"></a>監視 Azure CDN 資源的健康狀態
  
Azure CDN 資源健康狀態是 [Azure 資源健康狀態](../resource-health/resource-health-overview.md)的子集。  您可以使用 Azure 資源健康狀態來監視 CDN 資源的健康狀態，以及接收可行的指引來進行問題的疑難排解。

>[!IMPORTANT] 
>Azure CDN 資源健康狀態目前只負責全域 CDN 傳遞和 API 功能的健康狀態。  Azure CDN 資源健康情況不會驗證個別的 CDN 端點。
>
>摘要 Azure CDN 資源健康狀態的訊號最多可能延遲 15 分鐘。

## <a name="how-to-find-azure-cdn-resource-health"></a>如何尋找 Azure CDN 資源健康狀態

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽到您的 CDN 設定檔。

2. 按一下 [設定]  按鈕。

    ![設定按鈕](./media/cdn-resource-health/cdn-profile-settings.png)

3. 在 [支援與疑難排解] 下方，按一下 [資源健康狀態]。

    ![CDN 資源健康狀態](./media/cdn-resource-health/cdn-resource-health3.png)

>[!TIP] 
>您也可以尋找 [說明 + 支援] 刀鋒視窗的 [資源健康狀態] 圖格中所列出的 CDN 資源。  您可以在入口網站的右上角，按一下圓框 **？** 快速取得 [說明 + 支援] 。
>
> ![說明 + 支援](./media/cdn-resource-health/cdn-help-support.png)

## <a name="azure-cdn-specific-messages"></a>Azure CDN 特定的訊息

以下提供與 Azure CDN 資源健康狀態相關的狀態。

|訊息 | 建議的動作 |
|---|---|
|您可能已停止、移除或錯誤設定一或多個 CDN 端點 | 您可能已停止、移除或錯誤設定一或多個 CDN 端點。|
|很抱歉，目前無法使用 CDN 管理服務 | 請返回這裡以查看狀態更新。如果您的問題在預期解決時間之後持續發生，請連絡支援人員。|
|很抱歉，您的 CDN 端點可能受到我們的部分 CDN 提供者持續出現之問題所影響 | 請返回這裡以查看狀態更新。使用疑難排解工具來了解如何測試您的原始和 CDN 端點。如果您的問題在預期解決時間之後持續發生，請連絡支援人員。 |
|很抱歉，CDN 端點設定變更發生了傳播延遲 | 請返回這裡以查看狀態更新。如果您的組態變更在預期時間內並未完全傳播，請連絡支援人員。|
|很抱歉，我們遇到了載入補充入口網站的問題 | 請返回這裡以查看狀態更新。如果您的問題在預期解決時間之後持續發生，請連絡支援人員。|
很抱歉，我們的部分 CDN 提供者發生了問題 | 請返回這裡以查看狀態更新。如果您的問題在預期解決時間之後持續發生，請連絡支援人員。 |

## <a name="next-steps"></a>後續步驟

- [閱讀 Azure 資源健康狀態的概觀](../resource-health/resource-health-overview.md)
- [針對 CDN 壓縮的問題進行疑難排解](./cdn-troubleshoot-compression.md)
- [針對 404 錯誤的問題進行疑難排解](./cdn-troubleshoot-endpoint.md)