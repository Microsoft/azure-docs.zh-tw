---
title: 包含檔案
description: 包含檔案
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 09/06/2018
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 3a3ee0cda3643ac73581f868e47905841d9f1563
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44095586"
---
> [!IMPORTANT]
> - 適用於 Blob 和佇列的 Azure AD 驗證預覽版僅供用於非生產環境。 生產環境的服務等級協定 (SLA) 目前無法使用。 如果您的案例尚未支援 Azure AD 驗證，請繼續在應用程式中使用您的共用金鑰授權或 SAS 權杖。
>
> - 在預覽期間，RBAC 角色指派可能需要五分鐘的時間傳播。
>
> - 在呼叫 Blob 和佇列作業時，您必須使用 HTTPS 來向 Azure AD 進行驗證。
>
> - 在預覽版本中，Azure 入口網站不會使用 Azure AD 認證來讀取和寫入 Blob 與佇列資料。 相反地，入口網站會繼續依賴帳戶存取金鑰。 若要在入口網站檢視或更新 Blob 或佇列資料，使用者必須獲派包含 [Microsoft.Storage/storageAccounts/listkeys/action](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-account-key-operator-service-role) 的 RBAC 角色，其可授與呼叫[儲存體帳戶 - 列出金鑰](https://docs.microsoft.com/rest/api/storagerp/storageaccounts/listkeys)的權限。 Blob 和佇列的「參與者」和「讀取者」角色目前未在預覽版本中包含 **listkeys** 動作，因此無法提供透過 Azure 入口網站來存取資料的功能。 若要探索如何在預覽版本期間以身分識別存取 Blob 和佇列資料，請使用 PowerShell 或 Azure CLI。
