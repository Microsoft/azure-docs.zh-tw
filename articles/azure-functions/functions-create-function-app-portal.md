---
title: 從 Azure 入口網站建立函數應用程式 | Microsoft Docs
description: 從入口網站，在 Azure App Service 中建立新的函數應用程式。
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
editor: ''
tags: ''
ms.assetid: ''
ms.service: functions
ms.devlang: multiple
ms.topic: quickstart
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 04/11/2017
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: 4716d29a7b8c0830e2311398f033f8358ef24529
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38467640"
---
# <a name="create-a-function-app-from-the-azure-portal"></a>從 Azure 入口網站建立函數應用程式

Azure 函數應用程式使用 Azure App Service 基礎結構。 本主題說明如何在 Azure 入口網站中建立函數應用程式。 函數應用程式是裝載個別函式執行的容器。 當您在 App Service 主控方案中建立函數應用程式時，您的函數應用程式可以使用 App Service 的所有功能。

## <a name="create-a-function-app"></a>建立函數應用程式

[!INCLUDE [functions-create-function-app-portal](../../includes/functions-create-function-app-portal.md)]

當您建立函數應用程式時，請提供有效的**應用程式名稱**，其中只能包含字母、數字和連字號。 不允許使用底線 (**_**) 字元。

儲存體帳戶名稱必須介於 3 到 24 個字元的長度，而且只能包含數字和小寫字母。 儲存體帳戶名稱必須在 Azure 中是獨一無二的。 

函數應用程式建立之後，您可以使用一或多個不同的語言建立個別的函式。 使用[入口網站](functions-create-first-azure-function.md#create-function)、[連續部署](functions-continuous-deployment.md)或者[透過 FTP 上傳](https://github.com/projectkudu/kudu/wiki/Accessing-files-via-ftp)來建立函式。

## <a name="service-plans"></a>服務方案

Azure Functions 有兩個不同的服務方案︰取用方案和 App Service 方案。 取用方案會在您的程式碼執行時自動配置計算能力、視需要相應放大來處理負載，然後在程式碼未執行時相應縮小。 App Service 方案可讓您的函數應用程式存取 App Service 的所有功能。 建立您的函數應用程式時，您必須選擇您的服務方案，且目前無法變更。 如需詳細資訊，請參閱[選擇 Azure Functions 主控方案](functions-scale.md)。

若計畫在 App Service 方案上執行 JavaScript 函式，您應該選擇核心數目較少的方案。 如需詳細資訊，請參閱 [JavaScript 函式參考資料](functions-reference-node.md#choose-single-vcpu-app-service-plans)。

<a name="storage-account-requirements"></a>

## <a name="storage-account-requirements"></a>儲存體帳戶需求

在 App Service 中建立函數應用程式時，您必須建立或連結支援 Blob、「佇列」及「表格」儲存體的一般用途「Azure 儲存體」帳戶。 Azure Functions 會在內部使用「Azure 儲存體」來進行作業，例如管理觸發程序和記錄函式執行。 有些儲存體帳戶並不支援佇列和表格，例如僅限 Blob 的儲存體帳戶、Azure 進階儲存體和搭配 ZRS 複寫的一般用途儲存體帳戶。 建立函數應用程式時，[儲存體帳戶] 刀鋒視窗中會過濾掉這些帳戶。

>[!NOTE]
>當使用「使用情況主控方案」時，您的函式程式碼和繫結組態檔會儲存在主要儲存體帳戶中的 Azure 檔案儲存體中。 當您刪除主要儲存體帳戶時，會刪除此內容且無法復原。

若要深入了解儲存體帳戶類型，請參閱 [Azure 儲存體服務簡介](../storage/common/storage-introduction.md#azure-storage-services)。 

## <a name="next-steps"></a>後續步驟

[!INCLUDE [Functions quickstart next steps](../../includes/functions-quickstart-next-steps.md)]



