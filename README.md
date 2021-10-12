## 0. 事前準備
1. [IBM Cloudライトアカウント作成](https://cloud.ibm.com/login) ※アカウント取得方法は[こちら](https://github.com/IBMDeveloperTokyo/openshift-s2i-lab/blob/ce5a9313c68f8dfc496c9dc72fea8ad3a69d391d/%E3%82%AF%E3%83%AC%E3%82%B7%E3%82%99%E3%83%83%E3%83%88%E7%99%BB%E9%8C%B2%E3%81%AA%E3%81%97%E3%81%ABIBMCloud%E3%81%B8%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3.pdf)で公開しています。
2. [GitHubアカウント作成](https://github.com/)(参考URL: [GitHubアカウントの作成方法 (2021年版)](https://qiita.com/ayatokura/items/9eabb7ae20752e6dc79d))

### 免責
本ハンズオンワークショップではOpenShiftのクラスタを利用します。
これは、みなさまのIBM Cloudのライトアカウント(クレジットカード不要)に、IBMが準備した作成済みOpenShiftクラスタを持つ**別のアカウントを紐付けた上でOpehShiftを利用します**。
ですので、こちらに記載の手順に従ってお試し頂く中では課金は一切発生いたしません。
ですが、従量課金制のIBM Cloudアカウント(PAYGやサブスクリプション)上にOpenShiftのクラスタを作成したり、有償のサービスを作成したりした場合は課金が発生いたします。
万が一、ご自身のIBM Cloudアカウント(PAYGやサブスクリプション)に対してクラスタを作成するなどして課金が発生した場合、IBM及び本ワークショップの講師は責任を負いかねますので、十分ご注意の上実施下さい。

### OpenShiftへのいろいろな入力パターン
![](./images/001.png)

本ハンズオンワークショップではこの中からSource to Image (S2I) を試します。
![](./images/002.png)

### ハンズオンワークショップの流れ
1. OpenShift環境の準備
2. ソースコードのFork
3. アプリケーションのDeploy
4. Webhookの設定
5. ソースコードの修正及びDeploy(⾃動）

## 1. OpenShift環境の準備
ワークショップ⽤のIBM Cloud環境にご⾃⾝のIBM Cloud IDを関連付けます。

注意事項
```
・ブラウザはFirefox, Chromeをご利⽤ください
・本ワークショップ⽤のIBM Cloud環境はセミナー開催時から24時間限定でお使いいただけます
```

### 1.1 下記URLにFirefoxブラウザでアクセス
ワークショップの場合 **講師よりアクセス先をご案内いたします**。下記はサンプルURLとなります。

https://xxxxxx.mybluemix.net/

### 1.2 ハンズオン環境へSubmit
[Lab Key] 、[Your IBMid]にご⾃⾝のIDを⼊⼒し、チェックボックスにチェックを⼊れて[Submit]をクリックします。
![](./images/003.png)

### 1.3 IBM Cloudダッシュボードの起動
Congratulations! が表⽰されたら、指定したIBMid宛に送られるメールを確認します。<br>
※必要に応じて、IBM Cloud (no-reply@cloud.ibm.com) からのメールを受信できるように、ご使用されているメーラー設定などを行ってください。<br>
メール本文にある[Join now.]のリンクをクリックします。
![](./images/004-1.png)

その後IBM Cloudへアカウントを紐付けるための画面が表示されるので、[アカウントに参加]ボタンをクリックします。
![](./images/004-2.png)

自動でIBM Cloudへログインされます。ログインされない場合は https://cloud.ibm.com へアクセスして手動でログインして下さい。

### 1.4 アカウントの切り替え
IBM Cloudダッシュボードにログインしたら、右上のアカウント情報の右横の「v」をクリックして、ワークショップ用のアカウントへ切り替えます。<br>
[1840867 – Advowork] のような「数字の羅列 - Advowork」がワークショップ用に準備されたアカウントです。 (数字部分は自動的に割り当てられます)<br>
※このアカウントは1日程度で無効になる一時的なアカウントです。
![](./images/005.png)

### 1.5 OpenShiftクラスタへのアクセス
IBM Cloudダッシュボードの右上のアカウント情報が変更されたことを確認し、[リソースの要約]の[Clusters]をクリックします。
![](./images/006.png)

Clustersの下のクラスター名をクリックします。 (クラスター名は⾃動的に割り当てられます)
![](./images/007.png)

### 1.6 OpenShift Webコンソールの起動
[OpenShift Webコンソール]ボタンをクリックします。<br>
※ポップアップが制御されていると動作しませんので解除してください。
![](./images/008.png)

新しいウィンドウ（またはタブ）でOpenShiftのコンソールが開けばOKです。アクセスするURLはポート30000番台（⾃動で割り当てられます）を使っているので、社内プロキシなどで制限している場合はポートを開いておいてください。<br>
Webコンソールは、通常以下のようなURLでリダイレクトされます。
```https://c100-e.jp-tok.containers.cloud.ibm.com:31379/```
![](./images/009.png)

これでOpenShiftワークショップの環境準備は完了です。


## 2. ソースコードのFork
ここからはGitHubへアクセスして自分のリポジトリへサンプルソースコードをForkしていきます。

### 2.1 GitHubへのサインイン
GitHubにサインイン(Sign in)してください。まだアカウント登録されていない方は[こちら](https://github.com/)からサインアップ(Sign up)してください。<br>
![](./images/010.png)

### 2.2 リポジトリーのFork
ブラウザーで[https://github.com/osonoi/node-build-config-openshift](https://github.com/osonoi/node-build-config-openshift)を開いてください。<br>
[Fork]ボタンをクリックして、自分のアカウントを選択してください。
![](./images/011.png)

### 2.3 自分のリポジトリーの確認
Forkする際に指定した自分のリポジトリーへ、対象のプロジェクトがForkされたことを確認します。<br>
リポジトリーのパスの最初の部分が自分のGitHubアカウントになっていればOKです。
![](./images/012.png)


## 3. アプリケーションのDeploy
ここからは、先程用意したOpenShiftの環境へ、自分のGitHubリポジトリーにあるアプリケーションをデプロイします。

### 3.1 OpenShift Projectの作成
OpenShiftのWebコンソールへ戻り、[プロジェクト]ボタンをクリックします。<br>
その後[Projectの作成]ボタンをクリックするとCreate Project画面が開きますので、任意のプロジェクト名を入力し[Create]ボタンをクリックしてください。<br>なお、Nameにはすべて小文字をお使いください。
![](./images/013.png)

### 3.2 OpenShiftユーザータイプの切り替え
左上のメニューにて、[管理者]から[Developer]に切り替えます。<br>
切り替えたら[ソース:Git]をクリックしてください。
![](./images/014.png)

### 3.3 デプロイするアプリケーションのソースコードを指定
先ほどコピーした、自分のGitHubリポジトリーのURLを[GitリポジトリーURL]に入力します。<br>
下の[詳細のGitオプションの表示]をクリックすると入力エリアが展開するので、[コンテキストディレクトリー]に「/site」と入力してください。<br>
今回デプロイする対象のアプリケーションはGitHubリポジトリー[node-build-config-openshift]の「site」ディレクトリ配下のため、ここでディレクトリを指定します。
![](./images/015.png)

### 3.4 デプロイするアプリケーションのタイプを選択
言語やタイプの一覧がタイルで表示され、Node.jsが選択されていることを確認します。今回GitHubリポジトリーへForkしたプロジェクトはNode.jsアプリケーションだからです。<br>
選択したら最下段の[作成]ボタンをクリックしてください。（他のオプションはすべてデフォルトで構いません)
![](./images/016.png)

### 3.5 アプリケーションのデプロイ
アプリケーションのデプロイが始まります。1分弱お待ちください。中の丸が青くなったら完成です。丸の中をクリックすると右側にメニューが出てくるので[Routes]の下のURLをクリックするとWebへ公開されたアプリケーションへアクセスできます。
![](./images/017.png)

### 3.6 アプリケーションへのアクセス
デプロイされ、Webへ公開されたアプリケーションへアクセスできました。<br>
実際にこのアプリケーションへログインしてみましょう。ID,Passwordともに「test」と入れてログインしてください。<br>
医療関連のデータを管理するサンプルアプリケーションへログインできたかと思います。
![](./images/018.png)

ここまでで、GitHub上のソースコードをダイレクトにOpenShiftへデプロイする方法を学びました。


## 4. Webhookの設定
ここでは、GitHub上のソースコードが変更された際に、自動的にOpenShiftへデプロイされるようにWebhookをGitHub上へ設定していきたいと思います。

### 4.1 OpenShiftのWebhook URLの取得
OpenShiftのWebコンソールへアクセスします。左側のメニューから[ビルド]を選択し、右側のワークスペースに表示される[node-build-config-openshift]をクリックします。
![](./images/019.png)

下にスクロールして一番右の[Copy URL with Secret]をクリックしてWebhookのURLとSecretをクリップボードにコピーしてください。
![](./images/020.png)

### 4.2 GitHubにWebhookを設定
GitHubの自分のリポジトリーへ戻り、[Settings] -> [Webhooks] -> [Add webhook]を選択します。
![](./images/021.png)

先ほどクリップボードにコピーしたURL+secretを[Payload URL]に貼り付けてください。[Control type]は[application/json]を選択してください。
![](./images/022.png)

入力後、[Add webhook]を選択します。<br>
以下の図の様に緑のチェックマークが付いたら設定成功です。（チェックマークが表示されない場合はページを再読み込みしてください。）
![](./images/023.png)

これでwebhookの設定は完了です。後はソースコードの修正で自動的にアプリケーションがデプロイされます。


## 5. ソースコードの修正及びDeploy(自動)
最後に、GitHub上のソースコードを修正し、それが自動でOpenShiftへ反映されることを試していきます。

### 5.1 ソースコードの修正
GitHubの自分のリポジトリ画面から[Site]フォルダーを選択します。
![](./images/024.png)

[public]フォルダ配下の[index.html]を選択しペンのアイコンをクリックして編集モードにします。<br>
ここでは、GitHubのGUIから編集を行いますが、ローカルにcloneして編集したファイルをcommit、pushしてもOKです。
![](./images/025.png)

変更点をクイックに確認するために、ここでは23行目の英文「Example Health」を日本語の「医療管理」に変更してみます。<br>
変更したらコミットしてください。自分所有のリポジトリーなので、そのまま反映されます。
![](./images/026.png)

OpenShiftのWebコンソールへ戻り、左側の[トポロジー]を確認すると、再度デプロイがおこなわれていることが分かります。
![](./images/027.png)

再デプロイが完了したらデプロイしたアプリケーションを起動し直してください。（既に表示されている場合はリロードしてください）<br>
アプリケーション内にある見出しが「Example Health」から「医療管理」に変更されたことが確認できました。
![](./images/028.png)

お疲れさまでした！これで、OpenShiftのS2Iを使ったハンズオンワークショップは完了です。


## その他のアプリケーション
もし、PHPのサンプルアプリケーションで試してみたい方は下記のリポジトリーのソースコードを試してみてください。
[https://github.com/osonoi/php-s2i-openshift](https://github.com/osonoi/php-s2i-openshift)
![](./images/029.png)

