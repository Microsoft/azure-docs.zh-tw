---
title: 管理 Azure IoT Central 應用程式 | Microsoft Docs
description: 了解如何以系統管理員的身分管理 Azure IoT Central 應用程式
author: tbhagwat3
ms.author: tanmayb
ms.date: 04/16/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: a43febf1e78f80451b6aeed19e095b2c313d3216
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39284848"
---
# <a name="how-to-administer-your-application"></a>如何管理您的應用程式

在您建立 Microsoft Azure IoT Central 應用程式之後，就可以使用 Azure IoT Central 使用者介面的 [管理] 區段來管理應用程式。 若要瀏覽至 [管理] 區段，請選擇左側導覽功能表上的 [管理]。

[管理] 區段可讓您：

- 管理使用者

- 管理角色

- 檢視帳單資訊

- 管理應用程式設定

- 延長免費試用

[管理] 區段中有一個次要導覽功能表，其中具有各種管理工作的連結。

若要存取並使用 [管理] 區段，您在 Azure IoT Central 應用程式中擔任 [系統管理員] 角色。 如果您建立了 Azure IoT Central 應用程式，便會自動將該應用程式的 [系統管理員] 角色指派給您。 本文的＜管理使用者＞一節會進一步說明將 [系統管理員] 角色指派給其他使用者的方法。

## <a name="change-application-name"></a>變更應用程式名稱

若要變更應用程式的名稱，請使用次要導覽功能表來瀏覽至 [管理] 區段中的 [應用程式設定] 頁面。

在 [應用程式設定] 頁面上，在 [應用程式名稱] 欄位中輸入您選擇的名稱，然後選擇 [儲存]。

## <a name="change-the-application-url"></a>變更應用程式 URL

若要變更應用程式的 URL，請使用次要導覽功能表來瀏覽至 [管理] 區段中的 [應用程式設定] 頁面。

![[應用程式設定] 頁面](media\howto-administer\image0-a.png)

在 [應用程式設定] 頁面上，在 [URL] 欄位中輸入您選擇的 URL，然後選擇 [儲存]。 URL 的長度最多不能超過 200 個字元。 如果該 URL 不可用，您會看見驗證錯誤

> [!Note]
> 如果您變更 URL，舊的 URL 可能會被其他 Azure IoT Central 客戶所取用。 在該情況下，您將會無法使用該 URL。 當您變更 URL 時，舊 URL 將不會繼續運作，且您必須通知使用者應使用新的 URL。

## <a name="change-the-application-image"></a>變更應用程式影像

如需在 Azure IoT Central 應用程式中使用影像的詳細資訊，請參閱[準備及上傳影像至您的 Azure IoT Central 應用程式](howto-prepare-images.md)。

## <a name="copy-an-application"></a>複製應用程式

您可以建立任何應用程式複本，並減去其中的任何裝置執行個體、裝置資料歷程記錄和使用者資料。 這個複本會是付費應用程式，所以您必須支付費用。 您無法藉由複製另一個應用程式來建立試用版應用程式。

若要複製應用程式，請瀏覽至 [應用程式設定] 頁面，然後按一下 [複製] 按鈕。

![[應用程式設定] 頁面](media\howto-administer\appCopy1.png)

按一下 [複製] 按鈕會開啟一個對話方塊，您可以在其中針對藉由複製應用程式所建立的新應用程式，選取其名稱、URL、AAD 目錄、訂用帳戶和 Azure 區域。 選取上述每個欄位的值，然後按一下 [複製] 按鈕，以確認您要繼續進行。 您可以在關於[如何建立應用程式](howto-create-application.md)的文章中，深入了解要對這些值輸入什麼內容。

![[應用程式設定] 頁面](media\howto-administer\appCopy2.png)

應用程式複製作業成功之後，您就能夠藉由按一下 [應用程式設定] 頁面上所出現的連結，來瀏覽至透過複製應用程式所建立的新應用程式。

![[應用程式設定] 頁面](media\howto-administer\appCopy3.png)

> [!Note]
> 複製應用程式時會複製規則或動作的定義。 不過，由於有原始應用程式存取權的使用者不會複製到您所複製的應用程式，您必須手動將使用者新增至動作，例如向必要使用者寄送電子郵件。

## <a name="delete-an-application"></a>刪除應用程式

若要刪除應用程式，請使用次要導覽功能表來瀏覽至 [管理] 區段中的 [應用程式設定] 頁面。

選擇 [刪除]。

> [!Note]
> 刪除應用程式將會永久刪除所有與該應用程式相關聯的資料。 若要刪除應用程式，在建立應用程式時所選擇的 Azure 訂用帳戶中，您必須具有刪除資源的權限。 若要深入了解，請參閱[使用角色型存取控制來管理 Azure 訂用帳戶資源的存取](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure)。

## <a name="roles-in-azure-iot-central"></a>Azure IoT Central 中的角色

角色可讓您控制有哪些組織中的人員可以執行各種不同的 Azure IoT Central 工作。 Azure IoT Central 具有三個角色，可讓您指派給應用程式的使用者。 角色是根據每個應用程式進行指派。 相同的使用者在不同的應用程式中可以擔任不同的角色。 您可以在應用程式內將多個角色指派給相同的使用者。

### <a name="administrator"></a>系統管理員

擔任 [系統管理員] 角色的使用者能夠存取 Azure IoT Central 應用程式中的所有功能。

系統會自動將 [系統管理員] 角色指派給建立應用程式的使用者。 必須至少有一個使用者擔任 [系統管理員] 角色。

### <a name="application-builder"></a>應用程式建置者

擔任 [應用程式建置者] 角色的使用者，除了管理應用程式之外，能夠執行 Azure IoT Central 應用程式中的所有工作。

### <a name="application-operator"></a>應用程式操作員

擔任 [應用程式操作員] 角色的使用者將無法存取 [應用程式建置者] 頁面。 他們無法管理應用程式。

## <a name="manage-users"></a>管理使用者

應用程式系統管理員可以將應用程式中的角色指派給使用者。

### <a name="add-users"></a>新增使用者

每個使用者都必須具有使用者帳戶，才能登入並存取 Azure IoT Central 應用程式。 Azure IoT Central 支援 Microsoft 帳戶 (MSA) 和 Azure Active Directory (AAD) 帳戶。 Azure IoT Central 目前不支援 Azure Active Directory 群組。

若要深入了解，請參閱 [Microsoft 帳戶的說明](https://support.microsoft.com/products/microsoft-account?category=manage-account)和[快速入門：將使用者新增至 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/add-users-azure-active-directory)。

1. 若要將使用者帳戶新增至 Azyre IoT Central 應用程式，請使用次要導覽功能表來瀏覽至 [管理] 區段中的 [使用者] 頁面。

    ![使用者清單](media\howto-administer\image1.png)

1. 在 [使用者] 頁面上，選擇 [+ 新增使用者] 來新增使用者。

    ![新增使用者](media\howto-administer\image2.png)

1. 當您將使用者新增至 Azure IoT Central 應用程式時，請從 [角色] 下拉式清單為該使用者選擇角色。 請在本文的＜Azure IoT Central 中的角色＞一節中深入了解角色。

    ![角色選取](media\howto-administer\image3.png)

    > [!NOTE]
    >  若要大量新增使用者，請針對所有想要新增的使用者，輸入以分號分隔的使用者識別碼。 從 [角色] 下拉式清單選擇角色，然後選擇 [儲存]。

1. 在您新增使用者之後，[使用者] 頁面上便會出現該使用者的項目。

    ![使用者清單](media\howto-administer\image4.png)

### <a name="edit-the-roles-assigned-to-users"></a>編輯已指派給使用者的角色

角色在指派之後便無法再次變更。 若要變更指派給使用者的角色，請刪除該使用者，並以不同的角色再次新增該使用者。

### <a name="delete-users"></a>刪除使用者

若要刪除使用者，請按一下 [使用者] 頁面上的一或多個核取方塊，然後選擇 [刪除]。

## <a name="view-your-bill"></a>檢視您的帳單

若要檢視帳單，請瀏覽至 [管理] 區段中的 [帳單] 頁面，然後選擇 [帳單]。 Azure 帳單頁面會在新的索引標籤中開啟，您將能看見每個 Azure IoT Central 應用程式的帳單。

## <a name="convert-your-trial-to-a-paid-application"></a>將試用轉換為付費應用程式

您在評估完 IoT Central 之後，便能將試用轉換為付費應用程式。 若要完成此自助程序，請遵循這些步驟：

1. 使用次要導覽功能表來瀏覽至 [管理] 區段中的 [帳單] 頁面。 如果您尚未延長試用，該頁面看起來會如下所示：

    ![免費試用狀態](media/howto-administer/freetrial.png)

2. 按一下 [轉換為付費]。 如果您尚未延長試用，快顯示窗看起來會如下所示：

    在快顯示窗中，選取適當的 Azure Active Directory 租用戶，然後選取您要用於 IoT Central 應用程式的 Azure 訂用帳戶。

    ![延長免費試用](media/howto-administer/extend.png)

3. 按一下 [轉換] 之後，您的試用會轉換為付費應用程式並開始計費。

## <a name="extend-your-free-trial"></a>延長免費試用

根據預設，所有免費試用的可用期間都是 7 天。 如果您想要將試用期間延長至 30 天，請遵循這些步驟：

1. 使用次要導覽功能表來瀏覽至 [管理] 區段中的 [帳單] 頁面：

1. 按一下 [延長試用]。 在快顯示窗中，選取適當的 Azure Active Directory 租用戶，然後選取要用於 IoT Central 應用程式的 Azure 訂用帳戶：

1. 然後按一下 [延長]。 您試用的有效期間現在已是 30 天。

## <a name="next-steps"></a>後續步驟

您現在已了解如何管理 Azure IoT Central 應用程式，以下是建議的後續步驟：

> [!div class="nextstepaction"]
> [設定裝置範本](howto-set-up-template.md)