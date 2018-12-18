---
title: 使用多個服務組態設定 Azure 專案 | Microsoft Docs
description: 了解如何透過變更 ServiceDefinition.csdef、ServiceConfiguration.Local.cscfg 和 ServiceConfiguration.Cloud.cscfg 檔案來設定 Azure 雲端服務專案。
services: visual-studio-online
author: ghogen
manager: douge
assetId: a4fb79ed-384f-4183-9f74-5cac257206b9
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 11/11/2017
ms.author: ghogen
ms.openlocfilehash: 56b01855a5996bb62f721bda2bfc207684227854
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140070"
---
# <a name="configuring-your-azure-project-in-visual-studio-to-use-multiple-service-configurations"></a>在 Visual Studio 中設定 Azure 專案，以便使用多個服務組態

Visual Studio 中的 Azure 雲端服務專案包含三個組態檔：`ServiceDefinition.csdef`、`ServiceConfiguration.Local.cscfg` 和 `ServiceConfiguration.Cloud.cscfg`：

- `ServiceDefinition.csdef` 會部署至 Azure，以描述雲端服務及其角色的需求，並提供適用於所有執行個體的設定。 在執行階段可以使用 Azure 服務裝載執行階段 API 來讀取設定。 唯有當雲端服務停止時，您才可以在 Azure 上更新此檔案。
- `ServiceConfiguration.Local.cscfg` 和 `ServiceConfiguration.Cloud.cscfg` 提供定義檔中的設定值，並指定每個角色要執行的執行個體數目。 "Local" 檔案含有用於本機偵錯的值；"Cloud" 檔案會部署至 Azure 以做為 `ServiceConfiguration.cscfg`，並提供伺服器環境的設定。 當您的雲端服務在 Azure 中執行時，可以更新此檔案。

您可以在 Visual Studio 中使用適當角色的屬性頁來管理和修改組態設定 (以滑鼠右鍵按一下角色，然後選取 [屬性]，或按兩下角色)。 您可以將變更的範圍限定為在 [服務組態] 下拉式清單中選擇的任何組態。 除了以下各節描述的內容之外，Web 與背景工作角色的屬性相去不遠。

![VS_Solution_Explorer_Roles_Properties](./media/vs-azure-tools-multiple-services-project-configurations/IC784076.png)

如需服務定義檔和服務組態檔的基礎結構描述相關資訊，請參閱 [.csdef XML Schema](cloud-services/schema-csdef-file.md)和 [.cscfg XML Schema](cloud-services/schema-cscfg-file.md)。 如需服務組態的詳細資訊，請參閱 [如何設定雲端服務](cloud-services/cloud-services-how-to-configure-portal.md)。


## <a name="configuration-page"></a>組態頁面

### <a name="service-configuration"></a>服務組態

選取受變更影響的 `ServiceConfiguration.*.cscfg` 檔案。 根據預設，檔案中會有 Local 和 Cloud 變體，您可以使用 [管理...] 命令來複製、重新命名和移除組態檔案。 這些檔案會加入至您的雲端服務專案，且會出現在**方案總管**中。 不過，重新命名或移除組態只能從這個控制項進行。

### <a name="instances"></a>執行個體

將 [執行個體]  計數屬性設定為服務應對此角色執行的執行個體數目。

將 [VM 大小] 屬性設為 [超小]、[小型]、[中型]、[大型] 或 [特大]。  如需詳細資訊，請參閱 [雲端服務的大小](cloud-services/cloud-services-sizes-specs.md)。

### <a name="startup-action-web-role-only"></a>啟動動作 (僅限 Web 角色)

設定此屬性，可指定當您開始偵錯時，Visual Studio 應針對 HTTP 端點或 HTTPS 端點或兩者啟動網頁瀏覽器。

只有在您已為角色定義 HTTPS 端點時，才可使用 **HTTPS 端點**選項。 您可以在 [端點]  屬性頁面上定義 HTTPS 端點。

如果您已新增 HTTPS 端點，則預設會啟用 HTTPS 端點選項，而在您開始偵錯時，Visual Studio 除了會為 HTTP 端點啟動瀏覽器之外，還會為此端點啟動瀏覽器 (假設兩種啟動選項均已啟用)。

### <a name="diagnostics"></a>診斷

Web 角色預設會啟用診斷。 Azure 雲端服務專案和儲存體帳戶已設為使用本機儲存體模擬器。 當您準備好部署至 Azure 時，您可以選取產生器按鈕 ([...]) 改用 Azure 儲存體。 您可以隨選方式或在自動排程的間隔，將診斷資料傳輸至儲存體帳戶。 如需 Azure 診斷的詳細資訊，請參閱 [在 Azure 雲端服務和虛擬機器中啟用診斷](cloud-services/cloud-services-dotnet-diagnostics.md)。

## <a name="settings-page"></a>設定頁面

在 [設定] 頁面上，您可以利用成對的名稱和數值形式將設定加入組態。 在角色中執行的程式碼可以在執行階段使用 [Azure 受控程式庫](http://go.microsoft.com/fwlink?LinkID=171026) 所提供的類別 (具體而言，使用 [GetConfigurationSettingValue](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.getconfigurationsettingvalue.aspx) 方法) 讀取組態設定的值。

### <a name="configuring-a-connection-string-for-a-storage-account"></a>設定儲存體帳戶的連接字串

連接字串是一個設定，可為儲存體模擬器或 Azure 儲存體帳戶提供連線和驗證資訊。 每當角色中的程式碼存取 Azure 儲存體時 (Blob、佇列或資料表)，其皆需要連接字串。

> [!Note]
> Azure 儲存體帳戶的連接字串必須使用已定義的格式 (請參閱[設定 Azure 儲存體連接字串](storage/common/storage-configure-connection-string.md))。

您可以視需要將連接字串設定為使用本機儲存體，然後在將應用程式部署到雲端時，設定為使用 Azure 儲存體帳戶。 連接字串的設定不正確，可能會導致您的角色無法啟動，或在初始化中、忙碌中和停止中狀態間循環。

若要建立連接字串，請選取 [新增設定]，再將 [類型] 設定為 [連接字串]。

對於全新或現有的連接字串，請選取 [值] 欄位右側的 [...]* 以開啟 [建立儲存體連接字串] 對話方塊：

1. 在 [連接方式] 下方，選擇 [您的訂用帳戶] 選項以從訂用帳戶選取儲存體帳戶。 Visual Studio 接著會自動從 `.publishsettings` 檔案取得儲存體帳戶認證。
1. 選取 [手動輸入的認證]可讓您使用 Azure 入口網站的資訊直接指定帳戶名稱和金鑰。 若要複製帳戶金鑰：a. 在 Azure 入口網站上瀏覽至儲存體帳戶，然後選取 [管理金鑰]。
    2. 若要複製帳戶金鑰，請瀏覽至 Azure 入口網站上的儲存體帳戶，選取 [設定] > [存取金鑰]，然後使用 [複製] 按鈕將主要存取金鑰複製到剪貼簿。
1. 選取其中一個連線選項。 [指定自訂端點] 會要求您指定 Blob、資料表和佇列的特定 URL。 自訂端點允許您使用[自訂網域](storage/blobs/storage-custom-domain-name.md)更精確地控制存取。 請參閱[設定 Azure 儲存體連接字串](./storage/common/storage-configure-connection-string.md)。
1. 選取 [確定]，接著再選取 [檔案] > [儲存] 以新的連接字串更新組態。

再強調一次，當您將應用程式發佈至 Azure 時，請選擇包含連接字串之 Azure 儲存體帳戶的服務組態。 發佈您的應用程式之後，請確認應用程式如預期般對 Azure 儲存體服務運作。

如需關於如何更新服務組態的詳細資訊，請參閱[管理儲存體帳戶的連接字串](vs-azure-tools-configure-roles-for-cloud-service.md#manage-connection-strings-for-storage-accounts)一節。

## <a name="endpoints-page"></a>端點頁面

Web 角色通常會有一個位於連接埠 80 的 HTTP 端點。 另一方面，背景工作角色可以有任意數目的 HTTP、HTTPS 或 TCP 端點。 端點可以是可供外部用戶端使用的輸入端點，或是可供在服務中執行的其他角色使用的內部端點。

- 若要讓 HTTP 端點可供外部用戶端和網頁瀏覽器使用，請將端點類型變更為輸入，並指定名稱和公用連接埠號碼。
- 若要讓 HTTPS 端點可供外部用戶端和網頁瀏覽器使用，請將端點類型變更為 [輸入] ，並指定名稱、公用連接埠號碼和管理憑證名稱。 您必須先在 [憑證]  屬性頁面上定義憑證，才可以指定管理憑證。 
- 若要讓端點可供雲端服務中的其他角色用於內部存取，請將端點類型變更為內部，並指定此端點的名稱和可能的私人連接埠。

## <a name="local-storage-page"></a>本機儲存體頁面

您可以使用 [本機儲存體]  屬性頁面，為角色保留一或多個本機儲存體資源。 本機儲存體資源是執行中角色執行個體所在之 Azure 虛擬機器的檔案系統中的保留目錄。

## <a name="certificates-page"></a>憑證頁面

[憑證]  屬性頁面會將憑證的相關資訊新增至您的服務組態。 請注意，您的憑證不會與您的服務一起封裝；您必須透過 [Azure 入口網站](http://portal.azure.com)將憑證分別上傳至 Azure。

在這裡新增憑證會將憑證的相關資訊新增至您的服務組態。 憑證不會與您的服務一起封裝；您必須透過 Azure 入口網站分別上傳憑證。

若要讓憑證與角色產生關聯，請提供憑證的名稱。 在 [端點] 頁面上設定 HTTPS 端點時，您可以使用這個名稱來參照憑證。 接下來，指定憑證存放區為 [本機電腦] 或 [目前使用者] 和存放區的名稱。 最後，輸入憑證的指紋。 如果憑證在目前使用者 \ 個人 (我的) 存放區中，您可從填入的清單中選取憑證，以輸入憑證的指紋。 如果憑證位於任何其他位置，請以手動方式輸入指紋值。

當您從憑證存放區新增憑證時，所有中繼憑證都會自動新增至您的組態設定。 此外，這些中繼憑證也必須上傳至 Azure，以便針對 SSL 正確設定您的服務。

只在您的服務在雲端執行時，與您的服務相關聯的管理憑證才會套用至您的服務。 當您的服務在本機開發環境中執行時，它會使用由計算模擬器所管理的標準憑證。
