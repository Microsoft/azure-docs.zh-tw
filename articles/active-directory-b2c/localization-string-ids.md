---
title: 當地語系化字串識別碼 - Azure Active Directory B2C | Microsoft Docs
description: 在 Azure Active Directory B2C 的自訂原則中，使用 api.signuporsignin 的識別碼來指定適用於內容定義的識別碼。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 7619ebf038e164c9e586118c70e34955ca7c4767
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222659"
---
# <a name="localization-string-ids"></a>當地語系化字串識別碼

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

**Localization** 元素可讓您在適用於使用者旅程圖的原則中支援多個地區設定或語言。 此文章提供您可以在原則中使用的當地語系化識別碼清單。 若要熟悉 UI 當地語系化，請參閱[當地語系化](localization.md)。

## <a name="sign-up-or-sign-in-page-elements"></a>註冊或登入頁面元素

下列識別碼均可用於識別碼為 `api.signuporsignin` 的內容定義。

| ID | 預設值 |
| -- | ------------- |
| **local_intro_email** | 使用您的現有帳戶登入 |
| **logonIdentifier_email** | 電子郵件地址 |
| **requiredField_email** | 請輸入您的電子郵件 |
| **invalid_email** | 請輸入有效的電子郵件地址 |
| **email_pattern** | ^[a-zA-Z0-9.!#$%&’'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\\.[a-zA-Z0-9-]+)*$ |
| **local_intro_username** | 使用您的使用者名稱登入 |
| **logonIdentifier_username** | 使用者名稱 |
| **requiredField_username** | 請輸入您的使用者名稱 |
| **password** | 密碼 |
| **requiredField_password** | 請輸入您的密碼 |
| **invalid_password** | 您輸入的密碼未採用預期的格式。 |
| **forgotpassword_link** | 忘記密碼了嗎? |
| **createaccount_intro** | 還沒有帳戶嗎？ |
| **createaccount_link** | 立即註冊 |
| **divider_title** | 或 |
| **cancel_message** | 使用者忘記密碼 |
| **button_signin** | 登入 |
| **social_intro** | 使用您的社交帳戶登入 |
  **remember_me** |讓我保持登入|
| **unknown_error** | 我們無法將您登入。 請稍後再試一次。 |

下列範例示範在註冊或登入頁面中使用某些使用者介面元素的方式：

![註冊或登入頁面 UX 元素](./media/localization-string-ids/localization-susi.png)

在使用者旅程圖 **ClaimsExchange** 元素中設定識別提供者的識別碼。 若要將識別提供者的標題當地語系化，請將 **ElementType** 設定為 `ClaimsProvider`，同時將 **StringId** 設定為 `ClaimsExchange` 的識別碼。

```XML
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="MicrosoftExchange" TechnicalProfileReferenceId="MSA-OIDC" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccount" />
  </ClaimsExchanges>
</OrchestrationStep>
```

下列範例會將 Facebook 識別提供者當地語系化為阿拉伯文：

```XML
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">فيس بوك</LocalizedString>
```

## <a name="sign-up-or-sign-in-error-messages"></a>註冊或登入錯誤訊息

| ID | 預設值 |
| -- | ------------- |
| **UserMessageIfInvalidPassword** | 您的密碼不正確。 |
| **UserMessageIfClaimsPrincipalDoesNotExist** | 找不到您的帳戶。 |
| **UserMessageIfOldPasswordUsed** | 您使用的似乎是舊密碼。 |  
| **DefaultMessage** | 無效的使用者名稱或密碼。 |  
| **UserMessageIfUserAccountDisabled** | 您的帳戶已遭鎖定。 請連絡您的支援人員將其解除鎖定，然後再試一次。 |  
| **UserMessageIfUserAccountLocked** | 您的帳戶已暫時鎖定以防未經授權的使用。 請稍後再試。 |  
| **AADRequestsThrottled** | 目前的要求過多。 請稍後再重試。 |  

## <a name="sign-up-and-self-asserted-pages-user-interface-elements"></a>註冊和自我判斷頁面使用者介面元素

以下是適用於識別碼為 `api.localaccountsignup` 之內容定義的識別碼，或任何開頭為 `api.selfasserted` 之內容定義的識別碼，例如 `api.selfasserted.profileupdate` 和 `api.localaccountpasswordreset`。

| ID | 預設值 |
| -- | ------------- |
| **ver_sent** | 已將驗證碼傳送至： |
| **ver_but_default** | 預設值 |
| **cancel_message** | 使用者已取消輸入自動判斷的資訊 |
| **preloader_alt** | 請稍候 |
| **ver_but_send** | 傳送驗證碼 |
| **alert_yes** | 是 |
| **error_fieldIncorrect** | 有一或多個欄位填寫不正確。 請檢查您的輸入，然後再試一次。 |
| **year** | Year |
| **verifying_blurb** | 正在處理您的資訊，請稍候。 |
| **button_cancel** | 取消 |
| **ver_fail_no_retry** | 您已嘗試太多次，但都不正確。 請稍後再試一次。 |
| **month** | 月 |
| **ver_success_msg** | 已驗證電子郵件地址。 您現在可以繼續。 |
| **months** | 一月、二月、三月、四月、五月、六月、七月、八月、九月、十月、十一月、十二月 |
| **ver_fail_server** | 我們無法驗證您的電子郵件地址。 請輸入有效的電子郵件地址，然後再試一次。 |
| **error_requiredFieldMissing** | 遺漏必要欄位。 請填寫所有必要欄位，然後再試一次。 |
| **initial_intro** | 請提供下列詳細資料。 |
| **ver_but_resend** | 傳送新的驗證碼 |
| **button_continue** | 建立 |
| **error_passwordEntryMismatch** | 密碼輸入欄位不相符。 請在兩個欄位中輸入相同密碼，然後再試一次。 |
| **ver_incorrect_format** | 格式不正確。 |
| **ver_but_edit** | 變更電子郵件 |
| **ver_but_verify** | 驗證碼 |
| **alert_no** | 否 |
| **ver_info_msg** | 驗證碼已經傳送到您的收件匣。 請將其複製到下方輸入方塊。 |
| **day** | 天 |
| **ver_fail_throttled** | 驗證此電子郵件地址時有過多要求。 請稍候，然後再試一次。 |
| **helplink_text** | 這是什麼？ |
| **ver_fail_retry** | 該驗證碼不正確。 請再試一次。 |
| **alert_title** | 取消輸入您的詳細資料 |
| **required_field** | 此為必要資訊。 |
| **alert_message** | 確定要取消輸入您的詳細資料嗎? |
| **ver_intro_msg** | 必須進行驗證。 請按一下 [傳送] 按鈕。 |
| **ver_input** | 驗證碼 |

## <a name="sign-up-and-self-asserted-pages-error-messages"></a>註冊和自我判斷頁面錯誤訊息

| ID | 預設值 |
| -- | ------------- |
| **UserMessageIfClaimsPrincipalAlreadyExists** | 已存在具有所指定識別碼的使用者。 請選擇不同的識別碼。 |
| **UserMessageIfClaimNotVerified** | 宣告未驗證: {0} |
| **UserMessageIfIncorrectPattern** | 以下項目的模式不正確: {0} |
| **UserMessageIfMissingRequiredElement** | 遺失必要元素: {0} |
| **UserMessageIfValidationError** | 驗證時的錯誤，依: {0} |
| **UserMessageIfInvalidInput** | {0} 的輸入無效。 |
| **ServiceThrottled** | 目前的要求過多。 請稍後再重試。 |

下列範例示範在註冊頁面中使用某些使用者介面元素的方式：

![註冊頁面 UX 元素](./media/localization-string-ids/localization-sign-up.png)

下列範例示範在使用者按一下傳送驗證碼按鈕之後，於註冊頁面中使用某些使用者介面元素的方式：

![註冊頁面電子郵件驗證 UX 元素](./media/localization-string-ids/localization-email-verification.png)


## <a name="phone-factor-authentication-page-user-interface-elements"></a>電話要素驗證頁面使用者介面元素

以下為適用於識別碼為 `api.phonefactor` 之內容定義的識別碼。 

| ID | 預設值 |
| -- | ------------- |
| **button_verify** | 撥電話給我 |
| **country_code_label** | 國碼 (地區碼) |
| **cancel_message** | 使用者已取消多重要素驗證 |
| **text_button_send_second_code** | 傳送新的驗證碼 |
| **code_pattern** | \\d{6} |
| **intro_mixed** | 我們為您記錄了下列號碼。 我們將會透過簡訊傳送代碼以對您進行驗證。 |
| **intro_mixed_p** | 我們為您記錄了下列號碼。 請選擇一組電話號碼來讓我們傳送驗證碼簡訊或撥打，從而驗證您的身分。 |
| **button_verify_code** | 驗證碼 |
| **requiredField_code** | 請輸入您收到的驗證碼 |
| **invalid_code** | 請輸入您收到的 6 位數代碼 |
| **button_cancel** | 取消 |
| **local_number_input_placeholder_text** | 電話號碼 |
| **button_retry** | 重試 |
| **alternative_text** | 我的手機不在身邊 |
| **intro_phone_p** | 我們為您記錄了下列號碼。 請選擇一組電話號碼來讓我們撥打，從而驗證您的身分。 |
| **intro_phone** | 我們為您記錄了下列號碼。 我們將撥打電話來驗證您的身分。 |
| **enter_code_text_intro** | 請在下方輸入您的驗證碼，或  |
| **intro_entry_phone** | 請在下方輸入一組電話號碼來讓我們撥打，從而驗證您的身分。 |
| **intro_entry_sms** | 請在下方輸入一組電話號碼來讓我們傳送驗證碼簡訊，從而驗證您的身分。 |
| **button_send_code** | 傳送驗證碼 |
| **invalid_number** | 請輸入有效的電話號碼 |
| **intro_sms** | 我們為您記錄了下列號碼。 我們將會透過簡訊傳送代碼以對您進行驗證。 |
| **intro_entry_mixed** | 請在下方輸入一組電話號碼來讓我們傳送驗證碼簡訊或撥打，從而驗證您的身分。 |
| **number_pattern** | ^\\+(?:[0-9][\\x20-]?){6,14}[0-9]$ |
| **intro_sms_p** |我們為您記錄了下列號碼。 請選擇可以簡訊傳送驗證碼的號碼，以驗證您的身分。 |
| **requiredField_countryCode** | 請選取您的國碼 (地區碼) |
| **requiredField_number** | 請輸入您的電話號碼 |
| **country_code_input_placeholder_text** |國家或區域 |
| **number_label** | 電話號碼 |
| **error_tryagain** | 提供的電話號碼目前忙線中或不通。 請檢查號碼，然後再試一次。 |
| **error_incorrect_code** | 您輸入的驗證碼與我們的記錄不符。 請再試一次，或要求新的驗證碼。 |
| **countryList** | {\"DEFAULT\":\"國家/地區\",\"AF\":\"阿富汗\",\"AX\":\"奧蘭群島\",\"AL\":\"阿爾巴尼亞\",\"DZ\":\"阿爾及利亞\",\"AS\":\"美屬薩摩亞\",\"AD\":\"安道爾\",\"AO\":\"安哥拉\",\"AI\":\"安圭拉\",\"AQ\":\"南極洲\",\"AG\":\"安地卡及巴布達\",\"AR\":\"阿根廷\",\"AM\":\"亞美尼亞\",\"AW\":\"荷屬阿魯巴\",\"AU\":\"澳洲\",\"AT\":\"奧地利\",\"AZ\":\"亞塞拜然\",\"BS\":\"巴哈馬\",\"BH\":\"巴林\",\"BD\":\"孟加拉\",\"BB\":\"巴貝多\",\"BY\":\"白俄羅斯\",\"BE\":\"比利時\",\"BZ\":\"貝里斯\",\"BJ\":\"貝南\",\"BM\":\"百慕達\",\"BT\":\"不丹\",\"BO\":\"玻利維亞\",\"BQ\":\"波奈\",\"BA\":\"波士尼亞與赫塞哥維納\",\"BW\":\"波札那\",\"BV\":\"布威島\",\"BR\":\"巴西\",\"IO\":\"英屬印度洋領地\",\"VG\":\"英屬維京群島\",\"BN\":\"汶萊\",\"BG\":\"保加利亞\",\"BF\":\"布吉納法索\",\"BI\":\"蒲隆地\",\"CV\":\"維德角\",\"KH\":\"柬埔寨\",\"CM\":\"喀麥隆\",\"CA\":\"加拿大\",\"KY\":\"開曼群島\",\"CF\":\"中非共和國\",\"TD\":\"查德\",\"CL\":\"智利\",\"CN\":\"中國\",\"CX\":\"聖誕島\",\"CC\":\"科克斯 (基靈) 群島\",\"CO\":\"哥倫比亞\",\"KM\":\"葛摩\",\"CG\":\"剛果\",\"CD\":\"\",\"CK\":\"庫克群島\",\"CR\":\"哥斯大黎加\",\"CI\":\"象牙海岸 (科特迪瓦)\",\"HR\":\"克羅埃西亞\",\"CU\":\"古巴\",\"CW\":\"庫拉索\",\"CY\":\"賽普勒斯\",\"CZ\":\"捷克共和國\",\"DK\":\"丹麥\",\"DJ\":\"吉布地\",\"DM\":\"多米尼克\",\"DO\":\"多明尼加共和國\",\"EC\":\"厄瓜多\",\"EG\":\"埃及\",\"SV\":\"薩爾瓦多\",\"GQ\":\"赤道幾內亞\",\"ER\":\"厄利垂亞\",\"EE\":\"愛沙尼亞\",\"ET\":\"衣索比亞\",\"FK\":\"福克蘭群島\",\"FO\":\"法羅群島\",\"FJ\":\"斐濟\",\"FI\":\"芬蘭\",\"FR\":\"法國\",\"GF\":\"法屬圭亞那\",\"PF\":\"法屬玻里尼西亞\",\"TF\":\"法屬南部屬地\",\"GA\":\"加彭\",\"GM\":\"甘比亞\",\"GE\":\"喬治亞\",\"DE\":\"德國\",\"GH\":\"迦納\",\"GI\":\"直布羅陀\",\"GR\":\"希臘\",\"GL\":\"格陵蘭\",\"GD\":\"格瑞那達\",\"GP\":\"瓜地洛普\",\"GU\":\"關島\",\"GT\":\"瓜地馬拉\",\"GG\":\"根息\",\"GN\":\"幾內亞\",\"GW\":\"幾內亞比索\",\"GY\":\"蓋亞那\",\"HT\":\"海地\",\"HM\":\"赫德島及麥唐納群島\",\"HN\":\"宏都拉斯\",\"HK\":\"香港特別行政區\",\"HU\":\"匈牙利\",\"IS\":\"冰島\",\"IN\":\"印度\",\"ID\":\"印尼\",\"IR\":\"伊朗\",\"IQ\":\"伊拉克\",\"IE\":\"愛爾蘭\",\"IM\":\"曼島\",\"IL\":\"以色列\",\"IT\":\"義大利\",\"JM\":\"牙買加\",\"JP\":\"日本\",\"JE\":\"澤西島\",\"JO\":\"約旦\",\"KZ\":\"哈薩克\",\"KE\":\"肯亞\",\"KI\":\"吉里巴斯\",\"KR\":\"南韓\",\"KW\":\"科威特\",\"KG\":\"吉爾吉斯\",\"LA\":\"寮國\",\"LV\":\"拉脫維亞\",\"LB\":\"黎巴嫩\",\"LS\":\"賴索托\",\"LR\":\"賴比瑞亞\",\"LY\":\"利比亞\",\"LI\":\"列支敦斯登\",\"LT\":\"立陶宛\",\"LU\":\"盧森堡\",\"MO\":\"澳門特別行政區\",\"MK\":\"馬其頓 (FYRO)\",\"MG\":\"馬達加斯加\",\"MW\":\"馬拉威\",\"MY\":\"馬來西亞\",\"MV\":\"馬爾地夫\",\"ML\":\"馬利\",\"MT\":\"馬爾他\",\"MH\":\"馬紹爾群島\",\"MQ\":\"馬丁尼克\",\"MR\":\"茅利塔尼亞\",\"MU\":\"模里西斯\",\"YT\":\"馬約特島\",\"MX\":\"墨西哥\",\"FM\":\"密克羅尼西亞\",\"MD\":\"摩爾多瓦\",\"MC\":\"摩納哥\",\"MN\":\"蒙古\",\"ME\":\"蒙特內哥羅\",\"MS\":\"蒙哲臘\",\"MA\":\"摩洛哥\",\"MZ\":\"莫三比克\",\"MM\":\"緬甸\",\"NA\":\"納米比亞\",\"NR\":\"諾魯\",\"NP\":\"尼泊爾\",\"NL\":\"荷蘭\",\"NC\":\"新喀里多尼亞\",\"NZ\":\"紐西蘭\",\"NI\":\"尼加拉瓜\",\"NE\":\"尼日\",\"NG\":\"奈及利亞\",\"NU\":\"紐埃島\",\"NF\":\"諾福克島\",\"KP\":\"北韓\",\"MP\":\"北馬利安納群島\",\"NO\":\"挪威\",\"OM\":\"阿曼\",\"PK\":\"巴基斯坦\",\"PW\":\"帛琉\",\"PS\":\"巴勒斯坦民族權力機構\",\"PA\":\"巴拿馬\",\"PG\":\"巴布亞紐幾內亞\",\"PY\":\"巴拉圭\",\"PE\":\"秘魯\",\"PH\":\"菲律賓\",\"PN\":\"皮特肯群島\",\"PL\":\"波蘭\",\"PT\":\"葡萄牙\",\"PR\":\"波多黎各\",\"QA\":\"卡達\",\"RE\":\"留尼旺\",\"RO\":\"羅馬尼亞\",\"RU\":\"俄羅斯\",\"RW\":\"盧安達\",\"BL\":\"聖巴瑟米\",\"KN\":\"聖克里斯多福及尼維斯\",\"LC\":\"聖露西亞\",\"MF\":\"法屬聖馬丁\",\"PM\":\"聖皮埃與密克隆群島\",\"VC\":\"聖文森及格瑞那丁\",\"WS\":\"薩摩亞\",\"SM\":\"聖馬利諾\",\"ST\":\"聖多美普林西比\",\"SA\":\"沙烏地阿拉伯\",\"SN\":\"塞內加爾\",\"RS\":\"塞爾維亞\",\"SC\":\"塞席爾\",\"SL\":\"獅子山\",\"SG\":\"新加坡\",\"SX\":\"荷屬聖馬丁\",\"SK\":\"斯洛伐克\",\"SI\":\"斯洛維尼亞\",\"SB\":\"索羅門群島\",\"SO\":\"索馬利亞\",\"ZA\":\"南非\",\"GS\":\"南喬治亞與南三明治群島\",\"SS\":\"南蘇丹\",\"ES\":\"西班牙\",\"LK\":\"斯里蘭卡\",\"SH\":\"聖赫勒拿、阿森松、特里斯坦達庫尼亞群島\",\"SD\":\"蘇丹\",\"SR\":\"蘇利南\",\"SJ\":\"冷岸\",\"SZ\":\"史瓦濟蘭\",\"SE\":\"瑞典\",\"CH\":\"瑞士\",\"SY\":\"敘利亞\",\"TW\":\"台灣\",\"TJ\":\"塔吉克\",\"TZ\":\"坦尚尼亞\",\"TH\":\"泰國\",\"TL\":\"東帝汶\",\"TG\":\"多哥\",\"TK\":\"托克勞群島\",\"TO\":\"東加\",\"TT\":\"千里達及托巴哥\",\"TN\":\"突尼西亞\",\"TR\":\"土耳其\",\"TM\":\"土庫曼\",\"TC\":\"土克斯及開科斯群島\",\"TV\":\"吐瓦魯\",\"UM\":\"美國本土外小島嶼\",\"VI\":\"美屬維京群島\",\"UG\":\"烏干達\",\"UA\":\"烏克蘭\",\"AE\":\"阿拉伯聯合大公國\",\"GB\":\"英國\",\"US\":\"美國\",\"UY\":\"烏拉圭\",\"UZ\":\"烏茲別克\",\"VU\":\"萬那杜\",\"VA\":\"梵蒂岡\",\"VE\":\"委內瑞拉\",\"VN\":\"越南\",\"WF\":\"瓦利斯群島和富圖那群島\",\"YE\":\"葉門\",\"ZM\":\"尚比亞\",\"ZW\":\"辛巴威\"} |
| **error_448** | 提供的手機號碼不通。 |
| **error_449** | 使用者已超過重試次數限制。 |
| **verification_code_input_placeholder_text** | 驗證碼 |

下列範例示範在 MFA 註冊頁面中使用某些使用者介面元素的方式：

![註冊頁面電子郵件驗證 UX 元素](./media/localization-string-ids/localization-mfa1.png)

下列範例示範在 MFA 驗證頁面中使用某些使用者介面元素的方式：

![註冊頁面電子郵件驗證 UX 元素](./media/localization-string-ids/localization-mfa2.png)







