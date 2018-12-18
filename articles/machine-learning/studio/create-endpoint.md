---
title: 在 Machine Learning 中建立 Web 服務端點 | Microsoft Docs
description: 在 Azure Machine Learning 中建立 Web 服務端點
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 4657fc1b-5228-4950-a29e-bc709259f728
ms.service: machine-learning
ms.component: studio
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 10/04/2016
ms.openlocfilehash: 8cdf8c5ac3676d8abc9084fc842484aca5b6d1c7
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34833541"
---
# <a name="creating-endpoints"></a>建立端點
> [!NOTE]
>  此主題描述適用於**傳統** Machine Learning Web 服務的技巧。
> 
> 

當您建立會進而銷售給客戶的 Web 服務時，必須為每位客戶提供仍連結至建立 Web 服務之實驗的定型模型。 此外，實驗的任何更新都應可以選擇性地套用到整個端點，而不會覆寫自訂項目。

為了達到這個目的，Azure Machine Learning 允許您為已部署的 Web 服務建立多個端點。 Web 服務的每個端點都是個別定址、節流以及管理。 每個端點是一個唯一 URL 和授權金鑰，您可散發給您的客戶。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="adding-endpoints-to-a-web-service"></a>將端點新增至 Web 服務
有兩種方式可在 Web 服務中新增端點。

* 以程式設計方式
* 透過 Azure Machine Learning Web 服務入口網站

建立端點之後，您就可以透過同步 API、批次 API 以及 Excel 工作表來取用該端點。 除了透過此 UI 新增端點之外，您也可以使用端點管理 API，以程式設計方式加入端點。

> [!NOTE]
> 如果您已在 Web 服務中新增額外的端點，就無法刪除預設端點。
> 
> 

## <a name="adding-an-endpoint-programmatically"></a>以程式設計方式新增端點
您可以使用 [AddEndpoint](https://github.com/raymondlaghaeian/AML_EndpointMgmt/blob/master/Program.cs) 範例程式碼，以程式設計方式將端點新增至您的 Web 服務。

## <a name="adding-an-endpoint-using-the-azure-machine-learning-web-services-portal"></a>使用 Azure Machine Learning Web 服務入口網站新增端點
1. 在 Machine Learning Studio 中，按一下左側的 [Web 服務]。
2. 在 Web 服務儀表板底部，按一下 [管理端點]。 Azure Machine Learning Web 服務 入口網站會開啟 Web 服務的端點頁面。
3. 按一下 [新增] 。
4. 輸入新端點的名稱和描述。 端點名稱長度不可超過 24 個字元，而且必須由小寫字母或數字組成。 選取記錄層級，以及是否啟用範例資料。 如需有關記錄的詳細資訊，請參閱[為 Machine Learning Web 服務啟用記錄](web-services-logging.md)。

## <a name="next-steps"></a>後續步驟
[如何使用 Azure Machine Learning Web 服務](consume-web-services.md)。

