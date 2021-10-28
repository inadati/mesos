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
  * [Resource Role Weights](weights.md) for fair sharing.
  * [Resource Role Quota](quota.md) for how to configure Mesos to provide guaranteed resource allocations for use by a role.
  * [Reservations](reservation.md) for how operators and frameworks can reserve resources on individual agents for use by a role.
  * [Shared Resources](shared-resources.md) for how to share persistent volumes between tasks managed by different executors on the same agent.
* [Oversubscription](oversubscription.md) for how to configure Mesos to take advantage of unused resources to launch "best-effort" tasks.

## Security
* [Authentication](authentication.md)
* [Authorization](authorization.md)
* [SSL](ssl.md)
* [Secrets](secrets.md) for managing secrets within Mesos.

## Containerization
* [Containerizer Overview](containerizers.md)
  * [Containerizer Internals](containerizer-internals.md) for implementation details of containerizers.
  * [Docker Containerizer](docker-containerizer.md) for launching a Docker image as a Task, or as an Executor.
  * [Mesos Containerizer](mesos-containerizer.md) default containerizer, supports both Linux and POSIX systems.
    * [Container Images](container-image.md) for supporting container images in Mesos containerizer.
    * [Docker Volume Support](isolators/docker-volume.md)
    * [Nvidia GPU Support](gpu-support.md) for how to run Mesos with Nvidia GPU support.
* [Container Sandboxes](sandbox.md)
* [Container Volumes](container-volume.md)
* [Nested Container and Task Group (Pod)](nested-container-and-task-group.md)
* [Standalone Containers](standalone-containers.md)

## Networking
* [Networking Overview](networking.md)
  * [Networking in Detail](networking-for-mesos-managed-containers.md)
  * [Container Network Interface (CNI)](cni.md)
  * [Port Mapping Isolator](isolators/network-port-mapping.md)

## Storage
* [Multiple Disks](multiple-disk.md) for how to allow tasks to use multiple isolated disk resources.
* [Persistent Volume](persistent-volume.md) for how to allow tasks to access persistent storage resources.
* [Container Storage Interface (CSI) Support](csi.md)

## Scheduler and Executor Development
* [Running Workloads in Mesos](running-workloads.md) explains how a scheduler can specify and run tasks.
* [Framework Development Guide](app-framework-development-guide.md) describes how to build applications on top of Mesos.
* [Guide for Designing Highly Available Mesos Frameworks](high-availability-framework-guide.md)
* [Reconciliation](reconciliation.md) for ensuring a framework's state remains eventually consistent in the face of failures.
* [Task State Reasons](task-state-reasons.md) describes how task state reasons are used in Mesos.
* [Task Health Checking](health-checks.md)
* [v1 Scheduler HTTP API](scheduler-http-api.md) for communication between schedulers and the Mesos master.
* [v1 Executor HTTP API](executor-http-api.md) describes the new HTTP API for communication between executors and the Mesos agent.

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
