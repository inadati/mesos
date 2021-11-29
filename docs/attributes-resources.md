---
title: Apache Mesos - Attributes and Resources
layout: documentation
---

# Mesosの属性とリソース

Mesosには、クラスタを構成するエージェントを記述する2つの基本的な方法があります。1つはMesosマスターが管理するもので、もう1つはクラスタを使用するフレームワークに単純に渡されるものです。

## Types

MesosのAttributesとResourceでサポートされている値の種類は、スカラー、レンジ、セット、テキストです。

これらの型の定義を以下に示します。:

    scalar : floatValue

    floatValue : ( intValue ( "." intValue )? ) | ...

    intValue : [0-9]+

    range : "[" rangeValue ( "," rangeValue )* "]"

    rangeValue : scalar "-" scalar

    set : "{" text ( "," text )* "}"

    text : [a-zA-Z0-9_/.-]

## Attributes

属性は、Mesosがフレームワークにオファーを送信する際に渡すキーと値のペア（値はオプション）です。属性の値には、スカラー、レンジ、テキストの3つのタイプがあります。

    attributes : attribute ( ";" attribute )*

    attribute : text ":" ( scalar | range | text )

同じキーに対応する複数の属性を設定することは、スケジューラ側とMesos側の両方で、属性に基づくオファーのフィルタリングが複雑になるため、非常にお勧めできません（将来的には禁止される可能性もあります）。

## Resources
Mesosでは、スカラー、レンジ、セットの3種類のリソースを管理できます。これらは、Mesosエージェントが提供するさまざまなリソースを表すために使用されます。例えば、スカラーリソースタイプは、エージェントのメモリ量を表すのに使用できます。スカラーリソースは浮動小数点数で表され、小数の値を指定することができます（「1.5CPU」など）。Mesosはスカラリソースの精度として10進数3桁のみをサポートしています（例：「1.5123CPU」を予約することは「1.512CPU」を予約することと同等とみなされます）。GPUの場合、Mesosは整数値のみをサポートします。

リソースは、JSON配列またはキーと値のペアをセミコロンで区切った文字列で指定できます。以下の例を見てもJSONのフォーマットに疑問がある場合は、`include/mesos/mesos.proto`の`Resource` protobufメッセージ定義を確認してください。

JSONとしては:

    [
      {
        "name": "<resource_name>",
        "type": "SCALAR",
        "scalar": {
          "value": <resource_value>
        }
      },
      {
        "name": "<resource_name>",
        "type": "RANGES",
        "ranges": {
          "range": [
            {
              "begin": <range_beginning>,
              "end": <range_ending>
            },
            ...
          ]
        }
      },
      {
        "name": "<resource_name>",
        "type": "SET",
        "set": {
          "item": [
            "<first_item>",
            ...
          ]
        },
        "role": "<role_name>"
      },
      ...
    ]

key-valueのペアの一覧としては:

    resources : resource ( ";" resource )*

    resource : key ":" ( scalar | range | set )

    key : text ( "(" resourceRole ")" )?

    resourceRole : text | "*"

なお、`resourceRole`には有効なロール名を指定する必要があります。詳細は、[ロール](roles.md)のドキュメントを参照してください。

## 規定の用途と規約

リソースには、あらかじめ定義された動作があるものがいくつかあります。:

  - `cpus`
  - `gpus`
  - `disk`
  - `mem`
  - `ports`

`disk`および`mem`のリソースは、メガバイト単位で指定されることに注意してください。マスターのユーザ・インターフェースは、リソースの値をより人間が読みやすい形式に変換します。例えば、`15000`という値は`14.65GB`と表示されます。

`cpus` および `mem` リソースを持たないエージェントは、どのフレームワークにもそのリソースがアドバタイズされません。

## 例
デフォルトでは、`mesos-agent`の起動時に、Mesosはローカルマシンで利用可能なリソースを自動検出しようとします。また、エージェントが利用可能にするリソースを明示的に設定することもできます。

以下は、Mesosエージェントでリソースを設定する方法の例です。:

    --resources='cpus:24;gpus:2;mem:24576;disk:409600;ports:[21000-24000,30000-34000];bugs(debug_role):{a,b,c}'

    --resources='[{"name":"cpus","type":"SCALAR","scalar":{"value":24}},{"name":"gpus","type":"SCALAR","scalar":{"value":2}},{"name":"mem","type":"SCALAR","scalar":{"value":24576}},{"name":"disk","type":"SCALAR","scalar":{"value":409600}},{"name":"ports","type":"RANGES","ranges":{"range":[{"begin":21000,"end":24000},{"begin":30000,"end":34000}]}},{"name":"bugs","type":"SET","set":{"item":["a","b","c"]},"role":"debug_role"}]'

また、次のような内容のファイル`resources.txt`が与えられます。:

    [
      {
        "name": "cpus",
        "type": "SCALAR",
        "scalar": {
          "value": 24
        }
      },
      {
        "name": "gpus",
        "type": "SCALAR",
        "scalar": {
          "value": 2
        }
      },
      {
        "name": "mem",
        "type": "SCALAR",
        "scalar": {
          "value": 24576
        }
      },
      {
        "name": "disk",
        "type": "SCALAR",
        "scalar": {
          "value": 409600
        }
      },
      {
        "name": "ports",
        "type": "RANGES",
        "ranges": {
          "range": [
            {
              "begin": 21000,
              "end": 24000
            },
            {
              "begin": 30000,
              "end": 34000
            }
          ]
        }
      },
      {
        "name": "bugs",
        "type": "SET",
        "set": {
          "item": [
            "a",
            "b",
            "c"
          ]
        },
        "role": "debug_role"
      }
    ]

以下のようなことが可能です:

    $ path/to/mesos-agent --resources=file:///path/to/resources.txt ...

今回のケースでは、スカラー、範囲、セットの3種類のリソースが5つあります。`cpus`，`gpus`，`mem`，`disk`というスカラー，`ports`という範囲，`bugs`というセットです。`bugs`には`debug_role`というロールが割り当てられていますが，他のリソースにはロールが指定されていないため，デフォルトロールが割り当てられています。

**注意:**「デフォルトロール」は `--default_role` フラグで設定できます。

* `cpus`というスカラー、値は`24`
* `gpus`というスカラ、値は`2`
* `mem` という名のスカラ、値は `24576`
* `disk`という名のスカラ、値は`409600`
* `ports` という範囲には、`21000` から `24000`、`30000` から `34000`（いずれも含む）の値が入ります。
* `bugs`と呼ばれるセット、値は`a`、`b`、`c`で、役割は`debug_role`に割り当てられています。

Mesosエージェントの属性を設定するには、`mesos-agent`の`--attributes`コマンドラインフラグを使用できます。:

    --attributes='rack:abc;zone:west;os:centos5;level:10;keys:[1000-1500]'

その結果、以下の5つの属性が設定されることになります。:

* テキスト値がabcの`rack`
* テキスト値 west を持つ`zone`
* テキスト値 `centos5` を持つ `os`
* スカラー値10の`level`
* 範囲値`1000`～`1500`（含む）の`keys`
