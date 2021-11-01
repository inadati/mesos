---
title: Apache Mesos - Documentation Home
layout: documentation
---

## 基本編
* [Mesos アーキテクチャ](architecture.md) Mesosのコンセプトの概要
* [Mesosプレゼンテーションの動画とスライド](presentations.md)
* [学術論文とプロジェクトの履歴](https://www.usenix.org/conference/nsdi11/mesos-platform-fine-grained-resource-sharing-data-center)
* [Mesosのリリースとサポートポリシー](versioning.md)

## ビルド / インストール
* [ビルド](building.md) Mesosのコンパイルとインストールの基本的な手順
* [バイナリーパッケージ](binary-packages.md) Mesosのバイナリーパッケージの使用方法
* [設定](configuration.md) ビルド設定オプションを説明しています。
* [CMake](cmake.md) 新しいCMakeシステムの仕様についての詳細
* [Windowsサポート](windows.md) MesosでのWindowsサポートの状況

## 運用管理
* [設定](configuration.md) コマンドラインの引数の設定
* [高可用マスターのセットアップ](high-availability.md)
  * [ログのレプリケーション](replicated-log-internals.md) Mesosのレプリケーションされたログにおける情報
* [耐障害性エージェントのセットアップ](agent-recovery.md)
* [フレームワークの速度制限](framework-rate-limiting.md)
* [メンテナンス](maintenance.md) Mesosクラスターのメンテナンスのために
* [アップグレード](upgrades.md) Mesosクラスターのアップグレード
* [ダウングレード](downgrades.md) Mesosクラスターのダウングレード
* [ロギング](logging.md)
* [モニタリング / メトリクス](monitoring.md)
* [新しいCLIを使ったデバッグ](cli.md)
* [運用ガイド](operational-guide.md)
* [フェッチャーのキャッシュ設定](fetcher.md)
* [障害領域](fault-domains.md)
* [パフォーマンスのデバッグ](performance-profiling.md) Mesosにおけるパフォーマンスの問題のデバッグ
* [メモリのデバッグ](memory-profiling.md) Mesosにおける潜在的なメモリリークのデバッグ

## リソース管理
* [属性とリソース](attributes-resources.md) クラスターを構成するエージェントの説明
* [リソースロールの使用](roles.md)
  * [リソースロールウェイト](weights.md) 適切なリソース共有
  * [リソースロールクオータ](quota.md) リソースロールにクオータを設けるようにMesosを設定する方法
  * [予約](reservation.md) オペレータやフレームワークが、役割に応じて個々のエージェントのリソースを予約する方法
  * [共有リソース](shared-resources.md) 同一エージェント上の異なるエクゼキュータが管理するタスク間で永続ボリュームを共有する方法
* [過剰供給](oversubscription.md) 未使用のリソースを駆使して最善のタスクを起動するようMesosを設定する方法

## セキュリティ
* [認証](authentication.md)
* [認可](authorization.md)
* [SSL](ssl.md)
* [Secrets](secrets.md) Mesos内で秘密情報を管理するためのものです。

## コンテナ化
* [コンテナライザーの概要](containerizers.md)
  * [コンテナライザーの内部](containerizer-internals.md) コンテナライザーの実装の詳細
  * [Dockerコンテナライザー](docker-containerizer.md) タスクやエクゼキュータとしてDockerイメージを起動する
  * [Mesosコンテナライザー](mesos-containerizer.md) デフォルトのコンテナライザーでLinuxとPOSIXシステムの両方をサポートしています。
    * [コンテナイメージ](container-image.md) Mesosコンテナライザーでコンテナイメージをサポートする
    * [Dockerボリュームサポート](isolators/docker-volume.md)
    * [Nvidia GPUサポート](gpu-support.md) Nvidia GPUをサポートしたMesosを実行する方法
* [コンテナSandbox](sandbox.md)
* [コンテナボリューム](container-volume.md)
* [ネストされたコンテナとタスクグループ(Pod)](nested-container-and-task-group.md)
* [独立型コンテナ](standalone-containers.md)

## ネットワーク
* [ネットワーク概要](networking.md)
  * [ネットワークの詳細](networking-for-mesos-managed-containers.md)
  * [コンテナ・ネットワーク・インターフェイス (CNI)](cni.md)
  * [ポート・マッピング・アイソレータ](isolators/network-port-mapping.md)

## ストレージ
* [マルチディスク](multiple-disk.md) タスクが複数の独立したディスクリソースを使用できるようにする方法
* [永続ボリューム](persistent-volume.md) タスクが永続的なストレージにアクセスできるようにする方法
* [コンテナ・ストレージ・インターフェイス(CSI) サポート](csi.md)

## スケジューラーとエグゼキューターの開発
* [Mesosでのワークロードの実行](running-workloads.md) スケジューラーがタスクを指定して実行する方法
* [フレームワーク開発ガイド](app-framework-development-guide.md) Mesos上でアプリケーションを構築する方法
* [高可用性Mesosフレームワークを設計するためのガイド](high-availability-framework-guide.md)
* [整合性](reconciliation.md) フレームワークの障害時における整合性の保証
* [タスクのステートリーズン](task-state-reasons.md) Mesosにおけるタスクのステートリーズンの使用方法
* [タスクのヘルスチェック](health-checks.md)
* [v1 Scheduler HTTP API](scheduler-http-api.md) スケジューラーとMesosマスター間の為のAPI
* [v1 Executor HTTP API](executor-http-api.md) エクゼキュータとMesosエージェント間の通信のための新しいHTTP API

## APIs
* [HTTP Endpoints](endpoints/) for available HTTP endpoints.
* [API Client Libraries](api-client-libraries.md) lists client libraries for the HTTP APIs.
* [API Versioning](versioning.md#api-versioning) describes HTTP API and release versioning.
* [RecordIO](recordio.md) describes the RecordIO format used by the streaming endpoints of the HTTP API.
* API References:
  * [v0 Java API](/api/latest/java/)
  * [v0 C++ API](/api/latest/c++/namespacemesos.html)
  * [v1 Operator HTTP API](operator-http-api.md) for communication between operators and Mesos master/agent.
  * [v1 Scheduler HTTP API](scheduler-http-api.md) for communication between schedulers and the Mesos master.
  * [v1 Executor HTTP API](executor-http-api.md) describes the new HTTP API for communication between executors and the Mesos agent.
* [Mesos Modules](modules.md) for specifying Mesos modules for master, agent and tests.
  * [Allocation Module](allocation-module.md) for how to write custom resource allocators.

## Community
* [Getting Involved](/community/)
* [List of Mesos Users](powered-by-mesos.md)
* [3rd Party Frameworks](frameworks.md)
* [3rd Party Tools](tools.md) for developers and operators.
* [Development Roadmap](roadmap.md)
* [Design Docs](design-docs.md) list of design documents for various Mesos features.
* [Working groups](/community/#workinggroups) a listing of groups working on different components.

### Contributing
* [Reporting an Issue, Improvement, or Feature](reporting-an-issue.md) for getting started with JIRA.
* [Beginner Guide for Contributors](beginner-contribution.md) to get started contributing to Mesos for the first time.
* [Advanced Contribution Guide](advanced-contribution.md) to learn the typical workflow used when contributing to Mesos.
* [Engineering Principles and Practices](engineering-principles-and-practices.md) to serve as a shared set of project-level values for the community.
* Style Guides:
  * [Documentation Style Guide](documentation-guide.md)
  * [Developer Guide](developer-guide.md) for best practices and patterns used in Mesos.
  * [C++ Style Guide](c++-style-guide.md)
    * [Clang-Format](clang-format.md) for automatic formatting.
  * [Doxygen Style Guide](doxygen-style-guide.md)
  * [Markdown Style Guide](markdown-style-guide.md)
* [Testing Patterns](testing-patterns.md) for tips and tricks used in Mesos tests.
* [C++ Doxygen Reference](/api/latest/c++/) for internal APIs.
* [Committers and Maintainers](committers.md) a listing of project committers and component maintainers; useful when seeking feedback.
  * [Committing](committing.md) guidelines for committing changes.
  * [Release Guide](release-guide.md)
  * [Committer Candidate Guidelines](committer-candidate-guidelines.md) for becoming a committer.
* [Effective Code Reviewing](effective-code-reviewing.md) guidelines, tips, and learnings for how to do effective code reviews.
* [Reopening a Review](reopening-reviews.md) for our policy around reviving reviews on ReviewBoard.

## Books on Mesos

<div class="row">
  <div class="col-xs-6 col-md-4">
    <a href="https://www.packtpub.com/big-data-and-business-intelligence/apache-mesos-cookbook" class="thumbnail">
      <img src="https://d255esdrn735hr.cloudfront.net/sites/default/files/imagecache/ppv4_main_book_cover/9781785884627.png" alt="Apache Mesos Cookbook by David Blomquist, Tomasz Janiszewski">
    </a>
    <p class="text-center">Apache Mesos Cookbook by David Blomquist, Tomasz Janiszewski (Packt, August 2017)</p>
  </div>
  <div class="col-xs-6 col-md-4">
    <a href="https://www.manning.com/books/mesos-in-action" class="thumbnail">
      <img src="https://images.manning.com/255/340/resize/book/d/62f5c9b-0946-4569-ad50-ffdb84876ddc/Ignazio-Mesos-HI.png" alt="Mesos in Action by Roger Ignazio">
    </a>
    <p class="text-center">Mesos in Action by Roger Ignazio (Manning, 2016)</p>
  </div>
  <div class="col-xs-6 col-md-4">
    <a href="https://www.packtpub.com/big-data-and-business-intelligence/mastering-mesos" class="thumbnail">
      <img src="https://www.packtpub.com/sites/default/files/6249OS_5186%20Mastering%20Mesos.jpg" alt="Mastering Mesos by Dipa Dubhashi and Akhil Das">
    </a>
    <p class="text-center">Mastering Mesos by Dipa Dubhashi and Akhil Das (Packt, 2016)</p>
  </div>
  <div class="col-xs-6 col-md-4">
    <a href="https://www.packtpub.com/big-data-and-business-intelligence/apache-mesos-essentials" class="thumbnail">
      <img src="https://www.packtpub.com/sites/default/files/9781783288762.png" alt="Apache Mesos Essentials by Dharmesh Kakadia">
    </a>
    <p class="text-center">Apache Mesos Essentials by Dharmesh Kakadia (Packt, 2015)</p>
  </div>
  <div class="col-xs-6 col-md-4">
    <a href="http://shop.oreilly.com/product/0636920039952.do" class="thumbnail">
      <img src="http://akamaicovers.oreilly.com/images/0636920039952/lrg.jpg" alt="Building Applications on Mesos by David Greenberg">
    </a>
    <p class="text-center">Building Applications on Mesos by David Greenberg (O'Reilly, 2015)</p>
  </div>
</div>
