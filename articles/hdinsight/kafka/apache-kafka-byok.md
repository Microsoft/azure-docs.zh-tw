---
title: 在 Azure HDInsight 上攜帶您自己的 Apache Kafka 金鑰 (預覽)
description: 本文說明如何使用您自己在 Azure Key Vault 中的金鑰，來加密 Azure HDInsight 上的 Apache Kafka 中所儲存的資料。
services: hdinsight
ms.service: hdinsight
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 46105ee92a5c98cb8180b2499d0ad295702aac43
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953359"
---
# <a name="bring-your-own-key-for-apache-kafka-on-azure-hdinsight-preview"></a>在 Azure HDInsight 上攜帶您自己的 Apache Kafka 金鑰 (預覽)

Azure HDInsight 可針對 Apache Kafka 支援攜帶您自己的金鑰 (BYOK)。 這項功能可讓您擁有及管理用來加密待用資料的金鑰。 

HDInsight 中的所有受控磁碟都會使用 Azure 儲存體服務加密 (SSE) 來加以保護。 根據預設，這些磁碟上的資料會使用 Microsoft 所管理的金鑰來加密。 如果您啟用 BYOK，您會對 HDInsight 提供加密金鑰，以透過 Azure Key Vault 來進行使用及管理。 

BYOK 加密是單一步驟的程序，您可在叢集建立期間免費處理此程序。 您只需要使用 Azure Key Vault 將 HDInsight 註冊為受控識別，並在建立叢集時新增加密金鑰即可。

所有傳送到 Kafka 叢集的訊息 (包括 Kafka 所維護的複本) 都會使用對稱資料加密金鑰 (DEK) 來加密。 DEK 會使用金鑰保存庫中的金鑰加密金鑰 (KEK) 來加以保護。 加密和解密程序完全由 Azure HDInsight 來處理。 

您可以使用 Azure 入口網站或 Azure CLI，在金鑰保存庫中安全地輪替金鑰。 金鑰輪替時，HDInsight Kafka 叢集就會在幾分鐘內開始使用新的金鑰。 啟用「不要清除」和「虛刪除」金鑰保護功能，即可防範勒索軟體案例及意外刪除。 沒有這些保護功能的金鑰則不受支援。

## <a name="get-started-with-byok"></a>開始使用 BYOK

1. 建立 Azure 資源的受控識別。

   若要向 Key Vault 進行驗證，請使用 [Azure 入口網站](../../active-directory/managed-service-identity/how-to-manage-ua-identity-portal.md)、[Azure PowerShell](../../active-directory/managed-service-identity/how-to-manage-ua-identity-powershell.md)、[Azure Resource Manager](../../active-directory/managed-service-identity/how-to-manage-ua-identity-arm.md) 或 [Azure CLI](../../active-directory/managed-service-identity/how-to-manage-ua-identity-cli.md) 來建立使用者指派的受控識別。 必須有 Azure Active directory 才能使用受控識別和 Kafka 的 BYOK，但不需要企業安全性套件 (ESP)。 將受控識別資源識別碼新增至 Key Vault 存取原則時，請務必儲存起來。

   ![在 Azure 入口網站中建立使用者指派的受控識別](./media/apache-kafka-byok/user-managed-identity-portal.png)

2. 建立或匯入 Azure Key Vault。

   HDInsight 僅支援 Azure Key Vault。 如果您有自己的金鑰保存庫，則可以將自己的金鑰匯入 Azure Key Vault 中。 請記住，金鑰必須啟用「虛刪除」和「不要清除」。 「虛刪除」和「不要清除」功能可透過 REST、.NET/C#、PowerShell 及 Azure CLI 介面來使用。

   若要建立新的金鑰保存庫，請遵循 [Azure Key Vault](../../key-vault/key-vault-get-started.md) 快速入門。 如需如何匯入現有金鑰的詳細資訊，請瀏覽[關於金鑰、祕密和憑證](../../key-vault/about-keys-secrets-and-certificates.md)。

3. 將受控識別新增至金鑰保存庫存取原則。

   建立新的 Azure Key Vault 存取原則。

   ![建立新的 Azure Key Vault 存取原則](./media/apache-kafka-byok/add-key-vault-access-policy.png)

   將 [金鑰權限] 設定為 [取得]、[將金鑰解除包裝] 及 [包裝金鑰]。

   ![設定 Azure Key Vault 存取原則的金鑰權限](./media/apache-kafka-byok/add-key-vault-access-policy-keys.png)

   將 [祕密權限] 設定為 [取得]、[設定] 及 [刪除]。

   ![設定 Azure Key Vault 存取原則的金鑰權限](./media/apache-kafka-byok/add-key-vault-access-policy-secrets.png)

   在 [選取主體] 底下，選擇您所建立的使用者指派受控識別。

   ![為 Azure Key Vault 存取原則設定 [選取主體]](./media/apache-kafka-byok/add-key-vault-access-policy-select-principal.png)

4. 建立 HDInsight 叢集

   您現在可以開始建立新的 HDInsight 叢集。 BYOK 只能在叢集建立期間套用至新的叢集。 您無法從 BYOK 叢集移除加密，也無法將 BYOK 新增到現有叢集。

   ![Azure 入口網站中的 Kafka 磁碟加密](./media/apache-kafka-byok/apache-kafka-byok-portal.png)

   叢集建立期間，請提供完整的金鑰 URL，包括金鑰版本。 例如： `myakv.azure.com/KEK1/v1`。 您也需要將受控識別指派給叢集，並提供金鑰 URI。

## <a name="faq-for-byok-to-kafka"></a>Kafka 的 BYOK 常見問題集

**Kafka 叢集如何存取我的金鑰保存庫？**

   在叢集建立期間，將受控識別與 HDInsight Kafka 叢集相關聯。 您可以在叢集建立之前或建立期間，建立這個受控識別。 您也需要對受控識別授與權限，使其能夠存取金鑰儲存所在的金鑰保存庫。

**這項功能是否適用於 HDInsight 上的所有 Kafka 叢集？**

   BYOK 加密僅適用於 Kafka 1.1 和更新版本的叢集。

**不同主題/資料分割是否可以有不同金鑰？**

   不行，叢集中的所有受控磁碟都要由相同金鑰加密。

**如果金鑰已刪除，要如何復原叢集？**

   由於只支援啟用「虛刪除」的金鑰，所以如果在金鑰保存庫中還原金鑰，叢集應該就能重新存取金鑰。 若要還原 Azure Key Vault 的金鑰，請參閱 [Restore-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey)。

**是否可以讓生產者/消費者應用程式同時與 BYOK 叢集和非 BYOK 叢集搭配運作？**

   是。 生產者/消費者應用程式並不知道您有使用 BYOK。 加密會在 OS 層進行。 您不必對現有的生產者/消費者 Kafka 應用程式進行任何變更。

**是否也會加密 OS 磁碟/資源磁碟？**

   否。 不會加密 OS 磁碟和資源磁碟。

**如果將叢集相應增加，新的訊息代理程式是否也會順暢地支援 BYOK？**

   是。 叢集在相應增加期間需要存取金鑰保存庫中的金鑰。 叢集中的所有受控磁碟會使用同一個金鑰來加密。

**我所在的位置是否可以使用 BYOK？**

   Kafka BYOK 適用於所有公用雲端。

## <a name="next-steps"></a>後續步驟

* 如需 Azure 金鑰保存庫的詳細資訊，請參閱[什麼是 Azure 金鑰保存庫](../../key-vault/key-vault-whatis.md)？
* 若要開始使用 Azure Key Vault，請參閱[開始使用 Azure Key Vault](../../key-vault/key-vault-get-started.md)。
