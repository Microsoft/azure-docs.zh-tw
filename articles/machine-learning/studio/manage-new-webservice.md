---
title: 使用 Azure Machine Learning Web 服務入口網站 | Microsoft Docs
description: 管理 Azure 機器學習工作區的存取權，並部署和管理 ML API Web 服務
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: b62cf2ca-dd2a-4a83-bb54-469f948fb026
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/28/2017
ms.openlocfilehash: 4e1444f385859044b2dadd83c8e9841d8fc78bb0
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47096406"
---
# <a name="manage-a-web-service-using-the-azure-machine-learning-web-services-portal"></a>使用 Azure Machine Learning Web 服務入口網站管理 Web 服務
您可以使用 Microsoft Azure Machine Learning Web 服務入口網站，管理 Machine Learning 新式和傳統 Web 服務。 因為傳統 Web 服務和新式 Web 服務是根據不同的基礎技術，所以各有稍微不同的管理功能。

在 Machine Learning Web 服務入口網站中，您可以︰

* 監視 Web 服務的使用方式。
* 設定描述、更新 Web 服務的金鑰 (僅限新式)、更新您的儲存體帳戶金鑰 (僅限新式)、啟用記錄，以及啟用或停用範例資料。
* 刪除 Web 服務。
* 建立、刪除或更新計費方案 (僅限新式)。
* 新增和刪除端點 (僅限傳統)

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="permissions-to-manage-new-resources-manager-based-web-services"></a>管理以資源管理員為基礎的新 Web 服務的權限

新的 Web 服務會部署為 Azure 資源。 因此，您必須具備正確權限，才能部署和管理新的 Web 服務。  若要部署或管理新的 Web 服務，您必須獲得下列角色的指派：要部署 Web 服務之訂用帳戶上的參與者或管理員角色。 如果您邀請另一位使用者到 Machine Learning 工作區，就必須為他們指派訂用帳戶上的參與者或管理員角色，然後他們才能部署或管理 Web 服務。 

如果使用者沒有正確權限來存取 Azure Machine Learning Web 服務入口網站中的資源，他們將會在嘗試部署 Web 服務時，收到下列錯誤：

*Web 服務部署工作失敗。此帳戶沒有足夠權限來存取包含該工作區的 Azure 訂用帳戶。若要將 Web 服務部署到 Azure，必須邀請同一個帳戶到該工作區，並為該帳戶授予包含該工作區之 Azure 訂用帳戶的存取權。*

如需建立工作區的詳細資訊，請參閱[建立和共用 Azure Machine Learning 工作區](create-workspace.md)。

如需如何設定存取權限的詳細資訊，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../../role-based-access-control/role-assignments-portal.md)。


## <a name="manage-new-web-services"></a>管理新式 Web 服務
管理新式 Web 服務：

1. 使用您的 Microsoft Azure 帳戶登入 [Microsoft Azure Machine Learning Web 服務入口網站](https://services.azureml.net/quickstart) - 使用與 Azure 訂用帳戶相關聯的帳戶。
2. 在功能表上，按一下 [Web 服務] 。

這會針對您的訂用帳戶顯示一份已部署的 Web 服務清單。 

若要管理 Web 服務，請按一下 [Web 服務]。 您可以從 [Web 服務] 頁面上︰

* 按一下 Web 服務進行管理。
* 按一下 Web 服務的 [計費方案] 進行更新。
* 刪除 Web 服務。
* 複製 Web 服務並將它部署到另一個區域。

當您按一下某個 Web 服務時，Web 服務的 [快速入門] 頁面就會開啟。 Web 服務 [快速入門] 頁面有兩個功能表選項可讓您管理您的 Web 服務︰

* **儀表板** -可讓您檢視 Web 服務使用量。
* **設定** -可讓您新增描述性文字、更新與 Web 服務相關聯的儲存體帳戶金鑰，以及啟用和停用範例資料。

### <a name="monitoring-how-the-web-service-is-being-used"></a>監視 Web 服務的使用方式
按一下 [ **儀表板** ] 索引標籤。

您可以從儀表板中檢視您的 Web 服務經過一段時間的整體使用量。 您可以從使用量圖表右上角的 [期間] 下拉式功能表中選取要檢視的期間。 儀表板會顯示下列資訊：

* **一段時間的要求數** 顯示選取的一段時間內，要求數目的步階圖形。 它可以協助識別您所遇到的使用量尖峰。
* **要求-回應要求數** 顯示服務在選取的一段時間內收到的要求-回應呼叫總數，以及失敗的數目。
* **平均要求-回應計算時間** 顯示執行收到的要求所需的平均時間。
* **批次要求數** 顯示服務在選取的一段時間內收到的批次要求總數，以及失敗的數目。
* **平均作業延遲** 顯示執行收到的要求所需的平均時間。
* **錯誤數**顯示呼叫 Web 服務時所發生的錯誤彙總數目。
* **服務成本** 顯示與服務相關聯的計費方案費用。

### <a name="configuring-the-web-service"></a>設定 Web 服務
按一下 [設定]  功能表選項。

您可以更新下列屬性：

* [描述] 可讓您輸入 Web 服務的描述。
* **標題**可讓您輸入 Web 服務的標題。
* **金鑰** 可讓您交換您的主要和次要 API 金鑰。
* **儲存體帳戶金鑰**可讓您為與 Web 服務變更相關聯的儲存體帳戶更新金鑰。 
* [啟用範例資料] 可讓您提供範例資料，用來測試要求-回應服務。 如果您是在 Machine Learning Studio 中建立 Web 服務，範例資料會取自您用來訓練模型的資料。 如果您是以程式設計方式建立服務，資料會取自您提供做為 JSON 套件一部分的範例資料。

### <a name="managing-billing-plans"></a>管理計費方案
從 Web 服務 [快速入門] 頁面按一下 [方案]  功能表選項。 您也可以按一下與特定 Web 服務相關聯的方案來管理該方案。

* **新增** 可讓您建立新的方案。
* **新增/移除方案執行個體** 可讓您「相應放大」現有的方案以增加容量。
* **升級/降級** 可讓您「相應增加」現有的方案以增加容量。
* **刪除** 可讓您刪除方案。

按一下方案可檢視其儀表板。 儀表板可提供所選一段時間的快照或方案使用量。 若要選取時間期間來檢視，請按一下儀表板右上角的 [期間] 下拉式清單。 

方案儀表板會提供下列資訊：

* **方案描述** 顯示成本相關資訊和與方案相關聯的容量。
* **方案使用量** 顯示交易數目和已依方案計費的計算時數。
* **Web 服務數**顯示使用此方案的 Web 服務數目。
* **依呼叫數的前幾名 Web 服務**顯示依方案計費進行呼叫的前四個 Web 服務。
* **依計算時數的前幾名 Web 服務**顯示依方案計費使用計算資源的前四個 Web 服務。

## <a name="manage-classic-web-services"></a>管理傳統 Web 服務
> [!NOTE]
> 本節的程序是關於透過 Azure Machine Learning Web 服務入口網站來管理傳統 Web 服務。 如需透過 Machine Learning Studio 和 Azure 入口網站管理傳統 Web 服務的相關資訊，請參閱[管理 Azure Machine Learning 工作區](manage-workspace.md)。
> 
> 

管理傳統 Web 服務：

1. 使用您的 Microsoft Azure 帳戶登入 [Microsoft Azure Machine Learning Web 服務入口網站](https://services.azureml.net/quickstart) - 使用與 Azure 訂用帳戶相關聯的帳戶。
2. 在功能表上，按一下 [傳統 Web 服務]。

若要管理傳統 Web 服務，請按一下 [傳統 Web 服務]。 您可以從 [傳統 Web 服務] 頁面上︰

* 按一下 Web 服務以檢視相關聯的端點。
* 刪除 Web 服務。

管理傳統 Web 服務時是個別管理每個端點。 當您按一下 [Web 服務] 頁面中的 Web 服務時，將會開啟與服務相關聯的端點清單。 

在 [傳統 Web 服務端點] 頁面上，您可以新增和刪除服務的端點。 如需有關新增端點的詳細資訊，請參閱 [建立端點](create-endpoint.md)。

按一下其中一個端點，以開啟 Web 服務 [快速入門] 頁面。 [快速入門] 頁面有兩個功能表選項可讓您管理 Web 服務︰

* **儀表板** -可讓您檢視 Web 服務使用量。
* **設定** -可讓您新增描述性文字、開啟和關閉錯誤記錄、更新與 Web 服務相關聯的儲存體帳戶金鑰，以及啟用和停用範例資料。

### <a name="monitoring-how-the-web-service-is-being-used"></a>監視 Web 服務的使用方式
按一下 [ **儀表板** ] 索引標籤。

您可以從儀表板中檢視您的 Web 服務經過一段時間的整體使用量。 您可以從使用量圖表右上角的 [期間] 下拉式功能表中選取要檢視的期間。 儀表板會顯示下列資訊：

* **一段時間的要求數** 顯示選取的一段時間內，要求數目的步階圖形。 它可以協助識別您所遇到的使用量尖峰。
* **要求-回應要求數** 顯示服務在選取的一段時間內收到的要求-回應呼叫總數，以及失敗的數目。
* **平均要求-回應計算時間** 顯示執行收到的要求所需的平均時間。
* **批次要求數** 顯示服務在選取的一段時間內收到的批次要求總數，以及失敗的數目。
* **平均作業延遲** 顯示執行收到的要求所需的平均時間。
* **錯誤數**顯示呼叫 Web 服務時所發生的錯誤彙總數目。
* **服務成本** 顯示與服務相關聯的計費方案費用。

### <a name="configuring-the-web-service"></a>設定 Web 服務
按一下 [設定]  功能表選項。

您可以更新下列屬性：

* [描述] 可讓您輸入 Web 服務的描述。 [描述] 必要欄位。
* [記錄] 可讓您啟用或停用端點上的錯誤記錄。 如需有關記錄的詳細資訊，請參閱[為 Machine Learning Web 服務啟用記錄](web-services-logging.md)。
* [啟用範例資料] 可讓您提供範例資料，用來測試要求-回應服務。 如果您是在 Machine Learning Studio 中建立 Web 服務，範例資料會取自您用來訓練模型的資料。 如果您是以程式設計方式建立服務，資料會取自您提供做為 JSON 套件一部分的範例資料。


