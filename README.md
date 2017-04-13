CloudFormationテンプレート置き場
===

# テンプレート

## cfn_webap_env.json

- 事前に ```sample-key``` という名前でキーペアを作成しておく必要がある
- Amazon Linuxを1台、RDS（MySQL）を1台構築する
- ```SshLocation``` パラメータに自分のローカル環境のグローバルIPを設定してから使用する
  - サーバにSSHする際のセキュリティグループのインバウンドに設定される
  - 複数IPアドレスの記載には対応していない


## cfn_cicd_env.json

- 事前に ```cicd-ec2-key``` という名前でキーペアを作成しておく必要がある
- Amazon Linuxを1台構築する
  - EIPがフェッチされる
- Apache HTTP Server、Jenkins、Sonatype Nexusをインストールする
- ```SshLocation``` パラメータに自分のローカル環境のグローバルIPを設定してから使用する
  - サーバにSSHする際のセキュリティグループのインバウンドに設定される
  - 複数IPアドレスの記載には対応していない


## cfn_dev_env.json

- 「cfn_cicd_env.json」とはRedmine、SESが増える点のみ異なる
- 事前に ```dev-ec2-key``` という名前でキーペアを作成しておく必要がある
- 事前に SESを作成しておき、 **RedmineSmtpAddress** 、 **RedmineSmtpUsername** 、 **RedmineSmtpPassword** パラメータに値を設定しておく必要がある
  - SESはRedmineからメールを送信するために作成する
  - SESはTokyoリージョンに無いため、オレゴンリージョン（us-west-2）へ作成する
- Amazon Linuxを1台を構築する
  - EIPがフェッチされる
- Amazon LinuxにはApache HTTP Server、Jenkins、Sonatype Nexus、Redmine、SQLite（Redmine用）をインストールする
- ```SshLocation``` パラメータに自分のローカル環境のグローバルIPを設定してから使用する
  - サーバにSSHする際のセキュリティグループのインバウンドに設定される
  - 複数IPアドレスの記載には対応していない


# 設計

## ネットワーク

- VPCに割り当てるセグメントは **10.0.0.0/16**
  - 以下のIPはカッコの用途のため利用できない（前4つと最後1つ）
    - 10.0.0.0（ネットワークアドレス）
    - 10.0.0.1（VPC ルーター）
    - 10.0.0.2（DNS へのマッピング用）
    - 10.0.0.3（将来の利用のためにAWSで予約）
    - 10.0.255.255 （ネットワークブロードキャストアドレス）
- サブネットは冗長化のため2つ利用し、セグメントはそれぞれ以下
  - AZ-A: **10.0.0.0/24**
  - AZ-C: **10.0.1.0/24**
- **DBSubnetGroup** は上記のサブネットを横断して作成  

## セキュリティグループ

以下のセキュリティグループ（以降、SG）に分類する。  
なお、アウトバウンドはフリーとし、インバウンドは以下の通り受け付ける。

- WebパブリックSG（ *publicWebSecurityGroup* ）
  - インターネットへ公開しているサービスへのリクエストのみ受け付ける
  - LB、Webサーバなどを配置する
- アプリケーションSG（ *appSecurityGroup* ）
  - WebパブリックSGからのリクエストのみ受け付ける
  - オンライン・バッチアプリケーションサーバやLambdaを配置する
- データベースSG（ *dbSecurityGroup* ）
  - APからのアクセスのみ受け付ける
  - RDSなどデータベースを配置する
- 維持管理SG（ *sshSecurityGroup* ）
  - SSH、RDPのみ受け付ける
  - 上記には記載していないが、このSGからは全てのSGへアクセスできる
  - 踏み台を配置する

**NetworkAcl は使用しない** 。  
SSL終端はCloudFrontもしくはLB。

## EC2

- SELinuxはオフ
