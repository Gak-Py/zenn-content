---
title: "zennへローカルmdからの更新を学ぶ"
emoji: "🔥" 
type: "tech"     
topics: ["zenn", "markdown", "git", "cli"]
published: true 
---

# zennへローカルmdからの更新を学ぶ

Zennの記事は、ブラウザ上で直接編集しなくても  
**ローカルのMarkdown（md）ファイルから更新**できます。

この記事では、**Zenn CLIを使った基本的な更新フロー**を、実務目線で記録します。

---

## 前提条件

- Node.js がインストールされている
- ローカルでMarkdownを書ける（VS Code推奨）

node管理はvoltaを使用。

## 目次

1. ローカルでzenn cliを使うメリット
2. 実装
3. サンプルコード
4. まとめ

---

## ローカルでzenn cliを使うメリット

- ローカルで記事データとして残しておきたい
- 管理しやすそう
- vs codeから投稿する、というスキルを取得できる
---

## 実装

### 1. 環境構築

```bash
npm i -g zenn-cli
```

自動でフォルダを作ってくれる。  
基本はarticleにmdファイルを作って記事を書く。

### 2. git側でリポジトリを作る
zenn-contentsとかが主流らしい。

### 3. ローカルとgitを連携する
リポジトリを作った後にでるbashコードでローカルと連携できる。

```bash
git remote add origin https://github.com/アカウント/zenn-content.git
git push -u origin main
```

いったんこの時点でコミットしてもよさそう。

### 4. 記事を書く

```bash
zenn new:article
```
このコマンドで記事テンプレを作成してくれる。

vs codeだとzenn previewをつかわずともmdファイルはpreviewしてくれるのでよさそう。  
画像のプレビューを見たいときとかはzenn previewの方がわかりやすいのかもしれない。


### 5. 記事を投稿する

```bash
git add .
git commit -m "記事投稿"
git push
```

gitへのプッシュで投稿ができる。

というわけでこれが投稿できました。