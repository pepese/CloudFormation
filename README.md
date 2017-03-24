CloudFormationテンプレート置き場
===

# cfn_cicd_env.json

- 事前に ```cicd-ec2-key``` という名前でキーペアを作成しておく必要がある
- Amazon Linuxを1台構築する
- Apache HTTP Server、Jenkins、Sonatype Nexusをインストールする
- ```SshLocation``` パラメータに自分のローカル環境のグローバルIPを設定してから使用する
  - サーバにSSHする際のセキュリティグループのインバウンドに設定される
  - 複数のIPアドレスには対応していない
