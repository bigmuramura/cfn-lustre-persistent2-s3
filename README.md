# cfn-lustre-persistent2-s3
# Lustre 起動手順
FSx for Lustre 起動からマウントして利用可能な状態まで

1. CloudFormation から FSx for Lustre を作成（約10分）
      1. `CreateLustreToggle`を`true`設定
1. CloudFormation のデプロイ処理が完了
1. スタックの出力より`ExporetS3LinkCmd`をコピーし権限のある端末上でコマンドを実行
1. FSx for Lustre と所定の S3 バケットがリンクされる（約10分）
1. スタックの出力より`ExporetLustreMountCmd`をコピーしヘッドノード上でコマンドを実行
      1. ヘッドノードには Lustre Clinet のインストールが必須
      1. 規定のマウントパス`/mnt/lustre/`ディレクトリがヘッドノードに存在していること
1. ヘッドノードにマウントされたことを確認
1. `mnt/lustre/`から参照できる S3バケット名（規定`bucket1`）の所有者をrootから変更
      1. `sudo chown ec2-user. /mnt/lustre/bucket1`
      1. S3 バケットをリンクする度に所有者がrootに戻るため
1. コンピュートノード用のpostinstall.shのマウント設定を更新
      1. `ExporetLustreMountCmd`をコピペ

## postinstall.sh
実行スクリプトサンプル

### Amazon Linux2
```
#! /bin/bash

amazon-linux-extras install -y lustre2.10
mkdir -p /mnt/lustre
mount -t lustre -o noatime,flock fs-03e57799151f0807e.fsx.ap-northeast-1.amazonaws.com@tcp:/prz2fbmv /mnt/lustre
chown ec2-user. /mnt/lustre
```

# 削除手順
1. CloudFormation のスタックのパラメータを変更して更新
      1. `CreateLustreToggle`を`false`設定
1. FSx for Lustre は削除される
      1. データは Lustre と連携した S3 バケットに保存されている
            1. Lustre 削除時に S3 バケットの連携設定は自動的に削除される