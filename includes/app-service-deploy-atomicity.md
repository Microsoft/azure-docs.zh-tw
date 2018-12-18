---
title: 包含檔案
description: 包含檔案
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/08/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: ac2cf4d688b1bdc54ed2d7341f0e195d3b2fe42d
ms.sourcegitcommit: 4e36ef0edff463c1edc51bce7832e75760248f82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35236462"
---
## <a name="what-happens-to-my-app-during-deployment"></a>在部署期間我的應用程式會發生什麼事？

所有官方支援的部署方法都有一個共通處：會對應用程式 `/site/home/wwwroot` 資料夾中的檔案進行變更。 這些檔案是在生產環境中執行的相同檔案。 因此，部署可能會因為檔案鎖定而失敗，或者生產環境中的應用程式可能會因為並非所有檔案同時更新，所以在部署期間發生非預期的行為。 有幾種不同的方式可以避免這些問題：

- 停止您的應用程式，或在部署期間針對您的應用程式啟用離線模式。 如需詳細資訊，請參閱[在部署期間處理鎖定的檔案](https://github.com/projectkudu/kudu/wiki/Dealing-with-locked-files-during-deployment)。
- 部署到[預備位置](../articles/app-service/web-sites-staged-publishing.md)，並且啟用[自動交換](../articles/app-service/web-sites-staged-publishing.md#configure-auto-swap)。 
- 改為使用 [Run-from-Zip](https://github.com/Azure/app-service-announcements/issues/84)。
