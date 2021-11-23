---
title: Apache Mesos - Logging
layout: documentation
---

# ロギング
Mesosは、各Mesosコンポーネントのログを、Mesosがそのコンポーネントのソースコードをどの程度制御しているかによって、異なった方法で処理します。

大まかに言うと、これらのカテゴリは:

* [Internal](#Internal) - マスターとエージェント。
* [Containers](#Containers) - ExecutorとTask。
* External - Frameworksや[ZooKeeper](high-availability.md)など、Mesosの外部で起動されるコンポーネント。これらのコンポーネントは、独自のロギング・ソリューションを実装することが期待されます。

## <a name="Internal"></a>Internal

Mesos MasterとAgentは、[Googleのロギングライブラリ](https://github.com/google/glog)を使用しています。このライブラリの設定に使用されるコマンドラインオプションに関する情報は、[設定ドキュメント](configuration/master-and-agent.md#logging-options)を参照してください。Googleのロギング・オプションで明示的に言及されていないものは、環境変数で設定できます。

MasterとAgentは、[/logging/toggle](endpoints/logging/toggle.md) HTTPエンドポイントを公開しており、これは一時的に詳細なロギングに切り替えます。:

```
POST <ip:port>/logging/toggle?level=[1|2|3]&duration=VALUE
```

この効果は、マスター/エージェントの起動前に環境変数`GLOG_v`を設定した場合と似ていますが、指定された期間が経過するとログレベルが元に戻ります。

## <a name="Containers"></a>Containers

背景については、[containerizerのドキュメント](containerizers.md)を参照してください。

Mesosは、コンテナ内で実行されるエンティティの構造化ロギングを想定していません。代わりに、Mesosはコンテナの標準出力と標準エラーを[サンドボックス](sandbox.md#where-is-it)内にあるプレーンファイル（「stdout」と「stderr」）に格納します。

場合によっては、Mesosのデフォルトのコンテナロガーの動作が理想的ではないこともあります。:

* ロギングがコンテナ間で標準化されていない可能性があります。
* ログの集約が容易ではない。
* ログファイルのサイズは管理されていません。十分な時間があれば、「stdout」および「stderr」ファイルが Agent のディスクを埋め尽くす可能性があります。

## `ContainerLogger` モジュール
`ContainerLogger`モジュールはMesos 0.27.0で導入され、コンテナのデフォルトのログ動作の欠点に対処することを目的としています。このモジュールは、Mesosがコンテナのstdoutとstderrをリダイレクトする方法を変更するために使用できます。

[`ContainerLogger`のインターフェースはこちら](https://github.com/apache/mesos/blob/master/include/mesos/slave/container_logger.hpp)にあります。

Mesosには2つの`ContainerLogger`モジュールが付属しています。:

* `SandboxContainerLogger` は、既存のロギング動作を `ContainerLogger` として実装しています。これがデフォルトの動作です。
* `LogrotateContainerLogger` は、束縛されないログファイルサイズの問題に対処します。

### `LogrotateContainerLogger`

`LogrotateContainerLogger` は、コンテナの stdout および stderr ファイルの合計サイズを制限します。このモジュールは、モジュールへのパラメータに基づいてログファイルを回転させることでこれを行います。ログファイルが指定された最大サイズに達すると、ファイル名の最後に `.N` を追加して名前が変更されます（`N` はローテーションごとに増加します）。古いログファイルは、指定された最大ファイル数に達すると削除されます。

#### モジュールの起動
`LogrotateContainerLogger` は、エージェントの起動時に [`--modules` flag](modules.md#Invoking)でライブラリ `liblogrotate_container_logger.so` を指定し、 `--container_logger` Agent フラグに `org_apache_mesos_LogrotateContainerLogger` を設定することでロードできます。

#### モジュールのパラメーター

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Key
      </th>
      <th>
        説明
      </th>
    </tr>
  </thead>

  <tr>
    <td>
      <code>max_stdout_size</code>/<code>max_stderr_size</code>
    </td>
    <td>
      1つのstdout/stderrログファイルの最大サイズ（バイト）です。このサイズに達すると、ファイルのローテーションが行われます。
      デフォルトは10MBです。 1（メモリ）ページの最小サイズは、通常4KB程度です。
    </td>
  </tr>

  <tr>
    <td>
      <code>logrotate_stdout_options</code>/
      <code>logrotate_stderr_options</code>
    </td>
    <td>
      <code>logrotate</code>に渡す標準出力用の追加設定オプション。この文字列は、<code>logrotate</code>の設定ファイルに挿入されます。 例:「stdout」の場合。:
      <pre>
/path/to/stdout {
  [logrotate_stdout_options]
  size [max_stdout_size]
}</pre>
      注: <code>size</code>オプションは、このモジュールによって上書きされます。
    </td>
  </tr>

  <tr>
    <td>
      <code>environment_variable_prefix</code>
    </td>
    <td>
      起動している特定のコンテナに対する logrotate ロガーの動作を変更するための環境変数のプレフィックスです。ロガーは、コンテナの <code>CommandInfo</code> の <code>Environment</code> にある 4 つのプレフィックス付き環境変数を探します。:
      <ul>
        <li><code>MAX_STDOUT_SIZE</code></li>
        <li><code>LOGROTATE_STDOUT_OPTIONS</code></li>
        <li><code>MAX_STDERR_SIZE</code></li>
        <li><code>LOGROTATE_STDERR_OPTIONS</code></li>
      </ul>
      これらの変数が存在すると、モジュールのパラメータで設定されたグローバルな値を上書きします。デフォルトは <code>CONTAINER_LOGGER_</code> です。
    </td>
  </tr>

  <tr>
    <td>
      <code>launcher_dir</code>
    </td>
    <td>
      Mesosのバイナリのディレクトリパス。<code>LogrotateContainerLogger</code>は、このディレクトリの下にある<code>mesos-logrotate-logger</code>バイナリを見つけます。デフォルトは <code>/usr/local/libexec/mesos</code> です。
    </td>
  </tr>

  <tr>
    <td>
      <code>logrotate_path</code>
    </td>
    <td>
      指定された場合、<code>LogrotateContainerLogger</code> は、システムの <code>logrotate</code> の代わりに、指定された <code>logrotate</code> を使用します。<code>logrotate</code> が見つからない場合は、モジュールはエラーで終了します。
    </td>
  </tr>
</table>

#### 仕組みについて
1. コンテナが起動するたびに、`LogrotateContainerLogger`は`mesos-logrotate-logger`バイナリのコンパニオンサブプロセスを起動します。
2. このモジュールは、コンテナのstdout/stderrを`mesos-logrotate-logger`にリダイレクトするようMesosに指示します。
3. コンテナがstdout/stderrに出力すると、`mesos-logrotate-logger`はその出力を "stdout"/"stderr "ファイルにパイプします。ファイルが大きくなると、`mesos-logrotate-logger`は`logrotate`を呼び出して、ファイルを設定された最大サイズ以下に厳密に維持します。
4. コンテナが終了すると、`mesos-logrotate-logger`は終了する前にロギングを終了します。

`LogrotateContainerLogger`は、Agentのフェイルオーバーにも対応できるように設計されています。Agentプロセスが終了しても、`mesos-logrotate-logger`のインスタンスは継続して実行されます。

### カスタム`ContainerLogger`の作成
モジュールの書き方の基本については、[modulesのドキュメント](modules.md)をご覧ください。

新しい`ContainerLogger`を設計する際には、いくつかの注意点があります。:

* `ContainerLogger`によるロギングは、Agentのフェイルオーバーに強いものでなければなりません。Agentプロセス（`ContainerLogger`モジュールを含む）が死亡しても、ロギングは継続されるべきである。これは通常、サブプロセスを使用することで実現できます。
* コンテナがシャットダウンしても、`ContainerLogger`には明示的に通知されません。代わりに、コンテナのstdout/stderrで`EOF`に遭遇すると、コンテナが終了したことを示す。これにより、`ContainerLogger`が自分自身を終了する前にすべてのログを見たことがより強く保証されます。
* `ContainerLogger` は、特定の `ContainerLogger` でコンテナが起動されたと仮定してはいけません。Agent は、異なる `ContainerLogger` で再起動することができる。
* Agent上で動作する各[containerizer](containerizers.md)は、それぞれの`ContainerLogger`のインスタンスを使用します。つまり、1つのAgentで複数の`ContainerLogger`を実行することができます。ただし、各 Agent は 1 種類の `ContainerLogger` しか実行しません。
