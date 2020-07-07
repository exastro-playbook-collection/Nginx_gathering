# Nginx パラメータ生成ロール

# Trademarks
-----------
* Linuxは、Linus Torvalds氏の米国およびその他の国における登録商標または商標です。
* RedHat、RHEL、CentOSは、Red Hat, Inc.の米国およびその他の国における登録商標または商標です。
* Windows、PowerShellは、Microsoft Corporation の米国およびその他の国における登録商標または商標です。
* Ansibleは、Red Hat, Inc.の米国およびその他の国における登録商標または商標です。
* pythonは、Python Software Foundationの登録商標または商標です。
* Nginxは、Nginx Software Inc. の米国登録商標です。
* Crossplaneは、Upbound, Inc.の登録商標または商標です。
* NECは、日本電気株式会社の登録商標または商標です。
* その他、本ロールのコード、ファイルに記載されている会社名および製品名は、各社の登録商標または商標です。

## Description

本ロールでは、Nginx構築済み環境から設定情報を収集する機能と、収集した設定情報からNginx構築ロールで使用できるパラメータを生成する機能を提供します。

## Supports

本ロールは、以下環境をサポートします。

- 管理サーバー（Ansible実行サーバー）
  - OS：RHEL7.4（CentOS7.4）/RHEL8.0（CentOS8.0）
  - Ansible：Version 2.8
  - Python：2.7 or 3.6
  - [crossplane V0.5.3](https://github.com/nginxinc/crossplane)
- 対象サーバー
  - 利用しているロール、共通部品の仕様に準拠します。
  - Python：2.7 or 3.6

## Dependencies

本ロールでは、以下のロール、共通部品を利用しています。

- 収集機能（Nginx_gathering）
  - gathering ロール
- パラメータ生成機能（Nginx_extracting）
  - パラメータ生成共通部品

## Role Variables

本ロールで指定できる変数値について説明します。

### Mandatory Variables

ロール利用時に必ず指定しなければならない変数値はありません。

### Optional Variables

ロール利用時に以下の変数値を指定することができます。

- 共通

    | Name                            | Default Value | Description                         |
    | ------------------------------- | ------------- | ----------------------------------- |
    | `VAR_Nginx_gathering_dest`      | '{{ playbook_dir }}/_gathered_data' | 収集した設定情報の格納先パス |
    | `VAR_Nginx_tmpDir`              | '/tmp'        | 一時ファイルが保存されるディレクトリ、半角スペースを含まないでください |

- Nginx_extracting

    | Name                            | Default Value | Description                        |
    | ------------------------------- | ------------- | -----------------------------------|
    | `VAR_Nginx_extracting_rolename` | '['Nginx_Install','Nginx_Setup','Nginx_OSSetup']'  | パラメータ生成対象 (*1) |
    | `VAR_Nginx_extracting_dest`     | '{{ playbook_dir }}/_parameters' | 生成したパラメータの出力先パス |

(*1) 本変数値は収集・パラメータ生成時の識別子として使用するため、通常は変更しないでください。

## Results

本ロールの出力について説明します。

### 収集した設定情報の格納先

収集した設定情報は以下のディレクトリ配下に格納します。

- <VAR_Nginx_gathering_dest>/<ホスト名/IP>/Nginx_gathering/

本ロールを既定値で利用した場合、以下のように設定情報を格納します。

```none
    <Playbook実行ディレクトリ>/
      +-_gathered_data/
          +-<ホスト名/IP>/
              +-Nginx_gathering/       # 収集データ
                  +-nginx.json         # nginx構成ファイルデータ
                  +-nginxState.txt     # Nginxサービス状態データ       
```

### 生成したパラメータの出力例

生成したパラメータは以下のディレクトリ・ファイル名で出力します。

- <VAR_Nginx_extracting_dest>/<ホスト名/IP>/<VAR_Nginx_extracting_rolename>.yml

本ロールを既定値で利用した場合、以下のようにパラメータを出力します。

```none
    <Playbook実行ディレクトリ>/
      +-_parameters/
          +-<ホスト名/IPアドレス>/
              +-Nginx_Install.yml      # パラメータ
              +-Nginx_OSSetup.yml      # パラメータ
              +-Nginx_Setup.yml        # パラメータ
              
```

## Usage

本ロールの利用例について説明します。

### 設定情報収集およびパラメータ生成を行う場合

以下の例ではデフォルト設定で設定情報収集およびパラメータ生成を行います。

- `Nginx_pargen.yml` (Nginx用Playbook)

    ```yaml
    ---
    - hosts: linux
      gather_facts: true
      roles:
        - role: Nginx_gathering

    - hosts: linux
      gather_facts: false
      roles:
        - role: Nginx_extracting
    ```

- 以下のように設定情報とパラメータを出力します。

    ```none
    <Playbook実行ディレクトリ>/
      +-_gathered_data/
      |   +-<ホスト名/IPアドレス>/
      |       +-Nginx_gathering/       # 収集データ
      |           +-nginx.json         # nginx構成ファイルデータ
      |           +-nginxState.txt     # Nginxサービス状態データ 
      +-_parameters/
      |   +-<ホスト名/IPアドレス>/
      |       +-Nginx_Install.yml      # パラメータ
      |       +-Nginx_OSSetup.yml      # パラメータ
      |       +-Nginx_Setup.yml        # パラメータ
    ```

### パラメータ再利用

以下の例では、生成したパラメータを使用して構築済Nginxの設定を変更します。

- `Nginx_setup-playbook.yml`（Nginx構築ロールを使用）

    ```yaml
    ---
    - hosts: nginx
      roles:
        - Nginx_Setup
    ```

- 生成したパラメータを指定してplaybookを実行

    ```sh
    ansible-playbook Nginx_setup-playbook.yml -i hosts --extra-vars="@_parameters/<ホスト名/IPアドレス>/Nginx_Setup.yml"
    ```

# Copyright
Copyright (c) 2019 NEC Corporation

# Author Information
NEC Corporation
