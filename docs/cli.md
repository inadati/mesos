---
title: Apache Mesos - CLI
layout: documentation
---

# 新しいCLI

新しいMesos Command Line Interfaceは、1つの実行可能なPython 3スクリプトで、すべてのデフォルトコマンドと追加のカスタムプラグインを実行できます。

利用可能なサブコマンドのうち2つは、実行中のコンテナをデバッグすることができます。:

* `mesos task exec`: 実行中のタスクのコンテナ内でコマンドを実行します。
* `mesos task attach`: 実行中のタスクにローカル端末を接続し、入出力をストリームする。

## CLIのビルド
今のところ、Mesos CLIはまだ開発中であり、標準的なMesosディストリビューションの一部としてビルドされていません。

しかし、CLIは[Autotools](configuration/autotools.md)と[Cmakeのオプション](configuration/cmake.md)を使ってビルドすることができます。必要であれば、ビルドを開始する前にPython 3を設定するために、リンク先のページで説明されているオプションを確認してください。

このビルドの結果は、実行可能な`mesos`バイナリになります。

## CLIの使用

Mesosを構築せずにCLIを使用することも可能です。そのためには、以下の手順でCLIの仮想環境を起動します。:

```
$ cd src/python/cli_new/
$ PYTHON=python3 ./bootstrap
$ source activate
$ mesos
```

`mesos`を呼び出すとCLIが実行され、`mesos-cli-tests`を呼び出すと統合テストが実行されます。

##  CLIの設定
CLIは設定ファイルを使用して、クラスターのマスターがどこにあるかを知り、デフォルトで提供されているプラグインに加えて使用すべきプラグインをリストアップします。

設定ファイルは、デフォルトでは`~/.mesos/config.toml`にあり、以下のようになっています。:

```
# The `plugins` array lists the absolute paths of the
# plugins you want to add to the CLI.
plugins = [
  "</absolute/path/to/plugin-1/directory>",
  "</absolute/path/to/plugin-2/directory>"
]

# The `master` field is either composed of an `address` field
# or a `zookeeper` field, but not both. For example:
[master]
  address = "10.10.0.30:5050"
  # The `zookeeper` field has an `addresses` array and a `path` field.
  # [master.zookeeper]
  #   addresses = [
  #     "10.10.0.31:5050",
  #     "10.10.0.32:5050",
  #     "10.10.0.33:5050"
  #   ]
  #   path = "/mesos"
```
