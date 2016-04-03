## Packer で Windows Server AMI を作るぱかぱか

### 作成される AMI の概要

- Python をインストールする(msi パッケージのインストール試した)
- pyWin32 インストール(exe パッケージのインストールを試した)
- Ansible で構成管理出来るように設定(PowerShell スクリプトの実行を試した)

### インスタンス起動時に指定するユーザーデータ

- タイムゾーンを JST に変更
- ネットワークプロファイルを Public から Private に変更

```
<powershell>
tzutil.exe /s "Tokyo Standard Time"
Set-NetConnectionProfile -NetworkCategory Private
</powershell>
```

### 出来た AMI を Serverspec でテスト

```sh
% tree spec
spec
├── common
│   └── common_spec.rb
└── spec_helper.rb

1 directory, 2 files
```

### tips

- msi パッケージのインストール完了を待つ場合

```
Start-Process "C:\download_files\python-2.7.11.amd64.msi" /quiet -Wait
```

`Start-Process` コマンドレットを使って `-Wait` オプションを利用する。

- Administrator のパスワードをリセットする

```json
    "provisioners": [
        {
            "type": "powershell",
            "inline": [
                "C:\\PROGRA~1\\Amazon\\Ec2ConfigService\\Ec2Config.exe -sysprep"
            ]
        }
    ]
```

上記のように Packer の Provisioner において Ec2Config を `-sysprep` オプション付きで実行する。

- ネットワークプロファイルは NIC 作成時点で設定されるようなので AMI に設定を施してもダメ（っぽい） 
- タイムゾーンも同様に AMI に埋め込むよりインスタンス起動時に userdata で指定した

### 参考

- https://gist.github.com/mitchellh/ab754cffbb173b9f341a
