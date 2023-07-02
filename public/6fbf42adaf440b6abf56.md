---
title: GitHub Actions での環境変数の使い方
tags:
  - 自動化
  - Github-flow
  - GitHubActions
private: false
updated_at: '2021-12-03T11:13:01+09:00'
id: 6fbf42adaf440b6abf56
organization_url_name: cloud-creative-studios
---
# はじめに
GitHub Actions では環境変数を利用することができますが、ホストする環境によって環境変数の使い方が変わります。

環境変数を使うときには、ここが意外とハマりやすいポイントですので覚えておいた方が良いです。

[環境変数のサンプルプログラム](https://github.com/cloud-creative-studios/GitHub-Actions-Environment)

## Windows ホストランナー
```yaml
- name: Echo
  run: echo ${{ env.CONFIGURATION }}
```
## ubuntu/macOS ホストランナー
```yaml
- name: Echo
  run: echo $CONFIGURATION
```
上記を見ても分かりますが Windows 環境の場合、env コンテキストを介して環境変数にアクセスする必要があります。

ただし、ubuntu や macOS でも以下のような場合は env コンテキストを介してアクセスする必要があります。

```yaml
- name Echo
  env:
    TEMP_CONFIGURATION: ${{ env.CONFIGURATION }}
  run: echo $TEMP_CONFIGURATION
```

スクリプトを実行する場合は ```$ENV``` 形式でアクセスできますが、ステップの設定値を設定する際に環境変数のアクセスする場合は ```${{ env.???? }}``` 形式にする必要があります。
