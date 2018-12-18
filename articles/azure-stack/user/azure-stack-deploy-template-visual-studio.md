---
title: 使用 Visual Studio 部署 Azure Stack 中的範本 | Microsoft Docs
description: 了解如何以 Visual Studio 部署 Azure Stack 中的範本。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 628da2ae-64cc-42e0-b8b7-a6a3724cb974
ms.service: azure-stack
ms.custom: vs-azure
ms.workload: azure-vs
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/18/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: 1d61313018b0cd16594772676172737a7d8fc2cd
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46366688"
---
# <a name="deploy-templates-in-azure-stack-using-visual-studio"></a>使用 Visual Studio 部署 Azure Stack 中的範本

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用 Visual Studio 將 Azure Resource Manager 範本部署到 Azure Stack。

## <a name="to-deploy-a-template"></a>部署範本

1. 使用 Visual Studio [安裝並連接](azure-stack-install-visual-studio.md)至 Azure Stack。
2. 開啟 Visual Studio。
3. 選取 [檔案]，然後選取 [新增]。 在 [新增專案] 中，選取 [Azure 資源群組]。
4. 輸入新專案的 [名稱]，然後選取 [確定]。
5. 在 [選取 Azure 範本] 中，從下拉式清單中挑選 [Azure Stack 快速入門]。
6. 選取 [101-create-storage-account]，然後選取 [確定]。
7. 在您的新專案中，展開 [方案總管] 窗格中的 [範本] 節點，以查看可用的範本。
8. 在 [方案總管] 中，挑選您的專案名稱，然後選取 [部署]。 選取 [新增部署]。
9. 在 [部署至資源群組] 中，使用 [訂用帳戶] 下拉式清單來選取您的 Microsoft Azure Stack 訂用帳戶。
10. 從 [資源群組] 清單，選擇現有資源群組或建立新群組。
11. 從 [資源群組位置] 清單，選擇某個位置，然後選取 [部署]。
12. 在 [編輯參數] 中，提供參數值 (依範本而異)，然後選取 [儲存]。

## <a name="next-steps"></a>後續步驟

* [使用命令列部署範本](azure-stack-deploy-template-command-line.md)
* [開發適用於 Azure Stack 的範本](azure-stack-develop-templates.md)
