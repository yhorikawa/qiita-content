---
title: Advent Calenderをvscodeとqiita cliを使って書くために準備をしていたら、自分の日が来てしまったのでやっていたことをまとめます
tags:
  - 'qiita-cli'
private: false
updated_at: ''
id: null
organization_url_name: ateam-life-design
slide: false
ignorePublish: false
---
qiita cliが今年の8月にリリースされていました

https://github.com/increments/qiita-cli

せっかくので今回のAdvent Calenderをqiita cliを使って書いてみようと思っていました
そう思ったのは良いんですが、投稿日ぎりぎりに着手したために準備をしていたら投稿日になってしまいました

今から記事を書くと間に合わないので、やっていたことをまとめてみようかなと思いました
今回はdev containerを使って執筆環境を作りました

## 1. qiita cliのインストール

package.jsonにqiita cliを追加しておきます

```package.json
{
  "name": "qiita-content",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "devDependencies": {
    "@qiita/qiita-cli": "^1.3.0"
  }
}
```

## 2. credentials.jsonの作成

qiita cliのreadme.mdを参考にトークンを発行しておきます

https://github.com/increments/qiita-cli?tab=readme-ov-file#qiita-%E3%81%AE%E3%83%88%E3%83%BC%E3%82%AF%E3%83%B3%E3%82%92%E7%99%BA%E8%A1%8C%E3%81%99%E3%82%8B

こちらは本来`npx qiita login`を実行すると`$XDG_CONFIG_HOME/qiita-cli`もしくは`$HOME/.config/qiita-cli`にcredentials.jsonを作成してくれますが、今回はコンテナ内にマウントしたいためプロジェクト内に作ろうと思います
このファイルにはqiitaのAPIトークンが含まれているため、先に`.gitignore`に追加しておきます
ついでに`node_modules`やqiita cliで作られるフォルダも追加しておきます

```.gitignore
credentials.json
node_modules
.remote
```

credentials.jsonを作成します

```credentials.json
{
  "default": "qiita",
  "credentials": [
    {
      "name": "qiita",
      "accessToken": "先ほど発行したトークン"
    }
  ]
}
```

## 3. devcontainerの設定

`cmd + shift + p`を押して`Dev Containers: Add Dev Container Configuration Files...`を選択します

![addDevContainerConfigurationFiles](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/362594/b82b91d1-387c-0586-0aa2-0b4b5d4c3170.png)

qiita cliを動かすためにはnode.jsが必要なので、node.jsのdevcontainerを選択します
また、今回はpnpmを使おうと思うので、pnpmを選択しておきます

`.devcontainer`に`devcontainer.json`が作成されたと思います
qiita cliはデフォルトでは8888ポートを使うようなので、`forwardPorts`に8888を追加しておきます
今回はpnpmを使用しているので、`postCreateCommand`を`pnpm install`に変更します
先ほど作成したcredentials.jsonをコンテナ内にマウントするために`mounts`を追加しています

```devcontainer.json
{
  "name": "Node.js",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:1-20-bullseye",
  "features": {
    "ghcr.io/devcontainers-contrib/features/pnpm:2": {}
  },
  "forwardPorts": [8888],
  "postCreateCommand": "sudo chown node node_modules && pnpm install",
  "mounts": [
    "source=qiita-content-node_modules,target=${containerWorkspaceFolder}/node_modules,type=volume",
    "source=${localWorkspaceFolder}/credentials.json,target=/home/node/.config/qiita-cli/credentials.json,type=bind,consistency=cached"
  ],
  "customizations": {
    "vscode": {
      "extensions": [
        "DavidAnson.vscode-markdownlint",
        "streetsidesoftware.code-spell-checker"
      ]
    }
  }
}
```

`cmd + shift + p`を押して`Dev Containers: Reopen in Container`を選択し起動します

## 3. qiita cliでの記事作成

１回だけ`qiita init`を実行しておきます

```shell
pnpm qiita init
```

これによって`qiita.config.json`やgithub actionsの設定ファイルが作成されます

### preview

`pnpm qiita preview`でpreviewを確認できます

### 新規作成

`pnpm qiita new`を実行するかpreview上の新規作成ボタンから作成ができます

```shell
pnpm qiita new 記事のファイルのベース名
```

### 記事の公開

qiita cliのreadmeを参考にgithub actionsで公開するように設定します

https://github.com/increments/qiita-cli?tab=readme-ov-file#github-%E3%81%A7%E8%A8%98%E4%BA%8B%E3%82%92%E7%AE%A1%E7%90%86%E3%81%99%E3%82%8B

## 4. vscode拡張機能

`devcontainer.json`の`customizations`に拡張機能を追加することができます

```devcontainer.json
{
  "customizations": {
    "vscode": {
      "extensions": [
        "DavidAnson.vscode-markdownlint"
      ]
    }
  }
}
```

text-lintのような執筆が楽になるような拡張機能を追加しておきたいです

## 最後に

qiita cliによってvscodeで記事が書けるようになったので、記事の執筆が捗りますね
