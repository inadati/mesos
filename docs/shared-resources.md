---
title: Apache Mesos - Shared Persistent Volumes
layout: documentation
---

# 共有パーシステントボリューム

## 概要
タスクが[パーシステントボリューム](persistent-volume.md)を使用して起動すると、他のタスクはそのボリュームを使用することができず、ボリュームは、それを使用しているタスクが終了するまで、どのリソースオファーにも表示されません。

場合によっては、同じエージェント上で実行されている複数のタスク間でボリュームを共有することが有用な場合があります。例えば、複数のデータ分析タスク間で大規模なデータセットを効率的に共有するために使用することができます。

## 共有ボリュームの作成
共有パーシステントボリュームは、通常のパーシステントボリュームと同じワークフローを使用して作成されます。[予約されたリソース](reservation.md)から開始し、フレームワークスケジューラAPIまたは[/creat-volumes](endpoints/master/create-volumes.md) HTTPエンドポイントを介して`CREATE`操作を適用します。共有ボリュームを作成するには、ボリューム作成時に `shared` フィールドを設定します。

たとえば、`"engineering"`ロールにサブスクライブしているフレームワークが、2048MBの動的に予約されたディスクを含むリソースオファーを受け取ったとします。

```
{
  "allocation_info": { "role": "engineering" },
  "id" : <offer_id>,
  "framework_id" : <framework_id>,
  "slave_id" : <slave_id>,
  "hostname" : <hostname>,
  "resources" : [
    {
      "allocation_info": { "role": "engineering" },
      "name" : "disk",
      "type" : "SCALAR",
      "scalar" : { "value" : 2048 },
      "role" : "engineering",
      "reservation" : {
        "principal" : <framework_principal>
      }
    }
  ]
}
```

フレームワークは、このディスクリソースを使用して、次のようなオファー操作で共有永続性ボリュームを作成できます。:

```
{
  "type" : Offer::Operation::CREATE,
  "create": {
    "volumes" : [
      {
        "allocation_info": { "role": "engineering" },
        "name" : "disk",
        "type" : "SCALAR",
        "scalar" : { "value" : 2048 },
        "role" : "engineering",
        "reservation" : {
          "principal" : <framework_principal>
        },
        "disk": {
          "persistence": {
            "id" : <persistent_volume_id>
          },
          "volume" : {
            "container_path" : <container_path>,
            "mode" : <mode>
          }
        },
        "shared" : {
        }
      }
    ]
  }
}
```
`shared`フィールドが（空のJSONオブジェクトに）設定されていることに注意してください。これは、`CREATE`オペレーションが共有ボリュームを作成することを示しています。

## 共有ボリュームの使用
共有ボリュームを含むリソースオファーを受け取る資格を得るためには、フレームワークがマスターに登録する際に提供する`FrameworkInfo`で`SHARED_RESOURCES`ケイパビリティを有効にする必要があります。このケイパビリティを有効にしていないフレームワークは、共有リソースを提供されません。

フレームワークがリソースの提供を受けた場合、`shared`フィールドが設定されているかどうかを確認することで、ボリュームが共有されているかどうかを判断できます。通常のパーシステント・ボリュームとは異なり、タスクによって使用されている共有ボリュームは、そのボリュームのロールにサブスクライブされているフレームワークに提供され続けます。フレームワークは、1つの`ACCEPT`コールを使用してボリュームにアクセスする複数のタスクを起動することもできます。

なお、Mesosでは、ボリュームを共有するタスク間の分離や同時実行の制御は行われません。フレームワークの開発者は、同じボリュームにアクセスするタスクが互いに競合しないようにする必要があります。これは、アプリケーションレベルの同時実行制御を慎重に行うか、タスクがボリュームに読み取り専用でアクセスするようにすることで実現できます。[Persistent Volume](persistent-volume.md)のドキュメントに記載されているように、ボリューム上で起動されるタスクは、`"RO"`モードを指定してボリュームを読み取り専用モードで使用できます。

### 共有ボリュームの破棄

共有されているかどうかにかかわらず、永続的なボリュームは、そのボリュームを使用して実行中または保留中のタスクが起動していない場合にのみ破壊することができます。非共有ボリュームの場合、ボリュームを削除しても安全なタイミングを判断するのは通常簡単です。共有ボリュームの場合、ボリュームを使用してタスクを起動したフレームワークは、通常、ボリュームを破壊する前に、ボリュームがもはや使用されていないことを（例えば、参照カウントによって）確認するために調整する必要があります。

### リソースアロケーション

TODO：共有ボリュームがリソース配分に与える影響は？

## 参考資料

* [MESOS-3421](https://issues.apache.org/jira/browse/MESOS-3421)には、この機能の実装に関する追加情報が記載されています。

* 2016年8月31日に開催された「MesosCon Europe 2016」でのトークの題名
  "[Practical Persistent Volumes](http://schd.ws/hosted_files/mesosconeu2016/08/MesosConEurope2016PPVv1.0.pdf)".
