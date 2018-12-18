---
title: Machine Learning Web 服務的記錄 | Microsoft Docs
description: 了解如何為 Machine Learning Web 服務啟用記錄。 記錄提供可協助疑難排解 API 的其他資訊。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: c54d41e1-0300-46ef-bbfc-d6f7dca85086
ms.service: machine-learning
ms.component: studio
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/15/2017
ms.openlocfilehash: 4e1545c8fd05795c683b24c029376a3d1e6d85b8
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835826"
---
# <a name="enable-logging-for-machine-learning-web-services"></a>為 Machine Learning Web 服務啟用記錄
本文件提供 Machine Learning Web 服務記錄功能的相關資訊。 記錄會提供其他資訊，而不只是錯誤碼和訊息而已，可協助您針對 Machine Learning API 的呼叫進行疑難排解。  

## <a name="how-to-enable-logging-for-a-web-service"></a>如何為 Web 服務啟用記錄

從 [Azure Machine Learning Web 服務](https://services.azureml.net)入口網站啟用記錄。 

1. 登入 Azure Machine Learning Web 服務入口網站，網址為 [https://services.azureml.net](https://services.azureml.net)。 對於傳統 Web 服務，您也可以在 Machine Learning Studio 中的 Machine Learning Web 服務頁面上按一下 [新的 Web 服務體驗]，以進入入口網站。

   ![新的 Web 服務體驗連結](./media/web-services-logging/new-web-services-experience-link.png)

2. 在頂端的功能表列上，按一下 [Web 服務] 以取得新的 Web 服務，或按一下 [傳統 Web 服務] 以取得傳統 Web 服務。

   ![選取新的或傳統 Web 服務](./media/web-services-logging/select-web-service.png)

3. 對於新的 Web 服務，按一下 Web 服務名稱。 對於傳統 Web 服務，按一下 Web 服務名稱，然後在下一個頁面中按一下適當的端點。

4. 按一下頂端功能表列上的 [設定]。

5. 將 **[啟用記錄]** 選項設定為 *[錯誤]* \(僅記錄錯誤) 或 *[所有]* \(適用於完整記錄)。

   ![選取記錄等級](./media/web-services-logging/enable-logging.png)

6. 按一下 [檔案] 。

7. 對於傳統 Web 服務，請建立 **ml-診斷**容器。

   所有的 Web 服務記錄會保存在與 Web 服務相關聯的儲存體帳戶中名為 **ml-診斷**的 blob 容器中。 對於新的 Web 服務，此容器會在您第一次存取 Web 服務時建立。 對於傳統 Web 服務，如果容器不存在，則您需要建立容器。 

   1. 在 [Azure 入口網站](https://portal.azure.com)中，移至與 Web 服務相關聯的儲存體帳戶。

   2. 在 [Blob 服務] 下，按一下 [容器]。

   3. 如果容器 **ml-診斷**不存在，按一下 [+ 容器]，將容器命名為「ml-診斷」，[存取型別] 選取「Blob」。 按一下 [確定]。

      ![選取記錄等級](./media/web-services-logging/create-ml-diagnostics-container.png)

> [!TIP]
>
> 對於傳統 Web 服務，Machine Learning Studio 中的 Web 服務儀表板也有啟用記錄的參數。 不過，由於記錄現在是透過 Web 服務入口網站所管理，您需要透過入口網站來啟用記錄，如本文所述。 如果您已在 Studio 中啟用記錄，請在 Web 服務入口網站中停用記錄，然後再次啟用。


## <a name="the-effects-of-enabling-logging"></a>啟用記錄的效果
記錄啟用時，診斷和錯誤都會從 Web 服務端點記錄到與使用者工作區連結的 Azure 儲存體帳戶之 **ml-診斷** blob 容器中。 這個容器針對所有與此儲存體帳戶相關聯的工作區，存放所有 Web 服務端點的診斷資訊。

記錄可使用任何可用於探索 Azure 儲存體帳戶的多種工具來檢視。 最簡單的方法就是瀏覽至 Azure 入口網站中的儲存體帳戶，按一下 [容器]，然後按一下 [ml-診斷] 容器。  

## <a name="log-blob-detail-information"></a>記錄檔 blob 詳細資訊
在容器中的每個 blob，只會存放下列其中一項動作的診斷資訊：

* 批次執行方法的執行  
* 要求-回應方法的執行  
* 要求-回應容器的初始化

每個 blob 的名稱具有下列形式的前置詞︰ 


`{Workspace Id}-{Web service Id}-{Endpoint Id}/{Log type}`


其中的_記錄型別_是下列其中一個值：  

* 批次  
* 分數/要求  
* 分數/初始  

