---
title: 使用備份中心管理您的備份資產
description: 瞭解如何管理您的 Azure 環境，以確保您的所有資源都符合備份中心的備份觀點。
ms.topic: conceptual
ms.date: 09/01/2020
ms.openlocfilehash: 67b0591c7d7146d162687018854365d338105d76
ms.sourcegitcommit: 100390fefd8f1c48173c51b71650c8ca1b26f711
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98893841"
---
# <a name="govern-your-backup-estate-using-backup-center-preview"></a>使用備份中心 (預覽版來管理您的備份資產) 

備份中心可協助您管理 Azure 環境，以確保您的所有資源都符合備份的觀點。 以下是備份中心的一些治理功能：

* 查看並指派適用于備份的 Azure 原則

* 在所有用於備份的內建 Azure 原則上，查看資源的合規性。

* 查看尚未針對備份設定的所有資料來源。

## <a name="supported-scenarios"></a>支援的案例

* 如需支援和不支援案例的詳細清單，請參閱 [支援矩陣](backup-center-support-matrix.md) 。

## <a name="azure-policies-for-backup"></a>適用于備份的 Azure 原則

若要查看所有可用於備份的 [Azure 原則](../governance/policy/overview.md) ，請選取 [ **備份的 azure 原則** ] 功能表項目。 這會顯示適用于備份的所有內建和自訂 [Azure 原則定義](policy-reference.md) ，這些定義可以指派給您的訂用帳戶和資源群組。

選取任何定義，可讓您 [將原則指派](../governance/policy/tutorials/create-and-manage.md#assign-a-policy) 給某個範圍。

![選取 Azure 原則定義](./media/backup-center-govern-environment/azure-policy-definitions.png)

## <a name="backup-compliance"></a>備份合規性

按一下 [備份合規性] 功能表項目，可協助您根據已指派給 Azure 環境的各種內建原則，來查看資源的 [合規性](../governance/policy/how-to/get-compliance-data.md) 。 您可以查看所有原則都符合規範的資源百分比，以及有一或多個不符合規範之資源的原則。

![查看備份合規性](./media/backup-center-govern-environment/azure-policy-compliance.png)

## <a name="protectable-datasources"></a>可保護的資料來源

選取 [可保護的資料 **源** ] 功能表項目，可讓您查看尚未設定備份的所有資料來源。 您可以依 datasource 訂用帳戶、資源群組、位置、類型和標記來篩選清單。 一旦您識別出需要備份的資料來源之後，您可以在對應的格線專案上按一下滑鼠右鍵，然後選取 [ **備份** ] 來設定資源的備份。

![可保護的資料來源功能表](./media/backup-center-govern-environment/protectable-datasources.png)

> [!NOTE]
> 如果您選取 **AZURE VM 中的 SQL** 做為資料來源類型，可保護的資料 **源** 視圖會顯示沒有任何已設定備份之 SQL 資料庫的所有資源庫 vm 清單。
> 如果您選取 [ **Azure 儲存體 (Azure 檔案儲存體)** 作為資料來源類型，[可保護的資料 **源** ] 視圖會顯示 (支援檔案共用的所有儲存體帳戶清單) 該檔案共用沒有任何已設定進行備份的檔案共用。


## <a name="next-steps"></a>後續步驟

* [監視和操作備份](backup-center-monitor-operate.md)
* [使用備份中心執行動作](backup-center-actions.md)
* [取得您的備份見解](backup-center-obtain-insights.md)