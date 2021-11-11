---
title: Apache Mesos - Configuration
layout: documentation
---

# Mesosランタイムの設定

Mesosマスターとエージェントは、コマンドライン引数や環境変数でさまざまな設定オプションを取ることができます。利用可能なオプションの一覧は、`mesos-master -—help`または`mesos-agent -—help`を実行することで確認できます。各オプションは、次の2つの方法で設定できます。:

* `--option_name=value` を使って、値を直接指定するか、値が存在するファイルを指定して（`--option_name=file://path/to/file`）、バイナリに渡すことができます。パスは、現在の作業ディレクトリに対する絶対パスまたは相対パスです。

* 環境変数`MESOS_OPTION_NAME`（オプション名に`MESOS_`の接頭辞を付けたもの）を設定する。

設定値は、まず環境で検索され、次にコマンドラインで検索されます。

このドキュメントでは、Mesosのオプションの最近のスナップショットのみを掲載しています。お使いのバージョンのMesosがどのフラグをサポートしているかについての正確な情報は、`mesos-master -—help`のように`-—help`フラグを付けてバイナリを実行することで確認できます。

## マスターとエージェントのオプション

*これらは、Mesosマスターとエージェントの両方に共通するオプションです。*

See [configuration/master-and-agent.md](configuration/master-and-agent.md).

## マスターのオプション

See [configuration/master.md](configuration/master.md).

## エージェントのオプション

See [configuration/agent.md](configuration/agent.md).

## Libprocessのオプション

See [configuration/libprocess.md](configuration/libprocess.md).

# Mesosのビルド設定

## オートツールのオプション

特別なコンパイル要件がある場合は、Mesosを設定する際に`./configure —help`を参照してください。

See [configuration/autotools.md](configuration/autotools.md).

## CMakeのオプション

See [configuration/cmake.md](configuration/cmake.md).
