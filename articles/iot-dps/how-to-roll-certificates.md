---
title: 如何在 Azure IoT 中樞裝置佈建服務中輪替 X.509 憑證 | Microsoft Docs
description: 如何使用裝置佈建服務執行個體輪替 X.509 憑證
author: wesmc7777
ms.author: wesmc
ms.date: 08/06/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: 51e96065f726fadd528323157609034b5bb3f151
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387883"
---
# <a name="how-to-roll-x509-device-certificates"></a>如何輪替 X.509 裝置憑證

在 IoT 解決方案的生命週期內，您必須輪替憑證。 輪替憑證的兩大主因是安全性缺口和憑證到期。 

輪替憑證是一種最佳安全性做法，可在外洩事件中協助保護您的系統。 在[假想缺口方法](http://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf)中，Microsoft 主張備妥回應式安全程序和預防性措施的需求。 輪替裝置憑證應納入這些安全程序中。 您輪替憑證的頻率將取決於您解決方案的安全性需求。 有解決方案涉及高度敏感性資料的客戶可以每天輪替，然而其他客戶則是每隔幾年輪替其憑證。

輪替裝置憑證涉及更新儲存在裝置和 IoT 中樞上的憑證。 之後，裝置可以使用正常[自動佈建](concepts-auto-provisioning.md)搭配裝置佈建服務，本身向 IoT 中樞重新佈建。


## <a name="obtain-new-certificates"></a>取得新的憑證

有許多方式可為您的 IoT 裝置取得新憑證。 這些包括從裝置原廠取得憑證，產生自己的憑證，以及讓第三方為您管理憑證建立。 

憑證會彼此簽署，以構成從根 CA 憑證到[分葉憑證](concepts-security.md#end-entity-leaf-certificate)的信任鏈結。 簽署憑證是用來簽署信任鏈結尾端分葉憑證的憑證。 簽署憑證可以是信任鏈結的根 CA 憑證或中繼憑證。 如需詳細資訊，請參閱 [X.509 憑證](concepts-security.md#x509-certificates)。
 
有兩種不同的方式可取得簽署憑證。 第一種方式就是向根憑證授權單位 (CA) 購買簽署憑證，建議用於生產系統。 這種方式可將安全性向下鏈結到信任的來源。 

第二種方式就是使用 OpenSSL 之類的工具，建立自己的 X.509 憑證。 這種方法適合用來測試 X.509 憑證，但是提供很少的安全性保證。 除非您準備好作為自己的 CA 提供者，否則我們建議僅使用此方法進行測試。
 

## <a name="roll-the-certificate-on-the-device"></a>在裝置上輪替憑證

裝置上的憑證應該一律存放在安全的地方，像是[硬體安全性模組 (HSM)](concepts-device.md#hardware-security-module)。 輪替裝置憑證的方式將取決於它們一開始在裝置中建立和安裝的方式。 

如果您從第三方取得您的憑證，您必須研究它們如何輪替其憑證。 此程序可能包含在您與他們的約定中，或者可能是他們所提供的個別服務。 

如果您要管理自己的裝置憑證，您必須建置自己的管線，以便更新憑證。 請確定舊的和新的分葉憑證具有相同的一般名稱 (CN)。 具有相同的 CN，裝置本身即可重新佈建，而不需要建立重複的註冊記錄。 


## <a name="roll-the-certificate-in-the-iot-hub"></a>在 IoT 中樞輪替憑證

您可將裝置憑證手動新增至 IoT 中樞。 使用裝置佈建服務執行個體也可以自動處理憑證。 在本文中，我們假設會使用裝置佈建服務執行個體來支援自動佈建。

當裝置一開始透過自動佈建進行佈建時，它會啟動並連絡佈建服務。 佈建服務的回應方式如下：使用裝置的分葉憑證作為認證，在 IoT 中樞建立裝置身分識別之前，先執行身分識別檢查。 佈建服務會接著告知裝置獲派的 IoT 中樞，然後裝置會使用其分葉憑證進行驗證及連線到 IoT 中樞。 

新的分葉憑證輪替至裝置後，便無法再連線到 IoT 中樞，因為它目前使用新的憑證進行連線。 IoT 中樞只會辨識使用舊憑證的裝置。 裝置的連線嘗試結果會是「未經授權」的連線錯誤。 若要解決這個錯誤，您必須更新裝置的註冊項目，以將裝置的新分葉憑證納入考量。 然後佈建服務可以在重新佈建裝置時，視需要更新 IoT 中樞裝置登錄資訊。 

此連線失敗有一個可能的例外狀況：您已在佈建服務中為您的裝置建立[註冊群組](concepts-service.md#enrollment-group)。 在此情況下，如果您並未輪替裝置的信任鏈結中的根憑證或中繼憑證，只有新憑證屬於註冊群組中所定義的信任鏈結，就能辨識此裝置。 如果出現這種情況來反應安全性缺口，您應該至少將群組中被視為遭入侵的特定裝置憑證列入黑名單中。 如需詳細資訊，請參閱[將註冊群組中的特定裝置列入黑名單](https://docs.microsoft.com/azure/iot-dps/how-to-revoke-device-access-portal#blacklist-specific-devices-in-an-enrollment-group)。

在 [管理註冊] 頁面上，可以完成已輪替憑證的註冊項目更新。 若要存取該頁面，請遵循下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)，以瀏覽至具有您裝置註冊項目的 IoT 中樞裝置佈建服務執行個體。

2. 按一下 [管理註冊]。

    ![管理註冊](./media/how-to-roll-certificates/manage-enrollments-portal.png)


更新註冊項目的處理方式，將取決於您使用的是個別註冊或群組註冊。 此外，您是否因為安全性缺口或憑證到期而輪替憑證，建議的程序也會而有所不同。 下列各節說明如何處理這些更新。


## <a name="individual-enrollments-and-security-breaches"></a>個別註冊和安全性缺口

如果您是以輪替憑證的方式因應安全性缺口，則應該使用下列方法立即刪除目前的憑證：

1. 按一下 [個別註冊]，然後按一下清單中的註冊識別碼項目。 

2. 按一下 [刪除目前的憑證] 按鈕，然後按一下資料夾圖示，以選取要針對註冊項目上傳的新憑證。 完成後，請按一下 [儲存]。

    如果主要和次要憑證都遭到入侵，則應對這兩者完成下列步驟。

    ![管理個別註冊](./media/how-to-roll-certificates/manage-individual-enrollments-portal.png)

3. 將遭到入侵的憑證從佈建服務移除之後，只要服務中有憑證的裝置註冊存在，就仍可以使用該憑證建立裝置與 IoT 中樞的連線。 您可以透過兩種方式解決此問題： 

    第一種方式為手動瀏覽至 IoT 中樞，並立即移除與遭入侵憑證關聯的裝置註冊。 接著，使用已更新的憑證再次佈建裝置，此時將會建立新的裝置註冊。     

    ![移除 IoT 中樞裝置註冊](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    第二種方式為利用重新佈建支援將裝置重新佈建到相同的 IoT 中樞。 此方法可用來取代 IoT 中樞上裝置註冊的憑證。 如需詳細資訊，請參閱[如何重新佈建裝置](how-to-reprovision.md)。

## <a name="individual-enrollments-and-certificate-expiration"></a>個別註冊和憑證到期

如果您是以輪替憑證的方式來處理憑證過期，則應使用如下所示的次要憑證組態，使嘗試佈建的裝置減少停機時間。

稍後當次要憑證也快要到期，而且需要輪替時，您可以輪換成使用主要組態。 如此一來，輪流使用主要與次要憑證可使嘗試佈建的裝置減少停機時間。


1. 按一下 [個別註冊]，然後按一下清單中的註冊識別碼項目。 

2. 按一下 [次要憑證]，然後按一下資料夾圖示，以選取要針對註冊項目上傳的新憑證。 按一下 [檔案] 。

    ![使用次要憑證管理個別註冊](./media/how-to-roll-certificates/manage-individual-enrollments-secondary-portal.png)

3. 稍後當主要憑證到期時，返回並按一下 [刪除目前憑證] 按鈕，即可刪除該主要憑證。

## <a name="enrollment-groups-and-security-breaches"></a>註冊群組和安全性缺口

若要更新群組註冊來因應安全性缺口，您應該使用下列其中一種方法，立即刪除目前的根 CA 或中繼憑證。

#### <a name="update-compromised-root-ca-certificates"></a>更新遭到入侵的根 CA 憑證

1. 按一下您裝置佈建服務執行個體的 [憑證] 索引標籤。

2. 按一下清單中遭入侵的憑證，然後按一下 [刪除] 按鈕。 輸入憑證名稱來確認刪除，然後按一下 [確定]。 請針對所有遭入侵的憑證重複此程序。

    ![刪除根 CA 憑證](./media/how-to-roll-certificates/delete-root-cert.png)

3. 請遵循[設定已驗證的 CA 憑證](how-to-verify-certificates.md)中所述的步驟來新增並驗證新的根 CA 憑證。

4. 按一下您裝置佈建服務執行個體的 [管理註冊] 索引標籤，然後按一下 [註冊群組] 清單。 按一下清單中您的註冊群組名稱。

5. 按一下 [CA 憑證]，然後選取新的根 CA 憑證。 然後按一下 [儲存] 。 

    ![選取新的根 CA 憑證](./media/how-to-roll-certificates/select-new-root-cert.png)

6. 將遭到入侵的憑證從佈建服務移除之後，只要服務中有憑證的裝置註冊存在，就仍可以使用該憑證建立裝置與 IoT 中樞的連線。 您可以透過兩種方式解決此問題： 

    第一種方式為手動瀏覽至 IoT 中樞，並立即移除與遭入侵憑證關聯的裝置註冊。 接著，使用已更新的憑證再次佈建裝置，此時將會為每個裝置建立新的裝置註冊。     

    ![移除 IoT 中樞裝置註冊](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    第二種方式為利用重新佈建支援將裝置重新佈建到相同的 IoT 中樞。 此方法可用來取代 IoT 中樞上裝置註冊的憑證。 如需詳細資訊，請參閱[如何重新佈建裝置](how-to-reprovision.md)。



#### <a name="update-compromised-intermediate-certificates"></a>更新遭入侵的中繼憑證

1. 按一下 [註冊群組]，然後按一下清單中的群組名稱。 

2. 按一下 [中繼憑證]，然後按一下 [刪除目前的憑證]。 按一下資料夾圖示，以瀏覽至要針對註冊群組上傳的新中繼憑證。 完成時，請按一下 [儲存]。 如果主要和次要憑證都遭到入侵，則應對這兩者完成下列步驟。

    這個新中繼憑證應該由已經過驗證並新增至佈建服務的根 CA 憑證簽署。 如需詳細資訊，請參閱 [X.509 憑證](concepts-security.md#x509-certificates)。

    ![管理個別註冊](./media/how-to-roll-certificates/enrollment-group-delete-intermediate-cert.png)


3. 將遭到入侵的憑證從佈建服務移除之後，只要服務中有憑證的裝置註冊存在，就仍可以使用該憑證建立裝置與 IoT 中樞的連線。 您可以透過兩種方式解決此問題： 

    第一種方式為手動瀏覽至 IoT 中樞，並立即移除與遭入侵憑證關聯的裝置註冊。 接著，使用已更新的憑證再次佈建裝置，此時將會為每個裝置建立新的裝置註冊。     

    ![移除 IoT 中樞裝置註冊](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    第二種方式為利用重新佈建支援將裝置重新佈建到相同的 IoT 中樞。 此方法可用來取代 IoT 中樞上裝置註冊的憑證。 如需詳細資訊，請參閱[如何重新佈建裝置](how-to-reprovision.md)。


## <a name="enrollment-groups-and-certificate-expiration"></a>註冊群組與憑證到期

如果您是以輪替憑證的方式來處理憑證過期，則應使用如下所示的次要憑證組態，確保嘗試佈建的裝置不會停機。

稍後當次要憑證也快要到期，而且需要輪替時，您可以輪換成使用主要組態。 如此一來，輪流使用主要與次要憑證可確保嘗試佈建的裝置不會停機。 

#### <a name="update-expiring-root-ca-certificates"></a>更新即將到期的根 CA 憑證

1. 請遵循[設定已驗證的 CA 憑證](how-to-verify-certificates.md)中所述的步驟來新增並驗證新的根 CA 憑證。

2. 按一下您裝置佈建服務執行個體的 [管理註冊] 索引標籤，然後按一下 [註冊群組] 清單。 按一下清單中您的註冊群組名稱。

3. 按一下 [CA 憑證]，然後在 [次要憑證] 組態之下選取新的根 CA 憑證。 然後按一下 [儲存] 。 

    ![選取新的根 CA 憑證](./media/how-to-roll-certificates/select-new-root-secondary-cert.png)

4. 稍後當主要憑證到期時，按一下您裝置佈建服務執行個體的 [憑證] 索引標籤。 按一下清單中已到期的憑證，然後按一下 [刪除] 按鈕。 輸入憑證名稱來確認刪除，然後按一下 [確定]。

    ![刪除根 CA 憑證](./media/how-to-roll-certificates/delete-root-cert.png)



#### <a name="update-expiring-intermediate-certificates"></a>更新即將到期的中繼憑證


1. 按一下 [註冊群組]，然後按一下清單中的群組名稱。 

2. 按一下 [次要憑證]，然後按一下資料夾圖示，以選取要針對註冊項目上傳的新憑證。 按一下 [檔案] 。

    這個新中繼憑證應該由已經過驗證並新增至佈建服務的根 CA 憑證簽署。 如需詳細資訊，請參閱 [X.509 憑證](concepts-security.md#x509-certificates)。

   ![使用次要憑證管理個別註冊](./media/how-to-roll-certificates/manage-enrollment-group-secondary-portal.png)

3. 稍後當主要憑證到期時，返回並按一下 [刪除目前的憑證] 按鈕，即可刪除該主要憑證。


## <a name="reprovision-the-device"></a>重新佈建裝置

在裝置和裝置佈建服務上輪替憑證之後，裝置即可連絡裝置佈建服務來自行重新佈建。 

如果裝置在嘗試連線到 IoT 中樞時收到「未經授權」的錯誤，將裝置設計為連絡佈建服務以進行佈建流程，即可輕鬆地設計要重新佈建的裝置。

另一種方法適用於有效期短暫重疊的新舊憑證：使用 IoT 中樞將命令傳送到裝置，讓它們透過佈建服務重新註冊，以更新其 IoT 中樞連線資訊。 每個裝置可能會以不同方式處理命令，因此您必須設計您的裝置，才知道叫用此命令時要執行什麼動作。 您有數種方式可透過 IoT 中樞命令您的裝置，我們建議使用[直接方法](../iot-hub/iot-hub-devguide-direct-methods.md)或[作業](../iot-hub/iot-hub-devguide-jobs.md)來起始此程序。

重新佈建完成後，裝置就能夠使用新憑證來連線到 IoT 中樞。


## <a name="blacklist-certificates"></a>將憑證列入黑名單

為了回應安全性缺口，您可能需要將裝置憑證列入黑名單。 若要將裝置憑證列入黑名單，請停用目標裝置/憑證的註冊項目。 如需詳細資訊，請參閱[管理取消註冊](how-to-revoke-device-access-portal.md)一文中如何將裝置列入黑名單。

一旦將憑證納入已停用的註冊項目中，任何使用該憑證向 IoT 中樞註冊的嘗試都會失敗，即使該憑證已作為其他註冊項目的一部分啟用亦然。
 



## <a name="next-steps"></a>後續步驟

- 若要深入了解裝置佈建服務中的 X.509 憑證，請參閱[安全性](concepts-security.md) 
- 若要了解如何使用 Azure IoT 中樞的裝置佈建服務執行 X.509 CA 憑證擁有權證明，請參閱[如何驗證憑證](how-to-verify-certificates.md)
- 若要深入了解如何使用入口網站來建立註冊群組，請參閱[使用 Azure 入口網站管理裝置註冊](how-to-manage-enrollments.md)。










