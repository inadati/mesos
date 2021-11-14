---
title: Apache Mesos - Windows
layout: documentation
---

# Windows

Mesos 1.0.0では、Windowsを実験的にサポートしました。

## Mesosの構築

### システム要件

1. 最新の[Visual Studio 2017](https://www.visualstudio.com/downloads/)をインストールします。Community版で十分です。（無料）インストール時に、「C++によるデスクトップ開発」のワークロードを選択します。

2. CMake [CMake 3.8.0](https://cmake.org/download/)以降をインストールします。インストールの際、「Add CMake to the system PATH for all users」を選択してください。

3. Windows用の[GNUパッチ](http://gnuwin32.sourceforge.net/packages/patch.htm)をインストールする。

4. ソースからビルドする場合は、[Git](https://git-scm.com/download/win)をインストールします。

5. ビルドディレクトリにスペースがないことを確認してください。例えば、`C:/Program Files (x86)/mesos`は無効なビルドディレクトリです。

6. Mesosを開発している場合は、Python 2ではなく[Python 3](https://www.python.org/downloads/)をインストールしてください。これは、当社のサポートスクリプト（パッチの投稿や適用、ソースコードのlintなど）を使用するためです。

### 構築手順

以下は、Windows 10での手順です。

    # Clone (or extract) Mesos.
    git clone https://gitbox.apache.org/repos/asf/mesos.git
    cd mesos

    # Configure using CMake for an out-of-tree build.
    mkdir build
    cd build
    cmake .. -G "Visual Studio 15 2017 Win64" -T "host=x64"

    # Build Mesos.
    # To build just the Mesos agent, add `--target mesos-agent`.
    cmake --build .

    # The Windows agent exposes new isolators that must be used as with
    # the `--isolation` flag. To get started point the agent to a working
    # master, using eiher an IP address or zookeeper information.
    .\src\mesos-agent.exe --master=<master> --work_dir=<work folder> --launcher_dir=<repository>\build\src

## Mesosの起動

実行ファイルを別のマシンに展開する場合は、[Microsoft Visual C++ Redistributable for Visual Studio 2017](https://aka.ms/vs/15/release/VC_redist.x64.exe)もインストールする必要があります。

If you deploy the executables to another machine, you must also
install the [Microsoft Visual C++ Redistributable for Visual Studio 2017](https://aka.ms/vs/15/release/VC_redist.x64.exe).

## 既知の制限事項

現在の実装では、以下のような制限があることが分かっています。:

* Windowsではエージェントのみを実行する必要があります。Mesosマスターを起動することもできますが、マスターはWindowsでの高可用セットアップをサポートしていないため、テスト用にのみ使用してください。

* Mesosは内部的にNTFSのロングパスをサポートしていますが、ロングパスをサポートしていないタスクは、`-work_dir`がショートパスのエージェントで実行する必要があります。

* 対応するWindowsの最小バージョンは Windows 10 Creators Update（別名：バージョン1703、ビルド番号15063）、および[Windows Server（バージョン1709）][server]です。Windowsコンテナのサポートや移植を容易にする開発者向けの機能が進化しているため、今後も増える可能性があります。

* 非管理者として[シンボリックリンクを作成する][create symlinks]には、開発者モードを有効にする必要があります。それ以外の場合は、エージェントを管理者の下で実行する必要があります。

[server]: https://docs.microsoft.com/en-us/windows-server/get-started/get-started-with-1709
[create symlinks]: https://blogs.windows.com/buildingapps/2016/12/02/symlinks-windows-10/

## ビルド構成の例

### Ninjaでのビルド

MSBuildを使用する代わりに、[Ninja](https://ninja-build.org/)を使用してWindows上でMesosをビルドすることも可能で、その場合はビルドが大幅に速くなります。Ninjaを使用するには、Ninjaをダウンロードして、`ninja.exe`が`PATH`に含まれていることを確認する必要があります。

* [Windowsバイナリ]((https://github.com/ninja-build/ninja/releases))をダウンロードする。
* 解凍して、`ninja.exe`を`PATH`に入れてください。
* 環境を整えるために「x64 Native Tools Command Prompt for VS 2017」を開きます。
* そのコマンドプロンプトで、`powershell`と入力すると、より良いシェルを使うことができます。
* 上記と同様に、`cmake .. -G Ninja`でCMakeを設定します。
* これで、`ninja`を使って様々なターゲットを作ることができます。
* それ以外は暗黙なので、`ninja -v`を使って冗長にするとよいでしょう。

なお、Ninjaでは、64ビットのビルドツールを使用するために、正しい開発者用コマンドプロンプトを開く必要があります。（Ninjaでは64ビットのビルドツールを見つける方法がわからないため）

### Javaでのビルド

これにより、より多くのユニットテストが可能になりますが、まだ`mesos-master`を公式には作成していません。

WindowsでJavaを使ってビルドする場合は、[Maven][]ビルドツールをパスに追加する必要があります。また、`JAVA_HOME`環境変数を手動で設定する必要があります。Java SDKのインストールは、[Oracle][]社のフォームに記載されています。

[maven]: https://maven.apache.org/guides/getting-started/windows-prerequisites.html
[oracle]: http://www.oracle.com/technetwork/java/javase/downloads/index.html

この記事を書いている時点では、Java 9はまだサポートされていませんが、Java 8はテストされています。

Javaのビルドは遅いので、デフォルトでは`OFF`になっています。WindowsでJavaコンポーネントをビルドするには、次のようにして`ON`にします。:

```powershell
mkdir build; cd build
$env:PATH += ";C:\...\apache-maven-3.3.9\bin\"
$env:JAVA_HOME = "C:\Program Files\Java\jdk1.8.0_144"
cmake .. -DENABLE_JAVA=ON -G "Visual Studio 15 2017 Win64" -T "host=x64"
cmake --build . --target mesos-java
```
`mesos-java`ライブラリを手動で構築する必要はありません。Javaが有効な場合、`libmesos`がリンクします。

残念ながら、Windowsでは`FindJNI` CMakeモジュールが`JAVA_JVM_LIBRARY`に静的な`jvm.lib`へのパスを入力しますが、この変数は実行時にロードされる共有ライブラリ`jvm.dll`を指していなければなりません。次のように正しく設定してください。:

```
$env:JAVA_JVM_LIBRARY = "C:\Program Files\Java\jdk1.8.0_144\jre\bin\server\jvm.dll"
```

この場合、ランタイムにライブラリのロードに失敗し、次のようなエラーが発生することがあります。:

> "The specified module could not be found."

この場合、`jvm.dll`へのパスが正しいことが確認されると、エラーメッセージは実際には`jvm.dll`の依存関係が見つからないことを示しています。Windowsでは、DLLの検索パスには環境変数`PATH`が含まれていますので、`server\jvm.dll`を含むbinフォルダを`PATH`に追加してください。:

```
$env:PATH += ";C:\Program Files\Java\jdk1.8.0_144\jre\bin"
```

### OpenSSLでのビルド

WindowsでOpenSSLを使ってビルドする場合、Windows用のOpenSSLのディストリビューションをビルドまたはインストールする必要があります。一般的には、[Shining Light Productions社のOpenSSL][openssl]がよく使われています。

[openssl]: https://slproweb.com/products/Win32OpenSSL.html

この記事を書いている時点では、OpenSSL 1.1.xがサポートされています。

OpenSSLでビルドするには、`-DENABLE_SSL=ON`を使用してください。

OpenSSLに動的にリンクするので、ビルドした実行ファイルを別の場所に配置する場合は、そのマシンにもOpenSSLをインストールする必要があることに注意してください。

OpenSSLのインストールやMesos自体には証明書がバンドルされていないため、証明書の検証に失敗する可能性があることに注意してください。
