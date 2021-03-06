---

copyright:
  years: 2014, 2017
lastupdated: "2017-10-24"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}
{:tsSymptoms: .tsSymptoms}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}


# クラスターのトラブルシューティング
{: #cs_troubleshoot}

{{site.data.keyword.containershort_notm}} を使用する際は、ここに示すトラブルシューティング手法やヘルプの利用手法を検討してください。[{{site.data.keyword.Bluemix_notm}} システムの状況 ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/bluemix/support/#status) を確認することもできます。

いくつかの一般的な手順を実行することにより、クラスターが最新のものであることを確認できます。
- 定期的に[ワーカー・ノードをリブートして](cs_cli_reference.html#cs_worker_reboot)、IBM によってオペレーティング・システムに自動的にデプロイされる更新プログラムとセキュリティー・パッチがインストールされるようにします
- クラスターを {{site.data.keyword.containershort_notm}} 用の [Kubernetes の最新のデフォルト・バージョン](cs_versions.html)に更新します

{: shortdesc}

<br />


## クラスターのデバッグ
{: #debug_clusters}

クラスターをデバッグするためのオプションを確認し、障害の根本原因を探します。

1.  クラスターをリストし、クラスターの `State` を見つけます。

  ```
bx cs clusters```
  {: pre}

2.  クラスターの `State` を確認します。

  <table summary="表の行はすべて左から右に読みます。1 列目はクラスターの状態、2 列目は説明です。">
    <thead>
    <th>クラスターの状態</th>
    <th>説明</th>
    </thead>
    <tbody>
      <tr>
        <td>Deploying</td>
        <td>Kubernetes マスターがまだ完全にデプロイされていません。クラスターにアクセスできません。</td>
       </tr>
       <tr>
        <td>Pending</td>
        <td>Kubernetes マスターはデプロイされています。ワーカー・ノードはプロビジョン中であるため、まだクラスターでは使用できません。クラスターにはアクセスできますが、アプリをクラスターにデプロイすることはできません。</td>
      </tr>
      <tr>
        <td>Normal</td>
        <td>クラスター内のすべてのワーカー・ノードが稼働中です。クラスターにアクセスし、アプリをクラスターにデプロイできます。</td>
     </tr>
     <tr>
        <td>Warning</td>
        <td>クラスター内の 1 つ以上のワーカー・ノードが使用不可です。ただし、他のワーカー・ノードが使用可能であるため、ワークロードを引き継ぐことができます。</td>
     </tr>
     <tr>
      <td>Critical</td>
      <td>Kubernetes マスターにアクセスできないか、クラスター内のワーカー・ノードがすべてダウンしています。</td>
     </tr>
    </tbody>
  </table>

3.  クラスターが **Warning** 状態または **Critical** 状態の場合、あるいは **Pending** 状態が長時間続いている場合は、ワーカー・ノードの状態を確認してください。クラスターが **Deploying** 状態の場合は、クラスターが完全にデプロイされるまで待ってからクラスターの正常性を確認してください。**Normal** 状態のクラスターは、正常と見なされるので、この時点ではアクションは不要です。

  ```
    bx cs workers <cluster_name_or_id>
    ```
  {: pre}

  <table summary="表の行はすべて左から右に読みます。1 列目はクラスターの状態、2 列目は説明です。">
    <thead>
    <th>ワーカー・ノードの状態</th>
    <th>説明</th>
    </thead>
    <tbody>
      <tr>
       <td>Unknown</td>
       <td>次のいずれかの理由で、Kubernetes マスターにアクセスできません。<ul><li>Kubernetes マスターの更新を要求しました。更新中は、ワーカー・ノードの状態を取得できません。
</li><li>ワーカー・ノードを保護している追加のファイアウォールが存在するか、最近ファイアウォールの設定が変更された可能性があります。{{site.data.keyword.containershort_notm}} では、ワーカー・ノードと Kubernetes マスター間で通信を行うには、特定の IP アドレスとポートが開いている必要があります。
詳しくは、[ワーカー・ノードが再ロード・ループにはまった場合](#cs_firewall)を参照してください。
</li><li>Kubernetes マスターがダウンしています。[{{site.data.keyword.Bluemix_notm}} サポート・チケット](/docs/support/index.html#contacting-support)を開いて、{{site.data.keyword.Bluemix_notm}} サポートに連絡してください。</li></ul></td>
      </tr>
      <tr>
        <td>Provisioning</td>
        <td>ワーカー・ノードはプロビジョン中であるため、まだクラスターでは使用できません。CLI 出力の **Status** 列で、プロビジョニングのプロセスをモニターできます。ワーカー・ノードが長時間この状態であるのに、**Status** 列で処理の進行が見られない場合は、次のステップに進んで、プロビジョニング中に問題が発生していないか調べてください。</td>
      </tr>
      <tr>
        <td>Provision_failed</td>
        <td>ワーカー・ノードをプロビジョンできませんでした。次のステップに進んで、障害に関する詳細を調べてください。</td>
      </tr>
      <tr>
        <td>Reloading</td>
        <td>ワーカー・ノードは再ロード中であるため、クラスターでは使用できません。CLI 出力の **Status** 列で、再ロードのプロセスをモニターできます。ワーカー・ノードが長時間この状態であるのに、**Status** 列で処理の進行が見られない場合は、次のステップに進んで、再ロード中に問題が発生していないか調べてください。</td>
       </tr>
       <tr>
        <td>Reloading_failed</td>
        <td>ワーカー・ノードを再ロードできませんでした。次のステップに進んで、障害に関する詳細を調べてください。</td>
      </tr>
      <tr>
        <td>Normal</td>
        <td>ワーカー・ノードは完全にプロビジョンされ、クラスターで使用できる状態です。</td>
     </tr>
     <tr>
        <td>Warning</td>
        <td>ワーカー・ノードが、メモリーまたはディスク・スペースの限度に達しています。</td>
     </tr>
     <tr>
      <td>Critical</td>
      <td>ワーカー・ノードがディスク・スペースを使い尽くしました。</td>
     </tr>
    </tbody>
  </table>

4.  ワーカー・ノードの詳細情報をリストします。

  ```
  bx cs worker-get <worker_node_id>
  ```
  {: pre}

5.  一般的なエラー・メッセージを確認し、解決方法を調べます。

  <table>
    <thead>
    <th>エラー・メッセージ</th>
    <th>説明と解決</thead>
    <tbody>
      <tr>
        <td>{{site.data.keyword.Bluemix_notm}} Infrastructure Exception: Your account is currently prohibited from ordering 'Computing Instances'.</td>
        <td>ご使用の IBM Bluemix Infrastructure (SoftLayer) アカウントは、コンピュート・リソースの注文を制限されている可能性があります。[{{site.data.keyword.Bluemix_notm}} サポート・チケット](/docs/support/index.html#contacting-support)を開いて、{{site.data.keyword.Bluemix_notm}} サポートに連絡してください。</td>
      </tr>
      <tr>
        <td>{{site.data.keyword.Bluemix_notm}} Infrastructure Exception: Could not place order. There are insufficient resources behind router 'router_name' to fulfill the request for the following guests: 'worker_id'.</td>
        <td>選択した VLAN に関連付けられているデータ・センター内のポッドのスペースが不足しているため、ワーカー・ノードをプロビジョンできません。以下の選択肢があります。
<ul><li>別のデータ・センターを使用してワーカー・ノードをプロビジョンします。使用可能なデータ・センターをリストするには、<code>bx cs locations</code> を実行します。
<li>データ・センター内の別のポッドに関連付けられているパブリック VLAN とプライベート VLAN の既存のペアがある場合は、代わりにその VLAN ペアを使用します。<li>[{{site.data.keyword.Bluemix_notm}} サポート・チケット](/docs/support/index.html#contacting-support)を開いて、{{site.data.keyword.Bluemix_notm}} サポートに連絡してください。</ul></td>
      </tr>
      <tr>
        <td>{{site.data.keyword.Bluemix_notm}} Infrastructure Exception: Could not obtain network VLAN with id: &lt;vlan id&gt;.</td>
        <td>次のいずれかの理由で、選択した VLAN ID が見つからなかったため、ワーカー・ノードをプロビジョンできませんでした。
<ul><li>VLAN ID ではなく VLAN 番号を指定した可能性があります。VLAN 番号の長さは 3 桁または 4 桁ですが、VLAN ID の長さは 7 桁です。VLAN ID を取得するには、<code>bx cs vlans &lt;location&gt;</code> を実行してください。
<li>ご使用の IBM Bluemix Infrastructure (SoftLayer) アカウントに VLAN ID が関連付けられていない可能性があります。アカウントの使用可能な VLAN ID をリストするには、<code>bx cs vlans &lt;location&gt;</code> を実行します。IBM Bluemix Infrastructure (SoftLayer) アカウントを変更するには、[bx cs credentials-set](cs_cli_reference.html#cs_credentials_set) を参照してください。</ul></td>
      </tr>
      <tr>
        <td>SoftLayer_Exception_Order_InvalidLocation: The location provided for this order is invalid. (HTTP 500)</td>
        <td>ご使用の IBM Bluemix Infrastructure (SoftLayer) は、選択したデータ・センター内のコンピュート・リソースを注文するようにセットアップされていません。[{{site.data.keyword.Bluemix_notm}} サポート](/docs/support/index.html#contacting-support)に問い合わせて、アカウントが正しくセットアップされているか確認してください。</td>
       </tr>
       <tr>
        <td>{{site.data.keyword.Bluemix_notm}} Infrastructure Exception: The user does not have the necessary {{site.data.keyword.Bluemix_notm}} Infrastructure permissions to add servers
</br></br>
        {{site.data.keyword.Bluemix_notm}} Infrastructure Exception: 'Item' must be ordered with permission.</td>
        <td>IBM Bluemix Infrastructure (SoftLayer) ポートフォリオからワーカー・ノードをプロビジョンするために必要なアクセス権限がない可能性があります。[IBM Bluemix Infrastructure (SoftLayer) ポートフォリオへのアクセス権限を構成して標準の Kubernetes クラスターを作成する](cs_planning.html#cs_planning_unify_accounts)を参照してください。</td>
      </tr>
    </tbody>
  </table>

<br />


## アプリ・デプロイメントのデバッグ
{: #debug_apps}

アプリ・デプロイメントをデバッグするためのオプションを確認し、障害の根本原因を探します。

1. `describe` コマンドを実行して、サービス・リソースまたはデプロイメント・リソース内の異常を見つけます。

 例:
 <pre class="pre"><code>kubectl describe service &lt;service_name&gt; </code></pre>

2. [コンテナーが ContainerCreating 状態で停滞しているかどうかを確認します](#stuck_creating_state)。

3. クラスターが `Critical` 状態かどうかを確認します。クラスターが `Critical` 状態の場合、ファイアウォール・ルールを調べて、マスターがワーカー・ノードと通信できることを検査します。

4. サービスが正しいポートで listen していることを確認します。
   1. ポッドの名前を取得します。
     <pre class="pre"><code>kubectl get pods</code></pre>
   2. コンテナーにログインします。
     <pre class="pre"><code>kubectl exec -it &lt;pod_name&gt; -- /bin/bash</code></pre>
   3. コンテナー内からアプリに対して curl を実行します。ポートにアクセスできない場合、サービスは正しいポートで listen していないか、またはアプリに問題が生じている可能性があります。正しいポートを指定してサービスの構成ファイルを更新して再デプロイするか、またはアプリに存在する可能性がある問題を調査します。
     <pre class="pre"><code>curl localhost: &lt;port&gt;</code></pre>

5. サービスがポッドに正しくリンクされていることを確認します。
   1. ポッドの名前を取得します。
     <pre class="pre"><code>kubectl get pods</code></pre>
   2. コンテナーにログインします。
     <pre class="pre"><code>kubectl exec -it &lt;pod_name&gt; -- /bin/bash</code></pre>
   3. サービスのクラスター IP アドレスとポートに対して curl を実行します。IP アドレスとポートにアクセスできない場合、サービスのエンドポイントを調べます。エンドポイントがない場合は、サービスのセレクターがポッドと一致していません。エンドポイントがある場合は、サービスの宛先ポート・フィールドを調べて、ポッドに使用されているポートが宛先ポートと同じであることを確認します。
     <pre class="pre"><code>curl &lt;cluster_IP&gt;:&lt;port&gt;</code></pre>

6. Ingress サービスの場合、クラスター内からサービスにアクセスできることを検証します。
   1. ポッドの名前を取得します。
     <pre class="pre"><code>kubectl get pods</code></pre>
   2. コンテナーにログインします。
     <pre class="pre"><code>kubectl exec -it &lt;pod_name&gt; -- /bin/bash</code></pre>
   2. Ingress サービス用に指定された URL に対して curl を実行します。URL にアクセスできない場合、クラスターと外部エンドポイントの間にファイアウォールの問題が生じていないかを調べます。
     <pre class="pre"><code>curl &lt;host_name&gt;.&lt;domain&gt;</code></pre>

<br />


## kubectl のローカル・クライアントとサーバーのバージョンの特定

ローカルまたはクラスターで実行されている Kubernetes CLI のバージョンを調べるには、以下のコマンドを実行して、バージョンを確認します。


```
kubectl version  --short```
{: pre}

出力例:


```
        Client Version: v1.5.6
        Server Version: v1.5.6
        ```
{: screen}

<br />


## クラスターの作成中に IBM Bluemix Infrastructure (SoftLayer) アカウントに接続できない
{: #cs_credentials}

{: tsSymptoms}
新しい Kubernetes クラスターを作成すると、以下のメッセージを受け取ります。


```
We were unable to connect to your IBM Bluemix Infrastructure (SoftLayer) account. Creating a standard cluster requires that you have either a Pay-As-You-Go account that is linked to an IBM Bluemix Infrastructure (SoftLayer) account term or that you have used the IBM
{{site.data.keyword.Bluemix_notm}} Container Service CLI to set your {{site.data.keyword.Bluemix_notm}} Infrastructure API keys.
```
{: screen}

{: tsCauses}
リンクされていない {{site.data.keyword.Bluemix_notm}} アカウントのユーザーは、新しい従量課金アカウントを作成するか、{{site.data.keyword.Bluemix_notm}} CLI を使用して IBM Bluemix Infrastructure (SoftLayer) API キーを手動で追加する必要があります。

{: tsResolve}
{{site.data.keyword.Bluemix_notm}} アカウントの資格情報を追加するには、以下のようにします。

1.  IBM Bluemix Infrastructure (SoftLayer) 管理者に連絡して、IBM Bluemix Infrastructure (SoftLayer) のユーザー名と API キーを入手します。

    **注:** 標準クラスターを正常に作成するには、使用する IBM Bluemix Infrastructure (SoftLayer) アカウントに SuperUser 権限がセットアップされている必要があります。

2.  資格情報を追加します。

  ```
  bx cs credentials-set --infrastructure-username <username> --infrastructure-api-key <api_key>
  ```
  {: pre}

3.  標準クラスターを作成します。

  ```
  bx cs cluster-create --location dal10 --public-vlan my_public_vlan_id --private-vlan my_private_vlan_id --machine-type u1c.2x4 --name my_cluster --hardware shared --workers 2
  ```
  {: pre}

<br />


## SSH によるワーカー・ノードへのアクセスが失敗する
{: #cs_ssh_worker}

{: tsSymptoms}
SSH 接続を使用してワーカー・ノードにアクセスすることはできません。


{: tsCauses}
パスワードを使用した SSH は、ワーカー・ノードでは無効になっています。


{: tsResolve}
すべてのノードで実行する必要がある場合は [DaemonSets ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) を使用し、一回限りのアクションを実行する必要がある場合はジョブを使用してください。

<br />


## ポッドが保留状態のままである
{: #cs_pods_pending}

{: tsSymptoms}
`kubectl get pods` を実行すると、ポッドの状態が **Pending** になる場合があります。


{: tsCauses}
Kubernetes クラスターを作成したばかりの場合は、まだワーカー・ノードが構成中の可能性があります。クラスターが以前から存在するものである場合は、ポッドをデプロイするための十分な容量がクラスター内で不足している可能性があります。

{: tsResolve}
このタスクには、[管理者アクセス・ポリシー](cs_cluster.html#access_ov)が必要です。現在の[アクセス・ポリシー](cs_cluster.html#view_access)を確認してください。

Kubernetes クラスターを作成したばかりの場合は、以下のコマンドを実行して、ワーカー・ノードが初期化するまで待ちます。


```
kubectl get nodes```
{: pre}

クラスターが以前から存在するものである場合は、クラスターの容量を確認します。

1.  デフォルトのポート番号でプロキシーを設定します。

  ```
kubectl proxy```
   {: pre}

2.  Kubernetes ダッシュボードを開きます。

  ```
http://localhost:8001/ui```
  {: pre}

3.  ポッドをデプロイするための十分な容量がクラスター内にあるか確認します。


4.  クラスターの容量が足りない場合は、クラスターに別のワーカー・ノードを追加します。


  ```
bx cs worker-add <cluster name or id> 1```
  {: pre}

5.  ワーカー・ノードが完全にデプロイされたのにまだポッドが **pending** 状態のままである場合は、[Kubernetes の資料![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/#my-pod-stays-pending) を参照して、ポッドの pending 状態のトラブルシューティングを行ってください。

<br />


## ポッドが作成状態で停滞している
{: #stuck_creating_state}

{: tsSymptoms}
`kubectl get pods -o wide` を実行すると、同じワーカー・ノードで稼働している複数のポッドが `ContainerCreating` 状態で停滞していることが判明します。

{: tsCauses}
ワーカー・ノード上のファイル・システムは読み取り専用です。

{: tsResolve}
1. ワーカー・ノード上またはコンテナー内に保管されている可能性のあるデータをバックアップします。
2. 以下のコマンドを実行してワーカー・ノードを再作成します。

<pre class="pre"><code>bx cs worker-reload &lt;cluster_name&gt; &lt;worker_id&gt;</code></pre>

<br />


## コンテナーが開始しない
{: #containers_do_not_start}

{: tsSymptoms}
ポッドはクラスターに正常にデプロイされましたが、コンテナーが開始しません。

{: tsCauses}
レジストリーの割り当て量に到達すると、コンテナーが開始しないことがあります。

{: tsResolve}
[{{site.data.keyword.registryshort_notm}} 内のストレージを解放してください。](../services/Registry/registry_quota.html#registry_quota_freeup)

<br />


## 新しいワーカー・ノード上のポッドへのアクセスがタイムアウトで失敗する
{: #cs_nodes_duplicate_ip}

{: tsSymptoms}
クラスターのワーカー・ノードを削除した後にワーカー・ノードを追加しました。ポッドまたは Kubernetes サービスをデプロイすると、新しく作成したワーカー・ノードにリソースがアクセスできずに、接続がタイムアウトになります。

{: tsCauses}
クラスターからワーカー・ノードを削除した後にワーカー・ノードを追加すると、削除されたワーカー・ノードのプライベート IP アドレスが新しいワーカー・ノードに割り当てられる場合があります。Calico はこのプライベート IP アドレスをタグとして使用して、削除されたノードにアクセスし続けます。

{: tsResolve}
正しいノードを指すように、プライベート IP アドレスの参照を手動で更新します。

1.  2 つのワーカー・ノードの**プライベート IP** アドレスが同じであることを確認します。削除されたワーカーの**プライベート IP** と **ID** をメモします。

  ```
  bx cs workers <CLUSTER_NAME>
  ```
  {: pre}

  ```
  ID                                                 Public IP       Private IP       Machine Type   State     Status
  kube-dal10-cr9b7371a7fcbe46d08e04f046d5e6d8b4-w1   192.0.2.0.12   203.0.113.144   b1c.4x16       normal    Ready
  kube-dal10-cr9b7371a7fcbe46d08e04f046d5e6d8b4-w2   192.0.2.0.16   203.0.113.144   b1c.4x16       deleted    -
  ```
  {: screen}

2.  [Calico CLI](cs_security.html#adding_network_policies) をインストールします。
3.  Calico で使用可能なワーカー・ノードをリストします。<path_to_file> は、Calico 構成ファイルのローカル・パスに置き換えてください。

  ```
  calicoctl get nodes --config=<path_to_file>/calicoctl.cfg
  ```
  {: pre}

  ```
  NAME
  kube-dal10-cr9b7371a7fcbe46d08e04f046d5e6d8b4-w1
  kube-dal10-cr9b7371a7fcbe46d08e04f046d5e6d8b4-w2
  ```
  {: screen}

4.  Calico で重複しているワーカー・ノードを削除します。NODE_ID はワーカー・ノード ID に置き換えてください。

  ```
  calicoctl delete node NODE_ID --config=<path_to_file>/calicoctl.cfg
  ```
  {: pre}

5.  削除されていないワーカー・ノードをリブートします。

  ```
  bx cs worker-reboot CLUSTER_ID NODE_ID
  ```
  {: pre}


削除されたノードが Calico にリストされなくなります。

<br />


## ワーカー・ノードが接続できない
{: #cs_firewall}

{: tsSymptoms}
ワーカー・ノードを接続できない場合、さまざまな症状が出現することがあります。kubectl プロキシーに障害が起きると、またはクラスター内のサービスにアクセスしようとして接続が失敗すると、次のいずれかのメッセージが表示されることがあります。

  ```
Connection refused```
  {: screen}

  ```
Connection timed out```
  {: screen}

  ```
  Unable to connect to the server: net/http: TLS handshake timeout
  ```
  {: screen}

kubectl exec、attach、または logs を実行する場合、以下のメッセージが表示されることがあります。

  ```
Error from server: error dialing backend: dial tcp XXX.XXX.XXX:10250: getsockopt: connection timed out```
  {: screen}

kubectl プロキシーが正常に実行されても、ダッシュボードを使用できない場合は、次のエラー・メッセージが表示されることがあります。

  ```
timeout on 172.xxx.xxx.xxx```
  {: screen}



{: tsCauses}
追加のファイアウォールを設定したか、IBM Bluemix Infrastructure (SoftLayer) アカウントの既存のファイアウォール設定をカスタマイズした可能性があります。{{site.data.keyword.containershort_notm}} では、ワーカー・ノードと Kubernetes マスター間で通信を行うには、特定の IP アドレスとポートが開いている必要があります。
別の原因として、ワーカー・ノードが再ロード・ループにはまっている可能性があります。

{: tsResolve}
このタスクには、[管理者アクセス・ポリシー](cs_cluster.html#access_ov)が必要です。現在の[アクセス・ポリシー](cs_cluster.html#view_access)を確認してください。

カスタマイズ済みのファイアウォールで、以下のポートと IP アドレスを開きます。


1.  以下を実行して、クラスター内のすべてのワーカー・ノードのパブリック IP アドレスをメモします。


  ```
  bx cs workers '<cluster_name_or_id>'
  ```
  {: pre}

2.  ファイアウォールで、ワーカー・ノードからのアウトバウンド接続として、ソース・ワーカー・ノードから、`<each_worker_node_publicIP>` の宛先 TCP/UDP ポート範囲 (20000 から 32767 まで) およびポート 443 への発信ネットワーク・トラフィック、および以下の IP アドレスとネットワーク・グループへの発信ネットワーク・トラフィックを許可します。
    - **重要**: ブートストラッピング・プロセスの際にロードのバランスを取るため、ポート 443 と地域内のすべてのロケーションで相互に対する発信トラフィックを許可する必要があります。例えば、クラスターが米国南部にある場合、ポート 443 から dal10 と dal12 への、そして dal10 と dal12 から互いへのトラフィックを許可する必要があります。<p>
  <table summary="表の 1 行目は 2 列にまたがっています。残りの行は左から右に読みます。1 列目はサーバーの場所、2 列目は対応する IP アドレスです。">
    <thead>
      <th>地域</th>
      <th>ロケーション</th>
      <th>IP アドレス</th>
      </thead>
    <tbody>
      <tr>
         <td>南アジア太平洋地域</td>
         <td>mel01<br>syd01
</td>
         <td><code>168.1.97.67</code><br><code>168.1.8.195</code></td>
      </tr>
      <tr>
         <td>中欧</td>
         <td>ams03<br>fra02</td>
         <td><code>169.50.169.110</code><br><code>169.50.56.174</code></td>
        </tr>
      <tr>
        <td>英国南部</td>
        <td>lon02<br>lon04</td>
        <td><code>159.122.242.78</code><br><code>158.175.65.170</code></td>
      </tr>
      <tr>
        <td>米国東部</td>
         <td>wdc06<br>wdc07</td>
         <td><code>169.60.73.142</code><br><code>169.61.83.62</code></td>
      </tr>
      <tr>
        <td>米国南部</td>
        <td>dal10<br>dal12<br>dal13</td>
        <td><code>169.46.7.238</code><br><code>169.47.70.10</code><br><code>169.60.128.2</code></td>
      </tr>
      </tbody>
    </table>
</p>

3.  ワーカー・ノードから {{site.data.keyword.registrylong_notm}} への発信ネットワーク・トラフィックを許可します。
    - `TCP port 443 FROM <each_worker_node_publicIP> TO <registry_publicIP>`
    - <em>&lt;registry_publicIP&gt;</em> は、トラフィックを許可するレジストリー地域のすべてのアドレスに置き換えます。
        <p>      
<table summary="表の 1 行目は 2 列にまたがっています。残りの行は左から右に読みます。1 列目はサーバーの場所、2 列目は対応する IP アドレスです。">
    <thead>
        <th colspan=2><img src="images/idea.png"/> レジストリー IP アドレス</th>
        </thead>
      <tbody>
        <tr>
          <td>registry.au-syd.bluemix.net</td>
          <td><code>168.1.45.160/27</code></br><code>168.1.139.32/27</code></td>
        </tr>
        <tr>
          <td>registry.eu-de.bluemix.net</td>
          <td><code>169.50.56.144/28</code></br><code>159.8.73.80/28</code></td>
         </tr>
         <tr>
          <td>registry.eu-gb.bluemix.net</td>
          <td><code>159.8.188.160/27</code></br><code>169.50.153.64/27</code></td>
         </tr>
         <tr>
          <td>registry.ng.bluemix.net</td>
          <td><code>169.55.39.112/28</code></br><code>169.46.9.0/27</code></br><code>169.55.211.0/27</code></td>
         </tr>
        </tbody>
      </table>
</p>

4.  オプション: ワーカー・ノードから {{site.data.keyword.monitoringlong_notm}} サービスと {{site.data.keyword.loganalysislong_notm}} サービスへの発信ネットワーク・トラフィックを許可します。
    - `TCP port 443, port 9095 FROM <each_worker_node_publicIP> TO <monitoring_publicIP>`
    - <em>&lt;monitoring_publicIP&gt;</em> は、トラフィックを許可するモニタリング地域のすべてのアドレスに置き換えます。
        <p><table summary="表の 1 行目は 2 列にまたがっています。残りの行は左から右に読みます。1 列目はサーバーの場所、2 列目は対応する IP アドレスです。">
    <thead>
        <th colspan=2><img src="images/idea.png"/> モニタリング・パブリック IP アドレス</th>
        </thead>
      <tbody>
        <tr>
         <td>metrics.eu-de.bluemix.net</td>
         <td><code>159.122.78.136/29</code></td>
        </tr>
        <tr>
         <td>metrics.eu-gb.bluemix.net</td>
         <td><code>169.50.196.136/29</code></td>
        </tr>
        <tr>
          <td>metrics.ng.bluemix.net</td>
          <td><code>169.47.204.128/29</code></td>
         </tr>
         
        </tbody>
      </table>
</p>
    - `TCP port 443, port 9091 FROM <each_worker_node_publicIP> TO <logging_publicIP>`
    - <em>&lt;logging_publicIP&gt;</em> は、トラフィックを許可するロギング地域のすべてのアドレスに置き換えます。
        <p><table summary="表の 1 行目は 2 列にまたがっています。残りの行は左から右に読みます。1 列目はサーバーの場所、2 列目は対応する IP アドレスです。">
    <thead>
        <th colspan=2><img src="images/idea.png"/> ロギング・パブリック IP アドレス</th>
        </thead>
      <tbody>
        <tr>
         <td>ingest.logging.eu-de.bluemix.net</td>
         <td><code>169.50.25.125</code></td>
        </tr>
        <tr>
         <td>ingest.logging.eu-gb.bluemix.net</td>
         <td><code>169.50.115.113</code></td>
        </tr>
        <tr>
          <td>ingest.logging.ng.bluemix.net</td>
          <td><code>169.48.79.236</code><br><code>169.46.186.113</code></td>
         </tr>
        </tbody>
      </table>
</p>

5. プライベート・ファイアウォールがある場合、IBM Bluemix Infrastructure (SoftLayer) のプライベート IP のために適切な範囲を許可します。[このリンク](https://knowledgelayer.softlayer.com/faq/what-ip-ranges-do-i-allow-through-firewall)の **Backend (private) Network** で始まるセクションを参照してください。
    - 使用している[地域内のロケーション](cs_regions.html#locations)をすべて追加します
    - dal01 のロケーション (データ・センター) を追加する必要があることに注意してください
    - ポート 80 および 443 を開いて、クラスターのブートストラッピング処理を可能にします

<br />


## ワーカー・ノードを更新または再ロードした後に、重複するノードとポッドが表示される
{: #cs_duplicate_nodes}

{: tsSymptoms}
`kubectl get nodes` を実行すると、状況が **NotReady** の重複したワーカー・ノードが表示されます。**NotReady** のワーカー・ノードにはパブリック IP アドレスがありますが、**Ready** のワーカー・ノードにはプライベート IP アドレスがあります。

{: tsCauses}
以前のクラスターでは、クラスターのパブリック IP アドレスごとにワーカー・ノードがリストされていました。現在は、ワーカー・ノードはクラスターのプライベート IP アドレスごとにリストされています。ノードを再ロードまたは更新すると、IP アドレスは変更されますがパブリック IP アドレスへの参照は残ります。

{: tsResolve}
これらの重複によってサービスが中断されることはありませんが、以前のワーカー・ノード参照を API サーバーから除去する必要があります。

  ```
  kubectl delete node <node_name1> <node_name2>
  ```
  {: pre}

<br />


## Ingress 経由でアプリに接続できない
{: #cs_ingress_fails}

{: tsSymptoms}
クラスターでアプリ用の Ingress リソースを作成して、アプリをパブリックに公開しました。Ingress コントローラーのパブリック IP アドレスまたはサブドメインを経由してアプリに接続しようとすると、接続に失敗するかタイムアウトになります。

{: tsCauses}
次の理由で、Ingress が正しく機能していない可能性があります。
<ul><ul>
<li>クラスターがまだ完全にデプロイされていません。<li>クラスターが、ライト・クラスターとして、またはワーカー・ノードが 1 つしかない標準クラスターとしてセットアップされました。
<li>Ingress 構成スクリプトにエラーがあります。</ul></ul>

{: tsResolve}
Ingress のトラブルシューティングを行うには、以下のようにします。

1.  標準クラスターをセットアップしたこと、クラスターが完全にデプロイされていること、また、Ingress コントローラーの高可用性を確保するためにクラスターに 2 つ以上のワーカー・ノードがあることを確認します。

  ```
    bx cs workers <cluster_name_or_id>
    ```
  {: pre}

    CLI 出力で、ワーカー・ノードの **Status** に **Ready** と表示され、**Machine Type** に **free** 以外のマシン・タイプが表示されていることを確認します。

2.  Ingress コントローラーのサブドメインとパブリック IP アドレスを取得し、それぞれを ping します。

    1.  Ingress コントローラーのサブドメインを取得します。

      ```
      bx cs cluster-get <cluster_name_or_id> | grep "Ingress subdomain"
      ```
      {: pre}

    2.  Ingress コントローラーのサブドメインを ping します。

      ```
      ping <ingress_controller_subdomain>
      ```
      {: pre}

    3.  Ingress コントローラーのパブリック IP アドレスを取得します。

      ```
      nslookup <ingress_controller_subdomain>
      ```
      {: pre}

    4.  Ingress コントローラーのパブリック IP アドレスを ping します。

      ```
      ping <ingress_controller_ip>
      ```
      {: pre}

    Ingress コントローラーのパブリック IP アドレスまたはサブドメインで CLI からタイムアウトが返された場合は、カスタム・ファイアウォールをセットアップしてワーカー・ノードを保護しているのであれば、[ファイアウォール](#cs_firewall)で追加のポートとネットワーキング・グループを開く必要があります。

3.  カスタム・ドメインを使用している場合は、ドメイン・ネーム・サービス (DNS) プロバイダーで、カスタム・ドメインが IBM 提供の Ingress コントローラーのパブリック IP アドレスまたはサブドメインにマップされていることを確認します。
    1.  Ingress コントローラーのサブドメインを使用した場合は、正規名レコード (CNAME) を確認します。
    2.  Ingress コントローラーのパブリック IP アドレスを使用した場合は、カスタム・ドメインがポインター・レコード (PTR) でポータブル・パブリック IP アドレスにマップされていることを確認します。
4.  Ingress 構成ファイルを確認します。

    ```
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: myingress
    spec:
      tls:
      - hosts:
        - <ingress_subdomain>
        secretName: <ingress_tlssecret>
      rules:
      - host: <ingress_subdomain>
        http:
          paths:
          - path: /
            backend:
              serviceName: myservice
              servicePort: 80
    ```
    {: codeblock}

    1.  Ingress コントローラーのサブドメインと TLS 証明書が正しいことを確認します。IBM 提供のサブドメインと TLS 証明書を見つけるには、bx cs cluster-get <cluster_name_or_id> を実行します。
    2.  アプリが、Ingress の **path** セクションで構成されているパスを使用して listen していることを確認します。アプリがルート・パスで listen するようにセットアップされている場合は、**/** をパスとして含めます。
5.  Ingress デプロイメントを確認して、エラー・メッセージがないか探します。

  ```
  kubectl describe ingress <myingress>
  ```
  {: pre}

6.  Ingress コントローラーのログを確認します。
    1.  クラスター内で稼働している Ingress ポッドの ID を取得します。

      ```
      kubectl get pods -n kube-system |grep ingress
      ```
      {: pre}

    2.  Ingress ポッドごとにログを取得します。

      ```
      kubectl logs <ingress_pod_id> -n kube-system
      ```
      {: pre}

    3.  Ingress コントローラーのログでエラー・メッセージを探します。

<br />


## ロード・バランサー・サービス経由でアプリに接続できない
{: #cs_loadbalancer_fails}

{: tsSymptoms}
クラスター内にロード・バランサー・サービスを作成して、アプリをパブリックに公開しました。ロード・バランサーのパブリック IP アドレス経由でアプリに接続しようとすると、接続が失敗するかタイムアウトになります。

{: tsCauses}
次のいずれかの理由で、ロード・バランサー・サービスが正しく機能していない可能性があります。

-   クラスターが、ライト・クラスターであるか、またはワーカー・ノードが 1 つしかない標準クラスターです。
-   クラスターがまだ完全にデプロイされていません。
-   ロード・バランサー・サービスの構成スクリプトにエラーが含まれています。

{: tsResolve}
ロード・バランサー・サービスのトラブルシューティングを行うには、以下のようにします。

1.  標準クラスターをセットアップしたこと、クラスターが完全にデプロイされていること、また、ロード・バランサー・サービスの高可用性を確保するためにクラスターに 2 つ以上のワーカー・ノードがあることを確認します。

  ```
    bx cs workers <cluster_name_or_id>
    ```
  {: pre}

    CLI 出力で、ワーカー・ノードの **Status** に **Ready** と表示され、**Machine Type** に **free** 以外のマシン・タイプが表示されていることを確認します。

2.  ロード・バランサー・サービスの構成ファイルが正しいことを確認します。

  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: myservice
  spec:
    type: LoadBalancer
    selector:
      <selectorkey>:<selectorvalue>
    ports:
     - protocol: TCP
       port: 8080
  ```
  {: pre}

    1.  サービスのタイプとして **LoadBalancer** を定義したことを確認します。
    2.  アプリをデプロイするときに **label/metadata** セクションで使用したものと同じ **<selectorkey>** と **<selectorvalue>** を使用していることを確認します。
    3.  アプリで listen している **port** を使用していることを確認します。

3.  ロード・バランサー・サービスを確認し、**Events** セクションを参照して、エラーがないか探します。

  ```
    kubectl describe service <myservice>
    ```
  {: pre}

    次のようなエラー・メッセージを探します。
<ul><ul><li><pre class="screen"><code>Clusters with one node must use services of type NodePort</code>
</pre></br>ロード・バランサー・サービスを使用するには、2 つ以上のワーカー・ノードがある標準クラスターでなければなりません。
<li><pre class="screen"><code>No cloud provider IPs are available to fulfill the load balancer service request. Add a portable subnet to the cluster and try again</code>
</pre></br>このエラー・メッセージは、ロード・バランサー・サービスに割り振れるポータブル・パブリック IP アドレスが残っていないことを示しています。クラスター用にポータブル・パブリック IP アドレスを要求する方法については、[クラスターへのサブネットの追加](cs_cluster.html#cs_cluster_subnet)を参照してください。クラスターにポータブル・パブリック IP アドレスを使用できるようになると、ロード・バランサー・サービスが自動的に作成されます。
<li><pre class="screen"><code>Requested cloud provider IP <cloud-provider-ip> is not available. The following cloud provider IPs are available: <available-cloud-provider-ips</code>
</pre></br>**loadBalancerIP** セクションを使用してロード・バランサー・サービスのポータブル・パブリック IP アドレスを定義しましたが、そのポータブル・パブリック IP アドレスはポータブル・パブリック・サブネットに含まれていません。ロード・バランサー・サービスの構成スクリプトを変更して、使用可能なポータブル・パブリック IP アドレスを選択するか、またはスクリプトから **loadBalancerIP** セクションを削除して、使用可能なポータブル・パブリック IP アドレスが自動的に割り振られるようにします。
<li><pre class="screen"><code>No available nodes for load balancer services</code>
</pre>ワーカー・ノードが不足しているため、ロード・バランサー・サービスをデプロイできません。複数のワーカー・ノードを持つ標準クラスターをデプロイしましたが、ワーカー・ノードのプロビジョンが失敗した可能性があります。
<ol><li>使用可能なワーカー・ノードのリストを表示します。</br><pre class="codeblock"><code>kubectl get nodes</code></pre>
    <li>使用可能なワーカー・ノードが 2 つ以上ある場合は、ワーカー・ノードの詳細情報をリストします。</br><pre class="screen"><code>bx cs worker-get <worker_ID></code></pre>
    <li>「kubectl get nodes」コマンドと「bx cs worker-get」コマンドから返されたワーカー・ノードのパブリック VLAN ID とプライベート VLAN ID が一致していることを確認します。</ol></ul></ul>

4.  カスタム・ドメインを使用してロード・バランサー・サービスに接続している場合は、カスタム・ドメインがロード・バランサー・サービスのパブリック IP アドレスにマップされていることを確認します。
    1.  ロード・バランサー・サービスのパブリック IP アドレスを見つけます。

      ```
      kubectl describe service <myservice> | grep "LoadBalancer Ingress"
      ```
      {: pre}

    2.  カスタム・ドメインが、ポインター・レコード (PTR) でロード・バランサー・サービスのポータブル・パブリック IP アドレスにマップされていることを確認します。

<br />


## Calico CLI 構成の ETCD url の取得に失敗する
{: #cs_calico_fails}

{: tsSymptoms}
[ネットワーク・ポリシーを追加するための](cs_security.html#adding_network_policies) `<ETCD_URL>` を取得するときに、`calico-config not found` エラー・メッセージが出されます。

{: tsCauses}
クラスターが (Kubernetes バージョン 1.7)[cs_versions.html] 以降ではありません。

{: tsResolve}
(クラスターを更新する)[cs_cluster.html#cs_cluster_update] か、または以前のバージョンの Kubernetes と互換性のあるコマンドによって `<ETCD_URL>` を取得します。

`<ETCD_URL>` を取得するには、以下のいずれかのコマンドを実行します。

- Linux および OS X:

    ```
kubectl describe pod -n kube-system `kubectl get pod -n kube-system | grep calico-policy-controller | awk '{print $1}'` | grep ETCD_ENDPOINTS | awk '{print $2}'```
    {: pre}

- Windows:<ol>
    <li> kube-system 名前空間内のポッドのリストを取得し、Calico コントローラー・ポッドを見つけます。</br><pre class="codeblock"><code>kubectl get pod -n kube-system</code></pre></br>例:</br><pre class="screen"><code>calico-policy-controller-1674857634-k2ckm</code></pre>
    <li> Calico コントローラー・ポッドの詳細を表示します。</br> <pre class="codeblock"><code>kubectl describe pod -n kube-system calico-policy-controller-&lt;ID&gt;</code></pre>
    <li> ETCD エンドポイント値を見つけます。例: <code>https://169.1.1.1:30001</code>
            </ol>

`<ETCD_URL>` を取得するときには、(ネットワーク・ポリシーの追加)[cs_security.html#adding_network_policies] にリストされている手順を続行します。

<br />


## 既知の問題
{: #cs_known_issues}

既知の問題について説明します。
{: shortdesc}

### クラスター
{: #ki_clusters}

<dl>
  <dt>同じ {{site.data.keyword.Bluemix_notm}} スペース内の Cloud Foundry アプリがクラスターにアクセスできない</dt>
    <dd>Kubernetes クラスターを作成すると、クラスターはアカウント・レベルで作成され、スペースを使用しません ({{site.data.keyword.Bluemix_notm}} サービスをバインドする場合を除く)。クラスターからアクセスする Cloud Foundry アプリがある場合は、その Cloud Foundry アプリをパブリックに利用可能にするか、クラスター内のアプリを[パブリックに利用可能にする](cs_planning.html#cs_planning_public_network)必要があります。</dd>
  <dt>Kubernetes ダッシュボード NodePort サービスが無効になっている</dt>
    <dd>セキュリティー上の理由により、Kubernetes ダッシュボード NodePort サービスは無効になっています。
Kubernetes ダッシュボードにアクセスするには、以下のコマンドを実行します。
</br><pre class="codeblock"><code>kubectl proxy</code></pre></br>これにより、`http://localhost:8001/ui` から Kubernetes ダッシュボードにアクセスできるようになります。</dd>
  <dt>ロード・バランサーのサービス・タイプに制約がある</dt>
    <dd><ul><li>プライベート VLAN では、ロード・バランシングは使用できません。
<li>service.beta.kubernetes.io/external-traffic サービスと service.beta.kubernetes.io/healthcheck-nodeport サービスのアノテーションを使用することはできません。
これらのアノテーションについて詳しくは、[Kubernetes の資料![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://kubernetes.io/docs/tutorials/services/source-ip/) を参照してください。</ul></dd>
  <dt>いくつかのクラスターで水平自動スケーリングが機能しない</dt>
    <dd>セキュリティー上の理由で、以前のクラスターでは、Heapster によって使用される標準のポート (10255) はすべてのワーカー・ノードで閉じられています。このポートが閉じられていて Heapster がワーカー・ノードのメトリックを報告できないので、水平自動スケーリングは Kubernetes 資料の [Horizontal Pod Autoscaling ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) の説明のようには機能しません。この問題を回避するには、別のクラスターを作成します。</dd>
</dl>

### 永続ストレージ 
{: #persistent_storage}

`kubectl describe <pvc_name>` コマンドを実行すると、永続ボリュームの請求に対して **ProvisioningFailed** が表示されます。
<ul><ul>
<li>永続ボリューム請求を行う時に、利用できる永続ボリュームがないため、Kubernetes はメッセージ **ProvisioningFailed** を返します。
<li>永続ボリュームが作成され、それが請求にバインドされると、Kubernetes はメッセージ **ProvisioningSucceeded** を返します。
この処理には数分かかる場合があります。
</ul></ul>

## ヘルプとサポートの取得
{: #ts_getting_help}

コンテナーのトラブルシューティングを開始するには、以下の方法があります。

-   {{site.data.keyword.Bluemix_notm}} が使用可能かどうかを確認するために、[{{site.data.keyword.Bluemix_notm}} 状況ページを確認します![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/bluemix/support/#status)。
-   [{{site.data.keyword.containershort_notm}} Slack![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://ibm-container-service.slack.com) に質問を投稿します。{{site.data.keyword.Bluemix_notm}} アカウントに IBM ID を使用していない場合は、[crosen@us.ibm.com](mailto:crosen@us.ibm.com) に問い合わせて、この Slack への招待を要求してください。
-   フォーラムを確認して、同じ問題が他のユーザーで起こっているかどうかを調べます。
フォーラムを使用して質問するときは、{{site.data.keyword.Bluemix_notm}} 開発チームの目に止まるように、質問にタグを付けてください。


    -   {{site.data.keyword.containershort_notm}} を使用したクラスターまたはアプリの開発やデプロイに関する技術的な質問がある場合は、[Stack Overflow![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](http://stackoverflow.com/search?q=bluemix+containers) に質問を投稿し、`ibm-bluemix`、`kubernetes`、`containers` のタグを付けてください。
    -   サービスや概説の説明について質問がある場合は、[IBM developerWorks dW Answers ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/answers/topics/containers/?smartspace=bluemix) フォーラムを使用してください。`bluemix` と `containers` のタグを含めてください。
フォーラムの使用について詳しくは、[ヘルプの取得](/docs/support/index.html#getting-help)を参照してください。


-   IBM サポートにお問い合わせください。
IBM サポート・チケットを開く方法や、サポート・レベルとチケットの重大度については、[サポートへのお問い合わせ](/docs/support/index.html#contacting-support)を参照してください。

