---
title: Apache Mesos - Quota
layout: documentation
---

# クォータ

複数のユーザがクラスタを共有している場合、オペレータは各ユーザが使用できるリソースの数に制限を設けたい場合があります。Quotaはこのようなニーズに対応し、オペレータはロールごとに制限を設定することができます。

* [サポートされているリソース](#supported-resources)
* [クオータの更新](#updating-quotas)
* [クオータの表示](#viewing-quotas)
* [非推奨: クォータの保証](#deprecated-quota-guarantees)
* [実装上の注意](#implementation-notes)

## サポートされているリソース

以下のリソースはクォータをサポートしています。

* `cpus`
* `mem`
* `disk`
* `gpus`
* `SCALAR`タイプの任意のカスタムリソース

以下のリソースはクォータをサポートしていません。:

* `ports`
* `RANGES` または `SET`タイプの任意のカスタムリソース


## クオータの更新
デフォルトでは、すべてのロールにリソース制限はありません。1つまたは複数のロールのリソース制限を変更するには、v1 API `UPDATE_QUOTA`コールを使用します。このコールは、すべてか無かの方法で更新を適用することに注意してください。したがって、ロールのクォータ更新の1つが無効または未承認の場合は、要求全体が通過しません。

例:

```
curl --request POST \
     --url http://<master-ip>:<master-port>/api/v1/ \
     --header 'Content-Type: application/json' \
     --data '{
               "type": "UPDATE_QUOTA",
               "update_quota": {
                 "force": false,
                 "quota_configs": [
                   {
                     "role": "dev",
                     "limits": {
                       "cpus": { "value": 10 },
                       "mem":  { "value": 2048 },
                       "disk": { "value": 4096 }
                     }
                   },
                   {
                     "role": "test",
                     "limits": {
                       "cpus": { "value": 1 },
                       "mem":  { "value": 256 },
                       "disk": { "value": 512 }
                     }
                   }
                 ]
               }
             }'
```
* 現在のクォータ消費量が提供された制限値を超えている場合、リクエストは拒否されることに注意してください。このチェックは、`force`を`true`に設定することにより、上書きすることができます。
* マスターは、ロールがその制限を超えられないように、十分な数のオファーを取り消すことを試みますので注意してください。


## クオータの表示

### Web UI
Web UIの「Roles」タブには、すべての既知のロールのリソースアカウンティング情報が表示されます。これには、設定されたクォータとクォータの消費量が含まれます。

### API
クォータ関連の情報を見るためのエンドポイントがいくつかあります。

v1 APIの`GET_QUOTA`コールは、クォータの設定を返します。:

```
$ curl --request POST \
     --url http://<master-ip>:<master-port>/api/v1/ \
     --header 'Content-Type: application/json' \
     --header 'Accept: application/json' \
     --data '{ "type": "GET_QUOTA" }'
```

レスポンス:

```
{
  "type": "GET_QUOTA",
  "get_quota": {
    "status": {
      "infos": [
        {
          "configs" : [
            {
              "role": "dev",
              "limits": {
                "cpus": { "value": 10.0 },
                "mem":  { "value": 2048.0 },
                "disk": { "value": 4096.0 }
              }
            },
            {
              "role": "test",
              "limits": {
                "cpus": { "value": 1.0 },
                "mem":  { "value": 256.0 },
                "disk": { "value": 512.0 }
              }
            }
          ]
        }
      ]
    }
  }
}
```
クオータの消費量を表示するには、`/roles`エンドポイントを使用します。:

```
$ curl http://<master-ip>:<master-port>/roles
```

レスポンス

```
{
  "roles": [
    {
      "name": "dev",
      "weight": 1.0,
      "quota":
      {
        "role": "dev",
        "limit": {
          "cpus": 10.0,
          "mem":  2048.0,
          "disk": 4096.0
        },
        "consumed": {
          "cpus": 2.0,
          "mem":  1024.0,
          "disk": 2048.0
        }
      },
      "allocated": {
        "cpus": 2.0,
        "mem":  1024.0,
        "disk": 2048.0
      },
      "offered": {},
      "reserved": {
        "cpus": 2.0,
        "mem":  1024.0,
        "disk": 2048.0
      },
      "frameworks": []
    }
  ]
}
```

### クォータ消費

ロールのクォータ消費量は、割り当てと予約で構成されています。言い換えれば、予約が割り当てられていなくても、クォータ消費量に含まれます。提供されたリソースはクォータに対して課金されません。

### メトリクス

quotaでは、以下のメトリクスキーが公開されます。:

* `allocator/mesos/quota/roles/<role>/resources/<resource>/guarantee`
* `allocator/mesos/quota/roles/<role>/resources/<resource>/limit`
* クオータ消費量の指標は、次の方法で追加されます。[MESOS-9123](https://issues.apache.org/jira/browse/MESOS-9123)


## 非推奨: クォータの保証
Mesos 1.9より前のクォータ関連APIでは、クォータの「保証」のみが公開されており、ロールが利用できるリソースの最低量を保証していました。保証を設定すると、暗黙のクォータ制限も設定されます。Mesos 1.9+では、クォータ制限は上記のドキュメントに従って直接公開されるようになりました。

クォータ保証は非推奨となり、クォータ制限のみを使用するようになりました。クォータ保証を実施するには、未充足のクォータ保証をすべて満たすために、Mesosが十分なリソースを保持する必要がありました。Mesosは楽観的オファーモデルに移行しているため（マルチロール／マルチスケジューラのスケーラビリティを向上させるため、[MESOS-1607](https://issues.apache.org/jira/browse/MESOS-1607)を参照）、リソースを保留してクォータ保証を実施することはできなくなります。このようなモデルでは、クォータ制限は簡単に実施できますが、クォータ保証では、未充足の保証のためのスペースを確保するために、複雑な「有効な制限」の伝搬モデルが必要になります。

これらの理由から、クォータ保証はMesos 1.9ではまだ機能していますが、現在は非推奨となっています。楽観的オファーモデルでは、制限と優先度ベースの先取りの組み合わせがよりシンプルになります。

クォータ保証に関するドキュメントは、以前のドキュメントを参照してください： [https://github.com/apache/mesos/blob/1.8.0/docs/quota.md](https://github.com/apache/mesos/blob/1.8.0/docs/quota.md)

## 実装上の注意点
* クオータはネストしたロール（例：`eng/prod`）ではサポートされていません。
* フェイルオーバー時には、制限を正しく適用するために、アロケータは一時停止し、少なくとも80％のエージェントが再登録するか、10分が経過するまでオファーを発行しません。これらのパラメータは設定可能になる予定です。[MESOS-4073](https://issues.apache.org/jira/browse/MESOS-4073)
* クオータは現在、デフォルトのロールでサポートされています。[MESOS-3938](https://issues.apache.org/jira/browse/MESOS-3938)
