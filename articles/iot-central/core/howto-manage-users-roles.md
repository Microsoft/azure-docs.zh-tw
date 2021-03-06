---
title: 在 Azure IoT Central 應用程式中管理使用者和角色 |Microsoft Docs
description: 以系統管理員身分，如何在 Azure IoT Central 應用程式中管理使用者和角色
author: lmasieri
ms.author: lmasieri
ms.date: 12/05/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: corywink
ms.openlocfilehash: f6c45b8d9804f16c4e59d259f562cc03f187e6a0
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2020
ms.locfileid: "92122972"
---
# <a name="manage-users-and-roles-in-your-iot-central-application"></a>在 IoT Central 應用程式中管理使用者和角色

本文描述如何以系統管理員身分，在您的 Azure IoT Central 應用程式中新增、編輯和刪除使用者。 本文也說明如何在您的 Azure IoT Central 應用程式中管理角色。

若要存取並使用 [管理]**** 區段，您必須屬於 Azure IoT Central 應用程式的**管理員**角色。 如果您建立 Azure IoT Central 應用程式，系統就會自動將您新增至該應用程式的 **系統管理員** 角色。

## <a name="add-users"></a>新增使用者

每個使用者都必須具有使用者帳戶，才能登入並存取 Azure IoT Central 應用程式。 Azure IoT Central 支援 Microsoft 帳戶和 Azure Active Directory 帳戶。 Azure IoT Central 目前不支援 Azure Active Directory 群組。

如需詳細資訊，請參閱 [Microsoft 帳戶說明](https://support.microsoft.com/products/microsoft-account?category=manage-account)和[快速入門：將使用者新增至 Azure Active Directory](../../active-directory/fundamentals/add-users-azure-active-directory.md)。

1. 若要將使用者新增至 IoT Central 應用程式，請前往 [管理]**** 區段中的 [使用者]**** 頁面。
    
    > [!div class="mx-imgBorder"]
    >![管理使用者](media/howto-manage-users-roles/manage-users-pnp.png)

1. 若要新增使用者，在 [使用者]**** 頁面上，選擇 [+ 新增使用者]****。

1. 從 [角色]**** 下拉式清單中選擇使用者的角色。 請在本文的[管理角色](#manage-roles)一節中深入了解角色。

    > [!div class="mx-imgBorder"]
    >![新增使用者並選取角色](media/howto-manage-users-roles/add-user-pnp.png)

    > [!NOTE]
    > 身為自訂角色的使用者，授與他們新增其他使用者的許可權，只能將使用者新增至許可權與他們自己的角色相同或更少的角色。

如果從 Azure Active Directory 中刪除 IoT Central 的使用者識別碼，然後 readded，使用者將無法登入 IoT Central 應用程式。 若要重新啟用存取權，IoT Central 系統管理員應刪除應用程式中的使用者，並重新新增該使用者。

### <a name="edit-the-roles-that-are-assigned-to-users"></a>編輯指派給使用者的角色

角色指派之後就無法變更。 若要變更指派給使用者的角色，請刪除該使用者，然後以不同的角色再次新增該使用者。

> [!NOTE]
> 指派的角色是 IoT Central 應用程式特有的角色，無法從 Azure 入口網站進行管理。

## <a name="delete-users"></a>刪除使用者

若要刪除使用者，請選取 [使用者]**** 頁面上的一或多個核取方塊。 然後選取 [刪除]。

## <a name="manage-roles"></a>管理角色

角色可讓您控制組織中的哪些人員可以在 IoT Central 中執行各種工作。 您可以將三個內建角色指派給應用程式的使用者。 如果您需要更精細的控制，您也可以 [建立自訂角色](#create-a-custom-role) 。

> [!div class="mx-imgBorder"]
> ![管理角色選取專案](media/howto-manage-users-roles/manage-roles-pnp.png)

### <a name="administrator"></a>系統管理員

**系統管理員**角色中的使用者可以管理及控制應用程式的每個部分，包括計費。

系統會將建立應用程式的使用者自動指派給**管理員**角色。 必須至少有一個使用者擔任 [系統管理員]**** 角色。

### <a name="builder"></a>建立器

**Builder**角色中的使用者可以管理應用程式的每個部分，但無法在 [系統管理] 或 [連續資料匯出] 索引標籤上進行變更。

### <a name="operator"></a>運算子

**操作員**角色中的使用者可以監視裝置的健康情況和狀態。 不允許對裝置範本進行變更或管理應用程式。 操作員可以新增和刪除裝置、管理裝置集合，以及執行分析和作業。 

## <a name="create-a-custom-role"></a>建立自訂角色

如果您的解決方案需要更細微的存取控制，您可以使用自訂的許可權集來建立自訂角色。 若要建立自訂角色，請流覽至應用程式的 [**管理**] 區段中的 [**角色**] 頁面。 然後選取 [ **+ 新增角色**]，並新增角色的名稱和描述。 選取您的角色所需的許可權，然後選取 [ **儲存**]。

您可以使用將使用者新增至內建角色的相同方式，將使用者新增至您的自訂角色。

> [!div class="mx-imgBorder"]
> ![建立自訂角色](media/howto-manage-users-roles/create-custom-role-pnp.png)

### <a name="custom-role-options"></a>自訂角色選項

當您定義自訂角色時，如果使用者是角色的成員，您可以選擇要授與使用者的許可權集。 有些許可權相依于其他許可權。 例如，如果您將 [ **更新應用程式儀表板** ] 許可權加入至角色，就會自動加入 [ **View application 儀表板** ] 許可權。 下表摘要說明您可以在建立自訂角色時使用的可用許可權及其相依性。

#### <a name="managing-devices"></a>管理裝置

**裝置範本許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 管理 | 檢視 <br/> 其他相依性：查看裝置實例  |
| 完全控制 | 查看、管理 <br/> 其他相依性：查看裝置實例 |

**裝置實例許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無 <br/> 其他相依性：查看裝置範本和裝置群組 |
| 更新 | 檢視 <br/> 其他相依性：查看裝置範本和裝置群組  |
| 建立 | 檢視 <br/> 其他相依性：查看裝置範本和裝置群組  |
| 刪除 | 檢視 <br/> 其他相依性：查看裝置範本和裝置群組  |
| 執行命令 | 更新、觀看 <br/> 其他相依性：查看裝置範本和裝置群組  |
| 完全控制 | 查看、更新、建立、刪除、執行命令 <br/> 其他相依性：查看裝置範本和裝置群組  |

**裝置群組許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無 <br/> 其他相依性：查看裝置範本和裝置實例 |
| 更新 | 檢視 <br/> 其他相依性：查看裝置範本和裝置實例   |
| 建立 | 查看、更新 <br/> 其他相依性：查看裝置範本和裝置實例   |
| 刪除 | 檢視 <br/> 其他相依性：查看裝置範本和裝置實例   |
| 完全控制 | 查看、更新、建立、刪除 <br/> 其他相依性：查看裝置範本和裝置實例 |

**裝置連線能力管理許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 讀取實例 | 無 <br/> 其他相依性：查看裝置範本、裝置群組、裝置實例 |
| 管理實例 | 無 |
| 讀取全域 | 無   |
| 管理全球 | 讀取全域 |
| 完全控制 | 讀取實例、管理實例、讀取全域、管理全域。 <br/> 其他相依性：查看裝置範本、裝置群組、裝置實例 |

**作業許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無 <br/> 其他相依性：查看裝置範本、裝置實例和裝置群組 |
| 更新 | 檢視 <br/> 其他相依性：查看裝置範本、裝置實例和裝置群組 |
| 建立 | 查看、更新 <br/> 其他相依性：查看裝置範本、裝置實例和裝置群組 |
| 刪除 | 檢視 <br/> 其他相依性：查看裝置範本、裝置實例和裝置群組 |
| 執行 | 檢視 <br/> 其他相依性：查看裝置範本、裝置實例和裝置群組;更新裝置實例;在裝置實例上執行命令 |
| 完全控制 | 查看、更新、建立、刪除、執行 <br/> 其他相依性：查看裝置範本、裝置實例和裝置群組;更新裝置實例;在裝置實例上執行命令 |

**規則許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無 <br/> 其他相依性：查看裝置範本 |
| 更新 | 檢視 <br/> 其他相依性：查看裝置範本 |
| 建立 | 查看、更新 <br/> 其他相依性：查看裝置範本 |
| 刪除 | 檢視 <br/> 其他相依性：查看裝置範本 |
| 完全控制 | 查看、更新、建立、刪除 <br/> 其他相依性：查看裝置範本 |

#### <a name="managing-the-app"></a>管理應用程式

**應用程式設定許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 更新 | 檢視   |
| 複製 | 檢視 <br/> 其他相依性：查看裝置範本、裝置實例、裝置群組、儀表板、資料匯出、商標、說明連結、自訂角色、規則 |
| 刪除 | 檢視   |
| 完全控制 | 查看、更新、複製、刪除 <br/> 其他相依性：查看裝置範本、裝置群組、應用程式儀表板、資料匯出、商標、說明連結、自訂角色、規則 |

**應用程式範本匯出許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 匯出 | 檢視 <br/> 其他相依性：查看裝置範本、裝置實例、裝置群組、儀表板、資料匯出、商標、說明連結、自訂角色、規則 |
| 完全控制 | 查看、匯出 <br/> 其他相依性：查看裝置範本、裝置群組、應用程式儀表板、資料匯出、商標、說明連結、自訂角色、規則 |

**帳單許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 管理 | 無     |
| 完全控制 | 管理 |

#### <a name="managing-users-and-roles"></a>管理使用者和角色

**自訂角色許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無 |
| 更新 | 檢視 |
| 建立 | 查看、更新 |
| 刪除 | 檢視 |
| 完全控制 | 查看、更新、建立、刪除 |

**使用者管理權限**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無 <br/> 其他相依性：查看自訂角色 |
| 加 | 檢視 <br/> 其他相依性：查看自訂角色 |
| 刪除 | 檢視 <br/> 其他相依性：查看自訂角色 |
| 完全控制 | View、Add、Delete <br/> 其他相依性：查看自訂角色 |

> [!NOTE]
> 身為自訂角色的使用者，授與他們新增其他使用者的許可權，只能將使用者新增至許可權與他們自己的角色相同或更少的角色。

#### <a name="customizing-the-app"></a>自訂應用程式

**應用程式儀表板許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 更新 | 檢視   |
| 建立 | 查看、更新 |
| 刪除 | 檢視   |
| 完全控制 | 查看、更新、建立、刪除 |

**個人儀表板許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 更新 | 檢視   |
| 建立 | 查看、更新   |
| 刪除 | 檢視   |
| 完全控制 | 查看、更新、建立、刪除 |

**商標、favicon 和色彩許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 更新 | 檢視   |
| 完全控制 | 查看、更新 |

**說明連結許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 更新 | 檢視   |
| 完全控制 | 查看、更新 |

#### <a name="extending-the-app"></a>擴充應用程式

**資料匯出許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 更新 | 檢視   |
| 建立 | 查看、更新  |
| 刪除 | 檢視   |
| 完全控制 | 查看、更新、建立、刪除 |

**API 權杖許可權**

| 名稱 | 相依性 |
| ---- | -------- |
| 檢視 | 無     |
| 建立 | 檢視   |
| 刪除 | 檢視   |
| 完全控制 | View、Create、Delete |

## <a name="next-steps"></a>下一步

現在您已瞭解如何在 Azure IoT Central 應用程式中管理使用者和角色，建議的下一個步驟是瞭解如何 [管理您的帳單](howto-view-bill.md)。