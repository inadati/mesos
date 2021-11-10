---
title: Apache Mesos - Building
layout: documentation
---

# ビルド

## Mesosのダウンロード

Mesosを入手する方法は様々です。:

1\. [Apache](http://mesos.apache.org/downloads/)から最新の安定版をダウンロード [Apache](http://mesos.apache.org/downloads/) (***推奨***)

    $ wget https://downloads.apache.org/mesos/1.11.0/mesos-1.11.0.tar.gz
    $ tar -zxf mesos-1.11.0.tar.gz

2\. Mesosのgit[リポジトリ](https://gitbox.apache.org/repos/asf/mesos.git)をクローンする（***上級者向け***）

    $ git clone https://gitbox.apache.org/repos/asf/mesos.git

*注：上記のコマンドの実行に問題がある場合は、まず以下の**システム要件**のセクションを実行して、お使いのシステムに合ったwget、tar、gitの各ユーティリティをインストールする必要があるかもしれません。*

## システム要件

Mesosは、Linux（64ビット）およびMac OS X（64ビット）で動作します。Mesosをソースからビルドするには、GCC 4.8.1+またはClang 3.5+が必要です。

Linuxでは、ビルド時と実行時の両方で、カーネルバージョンが2.6.28以上であることが必要です。Linuxでプロセスの分離を完全にサポートするには、最新のカーネルが3.10以上必要です。

また、MesosエージェントはWindowsでも動作します。ソースからMesosをビルドするには、[Windows](windows.md)のセクションの指示に従ってください。

一部のMesosテストで必要となるDockerのホストネットワーク機能を完全にサポートするために、ホスト名がDNSまたは/etc/hostsで解決可能であることを確認してください。疑わしい場合は、/etc/hostsにあなたのホスト名が含まれていることを確認してください。

### Ubuntu 14.04

以下は、純正のUbuntu 14.04での手順です。他のOSをお使いの方は、適宜パッケージをインストールしてください。

    # Update the packages.
    $ sudo apt-get update

    # Install a few utility tools.
    $ sudo apt-get install -y tar wget git

    # Install the latest OpenJDK.
    $ sudo apt-get install -y openjdk-7-jdk

    # Install autotools (Only necessary if building from git repository).
    $ sudo apt-get install -y autoconf libtool

    # Install other Mesos dependencies.
    $ sudo apt-get -y install build-essential python-dev python-six python-virtualenv libcurl4-nss-dev libsasl2-dev libsasl2-modules maven libapr1-dev libsvn-dev

### Ubuntu 16.04

以下は、純正のUbuntu 16.04での手順です。他のOSをお使いの方は、適宜パッケージをインストールしてください。

    # Update the packages.
    $ sudo apt-get update

    # Install a few utility tools.
    $ sudo apt-get install -y tar wget git

    # Install the latest OpenJDK.
    $ sudo apt-get install -y openjdk-8-jdk

    # Install autotools (Only necessary if building from git repository).
    $ sudo apt-get install -y autoconf libtool

    # Install other Mesos dependencies.
    $ sudo apt-get -y install build-essential python-dev python-six python-virtualenv libcurl4-nss-dev libsasl2-dev libsasl2-modules maven libapr1-dev libsvn-dev zlib1g-dev iputils-ping

### Mac OS X 10.11 (El Capitan), macOS 10.12 (Sierra)

以下は、Mac OS X El Capitanでの手順です。Appleが提供するツールチェーンを使用してMesosを構築する場合、XCode >= 8.0のコマンドラインツールが必要です。

    # Install Python 3: https://www.python.org/downloads/

    # Install Command Line Tools. The Command Line Tools from XCode >= 8.0 are required.
    $ xcode-select --install

    # Install Homebrew.
    $ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

    # Install Java.
    $ brew install Caskroom/cask/java

    # Install libraries.
    $ brew install wget git autoconf automake libtool subversion maven xz

    # Install Python dependencies.
    $ sudo easy_install pip
    $ pip install virtualenv

macOS 10.12でコンパイルする場合、以下が必要です。:

    # There is an incompatibility with the system installed svn and apr headers.
    # We need the svn and apr headers from a brew installation of subversion.
    # You may need to unlink the existing version of subversion installed via
    # brew in order to configure correctly.
    $ brew unlink subversion # (If already installed)
    $ brew install subversion

    # When configuring, the svn and apr headers from brew will be automatically
    # detected, so no need to explicitly point to them.
    # If the build fails due to compiler warnings, `--disable-werror` can be passed
    # to configure to not treat warnings as errors.
    $ ../configure

    # Lastly, you may encounter the following error when the libprocess tests run:
    $ ./libprocess-tests
    Failed to obtain the IP address for '<hostname>'; the DNS service may not be able to resolve it: nodename nor servname provided, or not known

    # If so, turn on 'Remote Login' within System Preferences > Sharing to resolve the issue.

*注：YosemiteからEl Capitanにアップグレードする場合は、アップグレード後に必ず`xcode-select --install`を再実行してください。*

### CentOS 6.6

以下は、純正のCentOS 6.6を使用した場合の手順です。それ以外のOSをお使いの場合は、適宜パッケージをインストールしてください。

    # Install a recent kernel for full support of process isolation.
    $ sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    $ sudo rpm -Uvh http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm
    $ sudo yum --enablerepo=elrepo-kernel install -y kernel-lt

    # Make the just installed kernel the one booted by default, and reboot.
    $ sudo sed -i 's/default=1/default=0/g' /boot/grub/grub.conf
    $ sudo reboot

    # Install a few utility tools. This also forces an update of `nss`,
    # which is necessary for the Java bindings to build properly.
    $ sudo yum install -y tar wget git which nss

    # 'Mesos > 0.21.0' requires a C++ compiler with full C++11 support,
    # (e.g. GCC > 4.8) which is available via 'devtoolset-2'.
    # Fetch the Scientific Linux CERN devtoolset repo file.
    $ sudo wget -O /etc/yum.repos.d/slc6-devtoolset.repo http://linuxsoft.cern.ch/cern/devtoolset/slc6-devtoolset.repo

    # Import the CERN GPG key.
    $ sudo rpm --import http://linuxsoft.cern.ch/cern/centos/7/os/x86_64/RPM-GPG-KEY-cern

    # Fetch the Apache Maven repo file.
    $ sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo

    # 'Mesos > 0.21.0' requires 'subversion > 1.8' devel package, which is
    # not available in the default repositories.
    # Create a WANdisco SVN repo file to install the correct version:
    $ sudo bash -c 'cat > /etc/yum.repos.d/wandisco-svn.repo <<EOF
    [WANdiscoSVN]
    name=WANdisco SVN Repo 1.8
    enabled=1
    baseurl=http://opensource.wandisco.com/centos/6/svn-1.8/RPMS/$basearch/
    gpgcheck=1
    gpgkey=http://opensource.wandisco.com/RPM-GPG-KEY-WANdisco
    EOF'

    # Install essential development tools.
    $ sudo yum groupinstall -y "Development Tools"

    # Install 'devtoolset-2-toolchain' which includes GCC 4.8.2 and related packages.
    # Installing 'devtoolset-3' might be a better choice since `perf` might
    # conflict with the version of `elfutils` included in devtoolset-2.
    $ sudo yum install -y devtoolset-2-toolchain

    # Install other Mesos dependencies.
    $ sudo yum install -y apache-maven python-devel python-six python-virtualenv java-1.7.0-openjdk-devel zlib-devel libcurl-devel openssl-devel cyrus-sasl-devel cyrus-sasl-md5 apr-devel subversion-devel apr-util-devel

    # Enter a shell with 'devtoolset-2' enabled.
    $ scl enable devtoolset-2 bash
    $ g++ --version  # Make sure you've got GCC > 4.8!

    # Process isolation is using cgroups that are managed by 'cgconfig'.
    # The 'cgconfig' service is not started by default on CentOS 6.6.
    # Also the default configuration does not attach the 'perf_event' subsystem.
    # To do this, add 'perf_event = /cgroup/perf_event;' to the entries in '/etc/cgconfig.conf'.
    $ sudo yum install -y libcgroup
    $ sudo service cgconfig start

### CentOS 7.1

以下は、純正のCentOS 7.1を使用した場合の手順です。それ以外のOSをお使いの場合は、適宜パッケージをインストールしてください。

    # Install a few utility tools
    $ sudo yum install -y tar wget git

    # Fetch the Apache Maven repo file.
    $ sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo

    # Install the EPEL repo so that we can pull in 'libserf-1' as part of our
    # subversion install below.
    $ sudo yum install -y epel-release

    # 'Mesos > 0.21.0' requires 'subversion > 1.8' devel package,
    # which is not available in the default repositories.
    # Create a WANdisco SVN repo file to install the correct version:
    $ sudo bash -c 'cat > /etc/yum.repos.d/wandisco-svn.repo <<EOF
    [WANdiscoSVN]
    name=WANdisco SVN Repo 1.9
    enabled=1
    baseurl=http://opensource.wandisco.com/centos/7/svn-1.9/RPMS/\$basearch/
    gpgcheck=1
    gpgkey=http://opensource.wandisco.com/RPM-GPG-KEY-WANdisco
    EOF'

    # Parts of Mesos require systemd in order to operate. However, Mesos
    # only supports versions of systemd that contain the 'Delegate' flag.
    # This flag was first introduced in 'systemd version 218', which is
    # lower than the default version installed by centos. Luckily, centos
    # 7.1 has a patched 'systemd < 218' that contains the 'Delegate' flag.
    # Explicity update systemd to this patched version.
    $ sudo yum update systemd

    # Install essential development tools.
    $ sudo yum groupinstall -y "Development Tools"

    # Install other Mesos dependencies.
    $ sudo yum install -y apache-maven python-devel python-six python-virtualenv java-1.8.0-openjdk-devel zlib-devel libcurl-devel openssl-devel cyrus-sasl-devel cyrus-sasl-md5 apr-devel subversion-devel apr-util-devel

### Windows

[Windows](windows.md)の項の説明に従ってください。
## Mesosの構築 (Posix)

    # Change working directory.
    $ cd mesos

    # Bootstrap (Only required if building from git repository).
    $ ./bootstrap

    # Configure and build.
    $ mkdir build
    $ cd build
    $ ../configure
    $ make

ビルドを高速化し、ログの冗長性を減らすために、`-j <コア数> V=0` を `make` に追加することができます。

    # Run test suite.
    $ make check

    # Install (Optional).
    $ make install

## 例

Mesosには、C++、Java、Pythonで書かれたサンプルフレームワークがバンドルされています。
フレームワークのバイナリは、上記の**Mesosの構築**のセクションで説明したように、`make check`を実行して初めて利用できるようになります。

    # Change into build directory.
    $ cd build

    # Start Mesos master (ensure work directory exists and has proper permissions).
    $ ./bin/mesos-master.sh --ip=127.0.0.1 --work_dir=/var/lib/mesos

    # Start Mesos agent (ensure work directory exists and has proper permissions).
    $ ./bin/mesos-agent.sh --master=127.0.0.1:5050 --work_dir=/var/lib/mesos

    # Visit the Mesos web page.
    $ http://127.0.0.1:5050

    # Run C++ framework (exits after successfully running some tasks).
    $ ./src/test-framework --master=127.0.0.1:5050

    # Run Java framework (exits after successfully running some tasks).
    $ ./src/examples/java/test-framework 127.0.0.1:5050

    # Run Python framework (exits after successfully running some tasks).
    $ ./src/examples/python/test-framework 127.0.0.1:5050

*注：これらの例は、ローカルマシンでMesosを実行していることを前提としています。
例に従うと、本番環境（AWSなど）でMesosのWebページにアクセスすることはできません。
環境（例：AWS）ではアクセスできません。そのためには、Mesosマスターの起動時にホストの実際のIPを指定し、ファイアウォールの設定で外部からのポート5050へのアクセスを許可する必要があります。*
