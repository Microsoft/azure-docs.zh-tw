---
title: 如何從 Azure IoT 中樞裝置佈建服務中取消註冊裝置
description: 如何消註冊裝置以避免透過 Azure IoT 中樞裝置佈建服務進行佈建
author: wesmc7777
ms.author: wesmc
ms.date: 04/05/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: e66d896a7df48645dad39b5b978c4f7c2f8d8cb9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954546"
---
# <a name="how-to-disenroll-a-device-from-azure-iot-hub-device-provisioning-service"></a>如何從 Azure IoT 中樞裝置佈建服務中取消註冊裝置

對於受到廣泛使用的系統 (例如 IoT 解決方案) 來說，適當地管理裝置認證是非常重要的。 適用於這類系統的最佳做法是清楚地規劃當裝置認證 (不論是共用存取簽章 (SAS) 權杖還是 X.509 憑證) 可能外洩時，如何撤銷裝置的存取權。 

裝置佈建服務中的註冊功能可讓裝置[自動佈建](concepts-auto-provisioning.md)。 佈建的裝置是已向 IoT 中樞註冊的裝置，這讓裝置可以接收其最初的[裝置對應項](~/articles/iot-hub/iot-hub-devguide-device-twins.md)狀態，並開始回報遙測資料。 本文會說明如何從佈建服務執行個體中取消註冊裝置，使其之後無法再進行佈建。

> [!NOTE] 
> 針對您要撤銷其存取權的裝置，請注意這些裝置的重試原則。 例如，具有無限重試原則的裝置可能會持續嘗試向佈建服務進行註冊。 該情況會耗用服務資源，而且可能會影響效能。

## <a name="blacklist-devices-by-using-an-individual-enrollment-entry"></a>使用個別註冊項目將裝置列入封鎖清單

個別註冊適用於單一裝置，而且可以使用 X.509 憑證或 SAS 權杖 (適用於實際或虛擬 TPM) 作為證明機制 (針對使用 SAS 權杖作為證明機制的裝置，只能透過個別註冊進行佈建)。若要將具有個別註冊的裝置列入封鎖清單，您可以停用或刪除其註冊項目。 

停用裝置的註冊項目以將裝置暫時列入封鎖清單： 

1. 登入 Azure 入口網站，然後在左側功能表中選取 [所有資源]。
2. 在資源清單中，選取您要從中將裝置列入封鎖清單的佈建服務。
3. 在您的佈建服務中，選取 [管理註冊]，然後選取 [個別註冊] 索引標籤。
4. 選取您想要列入封鎖清單之裝置的註冊項目。 

    ![選取個別註冊](./media/how-to-revoke-device-access-portal/select-individual-enrollment.png)

5. 在註冊頁面上，捲動到底部，並在 [啟用項目] 切換開關選取 [停用]，然後選取 [儲存]。  

   ![在入口網站中停用個別註冊項目](./media/how-to-revoke-device-access-portal/disable-individual-enrollment.png)

刪除裝置的註冊項目以將裝置永久列入封鎖清單：

1. 登入 Azure 入口網站，然後在左側功能表中選取 [所有資源]。
2. 在資源清單中，選取您要從中將裝置列入封鎖清單的佈建服務。
3. 在您的佈建服務中，選取 [管理註冊]，然後選取 [個別註冊] 索引標籤。
4. 針對您想要列入封鎖清單的裝置，選取其註冊項目旁邊的核取方塊。 
5. 選取視窗頂端的 [刪除]，然後選取 [是] 以確認您要移除該註冊。 

   ![在入口網站中刪除個別註冊項目](./media/how-to-revoke-device-access-portal/delete-individual-enrollment.png)


完成此程序之後，您應該會看到您的項目已從個別註冊清單中移除。  

## <a name="blacklist-an-x509-intermediate-or-root-ca-certificate-by-using-an-enrollment-group"></a>使用註冊群組將 X.509 中繼或根 CA 憑證列入封鎖清單

X.509 憑證通常會配置在信任的信任鏈結中。 如果憑證在鏈結中的任何階段發生外洩，信任就會遭到破壞。 您必須將該憑證列入封鎖清單中，才能防止「裝置佈建服務」佈建在該憑證所屬任何鏈結中處於下游的裝置。 若要深入了解 X.509 憑證和其與佈建服務搭配使用的方式，請參閱 [X.509 憑證](./concepts-security.md#x509-certificates)。 

註冊群組是裝置的項目，這些裝置會共用由相同中繼或根 CA 所簽署之 X.509 憑證的通用證明機制。 註冊群組項目是使用與中繼或根 CA 關聯的 X.509 憑證所設定。 此項目的設定中也會包含其憑證鏈結中有該憑證之裝置所共用的任何設定值 (例如對應項狀態和 IoT 中樞連線)。 若要將憑證列入封鎖清單，您可以停用或刪除其註冊群組。

停用憑證的註冊群組以將憑證暫時列入封鎖清單： 

1. 登入 Azure 入口網站，然後在左側功能表中選取 [所有資源]。
2. 在資源清單中，選取您要從中將簽署憑證列入封鎖清單的佈建服務。
3. 在佈建服務中，選取 [管理註冊]，然後選取 [註冊群組] 索引標籤。
4. 若有註冊群組使用您想要列入封鎖清單的憑證，則將其選取。
5. 選取 [啟用項目] 切換開關上的 [停用]，然後選取 [儲存]。  

   ![在入口網站中停用註冊群組項目](./media/how-to-revoke-device-access-portal/disable-enrollment-group.png)

    
刪除憑證的註冊群組以將憑證永久列入封鎖清單：

1. 登入 Azure 入口網站，然後在左側功能表中選取 [所有資源]。
2. 在資源清單中，選取您要從中將裝置列入黑名單的佈建服務。
3. 在佈建服務中，選取 [管理註冊]，然後選取 [註冊群組] 索引標籤。
4. 針對您想要列入封鎖清單的憑證，選取其註冊群組旁邊的核取方塊。 
5. 選取視窗頂端的 [刪除]，然後選取 [是] 以確認您要移除該註冊群組。 

   ![在入口網站中刪除註冊群組項目](./media/how-to-revoke-device-access-portal/delete-enrollment-group.png)

完成此程序之後，您應該會看到您的項目已從註冊群組清單中移除。  

> [!NOTE]
> 如果您刪除某個憑證的註冊群組，其憑證鏈結中有該憑證的裝置可能仍可進行註冊，前提是根憑證或另一個在其憑證鏈結中更上層之中繼憑證的已啟用註冊群組存在。

## <a name="blacklist-specific-devices-in-an-enrollment-group"></a>將註冊群組中的特定裝置列入封鎖清單

實作 X.509 證明機制的裝置會使用裝置的信任鏈結和私密金鑰來進行驗證。 當裝置連線並向「裝置佈建服務」驗證時，服務會先尋找符合裝置認證的個別註冊。 接著，服務才會搜尋註冊群組，以判斷是否可以佈建該裝置。 如果服務找到裝置的已停用個別註冊，就會防止該裝置進行連線。 即使裝置憑證鏈結中之中繼或根 CA 的已啟用註冊群組存在，服務也會防止連線。 

若要將註冊群組中的個別裝置列入封鎖清單，請遵循下列步驟：

1. 登入 Azure 入口網站，然後在左側功能表中選取 [所有資源]。
2. 從資源清單中，選取包含您要列入封鎖清單之裝置註冊群組的佈建服務。
3. 在您的佈建服務中，選取 [管理註冊]，然後選取 [個別註冊] 索引標籤。
4. 選取頂端的 [新增個別註冊] 按鈕。 
5. 在 [新增註冊] 頁面上，選取 [X.509] 作為裝置的證明**機制**。

    上傳裝置憑證，然後輸入要列於封鎖清單之裝置的識別碼。 針對憑證，請使用裝置上所安裝的已簽署終端實體憑證。 裝置使用已簽署的終端實體憑證進行驗證。

    ![設定列於封鎖清單之裝置的裝置屬性](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group-1.png)

6. 捲動到 [新增註冊] 頁面底部，並在 [啟用項目] 切換開關選取 [停用]，然後選取 [儲存]。 

    [![在入口網站中，使用停用的個別註冊項目來停用群組註冊中的裝置](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group.png)](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group.png#lightbox)

成功建立註冊之後，您應該會看到已停用的裝置註冊列在 [個別註冊] 索引標籤上。 

## <a name="next-steps"></a>後續步驟

取消註冊也屬於較大型的取消佈建程序。 取消佈建裝置包括從佈建服務取消註冊，以及從 IoT 中樞取消登錄。 若要深入了解完整程序，請參閱[如何取消佈建先前自動佈建的裝置](how-to-unprovision-devices.md) 

