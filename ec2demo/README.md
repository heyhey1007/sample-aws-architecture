# 概要

EC2 を起動し、SSH 接続ができるようになるまでのデモ

- Amazon Elastic Compute Cloud (Amazon EC2) って？
  - AWS で最も基本的なコンピューティングリソース
    - リモート操作できる PC のイメージ
  - 柔軟にカスタムが可能
    - プロセッサ、メモリサイズ、ストレージ、ネットワーキング、オペレーティングシステム
  - スケーラビリティが高い
    - 簡単に増やしたり減らしたりできる

# 手順は以下のとおり

1. AWS **IAM アカウント**の準備
2. **キーペア**の作成
3. **セキュリティグループ**作成
4. **VPC**の作成

## AWSIAM アカウント準備

- IAM アカウントとは
  - IAM ユーザーとも
  - とにかくルートアカウントは使うな
  - AWS アカウント ≠ IAM アカウント

## キーペアの作成

- キーペアとは
  - キーペアは Amazon EC2 インスタンスに接続するときに使用します。
  - インスタンスを作成するときに Amazon EC2 にキーペアを指定する必要があり、インスタンスに接続するときには、そのキーペアを使用して認証する必要があります。
  - プライベートキーは AWS に保存されず、作成時にのみ取得することができます。後で復元することはできません。代わりに、プライベートキーを紛失した場合は、新しいキーペアを作成する必要があります。

## セキュリティグループ作成

- セキュリティグループとは

  - 今回は本質的に EC2 の**ファイアウォール**として動作します。
  - 「登録した IP アドレスからのみアクセスを受け付ける」などの設定が可能
  - EC2 インスタンス用のセキュリティグループを作成し、アクセス可能なネットワークトラフィックを決めるルールと共に作成することができます。
  - AWS では最も基本的なアクセス制限の方法

## インスタンス起動までのデモ

- VPC とは

  - EC2 や他のリソースを配置するための入れ物(ネットワーク)
  - リソースの配置、接続性、セキュリティなど、仮想ネットワーク環境を完全に制御することができます。
  - 最初のステップは、VPC を作成することです。
  - 次に、Amazon Elastic Compute Cloud (EC2) や Amazon Relational Database Service (RDS) インスタンスなどのリソースを追加できます。

  - 以下のリンクから VPC を作成
  - https://ap-northeast-1.console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/quickcreate?templateUrl=https%3A%2F%2Fnakata-sample-aws-cfn-template.s3.ap-northeast-1.amazonaws.com%2Fvpc.yaml&stackName=MyName-practice-stack&param_PrivateSubnetCidr=10.0.2.0%2F24&param_PublicSubnetCidr=10.0.1.0%2F24&param_SysName=yourSystemName&param_VpcCidr=10.0.0.0%2F16

## インスタンス起動

- インスタンスを起動
  - AMI 選択
    - Amazon Linux 2 AMI 64bit(x86)
  - インスタンスタイプ
    - t2.micro
  - インスタンスの設定
    - ネットワーク: 作成した VPC
    - サブネット: 作成したパブリックサブネット(10.0.1.0/24)
    - 自動割り当てパブリック IP: 有効
  - ストレージの追加
    - 8GB
    - 汎用 SSD(gp2)
  - タグの追加
    - Owner: [a-z]<自分の名前>
    - Name: <自分の名前>-sshdemo-ec2
  - セキュリティグループの設定
    - 作成したセキュリティグループを指定
  - キーペアの指定
    - 作成したキーペアを指定
- EC2 インスタンス一覧を確認
  - 作成したインスタンスのパブリック IP アドレスを確認
- Powershell 起動

  - ssh 確認

    ```
    $ ssh -v
    ```

  - ssh 接続

    ```
    $ ssh -i <作成したKeyPair> ec2-user@<インスタンスのパブリックIP>
    ```

- おまけ
  - Apache インストールしてテストページ出力
    - `$ sudo yum update`
      - 質問されたら`y`を入力
    - `$ sudo yum install httpd`
      - 質問されたら`y`を入力
    - `$ sudo systemctl start httpd.service`
    - `$ curl localhost`
    - test ページの html が出力されれば OK
  - 上記だけでは、手元のブラウザからは接続できない(ブラウザ URL にパブリック IP 入力)
    - セキュリティグループで HTTP アクセスが許可されていないため
    - セキュリティグループ->インバウンドルール追加
      - プロトコル:HTTP
      - ソース IP: マイ IP
      - 説明: <名前> IP
  - Hello,world!
    - ssh で ec2 に接続
    - `$ sudo vim /var/www/html/hello/index.html`
      - `hello,world`を入力
    - ブラウザで`<EC2のパブリックIP>/hello`を URL に入力する

# 後片付け

- EC2 の終了
- セキュリティグループ削除
- キーペア削除
- CloudFormation(VPC)の削除

#### 参考文献

Amazon EC2 インスタンスの起動、一覧表示、および終了
https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-services-ec2-instances.html
IP アドレスチェックツール
https://www.luft.co.jp/cgi/ipcheck.php
