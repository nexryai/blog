---
title: "Jenkinsでサーバーの監視をして幸せになる"
date: 2023-02-17
slug: "jenkins-server-monitoring"
description: "Jenkinsおじさんにサーバー監視をしてもらったら幸せになったというお話"
keywords: ["jenkins", "server", "selfhost"]
draft: false
tags: ["tech"]
math: false
toc: false
---


## 概要
Jenkinsおじさんにサーバー監視をしてもらったら幸せになったというお話

## PostgreSQLの監視
ここではJnekinsの構築方法については触れません。各自調べてください。ただし**Dockerを使う方法はおすすめしません。**<br>
今回はJenkinsおじさんにPostgreSQLのレプリケーションを監視してもらいます。

### 前提条件
pythonプラグインとdiscord通知プラグインが導入済み

### 指針
Jenkinsはジョブが異常終了（終了コードが0以外で終了）するとそのジョブを失敗したとマークします。  
適当にサーバーを監視しレプリケーションが止まっていると判断したら終了コード0以外で終了するスクリプト（ここではPython製）を書き、定期的に実行させ、失敗したらDiscordに通知を送るよう設定します。

### プロジェクトの作成
<img src="/images/img2-1.png"><br>
適当にジョブを作成します。「フリースタイルプロジェクトのビルド」を選択してください。

### ジョブの設定
プロジェクトを作成したら、そのプロジェクトの設定に飛び次の項目を設定します。

#### ビルド・トリガ 
<img src="/images/img2-2.png"><br>
ここでは15分に一回実行しています。

#### Build Steps 
言葉で説明するのが面倒なので画像で（  <br>
<img src="/images/img2-6.png"><br>
<img src="/images/img2-3.png"><br>

あくまでも例ですが中身のスクリプトは以下のように設定します。

```
import sys

import psycopg2
from psycopg2.extras import DictCursor


postgre_user_name = "USERNAME"
postgre_user_password = "PASSWORD"
postgre_server = "IP"
postgre_server_port = "PORT"
postgre_database_name = "DB_NAME"
postgre_table_name = "pg_stat_replication"

conn = psycopg2.connect("postgresql://"
                        + postgre_user_name
                        + ":" 
                        + postgre_user_password
                        + "@"
                        + postgre_server
                        + ":"
                        + postgre_server_port
                        + "/"
                        + postgre_database_name)
cur = conn.cursor(cursor_factory=DictCursor)
sql_sentence = "select pid from " + postgre_table_name + ";"
cur.execute(sql_sentence)
result = cur.fetchall()

try:
    print(f"PID: {result[0][0]}")

    if isinstance(result[0][0], int):
        sys.exit(0)
    else:
        raise

except Exception:
    sys.exit(1)
```

#### ビルド後の処理 
画面の指示に従ってDiscrod通知なりメール通知なりを追加します。

### 完了
設定を保存し、ジョブを実行します。ビルド履歴に成功したログが残れば上手く動いている証拠です。<br>
<img src="/images/img2-5.png"><br>

## まとめ
Jenkins、意外と汎用性が高いので今後も色々な場面で活用していきたいなと思ってます。
