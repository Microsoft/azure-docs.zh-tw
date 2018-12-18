---
title: 新增自訂 CA 憑證 - Azure API 管理 | Microsoft Docs
description: 了解如何在 Azure API 管理中新增自訂 CA 憑證。
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/20/2018
ms.author: apimpm
ms.openlocfilehash: 9d3399ba6ee724d91117486744ad1431f53edbce
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43053230"
---
# <a name="how-to-add-a-custom-ca-certificate-in-azure-api-management"></a>如何在 Azure API 管理中新增自訂 CA 憑證

Azure API 管理可允許在信任根存放區和中繼憑證存放區內的機器上，安裝 CA 憑證。 如果您的服務需要自訂 CA 憑證，則應該使用這項功能。

本文說明如何在 Azure 入口網站中，管理 Azure API 管理服務執行個體的 CA 憑證。

## <a name="step1"> </a>上傳 CA 憑證

![新增 CA 憑證](media/api-management-howto-ca-certificates/00.png)

請依照下列步驟來上傳新的 CA 憑證。 如果您尚未建立 API 管理服務執行個體，請參閱[建立 API 管理服務執行個體](get-started-create-service-instance.md)教學課程。

1. 在 Azure 入口網站中瀏覽至您的 Azure API 管理服務執行個體。

2. 從功能表中選取 [CA 憑證]。

3. 按一下 [+新增] 按鈕。  

    ![新增 CA 憑證](media/api-management-howto-ca-certificates/01.png)  

4. 瀏覽憑證並決定憑證存放區。 僅需要公開金鑰，因此不需要密碼。

    ![新增 CA 憑證](media/api-management-howto-ca-certificates/02.png)  

5. 按一下 [檔案] 。 這項作業可能需要幾分鐘的時間。

    ![新增 CA 憑證](media/api-management-howto-ca-certificates/03.png)  

> [!NOTE]
> 您可以使用 `New-AzureRmApiManagementSystemCertificate` Powershell 命令來上傳 CA 憑證。

## <a name="step1a"> </a>刪除用戶端憑證

若要刪除憑證，請按一下內容功能表的 **...**，然後選取憑證旁的 [刪除]。

![刪除 CA 憑證](media/api-management-howto-ca-certificates/04.png)  

[Upload a CA certificate]: #step1
[Delete a CA certificate]: #step1a