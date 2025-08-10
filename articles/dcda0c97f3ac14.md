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
開発したプロジェクト
https://github.com/Takuya0202/TechJamIteam

今回まとめるのは以下の内容です。
- prismaとsupabaseの設定。
- supabaseとnext.jsを使ったユーザー認証、ミドルウェア
- prismaを使ったcrud操作
- supabase storageを使って画像を保存する方法
この章ではsupabseとprismaを連携させてデータベースにテーブルを作成する方法を取り扱います。

## ディレクトリ構造
ディレクトリ構造は以下の通りとなります。
```bash
next-app/
├── app/
│   ├── api/                    # APIディレクトリ
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
│   ├── useAuth.ts
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
`NEXT_PUBLIC_SUPABSE_URL`と`NEXT_PUBLIC_SUPABASE_ANON_KEY`はnextからsupabaseにアクセスするときに使います。

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
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}
```
これでprismaとsupabaseを連携することができました。

## テーブルを定義する。
### モデルの定義方法
```prisma:prisma/schema.prisma
// モデルの定義
model Store {
  id          Int     @id @default(autoincrement())
  name        String
  description String?
  link        String?
  address     String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```
テーブルの定義方法については`schema.prisma`で実行します。
`カラム名 型 オプション`で定義していきます。
また、nullを許可する場合は`?`をつけることで許可できます。

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
  stores Store[]
}
```

