---
title: Apache Mesos - Getting started using Binaries.
layout: documentation
---

# バイナリパッケージ

## MesosのRPMをダウンロードする

[Bintrayリポジトリ](https://bintray.com/apache/mesos/)から最新の安定版RPMバイナリをダウンロードしてインストールします。:

    $ cat > /tmp/bintray-mesos-el.repo <<EOF
    #bintray-mesos-el - packages by mesos from Bintray
    [bintray-mesos-el]
    name=bintray-mesos-el
    baseurl=https://dl.bintray.com/apache/mesos/el7/x86_64
    gpgcheck=0
    repo_gpgcheck=0
    enabled=1
    EOF

    $ sudo mv /tmp/bintray-mesos-el.repo /etc/yum.repos.d/bintray-mesos-el.repo

    $ sudo yum update

    $ sudo yum install mesos

上記の手順は、RHEL 7用の最新バージョンのMesosをインストールする方法を示しています。
`baseurl` をお使いのオペレーティング・システムに適した URL に置き換えてください。
## Mesosマスターとエージェントを起動する

RPMインストールでは、作業用ディレクトリとして使用できる`/var/lib/mesos`ディレクトリが作成されます。

以下のコマンドでMesosマスターを起動します。:

    $ mesos-master --work_dir=/var/lib/mesos

別の端末でMesosエージェントを起動し、上記で起動したMesosマスターと関連付けます。:

    $ mesos-agent --work_dir=/var/lib/mesos --master=127.0.0.1:5050

この方法は、RPMをダウンロードした後にMesosを試す最も簡単な方法です。より複雑で本格的な設定方法については、ドキュメントの[管理](http://mesos.apache.org/documentation/latest/#administration)セクションを参照してください。
