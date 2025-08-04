---
title: "Supabase ✖️ Prisma ✖️ Next.jsでフルスタックアプリを作ってみる。"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs","supabase","prisma"]
published: false
---
# 開発経緯
チームでの開発でNext.jsとSupabaseを使ったアプリを開発した際の学習した内容としてまとめてみます。

## supabaseとprismaの設定
### prismaとsupabaseについて理解する。
#### supabaseとは何か
**supabase**とはPostgreSQLをベースとしたデータベースや認証、ストレージなどバックエンドに必要な機能を提供してくれるサービスです。

#### prismaとは何か
**prisma**とはデータベースツールキットで、ORM機能やマイグレーション、CRUD操作をメソッドで行うことができます。
Next.jsには標準のORMは含まれていないのでPrismaを使ってAPIからDBにアクセスします。

### supabaseでDBを作成する。
supabaseにログイン後、new projectをクリックすると以下のような画面になります。

![](/images/supabase_project.png)

project名とパスワードを設定することで作成できます。
`Database Password`は後ほど使うので覚えておきましょう
これでDBに関しては作成できました。

### prismaのセットアップ
次にprismaをセットアップします。
以下を実行
```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```
`.env`と`schema.prisma`ファイルが作成されていたら大丈夫です。

### ファイルを編集する。
`schema.prisma`を以下のように編集します。
```prisma:schema.prisma
generator client {
  provider = "prisma-client-js"
  // output   = "../app/generated/prisma" いらない
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}
```

次に取得先の`.env`にある`DATABASE_URL`と`DIRECT_URL`を取得します。
先ほど作成したsupabaseのプロジェクトの画面上部の`Connect`をクリックします。
そうすると以下の画面になるため、ORMsからPrismaを選択してコードを`.env`に張り付けしてください。

![](/images/supabase_connection.png)

おそらくurlの中身に`[YOUR-PASSWORD]`という文字があるのでそこを先ほど設定した`Database Password`に変更してください。
これでprismaとsupabaseを連携することができました。

### テーブルを定義する。
