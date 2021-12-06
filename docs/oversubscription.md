---
title: Apache Mesos - Oversubscription
layout: documentation
---

# オーバーサブスクリプション

優先度の高いユーザー向けサービスは、通常、ピーク時の負荷や予期せぬ負荷の急増に備えて大規模なクラスターにプロビジョニングされます。そのため、ほとんどの時間、プロビジョニングされたリソースは十分に活用されていません。オーバーサブスクリプションでは、一時的に使用されていないリソースを利用して、バックグラウンド分析、ビデオ/画像処理、チップシミュレーション、その他の低優先度のジョブなどのベストエフォート型のタスクを実行します。

## どのような仕組みになっているのですか？

オーバーサブスクリプションはMesos 0.23.0で導入され、既存のリソースアロケータ、リソースモニタ、Mesosエージェントの拡張に加えて、Resource EstimatorとQuality of Service (QoS) Controllerという2つの新しいエージェントコンポーネントが追加されました。新しいコンポーネントとその相互作用を以下に示します。

![Oversubscription overview](images/oversubscription-overview.jpg)

### リソースの推定
* (1) 最初のステップは、オーバーサブスクライブされたリソースの量を特定することです。リソース見積もり担当者は、リソースモニターを利用して、`ResourceStatistic`メッセージを介して使用状況の統計情報を定期的に取得します。resource estimatorは、収集したリソースの統計情報に基づいてロジックを適用し、オーバーサブスクライブされたリソースの量を決定する。これは、測定されたリソースの使用スラック（割り当てられたが未使用のリソース）と割り当てスラックに基づいた一連の制御アルゴリズムとすることができます。

* (2) エージェントは、リソース推定装置からの推定値をポーリングし続け、最新の推定値を追跡します。

* (3) 最新の推定値が前回の推定値と異なる場合、エージェントはマスターにオーバーサブスクライブされたリソースの合計量を送信します。

### リソーストラッキング＆スケジューリングアルゴリズム

* (4) 割り当て担当者は、オーバーサブスクライブされたリソースを通常のリソースとは別に記録し、それらのリソースを`revocable`なものとして注釈をつける。どのような種類のリソースをオーバーサブスクライブできるかは、リソース見積もり担当者に委ねられています。CPU シェアや帯域幅などの圧縮可能なリソースのみをオーバーサブスクライブすることを推奨する。

### フレームワーク
* (5)フレームワークは、通常の`launchTasks()`APIを使用して、取り消し可能なリソース上でタスクを起動することを選択できます。先取りに対処するように設計されていないフレームワークを保護するために、フレームワーク情報に`REVOCABLE_RESOURCES`（取り消し可能なリソース）ケイパビリティセットを登録したフレームワークだけが、取り消し可能なリソースのオファーを受け取ります。さらに、取り消し可能なリソースは動的に予約することができず、取り消し可能なディスクリソース上に永続的なボリュームを作成してはいけません。

### タスクの起動

* 取り消し可能なタスクは、エージェントで `runTask` リクエストを受信すると、通常通り起動します。取り消し可能なタスクと通常のタスクで特定のリソースを異なる方法で設定する必要がある場合、リソースは引き続き取り消し可能としてマークされ、アイソレータは適切なアクションを取ることができます。
> 注: タスクやエクゼキュータが使用するリソースが取り消し可能な場合、コンテナ全体が取り消し可能なコンテナとして扱われるため、QoS コントローラがキルやスロットルを行うことができます。

### 干渉の検出
* (6) 取り消し可能なタスクが実行されているときには、そのリソース上で実行されている元のタスクを常に監視し、SLAに基づいてパフォーマンスを保証することが重要である。検出された干渉に対応するために，QoS コントローラは，実行中の取り消し可能なタスクをキルまたはスロットルできる必要があります。

## オーバーサブスクライブのリソースを使用するフレームワークを可能にする
オーバーサブスクライブされたリソースの使用を計画しているフレームワークは、`REVOCABLE_RESOURCES`ケイパビリティセットに登録する必要があります。:

~~~{.cpp}
FrameworkInfo framework;
framework.set_name("Revocable framework");

framework.add_capabilities()->set_type(
    FrameworkInfo::Capability::REVOCABLE_RESOURCES);
~~~

この時点から、フレームワークは revocable のリソースをオファーで受け取るようになります。

>注: Mesosクラスタがオーバーサブスクリプションを有効にしていることは保証されません。そうでない場合は、取り消し可能なリソースはオファーされません。オーバーサブスクリプションのためにMesosを設定する方法については、以下を参照してください。

### `revocable`リソースを利用したタスクの起動
`revocable`リソースを使用したタスクの起動は、既存の`launchTasks` APIで行います。取り消し可能なリソースには、取り消し可能なフィールドが設定されます。以下に、通常のリソースと取り消し可能なリソースを使ったオファーの例を示します。

~~~{.json}
{
  "id": "20150618-112946-201330860-5050-2210-0000",
  "framework_id": "20141119-101031-201330860-5050-3757-0000",
  "agent_id": "20150618-112946-201330860-5050-2210-S1",
  "hostname": "foobar",
  "resources": [
    {
      "name": "cpus",
      "type": "SCALAR",
      "scalar": {
        "value": 2.0
      },
      "role": "*"
    }, {
      "name": "mem",
      "type": "SCALAR",
      "scalar": {
        "value": 512.0
      },
      "role": "*"
    },
    {
      "name": "cpus",
      "type": "SCALAR",
      "scalar": {
        "value": 0.45
      },
      "role": "*",
      "revocable": {}
    }
  ]
}
~~~

## カスタムリソースエスティメーターの作成
resource estimatorは、エージェントで使用される総リソースを推定・予測し、オーバーサブスクライブ可能なリソースをマスターに通知します。デフォルトでは、Mesosには`noop`と`fixed`のリソース推定器が付属しています。`noop` estimatorは、エージェントに空の見積もりを提供するだけで失速し、オーバーサブスクリプションを効果的に無効にします。`fixed` estimator は、実際に測定されたスラックを使用せず、固定のリソース量（コマンドラインフラグで定義）でノードをオーバーサブスクライブします。

インターフェースは以下のように定義されています。:

~~~{.cpp}
class ResourceEstimator
{
public:
  // Initializes this resource estimator. This method needs to be
  // called before any other member method is called. It registers
  // a callback in the resource estimator. The callback allows the
  // resource estimator to fetch the current resource usage for each
  // executor on agent.
  virtual Try<Nothing> initialize(
      const lambda::function<process::Future<ResourceUsage>()>& usage) = 0;

  // Returns the current estimation about the *maximum* amount of
  // resources that can be oversubscribed on the agent. A new
  // estimation will invalidate all the previously returned
  // estimations. The agent will be calling this method periodically
  // to forward it to the master. As a result, the estimator should
  // respond with an estimate every time this method is called.
  virtual process::Future<Resources> oversubscribable() = 0;
};
~~~

## カスタムQoSコントローラの作成

カスタムQoSコントローラを実装するためのインターフェースを以下に定義します。

~~~{.cpp}
class QoSController
{
public:
  // Initializes this QoS Controller. This method needs to be
  // called before any other member method is called. It registers
  // a callback in the QoS Controller. The callback allows the
  // QoS Controller to fetch the current resource usage for each
  // executor on agent.
  virtual Try<Nothing> initialize(
      const lambda::function<process::Future<ResourceUsage>()>& usage) = 0;

  // A QoS Controller informs the agent about corrections to carry
  // out, but returning futures to QoSCorrection objects. For more
  // information, please refer to mesos.proto.
  virtual process::Future<std::list<QoSCorrection>> corrections() = 0;
};
~~~

>注: QoS コントローラは `corrections()` をブロックしてはなりません。代わりに独自の libprocess アクタで QoS コントローラをバックアップします。

QoS コントローラは、特定の修正アクションを実行する必要があることをエージェントに通知します。各修正アクションには、実行者またはタスクに関する情報と、実行するアクションの種類が含まれます。

Mesosには、`noop`コントローラと`load` qosコントローラが付属しています。`noop`コントローラは、修正を行わないため、通常のタスクのサービス品質を保証しません。`load`コントローラは、システム全体の負荷が設定可能なしきい値を超えないようにし、その結果、ノード上のCPUの混雑を回避しようとします。負荷がしきい値を超えた場合、コントローラはすべてのリボカブル・エクゼキュータを退避させます。これらのしきい値は、2つのモジュールパラメータ `load_threshold_5min` と `load_threshold_15min` によって設定可能です。これらは、システムの標準的なunix負荷平均を表しています。1分間のシステム負荷は無視されます。なぜなら、オーバーサブスクリプションのユースケースでは、これは誤解を招く信号になる可能性があるからです。

~~~{.proto}
message QoSCorrection {
  enum Type {
    KILL = 1; // Terminate an executor.
  }

  message Kill {
    optional FrameworkID framework_id = 1;
    optional ExecutorID executor_id = 2;
  }

  required Type type = 1;
  optional Kill kill = 2;
}
~~~

## オーバーサブスクリプションの設定

エージェントに5つの新しいフラグが追加されました:

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        説明
      </th>
  </thead>

  <tr>
    <td>
      --oversubscribed_resources_interval=VALUE
    </td>
    <td>
      Tエージェントは、割り当てられていて利用可能なオーバーサブスクライブされたリソースの合計量に関する現在の推定値をマスターに定期的に更新します。更新の間隔は、このフラグによって制御されます。(デフォルト: 15秒)
    </td>
  </tr>

  <tr>
    <td>
      --qos_controller=VALUE
    </td>
    <td>
      オーバーサブスクリプションに使用するQoSコントローラーの名前
    </td>
  </tr>

  <tr>
    <td>
      --qos_correction_interval_min=VALUE
    </td>
    <td>
      エージェントは、実行中のタスクのパフォーマンスの観察結果に基づいて、QoS コントローラから QoS 補正をポーリングして実行します。これらの補正の最小間隔は、このフラグで制御します。(デフォルト: 0ns)
    </td>
  </tr>

  <tr>
    <td>
      --resource_estimator=VALUE
    </td>
    <td>
      オーバーサブスクリプションに使用するリソース・エスティメーターの名前です。
    </td>
  </tr>

</table>

`fixed`されたリソースの推定は、以下のように有効になります。

```
--resource_estimator="org_apache_mesos_FixedResourceEstimator"

--modules='{
  "libraries": {
    "file": "/usr/local/lib64/libfixed_resource_estimator.so",
    "modules": {
      "name": "org_apache_mesos_FixedResourceEstimator",
      "parameters": {
        "key": "resources",
        "value": "cpus:14"
      }
    }
  }
}'
```
上記の例では、固定量の14cpusがリボカブルリソースとして提供されます。

`load` qosコントローラを以下のように有効にします。

```
--qos_controller="org_apache_mesos_LoadQoSController"

--qos_correction_interval_min="20secs"

--modules='{
  "libraries": {
    "file": "/usr/local/lib64/libload_qos_controller.so",
    "modules": {
      "name": "org_apache_mesos_LoadQoSController",
      "parameters": [
        {
          "key": "load_threshold_5min",
          "value": "6"
        },
        {
	  "key": "load_threshold_15min",
	  "value": "4"
        }
      ]
    }
  }
}'
```
上記の例では、標準的なunixシステムのロードアベレージが5分間で6を超えた場合、または15分間で4を超えた場合、エージェントはすべての`revocable`エクゼキュータを退避させます。`LoadQoSController`は、効果的に20秒ごとに実行されます。

カスタムのリソース推定器と QoS コントローラをインストールするには、[モジュールのドキュメント](modules.md)を参照してください。
