## StackStormによる運用の自動化
[StackStorm で変わる運用](chapter1.md#stackstorm-で変わる運用)で述べられたもの以外にも、DevとOpsのワークフローをStackStormで結びつけることには様々なメリットがあります。

本章では、そのメリットの1つである「ARによる運用稼動の削減」について、Zabbix×StackStormを用いた例をご紹介します。

### ARという概念について
前提として、運用自動化のキーワードとして最近注目されている"AR = *Auto-Remediation*"という概念について説明します。

Auto-Remediation (直訳：自動修復)とは、その名の通り「障害が発生した際に自動で復旧すること」を示しています。

私の知る限り、サービス基盤レベルでARという言葉が使われ始めたのは、2011年にFacebookが発表した[Making Facebook Self-Healing](https://www.facebook.com/notes/facebook-engineering/making-facebook-self-healing/10150275248698920/)が初出です。

上記の記事で
> human engineers could focus on solving and preventing the larger, more complex outages.
と記されているように、ARを運用に取り入れることで「細々としたトラブル対応に時間を取らず、より大きな課題を解決するために時間を使う」ことができるようになります。

### StackStormによるARの実現
StackStormでも[Auto-Remediation Defined](https://stackstorm.com/2015/08/07/auto-remediation-defined/)で触れられているように、StackStormでもARを主要なユースケースの1つと位置付けているようです。

ここで、StackStormの掲げる"Event-Driven Automation"という思想に則り、ARの動作を考えてみます。

![Event-Driven Remidiate](images/st2-ar.png)

| 登場人物 | 機能 |
| --- | --- |
| Server | 自身の状態をMonitorに通知する |
| Monitor | Serverの状態を受け取り、任意のルールに従ってStackStormに通知する |
| StackStorm | Monitorからの通知をイベントとし、対象のServerをRemidiateする |

このように、非常に綺麗な構成でARの動作を定義することができます。

ZabbixやSensuなどの監視ツールはログの内容によって所定のスクリプトを実行する機能を備えていますが、StackStormという登場人物を増やすことによって
* 監視と処理の責任主体が明確になる
* 処理がActionとWorkflowによって定義されるので、再利用性が高まる

というメリットがあります。

では、実際にARの例を見ていきましょう。

### 運用例：DBクラスタの復旧
#### 状況設定
単純なDBクラスタの復旧を例にしたいと思います。
以下のような構成です。

//todo: 図を書く

さて、ここでロードバランサから「クラスタのうち1台に疎通できなくなった」というログがZabbixサーバに飛んできた場面からスタートです。

#### 前提条件
[StackStormによるARの実現](#StackStormによるARの実現)で示したようなAR処理を実現するためには、それぞれの登場人物に事前準備が必要です。
個々の具体的な設定手順について書くと本筋から外れてしまうので、以下に簡潔に記します。

| 登場人物 | 前提条件 |
| --- | --- |
| 

#### 処理フロー
例として、以下のようなフローで処理するものとします。

//todo: 図を書く

1. Zabbixによるログの分類
2. Zabbix→StackStormへのリクエスト発行
3. StackStormによる復旧処理の実行
	* 処理開始の通知
	* 復旧処理
	* 復旧判断
	* 処理終了の通知 or エスカレーション

項番について、具体的な処理と実装を見ていきましょう。

##### 1. Zabbixによるログの分類

