---
title: 使用 Azure Machine Learning Web 服務參數 | Microsoft Docs
description: 如何使用 Azure Machine Learning Web 服務參數來修改模型在 Web 服務受到存取時的行為。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: c49187db-b976-4731-89d6-11a0bf653db1
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/12/2017
ms.openlocfilehash: 91b3c9df8a7fd0e1abb79c21b1e1d833e57c24d5
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835921"
---
# <a name="use-azure-machine-learning-web-service-parameters"></a>使用 Azure Machine Learning Web 服務參數
藉由發行包含可設定參數模組的試驗，來建立 Azure Machine Learning Web 服務。 在某些情況下，您可能想要在執行 Web 服務時之際，變更模組的行為。 「Web 服務參數」可讓您執行這項工作。 

常見的範例是設定[匯入資料][reader]模組，讓已發佈之 Web 服務的使用者可以在存取 Web 服務時，指定不同的資料來源。 或者，設定[匯出資料][writer]模組，以便能夠指定不同的目的地。 部分其他範例包括變更[特徵雜湊][feature-hashing]模組的位元數，或變更[以篩選器為基礎的特徵選取][filter-based-feature-selection]模組的所需特徵數。 

您可以設定 Web 服務參數，並使其與實驗中的一個或多個模組參數產生關聯，而且您可以指定它們是必要還是選用參數。 然後 Web 服務的使用者就可以在呼叫 Web 服務時，提供這些參數的值。 

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="how-to-set-and-use-web-service-parameters"></a>如何設定及使用 Web 服務參數
您可以按一下模組參數旁邊的圖示，然後選取 [設為 Web 服務參數] 來定義 Web 服務參數。 這會建立新的 Web 服務參數並將它連線至該模組參數。 然後，在 Web 服務受到存取時，使用者可以指定 Web 服務參數的值，該值就會套用至模組參數。

在您定義 Web 服務參數之後，該參數就可以供試驗中任何其他的模組參數使用。 如果您定義和某個模組參數相關聯的 Web 服務參數，那麼只要參數預期具有相同類型的值，您就可以將該 Web 服務參數用於任何其他模組。 例如，如果 Web 服務參數是數值，那麼該參數就只能用於預期值為數值的模組參數。 當使用者設定 Web 服務參數的值時，該值將會套用至所有相關聯的模組參數。

您可以決定是否要為 Web 服務參數提供預設值。 如果您提供預設值，Web 服務的使用者就可以選擇是否使用該參數。 如果您沒有提供預設值，使用者就需要在存取 Web 服務時輸入值。

Web 服務的 API 文件會包含 Web 服務使用者在存取 Web 服務時，如何以程式設計方式指定 Web 服務參數的資訊。

> [!NOTE]
> 傳統 Web 服務的 API 文件是透過 Machine Learning Studio 中 Web 服務 [儀表板] 的 [API 說明頁面] 連結提供。 新的 Web 服務的 API 文件則是透過 [Azure Machine Learning Web Services](https://services.azureml.net/Quickstart) 入口網站中 Web 服務的 [取用] 和 [Swagger API] 頁面提供。
> 
> 

## <a name="example"></a>範例
舉例來說，假設我們有一個[匯出資料][writer]模組實驗，該模組會傳送資訊給 Azure Blob 儲存體。 我們將會定義名為 Blob path 的 Web 服務參數，以在服務被存取時允許 Web 服務使用者將路徑變更至 Blob 儲存體。

1. 在 Machine Learning Studio 中，按一下 [[匯出資料][writer]] 模組來選取它。 其屬性會顯示在試驗畫布右邊的 [屬性] 窗格中。
2. 指定儲存體類型：
   
   * 在 **[請指定資料目的地]** 底下，選取 [Azure Blob 儲存體]。
   * 在 **[請指定驗證類型]** 底下，選取 [帳戶]。
   * 輸入 Azure Blob 儲存體的帳戶資訊。 

3. 按一下 **[以容器參數為開頭的 Blob 路徑]** 右邊的圖示。 它看起來像這樣：
   
   ![Web 服務參數圖示][icon]
   
   選取 [設為 Web 服務參數]。
   
   就會在 **[屬性] 窗格底部的 [Web 服務參數]** 底下新增一個名為 [以容器為開頭的 Blob 路徑] 項目。 這就是現在與此[匯出資料][writer]模組參數關聯的「Web 服務參數」。
4. 若要重新命名 Web 服務參數，請按一下名稱，輸入 Blob path，然後按 **Enter** 鍵。 
5. 若要為 Web 服務參數提供預設值，請按一下名稱右邊的圖示，選取 [提供預設值]，輸入值 (例如 container1/output1.csv)，然後按 **Enter** 鍵。
   
   ![Web 服務參數][parameter]
6. 按一下 **[執行]**。 
7. 按一下 [部署 Web 服務]，然後選取 [部署 Web 服務 [傳統]] 或 [部署 Web 服務 [新]] 可部署 Web 服務。

> [!NOTE] 
> 若要部署新的 Web 服務，您必須在要部署 Web 服務的訂用帳戶中具備足夠的權限。 如需詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。 

Web 服務的使用者現在即可在存取 Web 服務時，為[匯出資料][writer]模組指定新的目的地。

## <a name="more-information"></a>詳細資訊
如需更詳細的範例，請參閱 [Machine Learning Blog](http://blogs.technet.com/b/machinelearning/archive/2014/11/25/azureml-web-service-parameters.aspx) 中的 [Web 服務參數](http://blogs.technet.com/b/machinelearning/archive/2014/11/25/azureml-web-service-parameters.aspx)項目。

有關存取 Machine Learning Web 服務的詳細資訊，請參閱[如何使用 Azure Machine Learning Web 服務](consume-web-services.md)。

<!-- Images -->
[icon]: ./media/web-service-parameters/icon.png
[parameter]: ./media/web-service-parameters/parameter.png


<!-- Module References -->
[feature-hashing]: https://msdn.microsoft.com/library/azure/c9a82660-2d9c-411d-8122-4d9e0b3ce92a/
[filter-based-feature-selection]: https://msdn.microsoft.com/library/azure/918b356b-045c-412b-aa12-94a1d2dad90f/
[reader]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
[writer]: https://msdn.microsoft.com/library/azure/7a391181-b6a7-4ad4-b82d-e419c0d6522c/

