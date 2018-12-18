---
title: 包含檔案
description: 包含檔案
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/16/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: a69a739f36854cacd6b361ca2bd17d904e9c4c96
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38753650"
---
資源鎖定可避免組織中的使用者不小心刪除或修改重要資源。 不同於角色型存取控制，資源鎖定會對所有使用者和角色套用限制。 

您可以將鎖定層級設定為 **CanNotDelete** 或 **ReadOnly**。 在入口網站中，鎖定層級分別會顯示為 [刪除] 和 [唯讀]。

* **CanNotDelete** 表示經過授權的使用者仍然可以讀取和修改資源，但無法刪除資源。 
* **ReadOnly** 表示經過授權的使用者可以讀取資源，但無法刪除或更新資源。 套用這個鎖定類似於限制所有經過授權使用者的權限是由「讀取者」角色所授與。 

> [!TIP]
> 套用**唯讀**鎖定時，請小心謹慎。 某些看起來像是讀取作業的作業實際上需要額外動作。 例如，儲存體帳戶上的**唯讀**鎖定會防止所有使用者列出金鑰。 清單金鑰作業是透過 POST 要求進行處理，因為傳回的金鑰可用於寫入作業。 App Service 資源上的**唯讀**鎖定會防止 Visual Studio 伺服器總管顯示該資源的檔案，因為該互動需要寫入存取權。

當您在父範圍套用鎖定時，該範圍內的所有資源都會都繼承相同的鎖定。 甚至您稍後新增的資源都會繼承父項的鎖定。 繼承中限制最嚴格的鎖定優先順序最高。

Resource Manager 鎖定只會套用於管理平面發生的作業，亦即要傳送至 `https://management.azure.com` 的作業。 鎖定並不會限制資源處理自有函式的方式。 限制資源的變更，但沒有限制資源的作業。 例如，SQL Database 上的唯讀鎖定會防止您刪除或修改資料庫。 它不會防止您建立、更新或刪除資料庫中的資料。 允許資料交易，因為這些作業不會傳送到 `https://management.azure.com`。

### <a name="who-can-create-or-delete-locks-in-your-organization"></a>誰可以建立或刪除您的組織中的鎖定
若要建立或刪除管理鎖定，您必須擁有 `Microsoft.Authorization/locks/*` 動作的存取權。 在內建角色中，只有 **擁有者** 和 **使用者存取管理員** 被授與這些動作的存取權。
