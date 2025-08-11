---
title: "Supabase ✖️ Prisma ✖️ Next.jsでフルスタックアプリを作って見た supabaseとprismaの連携編"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs","supabase","prisma","typescript"]
published: false
---
# はじめに
今回next.jsとsupabaseとprismaをつかってアプリケーションをチームで開発しました。
その際に学習した内容をまとめていきます
開発したプロジェクトはこちらです。
https://github.com/Takuya0202/meshiltupara


今回まとめるのは以下の内容です。
- prismaとsupabaseの設定。
- supabaseとnext.jsを使ったユーザー認証、ミドルウェア
- prismaを使ったcrud操作
- supabase storageを使って画像を保存する方法
この章ではsupabseとprismaを連携させてデータベースにテーブルを作成する方法を取り扱います。

## ディレクトリ構造
最終的なディレクトリ構造は以下の通りとなります。
```bash
next-app/
├── app/
│   ├── api/           
│   │   ├── auth/
│   │   │   └── register/
│   │   │       └── route.ts # ユーザー登録のAPI
│   │   ├── genre/
│   │   │   └── index/
│   │   │       └── route.ts
│   │   ├── store/
│   │   │   ├── [id]/
│   │   │   │   └── route.ts
│   │   │   ├── index/
│   │   │   │   └── route.ts
│   │   │   └── search/
│   │   │       └── [id]/
│   │   │           └── route.ts
│   │   └── user/
│   │       ├── edit/
│   │       │   └── route.ts
│   │       └── show/
│   │           └── route.ts
│   ├── components/             # コンポーネント
│   ├── actions/                # Server Actions
│   │   └── login.ts            # ログインのserver action
│   ├── login/                  # ログインページ
│   │   └── page.tsx
│   ├── profile/                # プロフィールページ
│   │   └── page.tsx
│   ├── register/               # ユーザー登録ページ
│   │   └── page.tsx
│   ├── store/                  # 店舗ページ
│   │   ├── [id]/
│   │   │   └── page.tsx
│   │   └── page.tsx
│   ├── top/                    # トップページ
│   │   └── page.tsx
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── hooks/                      # カスタムフック
│   ├── useAuth.ts              # ログインしてるユーザー情報を取得
├── lib/                        # Prismaクライアント
│   └── prisma.ts
├── prisma/                     # モデル定義、seeder
│   ├── schema.prisma           # Prismaスキーマ
│   └── seed.ts                 # シードファイル
├── utils/                      # ユーティリティ
│   └── supabase/
│       ├── client.ts           # Supabaseクライアント
│       ├── middleware.ts       # Supabaseミドルウェア
│       └── server.ts           # Supabaseサーバー
├── middleware.ts               # Next.jsミドルウェア
|-- .env                        # 環境設定
```

## supabaseとprismaの連携
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

### 環境設定をする
次にsupabaseにアクセスするための環境設定を行います。
以下の4つの環境変数を取得します。
```.env
DATABASE_URL=
DIRECT_URL=
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```
先ほど作成したsupabaseのプロジェクトの画面上部の`Connect`をクリックします。
以下の画面が開かれるので`Next.js`と`Prisma`を選択して貼り付けます。
![](/images/supabase_connection.png)
![](/images/supabase-env.png)

`DATABASE_URL`と`DIRECT_URL`はprismaからsupabaseにアクセスをする際に使用します。
`NEXT_PUBLIC_SUPABASE_URL`と`NEXT_PUBLIC_SUPABASE_ANON_KEY`はnextからsupabaseにアクセスするときに使います。

### prismaのセットアップ
次にprismaをセットアップします。
以下を実行
```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```
`.env`と`schema.prisma`ファイルが作成されていたら大丈夫です。
`prisma client`はデータベースをメソッド形式で操作するためのものです。
`prisma studio`というものを使えばブラウザからデータベースの内容を確認することができますが、
今回はsupabaseを使っているため、使用しないことにします。

### ファイルを編集する。
`schema.prisma`を以下のように編集します。
```prisma:schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")  // 追記
}
```
これでprismaとsupabaseを連携することができました。

## テーブルを定義する。
### モデルの定義方法
```prisma:prisma/schema.prisma
// モデルの定義
model Store {
  id          Int     @id @default(autoincrement()) // 主キー
  name        String
  description String?
  link        String?
  address     String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```
テーブルの定義方法については`schema.prisma`で実行します。
モデル名は`単数形 パスカルケース`で定義し、フィールド名は`キャメルケース`で定義します
各カラムについては`フィールド名 型 制約`で定義していきます。`?`をつけることで`null`を許可します。

### 1:多のリレーション定義
次に1:多のリレーションの方法を考えます。
今回はお店(Storeテーブル)とジャンル(Genreテーブル)の関係を例に考えます。

```prisma:prisma/schema.prisma
// モデルの定義
model Store {
  id          Int     @id @default(autoincrement())
  name        String
  description String?
  link        String?
  address     String?

  genreId Int // 外部キー
  genre   Genre @relation(fields: [genreId], references: [id], onDelete: Cascade) // 制約

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Genre {
  id   Int    @id @default(autoincrement())
  name String

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // 接続を定義
  stores Store[] // 配列で複数を持たせる
}
```
外部キーを持つ方は`Int`型でカラムを定義した後、
`@relation`で対象となるテーブルに関係を持たせます。
外部キーに接続する方は`モデル名の配列`で接続させます。
1:1のリレーションの場合、ここが`Store`だけでよくなります。

### 多：多の中間テーブル
次にユーザーがお店にいいねを持たせる場合を考えます。
この場合、両者は多：多の関係にあるので中間となるテーブルを考えます。
```prisma:schema.prisma
// 中間テーブル
model StoreLike {
  userId String
  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)

  storeId Int
  store   Store @relation(fields: [storeId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@id([userId, storeId])
}

model User {
   storeLikes    StoreLike[]
}

model Store {
   storeLikes    StoreLike[]
}
```
複合主キーを定義する場合、`@@id`で設定します。

### migrateに関わるコマンド
マイグレーションに関係するコマンドは以下のようなものがあります。

#### スキーマを反映させる
```bash
npx prisma migrate dev --name init
```
`--name`はコミットメッセージを表します。なんでもいいです。
実行すると`migrations`ディレクトリが作成され、SQLのログを確認できます。

#### データをリセットさせる
```bash
npx prisma migrate reset
```
データを完全消去します。

#### 本番環境でのマイグレーション
```bash
npx prisma migrate deploy
```
本番環境でマイグレーションを適用します。`migrate dev`と違い、スキーマの変更は行いません。

#### マイグレーション履歴の確認
```bash
npx prisma migrate status
```
マイグレーションの適用状況を確認できます。

#### データベースの状態を確認
```bash
npx prisma db pull
```
既存のデータベースからスキーマを取得します。

#### Prismaクライアントの再生成
```bash
npx prisma generate
```
prismaクライアントを利用するときのコマンドです。
prismaクライアントについてはcrud操作編で紹介します。

