---
title: 使用 Azure Media Clipper 建立剪輯 | Microsoft Docs
description: Azure Media Clipper (用來從資產建置媒體剪輯的工具) 的概觀
services: media-services
keywords: clip;subclip;encoding;media;剪輯;子剪輯;編碼;媒體
author: dbgeorge
manager: jasonsue
ms.author: dwgeo
ms.date: 11/10/2017
ms.topic: article
ms.service: media-services
ms.openlocfilehash: f3822386d0d16b1feaf16853424329558a18f910
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33782647"
---
# <a name="create-clips-with-azure-media-clipper"></a>使用 Azure Media Clipper 建立剪輯
Azure Media Clipper 是免費的 JavaScript 程式庫，可讓 Web 開發人員為其使用者提供用於建立媒體剪輯的介面。 此工具可整合到任何網頁中，並提供用於載入資產和提交剪輯作業的 API。

Azure Media Clipper 可讓您：
- 從即時封存修剪前置靜態圖像和後置靜態圖像 
- 從 AMS 即時事件、即時封存或 fMP4 VOD 檔案編輯精華影片 
- 串連來自多個來源的影片 
- 從您的 AMS 媒體資產產生摘要剪輯 
- 精準地針對每個畫面剪輯影片 
- 以圖片群組 (GOP) 的精確度，針對現有的即時和 VOD 資產產生動態資訊清單篩選 
- 針對您媒體服務帳戶中的資產產生編碼作業

若要要求新功能，或是提供想法或意見反應，請提交至 [Azure 媒體服務的 UserVoice](http://aka.ms/amsvoice/) \(英文\)。 如果您有特定問題或疑問，或是有發現任何錯誤 (Bug)，請以電子郵件通知媒體服務小組 (amcinfo@microsoft.com)。

下圖顯示 Clipper 介面：![Azure Media Clipper](media/media-services-azure-media-clipper-overview/media-services-azure-media-clipper-interface.PNG)

## <a name="release-notes"></a>版本資訊
如需 Clipper 部落格文章、各種已知問題，以及 Clipper 最新版本的變更記錄，請參閱下列清單：
- [部落格文章](https://azure.microsoft.com/blog/azure-media-clipper/)
- [已知問題清單](https://amp.azure.net/libs/amc/latest/docs/known_issues.html) \(英文\)
- [變更記錄](https://amp.azure.net/libs/amc/latest/docs/changelog.html) \(英文\)

## <a name="browser-support"></a>瀏覽器支援
Azure Media Clipper 是使用新式 HTML5 技術所建置，並支援下列瀏覽器：

- Microsoft Edge 13+
- Internet Explorer 11+
- Chrome 54+
- Safari 10+
- Firefox 50+

> [!NOTE]
> 目前僅支援使用 HTML5 播放來自 Azure 媒體服務的資料流。

## <a name="language-support"></a>語言支援
Clipper 小工具有以下列 18 種語言提供：
- 中文 (簡體)
- 中文 (繁體)
- 捷克文
- 荷蘭文，佛蘭德文
- English
- 法文
- 德文
- 匈牙利文
- 義大利文
- 日文
- 韓文
- 波蘭文
- 葡萄牙文 (巴西)
- 葡萄牙文 (葡萄牙)
- 俄文
- 西班牙文
- 瑞典文
- 土耳其文

## <a name="next-steps"></a>後續步驟
若要開始使用 Azure Media Clipper，請閱讀[開始使用](media-services-azure-media-clipper-getting-started.md)文章以了解如何部署該小工具的詳細資料。
