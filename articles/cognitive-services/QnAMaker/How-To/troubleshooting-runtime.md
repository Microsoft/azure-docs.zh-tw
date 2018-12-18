---
title: 疑難排解 - QnAMaker
titlesuffix: Azure Cognitive Services
description: QnAMaker 由使用者 Azure 帳戶中裝載的元件所組成。 在進行偵錯時，使用者可能需要操作其 QnAMaker Azure 資源，或將關於其設定的其他資訊提供給 QnAMaker 支援小組。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: ad3c01ebf37c8b544830b281144090694eeadfcd
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033717"
---
# <a name="qnamaker-troubleshooting"></a>QnAMaker 疑難排解
QnAMaker 由使用者 Azure 帳戶中裝載的元件所組成。 在進行偵錯時，使用者可能需要操作其 QnAMaker Azure 資源，或將關於其設定的其他資訊提供給 QnAMaker 支援小組。

## <a name="how-to-get-latest-qnamaker-runtime-updates"></a>如何取得最新的 QnAMaker 執行階段更新
QnAMaker 執行階段是您在 Azure 入口網站中[建立 QnAMaker 服務](./set-up-qnamaker-service-azure.md)時所部署的 Azure App Service 組件之一。 執行階段會定期進行更新。 若要將最新的更新套用至您的 QnAMaker 設定，您必須重新啟動 App Service。
1. 在 [Azure 入口網站](https://portal.azure.com)中移至 QnAMaker 服務 (資源群組)

    ![QnAMaker Azure 資源群組](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-resourcegroup.png)

2. 按一下 [App Service]，然後開啟 [概觀] 區段

     ![QnAMaker App Service](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-appservice.png)

3. 重新啟動 App Service。 此作業應會在數秒內完成。 請注意，在此重新啟動期間，使用者將無法使用您在此 QnAMaker 服務上建置的下游應用程式/聊天機器人。

    ![QnAMaker AppService 重新啟動](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-restart.png)

## <a name="how-to-get-the-qnamaker-service-hostname"></a>如何取得 QnAMaker 服務主機名稱
當您連絡 QnAMaker 支援人員或 UserVoice 時，QnAMaker 服務主機名稱將有助於偵錯的用途。 主機名稱是下列格式的 URL： https://*{hostname}*.azurewebsites.net.
    
1. 在 [Azure 入口網站](https://portal.azure.com)中移至 QnAMaker 服務 (資源群組)

    ![QnAMaker Azure 資源群組](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-resourcegroup.png)

2. 按一下 App Service

     ![QnAMaker App Service](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-appservice.png)

3. 主機名稱 URL 會顯示於 [概觀] 區段中

    ![QnAMaker 主機名稱](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-gethostname.png)
    

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 QnAMaker API](./upgrade-qnamaker-service.md)
