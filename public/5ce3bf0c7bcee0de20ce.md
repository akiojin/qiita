---
title: 何となくしか理解していないキーチェインを使いこなす
tags:
  - Mac
  - iOS
  - keychain
  - キーチェイン
private: false
updated_at: '2022-03-17T19:24:34+09:00'
id: 5ce3bf0c7bcee0de20ce
organization_url_name: cloud-creative-studios
---
# はじめに

キーチェイン…iOS の開発時に出てくるキーワードですが、意外とネットで調べてその場で解消して終わらせていることが多かったので備忘録として残しておこうかと思います。

## キーチェインとは？

キーチェインとはそもそも何なのか？
簡単に言うとパスワードとか証明書とか鍵とかを保存するファイルです。

デフォルトでは

- システムキーチェイン（`/Library/Keychains/System.keychain`）
- ログインキーチェイン（`~/Library/Keychains/login.keychain-db`）
- iCloud キーチェイン

などが存在します。

## キーチェインの種類

キーチェインには複数種類が存在します。

### システムキーチェイン

ユーザー間で共有されるシステム管理のキーチェインです。
もちろん、システム管理ですので、システム管理者アカウント（もしくは `sudo` とか）でないと操作は出来ません。

### デフォルトキーチェイン

デフォルトで使用されるユーザー管理のキーチェインです。
後述するカスタムキーチェインをデフォルトに設定したり、デフォルトで存在する `login.keychains-db` を指定したり出来ます。

### カスタムキーチェイン

ユーザーが自由に作ることが出来るキーチェインです。
もちろん、好きに作成・削除も出来ますし、デフォルトのキーチェインとして使用することも出来ます。

## キーチェインの操作方法

では、実際にキーチェインをどうやって操作するのかを説明していきます。
キーチェインの操作には以下の二種類の方法があります。

- キーチェインアクセス
- security コマンド

### キーチェインアクセス

キーチェインファイルを GUI で操作するアプリです。
カスタムキーチェインを作成したり一通りの操作をすることが出来ます。
（※唯一デフォルトのキーチェインの設定方法が分からないので結構ツラい）
また、キーチェインアクセスは反映がちょっと怪しい（security コマンドの内容がアプリを再起動しないと反映されない場合がある）ので気をつけてください。

### security コマンド

キーチェインアクセスで出来る操作は全て出来ます。
もちろん、デフォルトのキーチェインの設定も出来ますので、一覧で見る以外の操作はエンジニアであればこちらで操作した方が良いです。

## キーチェインの内容

特に何もしていなければキーチェインアクセスを開けば以下のような感じになっていると思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/19224/1e36ba4f-b30d-98f1-a7d6-170a23e67868.png)

- デフォルトキーチェイン
    - ログイン (`login.keychain-db` が『ログイン』と表示されます)
    - iCloud (Apple ID で mac にログインしていると表示されます)
- システムキーチェイン
    - システム　 (`System.keychain` が『システム』と表示されます)
    - システムルート

上記の状態でない場合は、何かしらキーチェインの操作をしてしまった後だと思います。

ちなみにシステムキーチェインは特に触らない（よっぽどシステムに精通していないのであれば触ってはいけない）と思いますので、今回は割愛します。

security コマンドの場合：
```sh
# デフォルトのキーチェイン一覧の表示
% security default-keychain
    "/Users/<ユーザー名>/Library/Keychains/login.keychain-db"
# 読み込まれているキーチェイン一覧の表示
security list-keychains
    "/Users/<ユーザー名>/Library/Keychains/login.keychain-db"
    "/Library/Keychains/System.keychain"
```

のような表示になると思います。

## キーチェインの問題が発生した場合

### `login.keychain-db` がなくなった！

稀に操作失敗時には `login.keychain-db` が無くなる場合があります。
その場合は慌てずに `~/Library/Keychains/login.keychain-db` が存在するか確認します。
もし、存在するのであれば、`security default-keychain -d user -s login.keychain-db` で復帰させることが可能です。

もし、`login.keychain-db` が存在せずに `login_renamed_??.keychaind-db` が存在するのであれば、それは `login.keychain-db` のバックアップになりますので、そのうちのどれかを `login.keychain-db` にリネームし、上記と同じ `security default-keychain -d user -s login.keychain-db` を実行すれば復帰するはずです。

それも存在しない場合は `security create-keychain -p <パスワード> login.keychain-db` を実行して、`login.keychain-db` を作成し、それを `security default-keychain -d user -s login.keychain-db` で設定することでログインキーチェインが使えるようになります。


## よく使いそうな security コマンド一覧

#### 新規キーチェインの作成
```sh
# キーチェイン名にパスを含めずに名前だけだと ~/Library/Keychains に作成される
security create-keychain -p <パスワード> <キーチェイン名>
```

#### キーチェインの削除
```sh
# デフォルトでは ~/Library/Keychains に配置されているキーチェインが削除される
security delete-keychain <キーチェイン名>
```

#### デフォルトキーチェインに設定・表示
```sh
# 設定
security default-keychain -d user -s <キーチェイン名>
# 表示
security default-keychain
```

#### 使用するキーチェイン一覧に設定・表示
```sh
# 設定
security list-keychains -d user -s <キーチェイン名1> <キーチェイン名2> ...
# 表示
security list-keychains
```

#### キーチェインのタイムアウトの設定
```sh
security set-keychain-settings -lut <ロックまでの秒数> <キーチェイン名>

# スリープ時にロック、もしくは 6 時間未操作時にロック
security set-keychain-settings -lut 21600 login.keychain-db
```

#### 登録されているパスワードをサービス名から検索する
```sh
security find-generic-password -s <サービス名>

# 例えば git-credential-manager-core の登録情報を検索するのであれば以下のようになります
security find-generic-password -s "git:https://github.com"
```

#### 登録されている証明書一覧を表示
```sh
# 登録されている証明書 (P12 ファイルなど) 一覧が表示される
security find-identity -p codesigning <キーチェイン名>
```

# おわりに

キーチェインって意外と難しく感じたりするのは、普段から使う事がないからだと思いますが、使ってみると意外と簡単に操作できるので、上記のコマンドを覚えておいてもよいかもしれません。
