---
title: Apache Mesos - Release and Support Policy
layout: documentation
---

# Mesosのリリースとサポートポリシー

Mesosのバージョニングとリリースポリシーは、運用者と開発者に以下の明確なガイドラインを与えています。

* 下位互換性に影響を与えることなく、既存のAPIに変更を加えること。
* APIのサポート期間
* Mesosのアップグレード

このドキュメントでは、Mesosの1.0.0以降のリリース戦略について説明します。


## リリーススケジュール

Mesosのリリースは時間ベースで行われますが、機能開発に合わせてリリーススケジュールを限定的に調整しています。 

これにより、ユーザーと開発者は、機能を使用したり制作したりするための予測可能なサイクルを得ることができ、同時に、各リリースにユーザーが待ち望んでいる開発成果を盛り込むことができます。

リリース時点で機能の準備ができていない場合、その機能は無効にすべきです。 

つまり、機能は、デフォルトではオプトインであり、簡単に無効化できるように開発すべきだということです。（例：フラグなど）

新しいMesosのリリースは、およそ**3ヶ月**ごとに行われます。バージョニング方式は[セマンティックバージョニング](http://semver.org)です。 通常、マイナーリリースのバージョンは、メジャーリリースでない限り、リリースごとに1ずつ増やされます。（例：1.1、1.2、1.3など）

すべての（マイナー）リリースは、安定したリリースであり、プロダクションでの使用を推奨しています。 これは、リリース候補が正式にリリースされる前に、厳格なテスト（ユニットテスト、統合テスト、ベンチマークテスト、クラスターテスト、スケーラビリティなど）を行うことを意味します。 通常のリリースが安定していないと判断された場合は、まれに安定させるためのパッチリリースがリリースされます。

常に、最新のリリースと2つ前のリリースの3つのリリースがサポートされています。 サポートとは、リリースに影響する**重要な問題**の修正を意味します。 問題が重要であると判断された場合、その問題は、**影響を受けた**リリースのうち、まだ**サポートされている**リリースのみで修正されます。 これをパッチリリースと呼び、パッチのバージョンを1ずつ増やしていきます。（例：1.2.1）あるリリースがサポート期間の終了になると、そのリリースに対するパッチリリースは行われなくなります。なお、これは後方互換性の保証や廃止期間（後述）とは関係ありません。

重要な問題の定義

* セキュリティに関する修正
* 互換性における問題
* 機能的な問題
* パフォーマンスの問題
* サードパーティの統合に関する修正（例：DockerリモートAPI)

問題が重要かどうかは、主観的である場合があります。 顕著な問題である場合もあれば、曖昧な場合もあります。 ユーザーはコミッターと協力して問題の重要性を把握し、サポートに対する同意とコミットの受け入れを行う必要があります。

パッチリリースは通常、**月に1回**程度行われます。

特定の問題がユーザーに影響を与えており、次の予定されたパッチリリースまで待てない場合、ユーザーは特定のサポートされたバージョンの予定外のパッチリリースを要求することができます。 開発者リストにメールでご連絡ください。

## アップグレード

すべての安定版リリースは、ゆるやかな互換性があります。ゆるやかな互換性とは:

* マスターやエージェントはエコシステム・コンポーネント（スケジューラー、エクゼキュータ、ズーキーパー、サービス・ディスカバリー・レイヤー、モニタリングなど）が非推奨の機能（非推奨のフラグや非推奨のメトリクスなど）に依存していない限り、新しいリリース・バージョンにアップグレードすることができます。
* 非推奨ではない、外部から見える動作に予期せぬ影響があってはならない。Mesos APIに期待されることについては、API互換性のセクションを参照してください。

> 注：互換性保証はまだモジュールには適用されません。詳しくは「モジュール」の項をご覧ください。


これは、ユーザーが（非推奨／削除された機能に依存していない限り）Mesosマスターまたはエージェントを、中間リリースバージョンを経由せずに、安定したリリースバージョンから別の安定したリリースバージョンに直接アップグレードできることを意味しています。 アップグレードの目的上、安定版リリースとは、最新のパッチバージョンが適用されたリリースを意味します。 たとえば、1.2.0、1.2.1、1.3.0、1.4.0、1.4.1のうち、1.2.1、1.3.0、1.4.1のリリースは安定していると考えられているので、ユーザーは1.2.1から1.4.1に直接アップグレードできるはずです。フレームワークがシームレスにアップグレードできる方法については、以下の「APIの互換性」のセクションをご覧ください。

機能の廃止期間は6ヶ月です。一定の期間を設けることで、Mesos開発者は技術的負債をいつまでも蓄積することなく、ユーザーはアップグレードを計画する時間を確保することができます。

特定のMesosバージョンへのアップグレードに関する詳細は[こちら](upgrade.md)に掲載されます。


## API バージョニング

Mesos API（Scheduler、Executor、Internal、Operator/Adminの各API）は、URLにバージョンが入ります。バージョンアップされたURLのプレフィックスは **`/api/vN`** となり、"N "はAPIのバージョンです。APIのリソースをWeb UIのパスと区別するために、"/api "というプレフィックスが選ばれています。

例:

* http://localhost:5050/api/v1/scheduler :  マスターが提供するスケジューラーのHTTP API
* http://localhost:5051/api/v1/executor  :  エージェントが提供するエグゼキューターのHTTP API

インストールするMesosによっては同じAPIの複数のバージョンをホストすることがあります。（Scheduler API v1やv2など）

### APIバージョンとリリースバージョンの対比

* 物事をシンプルにするため、APIの安定版はMesosのメジャーリリースバージョンに対応しています。
  * 例えば、APIのv1は、Mesosのリリースバージョン1.0.0、1.4.0、1.20.0などでサポートされます。
* vNバージョンのAPIは、N-1シリーズのリリースバージョンでもサポートされている可能性がありますが、vN APIはN-1シリーズの最後のリリースバージョンまでは安定していないと考えられています。
 *  たとえば、APIのv2はMesos 1.12.0リリースで導入されているかもしれませんが、「1」シリーズの最後のリリースである場合、Mesos 1.21.0リリースでのみ安定しているとみなされます。(常に、最新のリリースと2つ前のリリースの3つのリリースがサポートされるため)ただし、すべてのMesos 1.x.yバージョンでは、引き続きv1のAPIをサポートしています。
*  APIのバージョンが上がるのは、[後方互換性のないAPI](#api-compatibility)の修正が必要になった場合のみです。私たちは、所定のAPIのバージョンを少なくとも1年間はサポートするように努めます。
*  vN-1 APIの廃止の通知は、Mesosの「N.0.0」バージョンをリリースした時点で開始されます。vN-1 APIのサポートを終了する前に、フレームワークやオペレーターがvN APIにアップグレードするための十分な期間（例：6ヶ月）を設けるように努めます。

<a name="api-compatibility"></a>
### APIの互換性

APIの互換性は、対応するプロトコルバッファの保証によって決定されます。

例として、以下はスケジューラーAPIの「後方互換性」のある変更点と考えられます。:

* 新しいタイプのCall、即ち`/scheduler`への新しいタイプのHTTPリクエストの追加。
* 既存の`/scheduler`へのリクエストに新しいオプションフィールドを追加する。
* 新しいタイプのイベント、即ち`/scheduler`でストリームされる新しいタイプのチャンクの追加。
* `/scheduler`でストリームされるチャンクされたレスポンスに新しいヘッダーフィールドを追加する。
* `/scheduler`でストリームされるチャンクのボディに新しいフィールドを追加（またはフィールドの順序を変更）する。
* 新しいAPIリソースの追加。（例：`/foobar`）

以下は、スケジューラーAPIの後方互換性のない変更点と考えられます。:

* 既存の`/scheduler`へのリクエストに新しい必須項目を追加する。
* 既存のリクエストから`/scheduler`へのフィールドの名称変更/削除を行う。
* `/scheduler`でストリームされるチャンクからのフィールドの名称変更/削除。
* 既存のCallの名称変更/削除する。


## Implementation Details

### Release branches

For regular releases, the work is done on the master branch. There are no feature branches but there will be release branches.

When it is time to cut a minor release, a new branch (e.g., 1.2.x) is created off the master branch. We chose 'x' instead of patch release number to disambiguate branch names from tag names. Then the first RC (-rc1) is tagged on the release branch. Subsequent RCs, in case the previous RCs fail testing, should be tagged on the release branch.

Patch releases are also based off the release branches. Typically the fix for an issue that is affecting supported releases lands on the master branch and is then backported to the release branch(es). In rare cases, the fix might directly go into a release branch without landing on master (e.g.,  fix / issue is not applicable to master).

Having a branch for each minor release reduces the amount of work a release manager needs to do when it is time to do a release. It is the responsibility of the committer of a fix to commit it to all the affecting release branches. This is important because the committer has more context about the issue / fix at the time of the commit than a release manager at the time of release. The release manager of a minor release will be responsible for all its patch releases as well. Just like the master branch, history rewrites are not allowed in the release branch (i.e., no git push --force).


### API protobufs


Most APIs in Mesos accept protobuf messages with a corresponding JSON field mapping. To support multiple versions of the API, we decoupled the versioned protobufs backing the API from the "internal" protobufs used by the Mesos code.

For example, the protobufs for the v1 Scheduler API are located at:

```
include/mesos/v1/scheduler/scheduler.proto

package mesos.v1.scheduler;
option java_package = "org.apache.mesos.v1.scheduler";
option java_outer_classname = "Protos";
...
```

The corresponding internal protobufs for the Scheduler API are located at:

```
include/mesos/scheduler/scheduler.proto

package mesos.scheduler;
option java_package = "org.apache.mesos.scheduler";
option java_outer_classname = "Protos";
...
```

The users of the API send requests (and receive responses) based on the versioned protobufs. We implemented [evolve](https://github.com/apache/mesos/blob/master/src/internal/evolve.hpp)/[devolve](https://github.com/apache/mesos/blob/master/src/internal/devolve.hpp) converters that can convert protobufs from any supported version to the internal protobuf and vice versa.

Internally, message passing between various Mesos components would use the internal unversioned protobufs. When sending response (if any) back to the user of the API, the unversioned protobuf would be converted back to a versioned protobuf.
