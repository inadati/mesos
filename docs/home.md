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
* [HTTP エンドポイント](endpoints/) 利用可能なHTTPエンドポイント
* [APIクライアントライブラリ](api-client-libraries.md) HTTP APIのクライアントライブラリの一覧
* [APIのバージョニング](versioning.md#api-versioning) HTTP APIとリリースバージョニングについて
* [RecordIO](recordio.md) HTTP APIのストリーミング・エンドポイントで使用されるRecordIOフォーマットについて
* API References:
  * [v0 Java API](/api/latest/java/)
  * [v0 C++ API](/api/latest/c++/namespacemesos.html)
  * [v1 Operator HTTP API](operator-http-api.md) オペレーターとMesosマスター/エージェントの間のコミュニケーションについて
  * [v1 Scheduler HTTP API](scheduler-http-api.md) スケジューラーとMesosマスターの間の通信
  * [v1 Executor HTTP API](executor-http-api.md) エクゼキューターとMesosエージェント間の通信のための新しいHTTP APIについて
* [Mesosモジュール](modules.md) マスター、エージェント、およびテスト用のMesosモジュールを指定します。
  * [アロケーションモジュール](allocation-module.md) カスタム・リソース・アロケータの作成方法

## コミュニティ
* [参加するには](/community/)
* [Mesosユーザーの一覧](powered-by-mesos.md)
* [サードパーティのフレームワーク](frameworks.md)
* [サードパーティのツール](tools.md) 開発者とオペレーター向け
* [開発のロードマップ](roadmap.md)
* [設計資料](design-docs.md) 様々なMesosの機能の設計書のリストです。
* [ワーキンググループ](/community/#workinggroups) さまざまなコンポーネントに取り組んでいるグループのリスト

### 貢献
* [問題、修正、または新機能の要望](reporting-an-issue.md) JIRAを始めるために
* [コントリビューターへの入門](beginner-contribution.md) Mesosへの初めての貢献について
* [更に踏み込んだ貢献](advanced-contribution.md) Mesosに貢献する際の典型的なワークフロー
* [エンジニアリングの原則と実践](engineering-principles-and-practices.md) プロジェクトレベルの価値観をコミュニティで共有するため
* スタイルガイド:
  * [ドキュメントのスタイルガイド](documentation-guide.md)
  * [開発者ガイド](developer-guide.md) for best practices and patterns used in Mesos.
  * [C++ スタイルガイド](c++-style-guide.md)
    * [C言語フォーマット](clang-format.md) 自動整形
  * [Doxygenスタイルガイド](doxygen-style-guide.md)
  * [マークダウンスタイルガイド](markdown-style-guide.md)
* [テストパターン](testing-patterns.md) Mesosのテストで使われるヒントやコツの紹介
* [C++ Doxygen リファレンス](/api/latest/c++/) 内部API
* [メンテナーとコミッター](committers.md) プロジェクトのコミッターやコンポーネントのメンテナのリスト
  * [コミット](committing.md) 変更をコミットする際のガイドライン
  * [リリースガイド](release-guide.md)
  * [コミッター候補者のガイドライン](committer-candidate-guidelines.md) コミッターになるために
* [効果的なコードレビュー](effective-code-reviewing.md) 効果的なコードレビューを行うためのガイドライン、ヒント、学習事項
* [レビューの再開](reopening-reviews.md) レビューボードのレビューのreopenに関するポリシーについて

## Mesos関連書籍

<div class="row">
  <div class="col-xs-6 col-md-4">
    <a href="https://www.packtpub.com/big-data-and-business-intelligence/apache-mesos-cookbook" class="thumbnail">
      <img src="https://static.packt-cdn.com/products/9781785884627/cover/smaller" alt="Apache Mesos Cookbook by David Blomquist, Tomasz Janiszewski">
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
      <img src="https://learning.oreilly.com/library/cover/9781785886249/250w/" alt="Mastering Mesos by Dipa Dubhashi and Akhil Das">
    </a>
    <p class="text-center">Mastering Mesos by Dipa Dubhashi and Akhil Das (Packt, 2016)</p>
  </div>
  <div class="col-xs-6 col-md-4">
    <a href="https://www.packtpub.com/big-data-and-business-intelligence/apache-mesos-essentials" class="thumbnail">
      <img src="https://covers.zlibcdn2.com/covers299/books/7c/19/a3/7c19a3e3ccb83decdca2efe62d3297de.jpg" alt="Apache Mesos Essentials by Dharmesh Kakadia">
    </a>
    <p class="text-center">Apache Mesos Essentials by Dharmesh Kakadia (Packt, 2015)</p>
  </div>
  <div class="col-xs-6 col-md-4">
    <a href="http://shop.oreilly.com/product/0636920039952.do" class="thumbnail">
      <img src="https://books.google.com/books/publisher/content/images/frontcover/NLElCwAAQBAJ?fife=w200-h300" alt="Building Applications on Mesos by David Greenberg">
    </a>
    <p class="text-center">Building Applications on Mesos by David Greenberg (O'Reilly, 2015)</p>
  </div>
</div>
