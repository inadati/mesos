---
title: Apache Mesos - Performance Profiling
layout: documentation
---

# パフォーマンス・プロファイリング

このドキュメントには、Mesosのパフォーマンス分析を行うための様々なプロファイリングツールの使用方法に関する様々なガイドが掲載されています。

## Flamescope
[Flamescope](https://github.com/Netflix/flamescope)は、異なる時間範囲を[flamegraphs](https://github.com/brendangregg/FlameGraph)として探索するための可視化ツールです。このツールを使うためには、まずスタックトレースを取得する必要があります。ここでは、Linuxのperfを使って、100ヘルツのmesos masterプロセスの60秒の記録を取得する方法を紹介します。:

```
$ sudo perf record --freq=100 --no-inherit --call-graph dwarf -p <mesos-master-pid> -- sleep 60
$ sudo perf script --header | c++filt > mesos-master.stacks
$ gzip mesos-master.stacks
```
パフォーマンスデータの解析に協力を求めたい場合は、`mesos-master.stacks.gz`を一般にアクセス可能な場所にアップロードし、解析用に`dev@mesos.apache.org` でファイルを作成するか、[slack](http://mesos.slack.com)の#performanceチャンネルにファイルを送ってください。

また、自分で解析を行う場合は、mesos-master.stacksをflamescopeのgit checkoutの`examples`フォルダに入れてください。
