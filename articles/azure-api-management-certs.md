---
title: 上傳 Azure 服務管理憑證 | Microsoft Docs
description: 了解如何針對 Azure 入口網站上傳服務管理憑證。
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 1b813833-39c8-46be-8666-fd0960cfbf04
ms.service: na
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/01/2017
ms.author: jeconnoc
ms.openlocfilehash: 788ce1f2543b9ed76d718cf01d28292f49594414
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47037349"
---
# <a name="upload-an-azure-service-management-certificate"></a>上傳 Azure 服務管理憑證
管理憑證可讓您使用 Azure 所提供的傳統部署模型進行驗證。 許多程式和工具 (例如 Visual Studio 或 Azure SDK) 會使用這些憑證，將各種 Azure 服務的設定與部署自動化。 

> [!WARNING]
> 請務必小心！ 這些憑證類型允許使用它們進行驗證的任何人管理與他們相關聯的訂用帳戶。
>
>

如果您想要深入了解 Azure 憑證 (包括如何建立自我簽署憑證)，請參閱 [Azure 雲端服務的憑證概觀](cloud-services/cloud-services-certs-create.md#what-are-management-certificates)。

您也可以使用 [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) 驗證用於自動化用途的用戶端程式碼。

**注意：** 您必須是訂用帳戶的共同管理員，才能在 [管理憑證] 下執行任何作業。 [進一步了解](https://go.microsoft.com/fwlink/?linkid=849300)如何從新版 Azure 入口網站新增或移除共同管理員 

## <a name="upload-a-management-certificate"></a>上傳管理憑證
一旦建立管理憑證 (僅包含公開金鑰的 .cer 檔案) 之後，您就可以將其上傳到入口網站。 入口網站提供憑證時，具有符合的憑證 (私密金鑰) 的任何人都可以透過管理 API 連線，並存取相關聯訂用帳戶的資源。

1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 按一下 Azure 服務清單底部的 [所有服務]，然後選取 [一般] 服務群組中的 [訂用帳戶]。

    ![[訂用帳戶] 功能表](./media/azure-api-management-certs/subscriptions_menu.png)

3. 務必選取您想要用來與憑證建立關聯的正確訂用帳戶。     
4. 當您選取正確的訂用帳戶之後，按下 [設定] 群組中的 [管理憑證]。

    ![設定](./media/azure-api-management-certs/mgmtcerts_menu.png)

5. 按 [上傳]  按鈕。

    ![憑證頁面的 [上傳]](./media/azure-api-management-certs/certificates_page.png)
6. 填寫對話方塊資訊，然後按下 [上傳]。

    ![設定](./media/azure-api-management-certs/certificate_details.png)

## <a name="next-steps"></a>後續步驟
既然您已經擁有與訂用帳戶建立關聯的管理憑證，您就能 (在本機安裝相符的憑證之後) 以程式設計方式連線到[傳統部署模型 REST API](https://msdn.microsoft.com/library/azure/mt420159.aspx)，並將同樣與該訂用帳戶建立關聯的各種 Azure 資源自動化。
