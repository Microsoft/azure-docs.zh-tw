---
title: 使用 Visual Studio 來設定 Azure 雲端服務專案 | Microsoft Docs
description: 了解如何在 Visual Studio 中根據 Azure 雲端服務專案的需求來設定專案。
services: visual-studio-online
author: ghogen
manager: douge
assetId: 609d6965-05cc-47b1-82dc-c76a92d4f295
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/06/2017
ms.author: ghogen
ms.openlocfilehash: 0e2e03f7a492f5018b99145a63bb1b7432d3e0b3
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140072"
---
# <a name="configure-an-azure-cloud-service-project-with-visual-studio"></a>使用 Visual Studio 來設定 Azure 雲端服務專案
您可以根據 Azure 雲端服務專案的需求來設定專案。 您可以設定下列類別的專案的屬性：

- **將雲端服務發佈至 Azure** - 您可以設定屬性以確保部署至 Azure 的現有雲端服務不會被意外刪除。
- **在本機電腦上執行雲端服務或對其進行偵錯** - 您可以選取一個要使用的服務組態，並指示是否要啟動 Azure 儲存體模擬器。
- **在建立雲端服務套件時加以驗證** - 您可以決定將任何警告都視為錯誤，以便確保在部署雲端服務套件時不會發生任何問題。 

## <a name="steps-to-configure-an-azure-cloud-service-project"></a>設定 Azure 雲端服務專案的步驟
1. 在 Visual Studio 中開啟或建立雲端服務專案

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後從操作功能表中選取 [屬性]。
   
1. 在專案的屬性頁面中，選取 [開發] 索引標籤。

    ![專案屬性功能表](./media/vs-azure-tools-configuring-an-azure-project/solution-explorer-project-properties-menu.png)

1. 將 [刪除現有部署前先提示] 設定為 [True]。 這項設定可協助確保您不會意外刪除 Azure 中現有的部署

1. 選取想要的 [服務組態]，以指出您在本機執行雲端服務或對其進行偵錯時，所想要使用的服務組態。 如需有關如何修改某個角色之服務組態的詳細資訊，請參閱[如何使用 Visual Studio 來設定 Azure 雲端服務的角色](./vs-azure-tools-configure-roles-for-cloud-service.md)。

1. 將 [啟動 Azure 儲存體模擬器] 設定為 [True]，以在您於本機執行雲端服務或對其進行偵錯時，啟動 Azure 儲存體模擬器。

1. 將 [將警告視為錯誤] 設定為 [True]，以確保您在套件驗證發生錯誤時無法發佈。

1. 將 [使用 Web 專案連接埠] 設定為 [True]，以確保您的 Web 角色每次在 IIS Express 中於本機啟動時都使用相同的連接埠。

1. 從 Visual Studio 工具列中，選取 [儲存]。

## <a name="next-steps"></a>後續步驟
- [使用多個服務組態設定 Azure 專案](vs-azure-tools-multiple-services-project-configurations.md)

