---
title: 我的 ASP.NET 專案發生什麼情形？ | Microsoft Docs
description: 說明使用 Visual Studio 已連接服務將 Azure 儲存體新增至 ASP.NET 專案後，會發生什麼事
services: storage
author: ghogen
manager: douge
ms.assetid: e1fe1b6d-4e3d-476d-8b2f-f7ade050515e
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ms.openlocfilehash: a6e05d706d54d63695861b03cd9de0e65ebdd8bb
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141303"
---
# <a name="what-happened-to-my-aspnet-project-visual-studio-azure-storage-connected-service"></a>我的 ASP.NET 專案 (Visual Studio Azure 儲存體連接的服務) 發生什麼狀況？
## <a name="references-added"></a>加入參考
Azure 儲存體 NuGet 封裝已加入至 Visual Studio 專案。  
這個封裝會加入下列 .NET 參考：

* **Microsoft.Data.Edm**
* **Microsoft.Data.OData**
* **Microsoft.Data.Services.Client**
* **Microsoft.WindowsAzure.Configuration**
* **Microsoft.WindowsAzure.Storage**
* **Newtonsoft.Json**
* **System.Data**
* **System.Spatial**

## <a name="connection-string-for-azure-storage-added"></a>加入 Azure 儲存體的連接字串
在專案的 web.config 檔案中，已使用所選儲存體帳戶的連接字串和金鑰建立一個元素。

如需詳細資訊，請參閱 [ASP.NET](http://www.asp.net)。

