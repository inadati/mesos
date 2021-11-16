---
title: Apache Mesos - The Mesos Replicated Log
layout: documentation
---

# Mesosの複製されたログ
Mesosには、複製されたフォールトトレラントなアペンドオンリーログを作成するためのライブラリが用意されています。Mesosマスターはこのライブラリを使用して、複製された耐久性のある方法でクラスタの状態を保存します。このライブラリは、複製されたフレームワークの状態を保存したり、一般的な[複製されたステートマシン](https://en.wikipedia.org/wiki/State_machine_replication)パターンを実装したりするフレームワークでも使用できます。

## 複製されたログとは？

![Aurora and the Replicated Log](images/log-cluster.png)

複製されたログでは、ログエントリの保存は追記のみで、各ログエントリには任意のデータを含めることができます。ログは複製されるため、各ログエントリはシステム内に複数のコピーを持ちます。レプリケーションは、フォールトトレランスとハイアベイラビリティを提供します。以下の例では、Mesos上で動作するフォールト・トレラント・スケジューラ（すなわちフレームワーク）である[Apache Aurora](https://aurora.apache.org/)を使用して、典型的なレプリケートされたログのセットアップを示しています。

上図のように、複数のAuroraインスタンスが同時に動作しており（高可用性のため）、そのうちの1つがリーダーに選ばれています。Auroraが動作している各ホストには、ログのレプリカがあります。Auroraは、ログAPIを含むシンライブラリを介して、複製されたログにアクセスできます。

通常、リーダーはログにデータを追加する唯一のマスターです。各ログエントリは複製され、システム内のすべてのレプリカに送信されます。レプリカは強い一貫性を持っています。つまり、すべてのレプリカは、各ログ・エントリの値に同意します。ログが複製されているので、Auroraがフェイルオーバーを決定する際に、リモートホストからログをコピーする必要はありません。

## 利用例
複製されたログは、さまざまな分散型アプリケーションの構築に利用できます。たとえば、Auroraでは、複製されたログを使って、すべてのタスクの状態やジョブの設定を保存しています。また、Mesosマスターのレジストリは、複製されたログを活用して、クラスタ内のすべてのエージェントに関する情報を保存します。

複製されたログは、アプリケーションがレプリケートされた状態を強い一貫性のある方法で管理するためによく使われます。これを実現する1つの方法は、各ログエントリに状態を変更する操作を保存し、分散アプリケーションのすべてのインスタンスが同じ初期状態（例えば、空の状態）に同意することです。複製されたログは、各アプリケーション・インスタンスが同じ順序で同じログ・エントリのシーケンスを観察することを保証します。状態変更操作の適用が決定論的である限り、これによりすべてのアプリケーション・インスタンスが互いに一貫性を保つことができます。アプリケーションのいずれかのインスタンスがクラッシュした場合、初期状態から始めて、ログに記録されたすべての変異を順に適用することで、複製された状態の現在のバージョンを再構築することができます。

ログが大きくなりすぎた場合、アプリケーションはスナップショットを書き出し、そのスナップショット以前に発生したすべてのログエントリを削除することができます。このアプローチを使用すると、Mesosでは、複製されたログをバックエンドとして[分散状態](https://github.com/apache/mesos/blob/master/src/state/state.hpp)の抽象化を公開することになります。

同様に、複製されたログを使用して、[複製された状態のマシン](https://en.wikipedia.org/wiki/State_machine_replication)を構築することができます。このシナリオでは、各ログ・エントリはステート・マシン・コマンドを含んでいます。複製は強く一貫しているので、すべてのサーバーは同じコマンドを同じ順序で実行します。

## 実装

![Replicated Log Architecture](images/log-architecture.png)

複製されたログは、すべてのレプリカがすべてのログエントリの値に同意することを保証するために、[Paxosのコンセンサスアルゴリズム](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)を使用しています。これは、[このスライド](https://ramcloud.stanford.edu/~ongaro/userstudy/paxos.pdf)で説明されていることと同様です。Paxosに慣れている読者は、このセクションを読み飛ばしても構いません。

上の図は、実装の概要です。ユーザーがログにデータを追加したい場合、システムはログライターを作成します。ログ・ライターは内部でコーディネータを作成します。コーディネーターはすべてのレプリカに連絡を取り、Paxosアルゴリズムを実行して、すべてのレプリカが追記されたデータについて合意することを確認します。コーディネーターは、[プロポーザー](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)と呼ばれることもあります。

各レプリカは、ログ・エントリの配列を保持しています。配列のインデックスがログの位置になります。各ログ・エントリは、ユーザが書き込んだ値、関連するPaxosの状態、そして、このログ・エントリの値が合意されたことを意味する学習ビットの3つの要素で構成されています。したがって，我々の実装では，レプリカは[Acceptor](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)でもあり[Learner](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)でもあります。

### 1つのログエントリの合意を得るために
Paxosのラウンドは、すべてのレプリカが1つのログ・エントリの値について合意に達するのを助けることができます。ラウンドには2つのフェーズがあります：promiseフェーズとwriteフェーズです。[Paxosの原著論文](https://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf)とは若干異なる用語を使用していることに注意してください。我々の実装では、原著論文のprepareおよびacceptフェーズは、それぞれ promiseおよびwriteフェーズと呼ばれています。その結果、prepareリクエスト(レスポンス)はpromiseリクエスト(レスポンス)と呼ばれ、acceptリクエスト(レスポンス)はwriteリクエスト(レスポンス)と呼ばれています。

位置pのログに値Xを追加するために、コーディネータはまず、proposal番号nを持つすべてのレプリカにpromiseリクエストをブロードキャストし、nより小さいproposal番号の要求（promise／writeリクエスト）には応答しないことをレプリカに保証してもらう。

promiseリクエストを受け取ると、各レプリカは自分のPaxosの状態をチェックし、以前に配ったpromiseに応じて、そのリクエストに安全に応答できるかどうかを判断します。レプリカがpromiseを与えることができる場合（つまり、proposal番号のチェックをパスした場合）、まずそのpromise（proposal番号n）をディスクに永続化し、promiseレスポンスを返信します。もしレプリカが以前に書き込まれていた(つまりwriteリクエストを受け入れた)場合は、以前に書き込まれた値と、そのwriteリクエストで使われたproposal番号を、これから送ろうとしているpromiseレスポンスに含める必要があります。

[クォーラム](https://en.wikipedia.org/wiki/Quorum_%28distributed_computing%29)のレプリカからpromiseレスポンスを受け取ると、コーディネーターはまず、それらのレスポンスから以前に書き込まれた値が存在するかどうかをチェックします。以前に書き込まれた値が見つかった場合は、そのログエントリに対して既に値が合意されている可能性が高いため、追加操作を続行することはできません。これはPaxosの重要なアイデアの1つで、一貫性を確保するために書き込むことができる値を制限することです。

以前に書き込まれた値が見つからない場合、コーディネーターは、値Xとproposal番号nを持つすべてのレプリカにwriteリクエストをブロードキャストします。writeリクエストを受け取ると、各レプリカは自分が与えたpromiseを再度確認し、writeリクエストのproposal番号が自分が保証したproposal番号と同じかそれ以上であれば、writeレスポンスを返します。コーディネーターがレプリカの定足数からwriteレスポンスを受け取ると、追加操作は成功します。

### Multi-Paxosを使用した追加レイテンシの最適化

複製されたログを実装するための1つの素朴な解決策は、各ログエントリに対して完全なPaxosラウンド（promiseフェーズとwriteフェーズ）を実行することです。[Paxosの原著論文](https://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf)で議論されているように、リーダーが比較的安定している場合、Multi-Paxosを使用することで、ほとんどの追記操作でpromiseフェーズが不要となり、結果的にパフォーマンスが向上します。

これを実現するために、我々は暗黙のpromiseリクエストと呼ばれる新しいタイプのpromiseリクエストを導入しました。暗黙のpromiseリクエストは、(潜在的に無限の)ログ・エントリのセットに対するバッチ処理されたプロミス・リクエストと見なすことができます。暗黙のpromiseリクエストをブロードキャストすることは、概念的には、まだ値が合意されていないすべてのログエントリに対してpromiseリクエストをブロードキャストすることと同じです。コーディネータによってブロードキャストされた暗黙のpromiseリクエストがレプリカのクォーラムによって受け入れられた場合、このコーディネータは、まだ値が合意されていないログ・エントリに追加したい場合には、プロミス・フェーズを実行する必要はなくなる。したがって、この場合のコーディネーターは、選出された（別名、リーダー）と呼ばれ、複製されたログへの排他的なアクセス権を持っています。選出されたコーディネーターは、他のコーディネーターがより高いproposal番号で暗黙のpromiseリクエストをブロードキャストした場合、降格（または排他的アクセスを失う）する可能性があります。

残る問題は、まだ値が合意されていないログ・エントリをどうやって見つけるかということです。非常にシンプルな解決策があります。レプリカが暗黙のpromiseリクエストを受け入れた場合、そのレプリカの最大の既知のログ位置を応答に含めます。選出されたコーディネータは、pより大きい位置のログエントリのみを追加します。pはこれらの応答で見られるどのログ位置よりも大きいです。

Multi-Paxosは、リーダーが安定していればパフォーマンスが向上します。複製されたログ自体はリーダーの選出を行いません。代わりに、複製されたログのユーザーが安定したリーダーを選ぶことに依存します。例えば、Auroraはリーダーの選出に[ZooKeeper](https://zookeeper.apache.org/)を使用しています。

### ローカル読み取りの有効化
上述したように、私たちの実装では、各レプリカはAcceptorでもあり、Learnerでもあります。各レプリカをLearnerとして扱うことで、他のレプリカを介さずにローカルな読み取りを行うことができます。あるログ・エントリの値が合意されると、コーディネータは学習済みメッセージをすべてのレプリカにブロードキャストします。レプリカは学習済みメッセージを受け取ると、対応するログ・エントリの学習済みビットを設定し、そのログ・エントリの値が合意されたことを示します。学んだビットが設定されていれば、そのログ・エントリは「学習済み」になります。コーディネータはレプリカからの確認応答を待つ必要はありません。

読み取りを実行するために、ログリーダーは基礎となるローカルレプリカを直接検索します。対応するログエントリが学習されていれば、ログリーダはその値をユーザに返すだけです。そうでなければ、合意した値を発見するためにPaxosのフルラウンドが必要です。選出されたコーディネーターがいるレプリカが、常にすべてのログ・エントリを学習していることを確認します。これは、コーディネーターが選出された後に、学習されていないログ・エントリに対してPaxosのフルラウンドを実行することで実現しています。

### ガベージコレクションによるログサイズの削減

ログが大きくなった場合、アプリケーションはログを切り詰めるという選択肢を持っています。切り捨てを実行するには、特別なログ・エントリを追加します。その値は、ユーザが切り捨てたいログの位置です。レプリカは、この特別なログ・エントリを学習すると、実際にログを切り詰めることができます。

### 固有のproposal番号

[Paxosの研究論文](https://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf)の多くは、各proposal番号がグローバルに一意であり、コーディネーターは常にシステム内の他のproposal番号よりも大きいproposal番号を考え出すことができると仮定しています。しかし、これを実装することは、特に分散環境においては、簡単ではありません。[一部の研究者は、グローバルに一意なサーバIDを各proposal番号に連結することを提案しています。](https://ramcloud.stanford.edu/~ongaro/userstudy/paxos.pdf)しかし、各サーバーにグローバルに一意なIDを生成する方法はまだ明らかではありません。

我々のソリューションは、上記のような仮定をしていません。コーディネーターは、最初は任意のproposal番号を使うことができます。promiseフェーズで、レプリカがコーディネーターが使ったproposal番号よりも大きいproposal番号を知っている場合、そのレプリカはコーディネーターに最大の既知のproposal番号を送り返す。コーディネータは、より大きなproposal番号でpromiseフェーズを再試行します。

ライブロックを避けるために（例えば、2つのコーディネーターが完了した場合）、各リトライの前にTから2Tの間のランダムな遅延を注入します。Tは慎重に選ぶ必要があります。一方では、T >> ブロードキャスト時間にして、他が起動する前に1つのコーディネーターが通常タイムアウトして勝利するようにします。一方で、Tはできるだけ小さくして、待ち時間を短縮したいと考えています。現在、私たちはT = 100msを使用しています。このアイデアは、実は[Raft](https://ramcloud.stanford.edu/wiki/download/attachments/11370504/raft.pdf)からの借用です。

## Automatic replica recovery

The algorithm described above has a critical vulnerability: if a replica loses its durable state (i.e., log files) due to either disk failure or operational error, that replica may cause inconsistency in the log if it is simply restarted and re-added to the group. The operator needs to stop the application on all hosts, copy the log files from the leader's host, and then restart the application. Note that the operator cannot copy the log files from an arbitrary replica because copying an unlearned log entry may falsely assemble a quorum for an incorrect value, leading to inconsistency.

To avoid the need for operator intervention in this situation, the Mesos replicated log includes support for _auto recovery_. As long as a quorum of replicas is working properly, the users of the application won't notice any difference.

### Non-voting replicas

To enable auto recovery, a key insight is that a replica that loses its durable state should not be allowed to respond to requests from coordinators after restart. Otherwise, it may introduce inconsistency in the log as it could have accepted a promise/write request which it would not have accepted if its previous Paxos state had not been lost.

To solve that, we introduce a new status variable for each replica. A normal replica is said in VOTING status, meaning that it is allowed to respond to requests from coordinators. A replica with no persisted state is put in EMPTY status by default. A replica in EMPTY status is not allowed to respond to any request from coordinators.

A replica in EMPTY status will be promoted to VOTING status if the following two conditions are met:

1. a sufficient amount of missing log entries are recovered such that if other replicas fail, the remaining replicas can recover all the learned log entries, and
2. its future responses to a coordinator will not break any of the promises (potentially lost) it has given out.

In the following, we discuss how we achieve these two conditions.

### Catch-up

To satisfy the above two conditions, a replica needs to perform _catch-up_ to recover lost states. In other words, it will run Paxos rounds to find out those log entries whose values that have already been agreed. The question is how many log entries the local replica should catch-up before the above two conditions can be satisfied.

We found that it is sufficient to catch-up those log entries from position _begin_ to position _end_ where _begin_ is the smallest position seen in a quorum of VOTING replicas and _end_ is the largest position seen in a quorum of VOTING replicas.

Here is our correctness argument. For a log entry at position _e_ where _e_ is larger than _end_, obviously no value has been agreed on. Otherwise, we should find at least one VOTING replica in a quorum of replicas such that its end position is larger than _end_. For the same reason, a coordinator should not have collected enough promises for the log entry at position _e_. Therefore, it's safe for the recovering replica to respond requests for that log entry. For a log entry at position _b_ where _b_ is smaller than _begin_, it should have already been truncated and the truncation should have already been agreed. Therefore, allowing the recovering replica to respond requests for that position is also safe.

### Auto initialization

Since we don't allow an empty replica (a replica in EMPTY status) to respond to requests from coordinators, that raises a question for bootstrapping because initially, each replica is empty. The replicated log provides two choices here. One choice is to use a tool (`mesos-log`) to explicitly initialize the log on each replica by setting the replica's status to VOTING, but that requires an extra step when setting up an application.

The other choice is to do automatic initialization. Our idea is: we allow a replica in EMPTY status to become VOTING immediately if it finds all replicas are in EMPTY status. This is based on the assumption that the only time _all_ replicas are in EMPTY status is during start-up. This may not be true if a catastrophic failure causes all replicas to lose their durable state, and that's exactly the reason we allow conservative users to disable auto-initialization.

To do auto-initialization, if we use a single-phase protocol and allow a replica to directly transit from EMPTY status to VOTING status, we may run into a state where we cannot make progress even if all replicas are in EMPTY status initially. For example, say the quorum size is 2. All replicas are in EMPTY status initially. One replica will first set its status to VOTING because if finds all replicas are in EMPTY status. After that, neither the VOTING replica nor the EMPTY replicas can make progress. To solve this problem, we use a two-phase protocol and introduce an intermediate transient status (STARTING) between EMPTY and VOTING status. A replica in EMPTY status can transit to STARTING status if it finds all replicas are in either EMPTY or STARTING status. A replica in STARTING status can transit to VOTING status if it finds all replicas are in either STARTING or VOTING status. In that way, in our previous example, all replicas will be in STARTING status before any of them can transit to VOTING status.

## Non-leading VOTING replica catch-up

Starting with Mesos 1.5.0 it is possible to perform eventually consistent reads from a non-leading VOTING log replica. This makes possible to do additional work on non-leading framework replicas, e.g. offload some reading from a leader to standbys reduce failover time by keeping in-memory storage represented by the replicated log "hot".

To serve eventually consistent reads a replica needs to perform _catch-up_ to recover the latest log state in a manner similar to how it is done during [EMPTY replica recovery](#catch-up). After that the recovered positions can be replayed without fear of seeing "holes".

A truncation can take place during the non-leading replica catch-up. The replica may try to fill the truncated position if truncation happens after the replica has recovered _begin_ and _end_ positions, which may lead to producing inconsistent data during log replay. In order to protect against it we use a special tombstone flag that signals to the replica that the position was truncated and _begin_ needs to be adjusted. The replica is not blocked from truncations during or after catching-up, which means that the user may need to retry the catch-up procedure if positions that were recovered became truncated during log replay.

## Future work

Currently, replicated log does not support dynamic quorum size change, also known as _reconfiguration_. Supporting reconfiguration would allow us more easily to add, move or swap hosts for replicas. We plan to support reconfiguration in the future.
