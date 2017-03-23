CloudFormationテンプレート置き場
===

# cfn_cicd_env.json

- 事前に ```cicd-ec2-key``` という名前でキーペアを作成しておく必要がある
- Apache HTTP Server、Jenkins、Sonatype Nexusをインストールしたインスタンスを1台構築する
- ```SshLocation``` パラメータに自分のローカル環境のグローバルIPを設定してから使用する
  - サーバにSSHする際のセキュリティグループのインバウンドに設定される
  - 複数のIPアドレスを設定したい場合は、カンマ（ ```,``` ）区切りで
