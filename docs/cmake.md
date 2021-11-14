---
title: Apache Mesos - CMake
layout: documentation
---

# CMake 3.7+のインストール

[cmake-download]: https://cmake.org/download/

## Linux

[CMake.org][cmake-download]から最新版のCMakeをインストールします。自己解凍型のタールボールが用意されているので、この作業は簡単です。

現在、一般的なLinuxでは、十分なバージョンのCMakeがパッケージされているものはほとんどありません。Ubuntu バージョン 12.04 および 14.04 は CMake 2 を、Ubuntu 16.04 は CMake 3.5 を搭載しています。パッケージからcmakeをインストールした場合は、`apt-get purge cmake`で削除できます。

CentOSの標準パッケージはCMake 2ですが、残念ながらEPELの`cmake3`パッケージはCMake 3.6しかありませんので、次のようにして削除してください。  
`yum remove cmake cmake3`

## Mac OS X

HomeBrewのCMakeのバージョンで十分です。： `brew install cmake`

## Windows

[CMake.org][cmake-download]からMSIをダウンロードしてインストールします。

**注:** Windowsでは、CMake 3.7+ではなく、3.8+が必要です。

# Quick Start

CMakeを使って設定なしでビルドする最も基本的な方法は、非常に簡単です。:

```
mkdir build
cd build
cmake ..
cmake --build .
```

最後のステップである `cmake --build .` は、任意の特定のターゲットをビルドする `--target` コマンドを取ることもできます。 (例: `mesos-tests`、または `tests`で`mesos-tests`、`libprocess-tests`、`stout-tests`をビルドする)  `cmake --build . --target tests` 任意のフラグをネイティブのビルドシステム (`make` など) に送るには，コマンドに `-- <flags to be passed>` を追加します。`cmake --build . -- -j4`

また、`cmake --build` の代わりに、お好みのビルドシステムを使用することができます。例えば、Linux のデフォルトの CMake ジェネレーターは GNU Makefile を生成するので、`cmake ..` を設定した後は、通常通り `build` フォルダで `make tests` を実行すればよいのです。同様に、`-G Ninja` で設定して Ninja ジェネレーターを使用すると、`ninja tests` を実行して Ninja で`tests`をビルドすることができます。

# インストール可能なビルド

この例では、Mesosをビルドし、カスタムプレフィックスにインストールします。:
```
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/home/current_user/mesos
cmake --build . --target install
```

`mesos-tests`実行ファイルと関連するテストヘルパーを追加でインストールするには、`MESOS_INSTALL_TESTS`オプションを有効にします。（これは、インストールしたバイナリに対してMesosテストを実行するために使用できます）

別の場所にコピー/移動した後も動作するバイナリやライブラリのセットを作成するには、`MESOS_FINAL_PREFIX`を使用します。

以下の例では、`MESOS_FINAL_PREFIX`と`MESOS_INSTALL_TESTS`の両方を使用しています。

ビルドシステムで:
```
mkdir build && cd build
cmake -DMESOS_FINAL_PREFIX=/opt/mesos -DCMAKE_INSTALL_PREFIX=/home/current_user/mesos -DMESOS_INSTALL_TESTS=ON
cmake --build . --target install
tar -czf mesos.tar.gz mesos -C /home/current_user
```
ターゲットシステムで:
```
sudo tar -xf mesos.tar.gz -C /opt
# Run tests against Mesos installation
sudo /opt/mesos/bin/mesos-tests
# Start Mesos agent
sudo /opt/mesos/bin/mesos-agent --work-dir=/var/lib/mesos ...
```

# 対応オプション

See [設定オプション](configuration/cmake.md).

# 例

See [CMakeの例](cmake-examples.md).

# ドキュメント

[CMakeのドキュメント][]は、リファレンスモジュールとして書かれています。最もよく使われるセクションは:

* [ビルドシステムの概要](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html)
* [コマンド](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)
* [プロパティ](https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html)
* [変数](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html)

また、wikiには[便利な変数][]のセットがあります。

[CMakeのドキュメント]: https://cmake.org/cmake/help/latest/
[便利な変数]: https://cmake.org/Wiki/CMake_Useful_Variables

# 依存関係のグラフ

他のビルドシステムと同様に、CMakeは依存関係のグラフを持っています。他のビルドシステムとの違いは、CMakeの依存関係グラフのターゲットが他のビルドシステムに比べて非常に豊富であることです。CMake のターゲットは 「インターフェイス」 という概念を持っており、ビルドプロパティがターゲットの一部として保存され、これらのプロパティはグラフ内で過渡的に継承されます。

例えば、`mylib`というライブラリがあり、それをリンクする際には、`mylib/include`にあるそのヘッダをインクルードしなければなりません。ライブラリをビルドする際には、いくつかのプライベートヘッダもインクルードする必要がありますが、リンクする際にはインクルードしません。実行ファイルである`myprogram`をコンパイルする際には、`mylib`のパブリックヘッダをインクルードする必要がありますが、プライベートヘッダはインクルードしません。`myprogram`（および`mylib`にリンクする他のプログラム）に`mylib/include`を追加する手動のステップはなく、代わりに`mylib`のpublic interfaceプロパティから推論されます。これは以下のコードで表されます。:

```
# A new library with a single source file (headers are found automatically).
add_library(mylib mylib.cpp)

# The folder of private headers, not exposed to consumers of `mylib`.
target_include_directories(mylib PRIVATE mylib/private)

# The folder of public headers, added to the compilation of any consumer.
target_include_directories(mylib PUBLIC mylib/include)

# A new exectuable with a single source file.
add_executable(myprogram main.cpp)

# The creation of the link dependency `myprogram` -> `mylib`.
target_link_libraries(myprogram mylib)

# There is no additional step to add `mylib/include` to `myprogram`.
```

この考え方は、[`target_compile_definitions`][]のコンパイル定義、[`target_include_directories`][]のインクルードディレクトリ、[`target_link_libraries`][]のリンクライブラリ、[`target_compile_options`][]のコンパイルオプション、[`target_compile_features`][]のコンパイル機能など、ほぼすべてのビルドプロパティに当てはまります。

これらのコマンドはすべて、オプションで`<INTERFACE|PUBLIC|PRIVATE>`という引数を取り、グラフ内での遷移性を制約します。すなわち、`PRIVATE`インクルード・ディレクトリはターゲットのために記録されますが、ターゲットに依存するものには経時的に共有されません。`PUBLIC`はターゲットとその依存関係の両方に使用され、`INTERFACE`は依存関係にのみ使用されます。

このリストで注目すべきは、リンクディレクトリがないことです。CMake はライブラリの絶対パスを見つけて使用することを明確に推奨しており、リンクディレクトリは廃止されました。

[`target_compile_definitions`]: https://cmake.org/cmake/help/latest/command/target_compile_definitions.html
[`target_include_directories`]: https://cmake.org/cmake/help/latest/command/target_include_directories.html
[`target_link_libraries`]: https://cmake.org/cmake/help/latest/command/target_link_libraries.html
[`target_compile_options`]: https://cmake.org/cmake/help/latest/command/target_compile_options.html
[`target_compile_features`]: https://cmake.org/cmake/help/latest/command/target_compile_features.html

# よくある間違い

## Boolean

CMakeでは、`ON`、`OFF`、`TRUE`、`FALSE`、`1`、`0`をすべて真偽不明のBooleanとして扱います。さらに、`<target>-NOTFOUND`という形式の変数もfalseとして扱われます（これはパッケージの検索に使用されます）。

Mesosでは、`TRUE`と`FALSE`というBoolean型が好まれます。

詳しくは [`if`](https://cmake.org/cmake/help/latest/command/if.html) を参照してください。

## 条件式

歴史的な理由により、CMakeの`if`や`elseif`などの条件式は、自動的に変数名を補間します。もし、`${FOO}`が`BAR`と評価され、`BAR`が別の変数名であれば、`if (${FOO})`は`if (BAR)`となり、`BAR`は`if`によって再び評価されることになるため、手動で補間することは危険です。`${FOO}`の値を確認するには、`if (FOO)`にこだわってください。`if (${FOO})`は使用しないでください。

CMake ポリシー [CMP0012](https://cmake.org/cmake/help/latest/policy/CMP0012.html) および [CMP0054](https://cmake.org/cmake/help/latest/policy/CMP0054.html) も参照してください。

## 定義

`add_definitions()`を使用する場合（「グローバル」なコンパイル定義のためなので、めったに使用すべきではありません）、プリプロセッサ定義として扱われるためには、フラグの前に`-D`を付けなければなりません。しかし、`target_compile_definitions()`を使用する場合（特定のターゲットのために使用するため、好んで使用されるべきです）、フラグにはプレフィックスが必要ありません。
# 様式

一般的には、80行で折り返し、2スペースのインデントを使用します。引数を折り返す場合は、コマンドを別の行に、引数をそれ以降の行に書きます。:

```
target_link_libraries(
  program PRIVATE
  alpha
  beta
  gamma)
```

もしくは、一行で記述できます。:

```
target_link_libraries(program PUBLIC library)
```

括弧は常に最後の引数と一緒にしてください。

`if (FOO)`のような条件式とカッコの間には半角スペースを使用しますが、`add_executable(program)`のようなコマンドには使用しません。

カスタム関数やマクロ（`EXTERNAL`や`PATCH_CMD`など）の宣言と使用は大文字で行い、CMakeの組み込み（モジュールを含む）関数やマクロの使用は大文字で行いません。変数を大文字にします。

# CMakeのアンチパターン

[アンチパターン]: http://voices.canonical.com/jussi.pakkanen/2013/03/26/a-list-of-common-cmake-antipatterns/

CMake は他のビルドシステムよりも多くの 単純作業を代行するため、残念ながら新しい CMake コードを書く際に気をつけなければならない CMake の[アンチパターン][]がたくさんあります。以下は、新しい CMake コードを書く際に避けなければならない一般的な問題です。:

## `add_dependencies`の余分な使用

`target_link_libraries(a b)`でライブラリ`a`をライブラリ`b`にリンクした場合、CMakeのグラフはすでに依存関係の情報で更新されています。依存関係を(再)指定するために `add_dependencies(a b)` を使うのは冗長です。実際のところ、このコマンドはほとんど使われるべきではありません。

その例外として:

  1. `ExternalProject_Add`で追加されたターゲットに、インポートされたライブラリからの依存関係を設定する。
  2. 明示的なリンクが行われないため、Mesosモジュールへの依存関係を設定する。
  3. 実行ファイル間の依存関係を設定する（例：`mesos-agent`が`mesos-containerizer`実行ファイルを必要とする）。一般的に、実行時の依存関係は`add_dependency`で設定する必要がありますが、リンク依存関係は設定しません。

## `link_libraries`または `link_directories`の使用

これらのコマンドは決して使用してはいけません。ライブラリのリンクに使用される唯一の適切なコマンドは [`target_link_libraries`][] であり、CMake の依存関係グラフに情報を記録します。さらに、インポートされたサードパーティのライブラリは、それぞれのターゲットに正しい位置が記録されているはずなので、`link_directories`の使用は決して必要ではありません。
[公式ドキュメント][link-directories]では:

> Note that this command is rarely necessary. Library locations returned by
> `find_package()` and `find_library()` are absolute paths. Pass these absolute
> library file paths directly to the `target_link_libraries()` command. CMake
> will ensure the linker finds them.

[link-directories]: https://cmake.org/cmake/help/latest/command/link_directories.html

前者はグローバルな（またはディレクトリレベルの）副作用を設定し、後者はグラフに格納された特定のターゲット情報を設定するという違いがあります。

## `include_directories`の使用

インクルードディレクトリの情報が適切なターゲットにローカライズされたままになるように、`target_include_directories`を常に優先すべきであるということです。
## `endif ()`に何かを追加する

CMake の古いバージョンでは、`if (FOO)` ... `endif (FOO)` というスタイルを想定しており、`endif` に `if` コマンドと同じ式が含まれていました。しかし、これでは冗長になってしまうので、`endif ()`の括弧は空にしておきましょう。これは、`endforeach()`、`endwhile()`、`endmacro()`、`endfunction()`などの他の終わり方にも当てはまります。

## 余分なヘッダーファイルの指定

CおよびC++プロジェクトでCMakeを使用する際の明確な利点の1つは、ターゲットのソースリストにヘッダーファイルを追加する必要がないことです。CMakeは、ソースファイル（`.c`、`.cpp`など）を解析し、必要なヘッダーを自動的に決定するように設計されています。ただし、ビルドの一部として生成されたヘッダー（protobufやJNIヘッダーなど）は例外です。

## `CMAKE_BUILD_TYPE`を確認する

詳細については、[デバッグまたはリリース構成の構築](cmake-examples.md#building-debug-or-release-configurations)の例を参照してください。要するに、全ての[ジェネレーターが設定時][]に変数 `CMAKE_BUILD_TYPE` を尊重するわけではないので、CMake ロジックで使用してはいけません。サポートされている場合は、`$<$<CONFIG:Debug>:DEBUG_MODE>`のようなジェネレータ式を使用することができます。

[ジェネレーターが設定時]: https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#logical-expressions

# その他のHack

## `3RDPARTY_DEPENDENCIES`

Mesos on Windowsが安定するまでは、いくつかの依存関係を外部のリポジトリ[3rdparty](https://github.com/mesos/3rdparty)に保管しています。すべての依存関係がMesosにバンドルされるようになれば、この追加リポジトリは必要なくなります。それまでは、CMakeの変数`3RDPARTY_DEPENDENCIES`はデフォルトでこのURLを指しますが、レポのローカルクローンのディスク上の場所を指すこともできます。このオプションを使用すると、クリーンビルドのたびに GitHub からデータを取得する必要がなくなります。`-D3RDPARTY_DEPENDENCIES=C:/3rdparty`のように、フォワードスラッシュを含む絶対パスでなければならないことに注意してください。（Windowsでは失敗します）

## `EXTERNAL`

CMakeの`EXTERNAL`関数では、依存関係にあるディレクトリ構造を簡単に追跡するための変数がいくつか定義されています。具体的には、ライブラリの名前が`boost`の場合、次のように呼び出します。:

```
EXTERNAL(boost ${BOOST_VERSION} ${CMAKE_CURRENT_BINARY_DIR})
```

これにより、現在のスコープで以下の変数が副作用として定義されます。:

* `BOOST_TARGET` (depを入れるターゲットフォルダ名。例：`boost-1.53.0`)
* `BOOST_CMAKE_ROOT` (CMakeが圧縮されていないソースを置く場所。例：`build/3rdparty/boost-1.53.0`)
* `BOOST_ROOT` (ビルドの様々な段階でコードが置かれる場所。例えば、`build/.../boost-1.53.0/src`のように、依存関係にあるビルドの段階ごとに、`build-1.53.0-build`、`-lib`などのフォルダが含まれることがあります)

実装は`3rdparty/cmake/External.cmake`にあります。

これは、CMakeのモジュールである[ExternalProject][]と混同してはいけません。`ExternalProject_Add`を使用して、依存関係のダウンロード、抽出、設定、およびビルドを行います。

[ExternalProject]: https://cmake.org/cmake/help/latest/module/ExternalProject.html

## `CMAKE_NOOP`

これは、`3rdparty/CMakeLists.txt`で定義するCMake変数で、`ExternalProject`のステップをキャンセルできるようにします。`ExternalProject`のデフォルトの動作は、CMakeを使用してプロジェクトの設定、ビルド、インストールを試みることです。そのため、これらのステップの1つをスキップしなければならない場合、`CMAKE_NOOP`に設定して、代わりに何も実行されないようにします。

## `CMAKE_FORWARD_ARGS`

`3rdparty/CMakeLists.txt`に定義されている`CMAKE_FORWARD_ARGS`変数は、`CMAKE_ARGS`引数として`ExternalProject_Add`マクロに送られ（プロジェクトごとの引数も一緒に）、外部プロジェクトがCMakeプロジェクトとして設定されたときに使用されます。`ExternalProject_Add`の`CONFIGURE_COMMAND`または`BUILD_COMMAND`引数のいずれかが使用される場合、`CMAKE_ARGS`引数は無視されます。この変数は、コンパイラフラグなどのコンパイル構成がサードパーティの依存関係に適切に伝搬されることを保証します。

### `CMAKE_SSL_FORWARD_ARGS`

`3rdparty/CMakeLists.txt`で定義されている`CMAKE_SSL_FORWARD_ARGS`変数は、`CMAKE_FORWARD_ARGS`と似ていますが、OpenSSLを見つけてリンクする特定の外部プロジェクトにのみ使用されます。

## `LIBRARY_LINKAGE`

この変数は、`3rdparty/CMakeLists.txt`で使われるショートカットです。`BUILD_SHARED_LIBS`がtrueの場合は`SHARED`に、それ以外の場合は`STATIC`に設定されます。`SHARED`キーワードと`STATIC`キーワードは、ライブラリをどのように構築するかを宣言するために使用されますが、省略された場合は`BUILD_SHARED_LIBS`から自動的にタイプが推測されます。

## `MAKE_INCLUDE_DIR`

この関数は、`ExternalProject_Add`でビルドされたインポートライブラリのインクルードディレクトリの設定に関する[CMakeの問題][cmake-15052]を回避します。CMakeを快適に利用するために、`INTERFACE_INCLUDE_DIRECTORIES`を設定している各`IMPORTED`サードパーティ依存関係に対して、これを呼び出さなければなりません。例として、Glog:

```
MAKE_INCLUDE_DIR(glog)
```

[cmake-15052]: https://gitlab.kitware.com/cmake/cmake/issues/15052

## `GET_BYPRODUCTS`

この関数は、Ninjaジェネレーターがインポートされたライブラリを理解せず、代わりに`BUILD_BYPRODUCTS`を明示的に設定する必要があるという[CMakeの問題][cmake-060234]を回避します。これにより、単純に`ExternalProject_Add`とNinjaを使うことができます。Glogでは以下のようになります。:

```
GET_BYPRODUCTS(glog)
```

また、CMakeポリシー [CMP0058][]も参照してください。

[cmake-060234]: https://cmake.org/pipermail/cmake/2015-April/060234.html
[CMP0058]: https://cmake.org/cmake/help/latest/policy/CMP0058.html

## `PATCH_CMD`

CMakeの関数`PATCH_CMD`は、パッチファイルを与えられてパッチコマンドを生成します。パスが絶対パスでない場合は、現在のソースディレクトリに解決されます。コマンドは指定された変数名に格納されます。これは、サードパーティの依存関係に簡単にパッチを当てるために使われます。Glogの場合は以下のようになります。:

```
PATCH_CMD(GLOG_PATCH_CMD glog-${GLOG_VERSION}.patch)
ExternalProject_Add(
  ${GLOG_TARGET}
  ...
  PATCH_COMMAND     ${GLOG_PATCH_CMD})
```

実装は`3rdparty/cmake/PatchCommand.cmake`にあります。

### Windows `patch.exe`

Linuxでの`patch`の使用は簡単ですが、Windowsで同じことをするには少し手間がかかります。`PATH_CMD` は以下をカプセル化します。:

* `patch.exe` のキャッシュ変数 `PATCHEXE_PATH` をチェックします。
* 既定の場所にある`patch.exe`を検索します。
* `patch.exe`とカスタムマニフェストを一時ディレクトリにコピーします。
* UACプロンプトが表示されないようにマニフェストを適用します。
* パッチを当てた`patch.exe`を使用します。

そのため、`PATCH_CMD`を使えば、Linuxと同じように、管理者のプロンプトを必要とせずにパッチを適用することができます。

なお、Windowsでは、パッチファイルの改行コードはCRLFでなければなりません。改行コードがLFのファイルはエラーになります。"Assertion failed, hunk, file patch.c, line 343" このため、`git config core.autocrlf true`でMesos repoをチェックアウトする必要があります。
