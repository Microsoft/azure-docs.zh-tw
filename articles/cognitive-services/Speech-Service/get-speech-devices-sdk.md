---
title: 取得語音裝置 SDK
description: 了解如何取得語音裝置 SDK 的存取權。
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: v-jerkin
ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 05/07/2018
ms.author: v-jerkin
ms.openlocfilehash: f70b41cd7e3a7a6eddf32ae6ad024fa9ac040f29
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39281777"
---
# <a name="get-the-cognitive-services-speech-devices-sdk"></a>取得認知服務語音裝置 SDK

## <a name="requesting-access"></a>要求存取權

語音裝置 SDK 處於受限預覽狀態，要求您在程式中註冊。 目前，Microsoft 會偏好以大型公司作為存取這項產品的候選者。

若要取得語音裝置 SDK 的存取權，請遵循下列步驟：

1. 移至 Microsoft 語音裝置 SDK [註冊表單](https://aka.ms/sdsdk-signup)。
1. 閱讀[授權合約](speech-devices-sdk-license.md)。
1. 如果您同意此授權合約的條款，請選取 [我同意]。
1. 回答表單中的問題。
1. 提交表單。 
1. 如果您的電子郵件地址還不是 Azure Active Directory 的一部分，您會在核准時收到類似下面的邀請電子郵件。 如果您的電子郵件地址已經在 Azure Active Directory 中，您會在核准時收到來自 Microsoft 語音小組的電子郵件訊息，而且可以跳到[下載語音裝置 SDK](#download-the-speech-devices-sdk)。

## <a name="approval-e-mail"></a>核准電子郵件

```
From: Microsoft Speech Team from Microsoft (via Microsoft) <invites@microsoft.com> 
Subject: You're invited to the Microsoft organization 
```

![電子郵件訊息](media/speech-devices-sdk/get-sdk-1.png)

## <a name="accept-access"></a>接受存取
執行下列步驟，以您在註冊期間提供的電子郵件地址加入 Azure Active Directory。 此程序可供您存取語音裝置 SDK 的[下載網站](https://shares.datatransfer.microsoft.com/)。

1. 按一下您收到的電子郵件訊息中的 [開始使用]。 如果您的組織已經是 Office 365 客戶，系統將會提示您登入，而您可以跳到步驟 8。

2. 在啟動的瀏覽器視窗中按 [下一步]。

    ![驗證視窗](media/speech-devices-sdk/get-sdk-2.png)

3. 建立 Microsoft 帳戶 (如果您還沒有帳戶的話)。 輸入您在前述步驟 6 中用來接收邀請電子郵件的相同電子郵件地址。

    ![建立 Microsoft 帳戶](media/speech-devices-sdk/get-sdk-3.png)

4. 按 [下一步] 建立密碼。

5. 當系統提示您驗證電子郵件時，請回到您的電子郵件收件匣以取得傳送給您的驗證碼。
 
7. 在對話方塊中貼上或輸入電子郵件訊息中的安全碼。 在此範例中為 "8406"。 然後按 [下一步] 。

    ![驗證電子郵件](media/speech-devices-sdk/get-sdk-6.png)
 
8. 當您在瀏覽器視窗中看到 [存取面板應用程式] 時，您已確認您的電子郵件地址 (來自步驟 6) 現在是 Azure Active Directory 的一部分。 您現在可以存取語音裝置 SDK 下載網站。

## <a name="download-the-speech-devices-sdk"></a>下載語音裝置 SDK

移至[語音裝置 SDK 下載網站](https://shares.datatransfer.microsoft.com/)，並使用您稍早建立的 Microsoft 帳戶登入。 您現在可以依照下列步驟，下載語音裝置 SDK、相關聯的範例程式碼及參考資料。

![sdk 下載網站](media/speech-devices-sdk/get-sdk-7.png)

1. 當瀏覽器出現提示時，下載並安裝 Aspera Connect 工具。

    ![下載 Aspera Connect](media/speech-devices-sdk/get-sdk-8.png)
 
1. 按一下 [是] 切換至 Aspera Connect。

    ![切換為 Aspera Connect](media/speech-devices-sdk/get-sdk-9.png)
 
1. 按一下 [允許] 確認透過 Aspera Connect 下載檔案。

    ![透過 Aspera Connect 下載](media/speech-devices-sdk/get-sdk-10.png)
 
1. 已下載檔案之後，請關閉 Aspera Connect 的 [傳輸] 視窗。

    ![Aspera Connect 的 [傳輸] 視窗](media/speech-devices-sdk/get-sdk-11.png)
 
根據預設，檔案會下載到您的 [下載] 資料夾。 您現在可以登出此網站。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [開始使用語音裝置 SDK](speech-devices-sdk-qsg.md)
