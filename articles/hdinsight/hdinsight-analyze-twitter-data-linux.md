---
title: 使用 Apache Hive 分析 Twitter 資料 - Azure HDInsight
description: 了解如何使用 HDInsight 上的 Hive 與 Hadoop，將原始 Twitter 資料轉換成可搜尋的 Hive 資料表。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 06/26/2018
ms.author: jasonh
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.openlocfilehash: a20f9ef6e42027cf3f499654ac8a43eee7b41854
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43104073"
---
# <a name="analyze-twitter-data-using-hive-and-hadoop-on-hdinsight"></a>在 HDInsight 上使用 Hive 與 Hadoop 分析 Twitter 資料

了解如何使用 Apache Hive 來處理 Twitter 資料。 結果是一份傳送了最多包含特定文字之推文的 Twitter 使用者清單。

> [!IMPORTANT]
> 本文件中的步驟已在 HDInsight 3.6 上進行過測試。
>
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="get-the-data"></a>取得資料

Twitter 可讓您透過 REST API 抓取每則推文資料，作為 JavaScript 物件標記法 (JSON)。 [OAuth](http://oauth.net) (英文)。

### <a name="create-a-twitter-application"></a>建立 Twitter 應用程式

1. 從網頁瀏覽器登入 [https://apps.twitter.com/](https://apps.twitter.com/)。 如果您沒有 Twitter 帳戶，請按一下 [立即註冊] 連結。

2. 按一下 [建立新的應用程式] 。

3. 輸入 [名稱]、[說明]、[網站]。 您可以在 [網站] 欄位中自行設定 URL。 下表列出部分要使用的範例值：

   | 欄位 | 值 |
   |:--- |:--- |
   | 名稱 |MyHDInsightApp |
   | 說明 |MyHDInsightApp |
   | 網站 |http://www.myhdinsightapp.com |

4. 核取 [是，我同意] 然後按一下 [建立 Twitter 應用程式]。

5. 按一下 [權限]  索引標籤。預設權限為 [唯讀] 。

6. 按一下 **[金鑰和存取權杖** ] 索引標籤。

7. 按一下 [Create my access token] 。

8. 按一下位於頁面右上角的 [測試 OAuth]  。

9. 記下**消費者金鑰**、**消費者祕密**、**存取權杖**和**存取權杖祕密**。

### <a name="download-tweets"></a>下載的推文

下列 Python 程式碼會從 Twitter 下載 10,000 則推文，並儲存到名為 **tweets.txt**的檔案。

> [!NOTE]
> 由於已安裝 Python，下列步驟會在 HDInsight 叢集上執行。

1. 使用 SSH 連線到 HDInsight 叢集

    ```bash
    ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    ```

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

3. 使用下列命令來安裝 [Tweepy (英文)](http://www.tweepy.org/)、[Progressbar (英文)](https://pypi.python.org/pypi/progressbar/2.2) 和其他必要的套件：

   ```bash
   sudo apt install python-dev libffi-dev libssl-dev
   sudo apt remove python-openssl
   pip install virtualenv
   mkdir gettweets
   cd gettweets
   virtualenv gettweets
   source gettweets/bin/activate
   pip install tweepy progressbar pyOpenSSL requests[security]
   ```

4. 使用以下命令建立名為 **gettweets.py** 的檔案：

   ```bash
   nano gettweets.py
   ```

5. 使用以下文字作為 **gettweets.py** 檔案的內容：

   ```python
   #!/usr/bin/python

   from tweepy import Stream, OAuthHandler
   from tweepy.streaming import StreamListener
   from progressbar import ProgressBar, Percentage, Bar
   import json
   import sys

   #Twitter app information
   consumer_secret='Your consumer secret'
   consumer_key='Your consumer key'
   access_token='Your access token'
   access_token_secret='Your access token secret'

   #The number of tweets we want to get
   max_tweets=10000

   #Create the listener class that receives and saves tweets
   class listener(StreamListener):
       #On init, set the counter to zero and create a progress bar
       def __init__(self, api=None):
           self.num_tweets = 0
           self.pbar = ProgressBar(widgets=[Percentage(), Bar()], maxval=max_tweets).start()

       #When data is received, do this
       def on_data(self, data):
           #Append the tweet to the 'tweets.txt' file
           with open('tweets.txt', 'a') as tweet_file:
               tweet_file.write(data)
               #Increment the number of tweets
               self.num_tweets += 1
               #Check to see if we have hit max_tweets and exit if so
               if self.num_tweets >= max_tweets:
                   self.pbar.finish()
                   sys.exit(0)
               else:
                   #increment the progress bar
                   self.pbar.update(self.num_tweets)
           return True

       #Handle any errors that may occur
       def on_error(self, status):
           print status

   #Get the OAuth token
   auth = OAuthHandler(consumer_key, consumer_secret)
   auth.set_access_token(access_token, access_token_secret)
   #Use the listener class for stream processing
   twitterStream = Stream(auth, listener())
   #Filter for these topics
   twitterStream.filter(track=["azure","cloud","hdinsight"])
   ```

    > [!IMPORTANT]
    > 將下列項目的預留位置文字取代為您 twitter 應用程式中的資訊：
    >
    > * `consumer_secret`
    > * `consumer_key`
    > * `access_token`
    > * `access_token_secret`

    > [!TIP]
    > 調整最後一行上的主題篩選條件，以追蹤熱門關鍵字。 使用您執行指令碼當時熱門的關鍵字，可以更快擷取資料。

6. 依序按 **Ctrl + X**，然後 **Y** 儲存檔案。

7. 使用以下命令執行檔案，並下載推文：

    ```bash
    python gettweets.py
    ```

    進度指示器隨即出現。 隨著推文的下載，其進度會推進到 100%。

   > [!NOTE]
   > 如果需要花費很長的時間來讓進度列往前移動，則您應該變更篩選來追蹤趨勢主題。 當您的篩選中有許多關於該主題的推文時，您就能快速取得所需的 10000 則推文。

### <a name="upload-the-data"></a>上傳資料

若要將資料下載到 HDInsight 儲存體，請使用下列命令：

```bash
hdfs dfs -mkdir -p /tutorials/twitter/data
hdfs dfs -put tweets.txt /tutorials/twitter/data/tweets.txt
```

這些命令會將資料儲存在叢集中的所有節點都能存取的位置。

## <a name="run-the-hiveql-job"></a>執行 HiveQL 工作

1. 使用以下命令建立包含 HiveQL 陳述式的檔案：

   ```bash
   nano twitter.hql
   ```

    使用下列文字做為檔案的內容：

   ```hiveql
   set hive.exec.dynamic.partition = true;
   set hive.exec.dynamic.partition.mode = nonstrict;
   -- Drop table, if it exists
   DROP TABLE tweets_raw;
   -- Create it, pointing toward the tweets logged from Twitter
   CREATE EXTERNAL TABLE tweets_raw (
       json_response STRING
   )
   STORED AS TEXTFILE LOCATION '/tutorials/twitter/data';
   -- Drop and recreate the destination table
   DROP TABLE tweets;
   CREATE TABLE tweets
   (
       id BIGINT,
       created_at STRING,
       created_at_date STRING,
       created_at_year STRING,
       created_at_month STRING,
       created_at_day STRING,
       created_at_time STRING,
       in_reply_to_user_id_str STRING,
       text STRING,
       contributors STRING,
       retweeted STRING,
       truncated STRING,
       coordinates STRING,
       source STRING,
       retweet_count INT,
       url STRING,
       hashtags array<STRING>,
       user_mentions array<STRING>,
       first_hashtag STRING,
       first_user_mention STRING,
       screen_name STRING,
       name STRING,
       followers_count INT,
       listed_count INT,
       friends_count INT,
       lang STRING,
       user_location STRING,
       time_zone STRING,
       profile_image_url STRING,
       json_response STRING
   );
   -- Select tweets from the imported data, parse the JSON,
   -- and insert into the tweets table
   FROM tweets_raw
   INSERT OVERWRITE TABLE tweets
   SELECT
       cast(get_json_object(json_response, '$.id_str') as BIGINT),
       get_json_object(json_response, '$.created_at'),
       concat(substr (get_json_object(json_response, '$.created_at'),1,10),' ',
       substr (get_json_object(json_response, '$.created_at'),27,4)),
       substr (get_json_object(json_response, '$.created_at'),27,4),
       case substr (get_json_object(json_response,    '$.created_at'),5,3)
           when "Jan" then "01"
           when "Feb" then "02"
           when "Mar" then "03"
           when "Apr" then "04"
           when "May" then "05"
           when "Jun" then "06"
           when "Jul" then "07"
           when "Aug" then "08"
           when "Sep" then "09"
           when "Oct" then "10"
           when "Nov" then "11"
           when "Dec" then "12" end,
       substr (get_json_object(json_response, '$.created_at'),9,2),
       substr (get_json_object(json_response, '$.created_at'),12,8),
       get_json_object(json_response, '$.in_reply_to_user_id_str'),
       get_json_object(json_response, '$.text'),
       get_json_object(json_response, '$.contributors'),
       get_json_object(json_response, '$.retweeted'),
       get_json_object(json_response, '$.truncated'),
       get_json_object(json_response, '$.coordinates'),
       get_json_object(json_response, '$.source'),
       cast (get_json_object(json_response, '$.retweet_count') as INT),
       get_json_object(json_response, '$.entities.display_url'),
       array(
           trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[1].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[2].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[3].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[4].text')))),
       array(
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[1].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[2].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[3].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[4].screen_name')))),
       trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
       trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
       get_json_object(json_response, '$.user.screen_name'),
       get_json_object(json_response, '$.user.name'),
       cast (get_json_object(json_response, '$.user.followers_count') as INT),
       cast (get_json_object(json_response, '$.user.listed_count') as INT),
       cast (get_json_object(json_response, '$.user.friends_count') as INT),
       get_json_object(json_response, '$.user.lang'),
       get_json_object(json_response, '$.user.location'),
       get_json_object(json_response, '$.user.time_zone'),
       get_json_object(json_response, '$.user.profile_image_url'),
       json_response
   WHERE (length(json_response) > 500);
   ```

2. 依序按 **Ctrl + X**，然後 **Y** 儲存檔案。
3. 使用以下命令執行包含於檔案中的 HiveQL：

   ```bash
   beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -i twitter.hql
   ```

    此命令會執行 **twitter.hql** 檔案。 當查詢完成時，您會看到 `jdbc:hive2//localhost:10001/>` 提示字元。

4. 從 beeline 提示字元中，使用下列查詢來確認資料已匯入︰

   ```hiveql
   SELECT name, screen_name, count(1) as cc
   FROM tweets
   WHERE text like "%Azure%"
   GROUP BY name,screen_name
   ORDER BY cc DESC LIMIT 10;
   ```

    這個查詢會傳回最多 10 則訊息內容中包含文字 **Azure** 的推文。

    > [!NOTE]
    > 如果您變更了 `gettweets.py` 指令碼中的篩選條件，請將 **Azure** 取代為您使用的其中一個篩選條件。

## <a name="next-steps"></a>後續步驟

您已經學會如何將非結構化的 JSON 資料集轉換成結構化的 Hive 資料表。 若要深入了解 HDInsight 上的 Hive，請參閱下列文件：

* [開始使用 HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [使用 HDInsight 分析航班延誤資料](hdinsight-analyze-flight-delay-data-linux.md)

[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[apache-hive-tutorial]: https://cwiki.apache.org/confluence/display/Hive/Tutorial

[twitter-statuses-filter]: https://dev.twitter.com/docs/api/1.1/post/statuses/filter
