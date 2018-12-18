---
title: 疑難排解：建立及連接至機器學習工作區 | Microsoft Docs
description: 建立及連接 Azure Machine Learning 工作區之常見問題的解決方法
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 1a8aec4b-35f9-44e8-9570-2575b8979ab1
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2017
ms.openlocfilehash: 262c9af4e0f3ee34dc89986affacb6c0d8a0d801
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835717"
---
# <a name="troubleshooting-guide-create-and-connect-to-an-machine-learning-workspace"></a>疑難排解指南：建立及連接至機器學習服務工作區
本指南針對一些在設定 Azure Machine Learning 工作區時常發生的問題，提供解決方案。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="workspace-owner"></a>工作區擁有者
若要在 Machine Learning Studio 中開啟工作區，您必須使用建立工作區的 Microsoft 帳戶登入，或需要收到來自擁有者的邀請，才能加入工作區。 您可以從 Azure 入口網站管理工作區，其中包括設定存取的能力。

如需管理工作區的詳細資訊，請參閱 [管理 Azure Machine Learning 工作區]。

[管理 Azure Machine Learning 工作區]: manage-workspace.md

## <a name="allowed-regions"></a>允許區域
機器學習服務目前可用於數量有限的區域。 如果您的訂用帳戶並未包含這其中一個區域，您可能會看見錯誤訊息「您在允許的區域中沒有任何訂用帳戶」。

為了要求將區域新增至您的訂用帳戶，請從 Azure 入口網站中建立新的 Microsoft 支援要求，選取 [計費] 做為問題類型，並遵照提示來提交您的要求。

## <a name="storage-account"></a>儲存體帳戶
機器學習服務需要儲存體帳戶來儲存資料。 您可以使用現有的儲存體帳戶，或是在建立新的機器學習服務工作區時建立新的儲存體帳戶 (如果您有建立新儲存體帳戶的配額)。

新的 Machine Learning 工作區建立完成後，您可以使用建立工作區的 Microsoft 帳戶登入 Machine Learning Studio。 如果您遇到錯誤訊息「找不到工作區」(類似於下列螢幕擷取畫面)，請使用下列步驟來刪除您的瀏覽器 Cookie。

![找不到工作區][screen3]

**刪除瀏覽器 Cookie**

1. 如果您是使用 Internet Explorer，請按一下右上角的 [工具] 按鈕，然後選取 [網際網路選項]。  

![網際網路選項][screen4]

2. 在 [一般] 索引標籤下，按一下 [刪除...]

![[一般] 索引標籤][screen5]

3. 在 [刪除瀏覽歷程記錄] 對話方塊中，確定已選取 [Cookie 與網站資料]，然後按一下 [刪除]。

![刪除 cookie][screen6]

刪除 Cookie 之後，請重新啟動瀏覽器，然後前往 [Microsoft Azure Machine Learning](https://studio.azureml.net) 頁面。 出現輸入使用者名稱和密碼的提示時，請輸入建立工作區所使用的同一個 Microsoft 帳戶。

## <a name="comments"></a>註解

我們的目標是讓機器學習服務經驗盡可能完美。 請將任何建議或問題貼在 [Azure Machine Learning 論壇](http://social.msdn.microsoft.com/Forums/windowsazure/home?forum=MachineLearning) 中，以協助我們為您提供更好的服務。

[screen1]:media/troubleshooting-creating-ml-workspace/screen1.png
[screen2]:media/troubleshooting-creating-ml-workspace/screen2.png
[screen3]:media/troubleshooting-creating-ml-workspace/screen3.png
[screen4]:media/troubleshooting-creating-ml-workspace/screen4.png
[screen5]:media/troubleshooting-creating-ml-workspace/screen5.png
[screen6]:media/troubleshooting-creating-ml-workspace/screen6.png
